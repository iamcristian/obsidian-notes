---
tags:
  - software-engineering
  - devops
  - ci-cd
  - github-actions
created: 2026-01-02
status: 🔴
---
# 🤖 GitHub Actions

> *"Automate your workflow from idea to production."*

## 🎯 What is GitHub Actions?

GitHub Actions es una plataforma de CI/CD integrada en GitHub que permite automatizar builds, tests y deployments directamente desde tu repositorio.

---

## 📁 Workflow Structure

```
.github/
└── workflows/
    ├── ci.yml           # Continuous Integration
    ├── cd.yml           # Continuous Deployment
    ├── release.yml      # Release automation
    └── cron.yml         # Scheduled tasks
```

---

## 📝 Basic Workflow

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

# Triggers
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  workflow_dispatch:  # Manual trigger

# Jobs
jobs:
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Run linter
        run: npm run lint
```

---

## 🎯 Triggers (Events)

```yaml
on:
  # Push to specific branches
  push:
    branches:
      - main
      - 'releases/**'
    paths:
      - 'src/**'
      - '!src/**/*.md'
    tags:
      - 'v*'

  # Pull request events
  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened]

  # Scheduled (cron)
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight UTC

  # Manual trigger
  workflow_dispatch:
    inputs:
      environment:
        description: 'Deployment environment'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production

  # Repository dispatch (API trigger)
  repository_dispatch:
    types: [deploy]

  # After another workflow
  workflow_run:
    workflows: ["Build"]
    types: [completed]
```

---

## 🔄 Complete CI/CD Pipeline

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ==================== TEST ====================
  test:
    name: Test
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run unit tests
        run: npm run test:unit

      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgres://test:test@localhost:5432/testdb

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

  # ==================== LINT ====================
  lint:
    name: Lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck

  # ==================== BUILD ====================
  build:
    name: Build Docker Image
    runs-on: ubuntu-latest
    needs: [test, lint]
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    
    permissions:
      contents: read
      packages: write

    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=sha
            type=ref,event=branch

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # ==================== DEPLOY ====================
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: build
    environment:
      name: staging
      url: https://staging.example.com

    steps:
      - name: Deploy to staging
        run: |
          echo "Deploying ${{ needs.build.outputs.image-tag }} to staging"
          # kubectl set image deployment/app app=${{ needs.build.outputs.image-tag }}

  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: [build, deploy-staging]
    environment:
      name: production
      url: https://example.com

    steps:
      - name: Deploy to production
        run: |
          echo "Deploying to production"
```

---

## 📦 Matrix Strategy

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [18, 20, 22]
        exclude:
          - os: windows-latest
            node-version: 18
        include:
          - os: ubuntu-latest
            node-version: 20
            experimental: true

    steps:
      - uses: actions/checkout@v4
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test
```

---

## 🔐 Secrets & Environment Variables

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    
    # Environment with protection rules
    environment:
      name: production
      url: https://example.com

    env:
      # Workflow-level env var
      NODE_ENV: production

    steps:
      - name: Deploy
        env:
          # Step-level env var
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          API_KEY: ${{ secrets.API_KEY }}
        run: |
          echo "Deploying with secrets..."
```

### GitHub Contexts

```yaml
steps:
  - name: Show contexts
    run: |
      echo "Repository: ${{ github.repository }}"
      echo "Branch: ${{ github.ref_name }}"
      echo "SHA: ${{ github.sha }}"
      echo "Actor: ${{ github.actor }}"
      echo "Run ID: ${{ github.run_id }}"
      echo "Event: ${{ github.event_name }}"
```

---

## 📤 Artifacts & Caching

### Upload/Download Artifacts
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run build
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build
          path: dist/
          retention-days: 7

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: build
          path: dist/
```

### Caching Dependencies
```yaml
steps:
  - uses: actions/checkout@v4
  
  # Node.js with built-in cache
  - uses: actions/setup-node@v4
    with:
      node-version: '20'
      cache: 'npm'

  # Or manual caching
  - name: Cache node modules
    uses: actions/cache@v4
    with:
      path: ~/.npm
      key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
      restore-keys: |
        ${{ runner.os }}-node-
```

---

## 🔀 Reusable Workflows

### Caller Workflow
```yaml
# .github/workflows/ci.yml
name: CI
on: push

jobs:
  call-tests:
    uses: ./.github/workflows/reusable-tests.yml
    with:
      node-version: '20'
    secrets: inherit
```

### Reusable Workflow
```yaml
# .github/workflows/reusable-tests.yml
name: Reusable Tests

on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string
    secrets:
      NPM_TOKEN:
        required: false

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci
      - run: npm test
```

---

## 🎯 Common Patterns

### PR Checks with Status
```yaml
name: PR Checks

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  checks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Check PR title
        run: |
          if [[ ! "${{ github.event.pull_request.title }}" =~ ^(feat|fix|docs|chore|refactor): ]]; then
            echo "PR title must start with type (feat:, fix:, etc.)"
            exit 1
          fi
```

### Auto-merge Dependabot
```yaml
name: Dependabot Auto-merge

on: pull_request

permissions:
  contents: write
  pull-requests: write

jobs:
  auto-merge:
    runs-on: ubuntu-latest
    if: github.actor == 'dependabot[bot]'
    steps:
      - name: Auto-merge minor updates
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Release on Tag
```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
      
      - name: Create Release
        uses: softprops/action-gh-release@v1
        with:
          generate_release_notes: true
```

---

## 📋 Useful Actions

| Action | Purpose |
|--------|---------|
| `actions/checkout` | Checkout repository |
| `actions/setup-node` | Setup Node.js |
| `actions/cache` | Cache dependencies |
| `actions/upload-artifact` | Upload build artifacts |
| `docker/build-push-action` | Build and push Docker images |
| `softprops/action-gh-release` | Create GitHub releases |
| `codecov/codecov-action` | Upload code coverage |

---

← [[Programming/Software Engineering/DevOps/_Index|Back to DevOps]]
