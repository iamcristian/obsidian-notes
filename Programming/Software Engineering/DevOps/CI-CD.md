---
tags:
  - software-engineering
  - devops
  - ci-cd
created: 2026-01-02
status: 🔴
---
# 🔄 CI/CD Fundamentals

> *"Continuous Integration is a software development practice where members of a team integrate their work frequently."* — Martin Fowler

## 🎯 What is CI/CD?

**CI (Continuous Integration)**: Práctica de integrar código frecuentemente al repositorio principal, con builds y tests automatizados.

**CD (Continuous Delivery/Deployment)**: Extensión de CI que automatiza el release y deployment del software.

---

## 🏗️ CI/CD Pipeline Overview

```
┌────────────────────────────────────────────────────────────────────┐
│                        CI/CD PIPELINE                               │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│   ┌──────┐    ┌───────┐    ┌──────┐    ┌────────┐    ┌────────┐   │
│   │ Code │───►│ Build │───►│ Test │───►│ Deploy │───►│Monitor │   │
│   │      │    │       │    │      │    │Staging │    │        │   │
│   └──────┘    └───────┘    └──────┘    └────────┘    └────────┘   │
│       │                                     │                      │
│       │         CONTINUOUS INTEGRATION      │                      │
│       │◄───────────────────────────────────►│                      │
│       │                                     │                      │
│       │            CONTINUOUS DELIVERY      │    ┌────────┐        │
│       │◄───────────────────────────────────────►│ Deploy │        │
│       │                                         │  Prod  │        │
│       │            CONTINUOUS DEPLOYMENT        │(Manual)│        │
│       │◄───────────────────────────────────────►└────────┘        │
│                                                 (Automatic)        │
└────────────────────────────────────────────────────────────────────┘
```

---

## 📊 CI vs CD vs CD

| Aspect | Continuous Integration | Continuous Delivery | Continuous Deployment |
|--------|----------------------|--------------------|--------------------|
| **Goal** | Detect integration issues early | Always deployable | Always deployed |
| **Automation** | Build + Test | + Staging deploy | + Production deploy |
| **Manual Step** | None | Production deploy | None |
| **Risk** | Low | Medium | Requires maturity |

---

## 🔄 Continuous Integration

### CI Principles
1. **Commit frequently** - Al menos una vez al día
2. **Automate the build** - Un comando para compilar todo
3. **Self-testing build** - Tests automáticos con el build
4. **Fast feedback** - Build en < 10 minutos
5. **Fix broken builds immediately** - Prioridad #1

### CI Pipeline Stages
```yaml
# Example CI Pipeline
stages:
  - lint        # Code quality checks
  - test        # Unit & integration tests
  - build       # Compile/bundle
  - scan        # Security scanning
  - artifact    # Create deployable artifact
```

### CI Best Practices
```
┌─────────────────────────────────────────────────────────────┐
│                    CI BEST PRACTICES                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ✅ Trunk-based development                                  │
│     • Short-lived feature branches (< 1 day)                │
│     • Frequent merges to main                               │
│                                                             │
│  ✅ Fast feedback loop                                       │
│     • Build < 5 min, Full pipeline < 15 min                 │
│     • Parallelizar tests                                    │
│                                                             │
│  ✅ Fail fast                                                │
│     • Run fastest checks first (lint, unit tests)           │
│     • Stop pipeline on first failure                        │
│                                                             │
│  ✅ Reproducible builds                                      │
│     • Versioned dependencies                                │
│     • Containerized build environment                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🚀 Continuous Delivery / Deployment

### Deployment Stages
```
┌─────────────────────────────────────────────────────────────────┐
│                    DEPLOYMENT PIPELINE                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Development    Staging         Production                      │
│  ┌─────────┐   ┌─────────┐     ┌─────────┐                     │
│  │   DEV   │──►│ STAGING │──►  │  PROD   │                     │
│  │         │   │         │     │         │                     │
│  │Auto-    │   │Auto-    │     │Manual/  │                     │
│  │deploy   │   │deploy   │     │Auto     │                     │
│  └─────────┘   └─────────┘     └─────────┘                     │
│       │             │               │                           │
│       ▼             ▼               ▼                           │
│  Dev tests     E2E tests      Canary/Blue-Green                 │
│  Integration   Performance    Monitoring                        │
│  Smoke tests   Security       Alerts                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Deployment Strategies

