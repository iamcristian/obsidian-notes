---
tags:
  - software-engineering
  - devops
created: 2026-01-02
status: 🟡
---
# 🛠️ DevOps

> *"DevOps is the union of people, process, and products to enable continuous delivery of value to our end users."* — Donovan Brown

## 🗺️ Overview

DevOps es una cultura y conjunto de prácticas que unifica desarrollo (Dev) y operaciones (Ops) para entregar software de forma rápida, segura y continua.

---

## 📚 Topics

### 🐳 Containerization
- [[Programming/Software Engineering/DevOps/Docker|Docker]] - Containers fundamentals
- [[Programming/Software Engineering/DevOps/Docker Compose|Docker Compose]] - Multi-container apps
- [[Programming/Software Engineering/DevOps/Kubernetes|Kubernetes]] - Container orchestration

### 🔄 CI/CD
- [[Programming/Software Engineering/DevOps/CI-CD|CI/CD Fundamentals]] - Continuous Integration & Deployment
- [[Programming/Software Engineering/DevOps/GitHub Actions|GitHub Actions]] - GitHub automation
- [[Programming/Software Engineering/DevOps/Pipeline Design|Pipeline Design]] - Designing effective pipelines

### ☁️ Cloud & Infrastructure
- [[Programming/Software Engineering/DevOps/Cloud Fundamentals|Cloud Fundamentals]] - AWS, GCP, Azure
- [[Programming/Software Engineering/DevOps/Serverless|Serverless]] - FaaS architecture
- [[Programming/Software Engineering/DevOps/Infrastructure as Code|Infrastructure as Code]] - Terraform, CloudFormation

### 📊 Monitoring & Observability
- [[Programming/Software Engineering/DevOps/Monitoring|Monitoring]] - Metrics & alerting
- [[Programming/Software Engineering/DevOps/Logging|Logging]] - Centralized logging
- [[Programming/Software Engineering/DevOps/Observability|Observability]] - Traces, metrics, logs

---

## 🔄 DevOps Lifecycle

```
      ┌────────────────────────────────────────────────────────┐
      │                    CONTINUOUS INTEGRATION               │
      │  ┌─────┐    ┌──────┐    ┌───────┐    ┌───────────────┐  │
      │  │Plan │───►│ Code │───►│ Build │───►│     Test      │  │
      │  └─────┘    └──────┘    └───────┘    └───────────────┘  │
      │       ▲                                      │          │
      │       │                                      ▼          │
      │  ┌────────┐  ┌─────────┐  ┌────────┐  ┌──────────┐     │
      │  │Monitor │◄─│ Operate │◄─│ Deploy │◄─│ Release  │     │
      │  └────────┘  └─────────┘  └────────┘  └──────────┘     │
      │                    CONTINUOUS DEPLOYMENT                │
      └────────────────────────────────────────────────────────┘
```

---

## 📊 DevOps Maturity Model

| Level | Description | Practices |
|-------|-------------|-----------|
| **Level 0** | Manual | Manual deploys, no CI |
| **Level 1** | Scripted | Basic scripts, some automation |
| **Level 2** | CI | Automated builds, basic tests |
| **Level 3** | CD | Automated deployments, staging |
| **Level 4** | Continuous | Full automation, feature flags |
| **Level 5** | Optimized | Metrics-driven, self-healing |

---

## 🎯 Core DevOps Practices

### 1. Infrastructure as Code (IaC)
- Todo en version control
- Infraestructura reproducible
- Review de cambios como código

### 2. Continuous Integration
- Commits frecuentes
- Build automatizado
- Tests automatizados

### 3. Continuous Delivery
- Deploy automatizado
- Ambientes reproducibles
- Rollback rápido

### 4. Monitoring & Feedback
- Métricas en tiempo real
- Alertas proactivas
- Feedback loops cortos

---

## 📈 Key Metrics (DORA)

| Metric | Elite | High | Medium | Low |
|--------|-------|------|--------|-----|
| **Deployment Frequency** | Multiple/day | Weekly | Monthly | Monthly+ |
| **Lead Time** | < 1 hour | < 1 week | < 1 month | > 1 month |
| **MTTR** | < 1 hour | < 1 day | < 1 week | > 1 week |
| **Change Failure Rate** | 0-15% | 16-30% | 31-45% | 46%+ |

---

## 📋 Learning Path

```dataview
TABLE status as "Status", created as "Created"
FROM "Programming/Software Engineering/DevOps"
WHERE file.name != "_Index"
SORT created ASC
```

---

← [[Programming/Software Engineering/Software Engineering Roadmap|Back to Software Engineering]]
