---
tags:
  - software-engineering
  - devops
  - cloud
created: 2026-01-02
status: 🔴
---
# ☁️ Cloud Fundamentals

> *"The cloud is about how you do computing, not where you do computing."* — Paul Maritz

## 🎯 What is Cloud Computing?

Cloud computing es la entrega de servicios de computación (servidores, almacenamiento, bases de datos, redes, software) a través de Internet ("la nube").

---

## 📊 Service Models

```
┌─────────────────────────────────────────────────────────────────┐
│                    CLOUD SERVICE MODELS                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│    On-Premises      IaaS          PaaS          SaaS            │
│   ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐   │
│   │Application│  │Application│  │Application│  │███████████│   │
│   │───────────│  │───────────│  │███████████│  │███████████│   │
│   │   Data    │  │   Data    │  │███████████│  │███████████│   │
│   │───────────│  │───────────│  │███████████│  │███████████│   │
│   │  Runtime  │  │  Runtime  │  │███████████│  │███████████│   │
│   │───────────│  │───────────│  │███████████│  │███████████│   │
│   │Middleware │  │Middleware │  │███████████│  │███████████│   │
│   │───────────│  │───────────│  │███████████│  │███████████│   │
│   │    O/S    │  │    O/S    │  │███████████│  │███████████│   │
│   │───────────│  │───────────│  │███████████│  │███████████│   │
│   │Virtualizn │  │███████████│  │███████████│  │███████████│   │
│   │───────────│  │███████████│  │███████████│  │███████████│   │
│   │  Servers  │  │███████████│  │███████████│  │███████████│   │
│   │───────────│  │███████████│  │███████████│  │███████████│   │
│   │  Storage  │  │███████████│  │███████████│  │███████████│   │
│   │───────────│  │███████████│  │███████████│  │███████████│   │
│   │ Networking│  │███████████│  │███████████│  │███████████│   │
│   └───────────┘  └───────────┘  └───────────┘  └───────────┘   │
│                                                                 │
│   █ = Managed by Cloud Provider                                 │
│   □ = Managed by You                                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### IaaS (Infrastructure as a Service)
- **You manage**: Applications, data, runtime, middleware, OS
- **Provider manages**: Virtualization, servers, storage, networking
- **Examples**: AWS EC2, Azure VMs, Google Compute Engine
- **Use case**: Full control, migrate existing apps

### PaaS (Platform as a Service)
- **You manage**: Applications, data
- **Provider manages**: Everything else
- **Examples**: Heroku, AWS Elastic Beanstalk, Google App Engine
- **Use case**: Focus on code, not infrastructure

### SaaS (Software as a Service)
- **You manage**: Nothing (just use it)
- **Provider manages**: Everything
- **Examples**: Gmail, Slack, Salesforce
- **Use case**: Ready-to-use applications

---

## 🏗️ AWS vs Azure vs GCP

### Compute Services

| Service Type | AWS | Azure | GCP |
|-------------|-----|-------|-----|
| **VMs** | EC2 | Virtual Machines | Compute Engine |
| **Containers** | ECS, EKS | AKS, Container Instances | GKE, Cloud Run |
| **Serverless** | Lambda | Functions | Cloud Functions |
| **App Platform** | Elastic Beanstalk | App Service | App Engine |

### Storage Services

| Service Type | AWS | Azure | GCP |
|-------------|-----|-------|-----|
| **Object Storage** | S3 | Blob Storage | Cloud Storage |
| **Block Storage** | EBS | Managed Disks | Persistent Disk |
| **File Storage** | EFS | Azure Files | Filestore |
| **Archive** | Glacier | Archive Storage | Archive |

### Database Services

| Service Type | AWS | Azure | GCP |
|-------------|-----|-------|-----|
| **Relational** | RDS, Aurora | SQL Database | Cloud SQL |
| **NoSQL** | DynamoDB | Cosmos DB | Firestore, Bigtable |
| **Cache** | ElastiCache | Cache for Redis | Memorystore |
| **Data Warehouse** | Redshift | Synapse | BigQuery |

### Networking

| Service Type | AWS | Azure | GCP |
|-------------|-----|-------|-----|
| **VPN** | VPC | Virtual Network | VPC |
| **CDN** | CloudFront | CDN | Cloud CDN |
| **Load Balancer** | ELB/ALB | Load Balancer | Cloud Load Balancing |
| **DNS** | Route 53 | DNS | Cloud DNS |

---

## 🌐 Deployment Options Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                   DEPLOYMENT SPECTRUM                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Control ◄─────────────────────────────────────────────► Ease   │
│                                                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │Bare Metal│  │   VMs    │  │Containers│  │Serverless│        │
│  │          │  │          │  │          │  │          │        │
│  │ Full     │  │ OS-level │  │ App-level│  │ Function │        │
│  │ control  │  │ control  │  │ control  │  │ level    │        │
│  │          │  │          │  │          │  │          │        │
│  │ Months   │  │ Minutes  │  │ Seconds  │  │ Instant  │        │
│  │ to setup │  │ to setup │  │ to deploy│  │ scaling  │        │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘        │
│       │              │             │             │               │
│       ▼              ▼             ▼             ▼               │
│    $$$$$          $$$$           $$$            $$               │
│   (High)        (Medium)       (Lower)        (Pay per use)     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Bare Metal
```
✅ Pros:
- Maximum performance
- Full hardware control
- No noisy neighbors
- Compliance requirements