#### 1. Blue-Green Deployment
```
┌──────────────────────────────────────────────────────┐
│                 BLUE-GREEN DEPLOYMENT                 │
├──────────────────────────────────────────────────────┤
│                                                      │
│  Before:                                             │
│  ┌────────────┐                                      │
│  │   Users    │                                      │
│  └─────┬──────┘                                      │
│        │                                             │
│        ▼                                             │
│  ┌────────────┐     ┌────────────┐                  │
│  │  Blue v1   │     │ Green (-)  │                  │
│  │  (Active)  │     │  (Idle)    │                  │
│  └────────────┘     └────────────┘                  │
│                                                      │
│  After:                                              │
│  ┌────────────┐                                      │
│  │   Users    │                                      │
│  └─────┬──────┘                                      │
│        │                                             │
│        ▼                                             │
│  ┌────────────┐     ┌────────────┐                  │
│  │  Blue v1   │     │ Green v2   │                  │
│  │  (Standby) │     │ (Active)   │                  │
│  └────────────┘     └────────────┘                  │
│                                                      │
│  Rollback: Switch traffic back to Blue               │
└──────────────────────────────────────────────────────┘
```

#### 2. Canary Deployment
```
┌──────────────────────────────────────────────────────┐
│                 CANARY DEPLOYMENT                     │
├──────────────────────────────────────────────────────┤
│                                                      │
│  Phase 1: 5% traffic to canary                       │
│  ┌────────────────────────────────────────────┐      │
│  │ ████████████████████████████████████░░     │      │
│  │          v1 (95%)              v2 (5%)     │      │
│  └────────────────────────────────────────────┘      │
│                                                      │
│  Phase 2: 25% traffic (if metrics OK)                │
│  ┌────────────────────────────────────────────┐      │
│  │ ███████████████████████████░░░░░░░░░░      │      │
│  │          v1 (75%)          v2 (25%)        │      │
│  └────────────────────────────────────────────┘      │
│                                                      │
│  Phase 3: 100% traffic (fully rolled out)            │
│  ┌────────────────────────────────────────────┐      │
│  │ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   │      │
│  │                  v2 (100%)                 │      │
│  └────────────────────────────────────────────┘      │
│                                                      │
└──────────────────────────────────────────────────────┘
```

#### 3. Rolling Deployment
```
┌──────────────────────────────────────────────────────┐
│                 ROLLING DEPLOYMENT                    │
├──────────────────────────────────────────────────────┤
│                                                      │
│  Initial:  [v1] [v1] [v1] [v1]                       │
│                                                      │
│  Step 1:   [v2] [v1] [v1] [v1]                       │
│  Step 2:   [v2] [v2] [v1] [v1]                       │
│  Step 3:   [v2] [v2] [v2] [v1]                       │
│  Step 4:   [v2] [v2] [v2] [v2]                       │
│                                                      │
│  Pros: Zero downtime, gradual rollout                │
│  Cons: Multiple versions running simultaneously      │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## 📋 Pipeline Components

### 1. Source Stage
```yaml
source:
  - checkout code
  - fetch dependencies
  - validate branch policies
```

### 2. Build Stage
```yaml
build:
  - compile code
  - run linters
  - generate artifacts
  - create Docker image
```

### 3. Test Stage
```yaml
test:
  - unit tests
  - integration tests
  - contract tests
  - security scanning (SAST)
  - code coverage
```

### 4. Deploy Stage
```yaml
deploy:
  staging:
    - deploy to staging
    - run E2E tests
    - run performance tests
    - manual approval gate
  
  production:
    - deploy with strategy (canary/blue-green)
    - smoke tests
    - monitor metrics
    - auto-rollback on failure
```

---

## 🛠️ CI/CD Tools Comparison

| Tool | Type | Best For |
|------|------|----------|
| **GitHub Actions** | Cloud | GitHub repos |
| **GitLab CI** | Cloud/Self-hosted | GitLab users |
| **Jenkins** | Self-hosted | Enterprise, customization |
| **CircleCI** | Cloud | Docker-based |
| **Azure DevOps** | Cloud | Microsoft ecosystem |
| **ArgoCD** | GitOps | Kubernetes |

---

## 🎯 Pipeline Quality Gates

```yaml
quality_gates:
  code_coverage:
    minimum: 80%
    fail_on: decrease
  
  security:
    critical_vulnerabilities: 0
    high_vulnerabilities: 0
  
  performance:
    response_time_p95: < 200ms
    error_rate: < 0.1%
  
  code_quality:
    sonar_quality_gate: passed
    no_new_bugs: true
    no_new_code_smells: true
```

---

## 📊 CI/CD Metrics

| Metric | Description | Target |
|--------|-------------|--------|
| **Lead Time** | Code commit → Production | < 1 hour |
| **Deployment Frequency** | How often deployments happen | Multiple/day |
| **MTTR** | Mean Time To Recover | < 1 hour |
| **Change Failure Rate** | % of deployments causing issues | < 15% |
| **Build Duration** | Pipeline execution time | < 15 min |

---

## 📚 Related

- [[Programming/Software Engineering/DevOps/GitHub Actions|GitHub Actions]]
- [[Programming/Software Engineering/DevOps/Docker|Docker]]
- [[Programming/Software Engineering/Testing/Testing Complete Guide|Testing]]

---

← [[Programming/Software Engineering/DevOps/DevOps Complete Guide|Back to DevOps]]
