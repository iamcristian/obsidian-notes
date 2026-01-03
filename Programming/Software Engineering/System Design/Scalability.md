---
tags:
  - software-engineering
  - system-design
  - scalability
created: 2026-01-02
status: 🔴
---
# 📈 Scalability

> *"The ability of a system to handle growing amounts of work by adding resources."*

## 🎯 Types of Scaling

### Vertical Scaling (Scale Up)
```
BEFORE              AFTER
┌──────────┐        ┌──────────────────┐
│ 4 CPU    │   →    │ 16 CPU           │
│ 16GB RAM │        │ 64GB RAM         │
│ 500GB    │        │ 2TB SSD          │
└──────────┘        └──────────────────┘
   Small               Large Server
   Server              (More powerful)
```

**Pros:**
- Simple - no cambios de código
- Sin complejidad de distribución
- Datos en un solo lugar

**Cons:**
- Límite físico de hardware
- Single point of failure
- Costoso en escalas altas
- Downtime para upgrade

### Horizontal Scaling (Scale Out)
```
BEFORE              AFTER
┌──────────┐        ┌────────┐  ┌────────┐  ┌────────┐
│ Server   │   →    │Server 1│  │Server 2│  │Server 3│
└──────────┘        └────────┘  └────────┘  └────────┘
                          │          │          │
                    ┌─────┴──────────┴──────────┴─────┐
                    │         Load Balancer           │
                    └─────────────────────────────────┘
```

**Pros:**
- Escala "infinita"
- Alta disponibilidad
- Costo más lineal
- No single point of failure

**Cons:**
- Complejidad de coordinación
- Consistencia de datos
- Requiere código stateless

---

## 🏗️ Scaling Strategies

### 1. Application Layer Scaling
```
                    ┌─────────────────┐
                    │  Load Balancer  │
                    └────────┬────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
    ┌────┴────┐        ┌────┴────┐        ┌────┴────┐
    │ App 1   │        │ App 2   │        │ App 3   │
    └────┬────┘        └────┬────┘        └────┬────┘
         │                   │                   │
         └───────────────────┼───────────────────┘
                             │
                    ┌────────┴────────┐
                    │    Database     │
                    └─────────────────┘
```

**Requisitos:**
- Aplicación **stateless**
- Session storage externo (Redis)
- Shared file storage (S3)

```typescript
// ❌ Bad - Stateful
class UserController {
  private userCache = new Map(); // Estado local
  
  getUser(id: string) {
    if (this.userCache.has(id)) {
      return this.userCache.get(id);
    }
    // ...
  }
}

// ✅ Good - Stateless con cache externo
class UserController {
  constructor(private redis: Redis) {}
  
  async getUser(id: string) {
    const cached = await this.redis.get(`user:${id}`);
    if (cached) return JSON.parse(cached);
    // ...
  }
}
```

### 2. Database Layer Scaling

#### Read Replicas
```
                    ┌─────────────────┐
                    │  Master (Write) │
                    └────────┬────────┘
                             │
            ┌────────────────┼────────────────┐
            │                │                │
     ┌──────┴──────┐  ┌──────┴──────┐  ┌─────┴───────┐
     │ Replica 1   │  │ Replica 2   │  │ Replica 3   │
     │  (Read)     │  │  (Read)     │  │  (Read)     │
     └─────────────┘  └─────────────┘  └─────────────┘
```

#### Sharding (Partitioning)
```
        User ID: 1-1M        User ID: 1M-2M       User ID: 2M-3M
     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
     │   Shard 1    │     │   Shard 2    │     │   Shard 3    │
     │  Users DB    │     │  Users DB    │     │  Users DB    │
     └──────────────┘     └──────────────┘     └──────────────┘
```

```typescript
// Sharding by user ID
function getShardForUser(userId: number): number {
  return userId % NUM_SHARDS;
}

// Sharding by geography
function getShardForUser(user: User): string {
  const region = user.country;
  if (['US', 'CA', 'MX'].includes(region)) return 'shard-americas';
  if (['UK', 'FR', 'DE'].includes(region)) return 'shard-europe';
  return 'shard-asia';
}
```

### 3. Caching Strategy
```
              ┌─────────────┐
Request ──────►   CDN       │ (Static assets)
              └──────┬──────┘
                     │ cache miss
              ┌──────▼──────┐
              │ App Server  │
              └──────┬──────┘
                     │
              ┌──────▼──────┐
              │   Redis     │ (Session, hot data)
              └──────┬──────┘
                     │ cache miss
              ┌──────▼──────┐
              │  Database   │
              └─────────────┘
```

---

## 📊 Scaling Metrics

### Key Performance Indicators
```typescript
interface ScalingMetrics {
  // Throughput
  requestsPerSecond: number;
  transactionsPerSecond: number;
  
  // Latency
  p50ResponseTime: number;  // 50th percentile
  p95ResponseTime: number;  // 95th percentile
  p99ResponseTime: number;  // 99th percentile
  
  // Resources
  cpuUtilization: number;   // Target: < 70%
  memoryUtilization: number;
  diskIOPS: number;
  
  // Availability
  uptime: number;           // Target: 99.9%+
  errorRate: number;        // Target: < 0.1%
}
```

### Auto-Scaling Rules
```yaml
# Kubernetes HPA example
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-service
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

---

## 🎯 Scaling Decision Tree

```
¿Necesitas escalar?
        │
        ▼
┌───────────────────┐
│ ¿Cuál es el       │
│ bottleneck?       │
└───────┬───────────┘
        │
  ┌─────┼─────────────────┐
  │     │                 │
  ▼     ▼                 ▼
 CPU   Memory           Database
  │     │                 │
  ▼     ▼                 ▼
Scale   Add            Read-heavy? → Add Replicas
 Out    Memory          │
        Cache          Write-heavy? → Sharding
                        │
                       Both? → CQRS + Sharding
```

---

## ✅ Best Practices

| Practice | Description |
|----------|-------------|
| **Start Simple** | No over-engineer desde el inicio |
| **Measure First** | Identifica bottlenecks reales |
| **Stateless Apps** | Facilita horizontal scaling |
| **Cache Aggressively** | Reduce load en DB |
| **Database Indexing** | Antes de escalar, optimiza |
| **Async Processing** | Queues para trabajo pesado |

---

## 💡 Rules of Thumb

> [!tip] Scaling Numbers
> - **1 server** puede manejar ~1000 req/s (simple app)
> - **1 PostgreSQL** puede manejar ~10K queries/s
> - **1 Redis** puede manejar ~100K ops/s
> - **Rule of 10x**: Diseña para 10x tu carga actual

---

← [[Programming/Software Engineering/System Design/_Index|Back to System Design]]