❌ Cons:
- High upfront cost
- Long provisioning time
- Manual scaling
- Maintenance overhead
```

### Virtual Machines (VMs)
```
✅ Pros:
- Familiar model
- Good isolation
- Flexible OS choice
- Snapshot/backup easy

❌ Cons:
- Resource overhead
- Slower startup
- OS management needed
- Less efficient scaling
```

### Containers
```
✅ Pros:
- Fast startup
- Efficient resources
- Portable
- Consistent environments

❌ Cons:
- Orchestration complexity
- Networking complexity
- Stateful apps harder
- Security surface
```

### Serverless
```
✅ Pros:
- Zero ops
- Auto-scaling
- Pay per execution
- Focus on code

❌ Cons:
- Cold starts
- Vendor lock-in
- Limited runtime
- Debugging harder
- Stateless only
```

---

## 💰 Cost Optimization

### Pricing Models

| Model | Description | Savings |
|-------|-------------|---------|
| **On-Demand** | Pay per hour/second | 0% (baseline) |
| **Reserved** | 1-3 year commitment | 30-75% |
| **Spot/Preemptible** | Excess capacity | 60-90% |
| **Savings Plans** | Flexible commitment | 20-72% |

### Cost Optimization Strategies

```
┌─────────────────────────────────────────────────────────────────┐
│                   COST OPTIMIZATION                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. RIGHT-SIZING                                                │
│     • Analyze actual usage                                      │
│     • Downsize over-provisioned resources                       │
│     • Use auto-scaling                                          │
│                                                                 │
│  2. RESERVED CAPACITY                                           │
│     • Commit for predictable workloads                          │
│     • Use Savings Plans for flexibility                         │
│                                                                 │
│  3. SPOT INSTANCES                                              │
│     • Use for fault-tolerant workloads                          │
│     • Batch processing, testing                                 │
│                                                                 │
│  4. STORAGE TIERS                                               │
│     • Hot → Warm → Cold → Archive                               │
│     • Lifecycle policies                                        │
│                                                                 │
│  5. NETWORKING                                                  │
│     • Minimize data transfer                                    │
│     • Use regional endpoints                                    │
│     • CDN for static content                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔒 Security Best Practices

### Shared Responsibility Model
```
┌─────────────────────────────────────────────────────────────────┐
│               SHARED RESPONSIBILITY MODEL                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CUSTOMER RESPONSIBILITY ("Security IN the Cloud")              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ • Data encryption                                        │   │
│  │ • Identity & Access Management                           │   │
│  │ • Application security                                   │   │
│  │ • Network configuration (firewalls, security groups)     │   │
│  │ • OS patching (IaaS)                                     │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  PROVIDER RESPONSIBILITY ("Security OF the Cloud")              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ • Physical security                                      │   │
│  │ • Hardware maintenance                                   │   │
│  │ • Network infrastructure                                 │   │
│  │ • Virtualization layer                                   │   │
│  │ • Managed services (PaaS)                                │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Security Checklist
> [!check] Cloud Security Essentials
> - [ ] Enable MFA for all users
> - [ ] Use IAM roles, not access keys
> - [ ] Encrypt data at rest and in transit
> - [ ] Enable logging and monitoring
> - [ ] Use VPC/private subnets
> - [ ] Implement least privilege access
> - [ ] Regular security audits
> - [ ] Backup and disaster recovery plan

---

## 📚 Related

- [[Programming/Software Engineering/DevOps/Serverless|Serverless]]
- [[Programming/Software Engineering/System Design/Scalability|Scalability]]
- [[Programming/Software Engineering/System Design/CDN|CDN]]

---

← [[Programming/Software Engineering/DevOps/DevOps Complete Guide|Back to DevOps]]
