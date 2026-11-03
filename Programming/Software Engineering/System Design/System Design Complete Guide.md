---
tags:
  - software-engineering
  - system-design
  - guide
  - complete
created: 2026-01-02
---
# 🌐 System Design - Guía Completa

> *"El arte de diseñar sistemas distribuidos que sean escalables, confiables y mantenibles."*

---

## 📋 Tabla de Contenidos

1. [Framework para System Design](#-framework-para-system-design)
2. [Scalability](#-scalability)
3. [Load Balancing](#-load-balancing)
4. [Caching](#-caching)
5. [Databases](#-databases)
6. [CAP Theorem](#-cap-theorem)
7. [Message Queues](#-message-queues)
8. [API Design](#-api-design)
9. [CDN](#-cdn-content-delivery-network)
10. [ETL Pipelines](#-etl-pipelines)
11. [Rate Limiting](#-rate-limiting)
12. [Case Studies](#-case-studies)

---

## 🎯 Framework para System Design

### Paso 1: Clarificar Requisitos (3-5 min)

```markdown
Preguntas Funcionales:
- ¿Qué funcionalidades son core vs nice-to-have?
- ¿Quiénes son los usuarios?
- ¿Cómo van a usar el sistema?

Preguntas No Funcionales:
- ¿Cuántos usuarios? (DAU/MAU)
- ¿Read-heavy o write-heavy?
- ¿Consistencia o disponibilidad?
- ¿Latencia aceptable? (p99 < 200ms?)
- ¿Disponibilidad requerida? (99.9%?)
```

### Paso 2: Estimaciones (5 min)

```markdown
Ejemplo: Sistema de URL Shortener

Usuarios:
- 100M usuarios activos mensuales
- 10% crean URLs = 10M writes/mes
- 90% solo leen = 100:1 read/write ratio

Traffic:
- Writes: 10M / 30 días / 86400 seg ≈ 4 writes/seg
- Reads: 400 reads/seg

Storage (5 años):
- 10M URLs/mes × 12 × 5 = 600M URLs
- Cada URL ≈ 500 bytes
- Total: 600M × 500B = 300GB

Bandwidth:
- Writes: 4 × 500B = 2KB/seg
- Reads: 400 × 500B = 200KB/seg
```

### Paso 3: API Design (5 min)

```typescript
// Definir endpoints principales
POST   /api/v1/urls          // Crear short URL
GET    /api/v1/urls/{id}     // Obtener URL original
DELETE /api/v1/urls/{id}     // Eliminar URL
GET    /api/v1/urls/{id}/stats // Analytics
```

### Paso 4: High-Level Design (10 min)

```
┌──────────┐     ┌──────────────┐     ┌─────────────┐
│  Client  │────▶│ Load Balancer│────▶│ App Servers │
└──────────┘     └──────────────┘     └─────────────┘
                                            │
                 ┌──────────────┬───────────┴────────┐
                 ▼              ▼                    ▼
           ┌─────────┐   ┌───────────┐        ┌─────────┐
           │  Cache  │   │  Database │        │   CDN   │
           └─────────┘   └───────────┘        └─────────┘
```

### Paso 5: Deep Dive (15-20 min)

```markdown
- Identificar bottlenecks
- Proponer soluciones
- Discutir trade-offs
- Considerar edge cases
- Planes de monitoreo
```

---

## 📈 Scalability

### ¿Qué es Scalability?

La capacidad de un sistema para manejar crecimiento de carga de trabajo añadiendo recursos.

### Vertical Scaling (Scale Up)

```
┌─────────────────────┐
│   ANTES             │
│   ┌───────────┐     │
│   │  2 CPU    │     │
│   │  4GB RAM  │     │
│   │  100GB SSD│     │
│   └───────────┘     │
└─────────────────────┘
         ↓
┌─────────────────────┐
│   DESPUÉS           │
│   ┌───────────┐     │
│   │  16 CPU   │     │
│   │  64GB RAM │     │
│   │  1TB SSD  │     │
│   └───────────┘     │
└─────────────────────┘
```

**Ventajas:**
- ✅ Simple de implementar
- ✅ No requiere cambios en código
- ✅ Sin complejidad de datos distribuidos

**Desventajas:**
- ❌ Límite físico de hardware
- ❌ Single point of failure
- ❌ Costoso en hardware premium
- ❌ Downtime durante upgrade

### Horizontal Scaling (Scale Out)

```
ANTES:
┌───────────┐
│  Server   │
└───────────┘

DESPUÉS:
┌───────────┐  ┌───────────┐  ┌───────────┐
│  Server 1 │  │  Server 2 │  │  Server 3 │
└───────────┘  └───────────┘  └───────────┘
      ▲              ▲              ▲
      └──────────────┼──────────────┘
                     │
              Load Balancer
```

**Ventajas:**
- ✅ Escalado ilimitado (teóricamente)
- ✅ Alta disponibilidad
- ✅ Costo-efectivo con commodity hardware
- ✅ No hay single point of failure

**Desventajas:**
- ❌ Complejidad de código (stateless requerido)
- ❌ Consistencia de datos más difícil
- ❌ Network latency entre nodos
- ❌ Más complejo de mantener

### Stateless vs Stateful

```typescript
// ❌ STATEFUL - No escala horizontalmente
class ShoppingCart {
  private items: Item[] = []; // Estado en memoria
  
  addItem(item: Item) {
    this.items.push(item);
  }
}

// ✅ STATELESS - Escala horizontalmente
class ShoppingCartService {
  constructor(private cache: Redis, private db: Database) {}
  
  async addItem(userId: string, item: Item) {
    // Estado en storage externo
    await this.cache.lpush(`cart:${userId}`, JSON.stringify(item));
  }
  
  async getCart(userId: string): Promise<Item[]> {
    return await this.cache.lrange(`cart:${userId}`, 0, -1);
  }
}
```

### Cuándo Usar Cada Uno

| Escenario | Recomendación |
|-----------|---------------|
| Startup pequeño | Vertical primero |
| Base de datos | Vertical + Read replicas |
| Aplicación web | Horizontal |
| Procesamiento batch | Horizontal |
| Alta disponibilidad crítica | Horizontal |
| Presupuesto limitado | Vertical |

---

## ⚖️ Load Balancing

### ¿Qué es Load Balancing?

Distribuye el tráfico entrante entre múltiples servidores para asegurar que ningún servidor esté sobrecargado.

### Arquitectura

```
                    ┌────────────┐
                    │   Client   │
                    └─────┬──────┘
                          │
                          ▼
                 ┌─────────────────┐
                 │  Load Balancer  │
                 │   (L4 / L7)     │
                 └────────┬────────┘
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
  ┌───────────┐    ┌───────────┐    ┌───────────┐
  │  Server 1 │    │  Server 2 │    │  Server 3 │
  │  (Healthy)│    │  (Healthy)│    │  (Down)   │
  └───────────┘    └───────────┘    └───────────┘
```

### Tipos de Load Balancers

#### Layer 4 (Transport Layer)

```
Trabaja con: IP + Puerto
Ventajas: Muy rápido, simple
Desventajas: No puede inspeccionar contenido

Ejemplo: TCP/UDP forwarding
┌─────────┐      ┌──────┐      ┌─────────┐
│ Client  │─TCP─▶│  L4  │─TCP─▶│ Server  │
└─────────┘      │  LB  │      └─────────┘
                 └──────┘
```

#### Layer 7 (Application Layer)

```
Trabaja con: HTTP headers, cookies, URL path
Ventajas: Routing inteligente, SSL termination
Desventajas: Más overhead

Ejemplo: Routing por path
/api/*     → API Servers
/static/*  → Static Servers
/admin/*   → Admin Servers
```

### Algoritmos de Balanceo

#### 1. Round Robin

```typescript
class RoundRobinLB {
  private servers: string[];
  private current = 0;

  getNextServer(): string {
    const server = this.servers[this.current];
    this.current = (this.current + 1) % this.servers.length;
    return server;
  }
}

// Resultado: S1 → S2 → S3 → S1 → S2 → S3...
```

**Uso:** Servidores con capacidad similar.

#### 2. Weighted Round Robin

```typescript
class WeightedRoundRobinLB {
  // Server A: weight 3 (más potente)
  // Server B: weight 1 (menos potente)
  // Resultado: A, A, A, B, A, A, A, B...
  
  private servers = [
    { host: 'server-a', weight: 3 },
    { host: 'server-b', weight: 1 }
  ];
}
```

**Uso:** Servidores con diferente capacidad.

#### 3. Least Connections

```typescript
class LeastConnectionsLB {
  private connections: Map<string, number> = new Map();

  getNextServer(): string {
    // Retorna el servidor con menos conexiones activas
    let minConnections = Infinity;
    let selectedServer = '';
    
    for (const [server, count] of this.connections) {
      if (count < minConnections) {
        minConnections = count;
        selectedServer = server;
      }
    }
    return selectedServer;
  }
}
```

**Uso:** Cuando las requests tienen tiempos de respuesta variables.

#### 4. IP Hash

```typescript
class IPHashLB {
  private servers: string[];

  getServer(clientIP: string): string {
    const hash = this.hashIP(clientIP);
    const index = hash % this.servers.length;
    return this.servers[index];
  }

  private hashIP(ip: string): number {
    return ip.split('.').reduce((acc, octet) => acc + parseInt(octet), 0);
  }
}
```

**Uso:** Cuando necesitas session affinity (sticky sessions).

#### 5. Least Response Time

```typescript
// Combina:
// - Menor número de conexiones activas
// - Menor tiempo de respuesta promedio
// Ideal para optimizar experiencia de usuario
```

### Health Checks

```typescript
// Configuración típica de health check
const healthCheckConfig = {
  interval: 10,        // Cada 10 segundos
  timeout: 5,          // 5 segundos timeout
  unhealthyThreshold: 3, // 3 fallos = unhealthy
  healthyThreshold: 2,   // 2 éxitos = healthy de nuevo
  path: '/health',     // Endpoint a verificar
  expectedStatus: 200  // Código esperado
};

// Endpoint de health check
app.get('/health', (req, res) => {
  const dbHealthy = checkDatabaseConnection();
  const cacheHealthy = checkCacheConnection();
  
  if (dbHealthy && cacheHealthy) {
    res.status(200).json({ status: 'healthy' });
  } else {
    res.status(503).json({ status: 'unhealthy' });
  }
});
```

### Load Balancers Populares

| Tool | Tipo | Uso Común |
|------|------|-----------|
| **NGINX** | L7 | Web apps, reverse proxy |
| **HAProxy** | L4/L7 | High performance |
| **AWS ALB** | L7 | AWS applications |
| **AWS NLB** | L4 | Ultra-low latency |
| **Cloudflare** | L7 | Global CDN + LB |

---

## 🚀 Caching

### ¿Qué es Caching?

Almacenar datos frecuentemente accedidos en una capa de alta velocidad para reducir latencia y carga en la base de datos.

### Arquitectura de Cache

```
┌──────────┐    ┌─────────┐    ┌──────────────┐    ┌──────────┐
│  Client  │───▶│  Cache  │───▶│ App Server   │───▶│ Database │
└──────────┘    │ (Redis) │    └──────────────┘    └──────────┘
                └─────────┘           │
                     ▲                │
                     └────────────────┘
                     Cache population
```

### Cache Strategies

#### 1. Cache-Aside (Lazy Loading)

```typescript
class CacheAsideService {
  constructor(
    private cache: Redis,
    private db: Database
  ) {}

  async getUser(id: string): Promise<User> {
    // 1. Intentar obtener del cache
    const cached = await this.cache.get(`user:${id}`);
    if (cached) {
      return JSON.parse(cached); // Cache HIT
    }

    // 2. Cache MISS - obtener de DB
    const user = await this.db.findUser(id);
    
    // 3. Guardar en cache para próxima vez
    await this.cache.setex(`user:${id}`, 3600, JSON.stringify(user));
    
    return user;
  }
}
```

**Ventajas:** Solo cachea datos que se piden, resiliente a fallos de cache.
**Desventajas:** Primera request siempre lenta, datos pueden quedar stale.

#### 2. Write-Through

```typescript
class WriteThroughService {
  async updateUser(id: string, data: UserUpdate): Promise<User> {
    // 1. Escribir a DB
    const user = await this.db.updateUser(id, data);
    
    // 2. Actualizar cache inmediatamente
    await this.cache.setex(`user:${id}`, 3600, JSON.stringify(user));
    
    return user;
  }
}
```

**Ventajas:** Cache siempre consistente con DB.
**Desventajas:** Latencia de escritura mayor, cachea datos que quizás nunca se leen.

#### 3. Write-Behind (Write-Back)

```typescript
class WriteBehindService {
  private writeQueue: Map<string, any> = new Map();

  async updateUser(id: string, data: UserUpdate): Promise<void> {
    // 1. Escribir solo al cache (rápido)
    await this.cache.setex(`user:${id}`, 3600, JSON.stringify(data));
    
    // 2. Encolar para escritura async a DB
    this.writeQueue.set(`user:${id}`, data);
  }

  // Background job que sincroniza a DB
  @Cron('*/5 * * * * *') // Cada 5 segundos
  async flushToDatabase() {
    for (const [key, data] of this.writeQueue) {
      await this.db.update(key, data);
      this.writeQueue.delete(key);
    }
  }
}
```

**Ventajas:** Escrituras muy rápidas, reduce carga en DB.
**Desventajas:** Riesgo de pérdida de datos si cache falla.

#### 4. Read-Through

```typescript
// El cache maneja automáticamente los misses
// Configuración en Redis/Memcached enterprise

const cacheConfig = {
  readThrough: true,
  loader: async (key: string) => {
    return await database.get(key);
  }
};
```

### Cache Eviction Policies

| Policy | Descripción | Uso |
|--------|-------------|-----|
| **LRU** | Least Recently Used | General, más común |
| **LFU** | Least Frequently Used | Datos con popularidad variable |
| **FIFO** | First In First Out | Simple, predecible |
| **TTL** | Time To Live | Datos con expiración natural |
| **Random** | Evicción aleatoria | Cuando LRU es muy costoso |

### TTL Strategies

```typescript
// Diferentes TTL según tipo de dato
const TTL = {
  USER_SESSION: 30 * 60,      // 30 minutos
  USER_PROFILE: 60 * 60,      // 1 hora
  PRODUCT_LIST: 5 * 60,       // 5 minutos
  STATIC_CONFIG: 24 * 60 * 60 // 24 horas
};

// Cache con TTL
await redis.setex('user:123', TTL.USER_PROFILE, userData);
```

### Cache Invalidation

```typescript
// El problema más difícil en computer science

// 1. TTL-based (automático)
await cache.setex(key, 3600, value); // Expira en 1 hora

// 2. Event-based (manual)
class UserService {
  async updateUser(id: string, data: User) {
    await this.db.update(id, data);
    await this.cache.del(`user:${id}`);        // Invalidar específico
    await this.cache.del('users:list');         // Invalidar lista
    await this.eventBus.publish('user.updated', { id }); // Notificar otros servicios
  }
}

// 3. Version-based
const cacheKey = `user:${id}:v${user.version}`;
```

### Herramientas de Cache

| Tool | Tipo | Uso |
|------|------|-----|
| **Redis** | In-memory | Cache distribuido, session storage |
| **Memcached** | In-memory | Cache simple, alta velocidad |
| **CDN** | Edge cache | Archivos estáticos |
| **Browser Cache** | Client-side | Assets, API responses |
| **Application Cache** | In-process | Configuración, datos estáticos |

---

## 💾 Databases

### SQL vs NoSQL

#### SQL (Relational)

```sql
-- Estructura rígida, ACID compliant
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(100),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    total DECIMAL(10,2),
    status VARCHAR(20)
);

-- Relaciones con JOINs
SELECT u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id;
```

**Cuándo usar SQL:**
- Transacciones complejas (bancos, e-commerce)
- Datos relacionados que requieren JOINs
- Consistencia es crítica
- Schema bien definido

#### NoSQL (Document Store - MongoDB)

```javascript
// Estructura flexible, schema-less
{
  "_id": "user123",
  "email": "user@example.com",
  "name": "John Doe",
  "profile": {
    "bio": "Software Engineer",
    "avatar": "https://..."
  },
  "orders": [
    { "id": "order1", "total": 99.99, "items": [...] },
    { "id": "order2", "total": 149.99, "items": [...] }
  ]
}
```

**Cuándo usar NoSQL:**
- Esquema cambia frecuentemente
- Datos semi-estructurados (logs, eventos)
- Scale horizontal masivo
- Read-heavy workloads

### Database Scaling

#### 1. Read Replicas

```
                    ┌──────────────┐
    Writes ────────▶│    Master    │
                    └──────┬───────┘
                           │ Replication
              ┌────────────┼────────────┐
              ▼            ▼            ▼
        ┌──────────┐ ┌──────────┐ ┌──────────┐
Reads ──│ Replica 1│ │ Replica 2│ │ Replica 3│
        └──────────┘ └──────────┘ └──────────┘
```

```typescript
class DatabaseService {
  private master: Connection;
  private replicas: Connection[];
  private replicaIndex = 0;

  async write(query: string): Promise<any> {
    // Escrituras siempre al master
    return this.master.execute(query);
  }

  async read(query: string): Promise<any> {
    // Lecturas distribuidas entre réplicas
    const replica = this.replicas[this.replicaIndex];
    this.replicaIndex = (this.replicaIndex + 1) % this.replicas.length;
    return replica.execute(query);
  }
}
```

#### 2. Sharding (Particionamiento Horizontal)

```
User ID 1-1M      User ID 1M-2M     User ID 2M-3M
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   Shard 1    │  │   Shard 2    │  │   Shard 3    │
│  users 1-1M  │  │ users 1M-2M  │  │ users 2M-3M  │
└──────────────┘  └──────────────┘  └──────────────┘
```

**Estrategias de Sharding:**

```typescript
// 1. Range-based sharding
function getShard(userId: number): string {
  if (userId < 1000000) return 'shard1';
  if (userId < 2000000) return 'shard2';
  return 'shard3';
}

// 2. Hash-based sharding (más uniforme)
function getShard(userId: number, numShards: number): number {
  return hashFunction(userId) % numShards;
}

// 3. Directory-based sharding
const shardDirectory = {
  'user:123': 'shard1',
  'user:456': 'shard2'
};
```

**Problemas del Sharding:**
- JOINs entre shards muy costosos
- Rebalanceo complejo al añadir shards
- Transacciones distribuidas difíciles

#### 3. Indexes

```sql
-- Sin índice: O(n) full table scan
SELECT * FROM users WHERE email = 'user@example.com';
-- Con 1M rows = 1M comparaciones

-- Con índice: O(log n) B-tree lookup
CREATE INDEX idx_users_email ON users(email);
-- Con 1M rows = ~20 comparaciones

-- Índice compuesto para queries frecuentes
CREATE INDEX idx_orders_user_status 
ON orders(user_id, status);

-- Covering index (incluye datos en el índice)
CREATE INDEX idx_users_email_name 
ON users(email) INCLUDE (name);
```

### Normalización vs Denormalización

```sql
-- NORMALIZADO (3NF)
-- Ventaja: No redundancia, fácil de actualizar
-- Desventaja: Requiere JOINs

users: id, name, email
addresses: id, user_id, street, city
orders: id, user_id, total

SELECT u.name, a.city, o.total
FROM users u
JOIN addresses a ON u.id = a.user_id
JOIN orders o ON u.id = o.user_id;

-- DENORMALIZADO
-- Ventaja: Lecturas rápidas, sin JOINs
-- Desventaja: Redundancia, updates complejos

orders: id, user_id, user_name, user_email, 
        shipping_street, shipping_city, total
```

**Regla general:** Normaliza para escrituras, denormaliza para lecturas.

---

## 📐 CAP Theorem

### Los Tres Pilares

```
                    Consistency
                        /\
                       /  \
                      /    \
                     /  CP  \
                    /________\
                   /\        /\
                  /  \  CA  /  \
                 / AP \    /    \
                /______\  /______\
         Availability    Partition Tolerance
```

**Solo puedes elegir 2 de 3** (en realidad, debes elegir entre C y A cuando hay P).

### Definiciones

| Propiedad | Significado |
|-----------|-------------|
| **Consistency** | Todos los nodos ven los mismos datos al mismo tiempo |
| **Availability** | Cada request recibe respuesta (success o failure) |
| **Partition Tolerance** | El sistema sigue funcionando aunque se pierda comunicación entre nodos |

### En la Práctica

```
Red partitions SIEMPRE pueden ocurrir en sistemas distribuidos.
Por lo tanto, siempre debes elegir P.

La verdadera elección es: Consistency (CP) vs Availability (AP)
```

### Sistemas CP (Consistency + Partition Tolerance)

```typescript
// Ejemplo: Sistema bancario
// Preferimos rechazar operación que dar datos inconsistentes

async function transfer(from: string, to: string, amount: number) {
  const lock = await acquireDistributedLock([from, to]);
  try {
    // Si no podemos alcanzar todos los nodos, FALLA
    const fromBalance = await getBalanceStrong(from); // Espera consenso
    if (fromBalance < amount) {
      throw new InsufficientFundsError();
    }
    
    await updateBalanceAllNodes(from, fromBalance - amount);
    await updateBalanceAllNodes(to, await getBalanceStrong(to) + amount);
  } finally {
    await releaseLock(lock);
  }
}
```

**Ejemplos:** MongoDB (modo por defecto), Redis Cluster, HBase, Zookeeper

### Sistemas AP (Availability + Partition Tolerance)

```typescript
// Ejemplo: Feed de redes sociales
// Preferimos mostrar datos potencialmente desactualizados que no mostrar nada

async function getFeed(userId: string): Promise<Post[]> {
  try {
    // Intenta obtener del nodo más cercano
    return await nearestNode.getFeed(userId);
  } catch (e) {
    // Si falla, retorna datos cacheados (quizás stale)
    return await localCache.getFeed(userId);
  }
}

// Eventual consistency: los datos se sincronizarán eventualmente
```

**Ejemplos:** Cassandra, DynamoDB, CouchDB, DNS

### PACELC Extension

```
Si hay Partition (P):
  ¿Elegir Availability (A) o Consistency (C)?

Si NO hay Partition (E - Else):
  ¿Elegir Latency (L) o Consistency (C)?

Ejemplo:
- DynamoDB: PA/EL (Availability y Latency primero)
- MongoDB: PC/EC (Consistency primero)
- Cassandra: PA/EL (configurable)
```

---

## 📨 Message Queues

### ¿Por Qué Usar Message Queues?

```
SIN QUEUE (Síncrono):
┌────────┐     ┌─────────┐     ┌─────────┐
│ Client │────▶│ Service │────▶│ Email   │ ← Bloqueado esperando
└────────┘     │   API   │     │ Service │
               └─────────┘     └─────────┘
Total time: 200ms + 500ms = 700ms

CON QUEUE (Asíncrono):
┌────────┐     ┌─────────┐     ┌───────┐     ┌─────────┐
│ Client │────▶│ Service │────▶│ Queue │────▶│ Email   │
└────────┘     │   API   │     └───────┘     │ Worker  │
               └─────────┘                   └─────────┘
Response time: 200ms (email se envía async)
```

### Beneficios

1. **Desacoplamiento:** Servicios no dependen directamente entre sí
2. **Escalabilidad:** Añadir más workers para procesar mensajes
3. **Resiliencia:** Mensajes persisten si un servicio cae
4. **Rate Limiting:** Controlar velocidad de procesamiento
5. **Retry Logic:** Reintentar mensajes fallidos automáticamente

### Patrones de Mensajería

#### 1. Point-to-Point (Queue)

```
Productor ───▶ [Queue] ───▶ Consumidor

Un mensaje = Un consumidor
Ejemplo: Job processing, task distribution
```

```typescript
// Productor
await queue.send('email-queue', {
  to: 'user@example.com',
  subject: 'Welcome!',
  body: 'Thanks for signing up'
});

// Consumidor (Worker)
queue.consume('email-queue', async (message) => {
  await emailService.send(message);
  message.ack(); // Confirmar procesamiento
});
```

#### 2. Pub/Sub (Topic)

```
                    ┌──────────────┐
                    │  Subscriber  │ (Notification Service)
                    └──────────────┘
                          ▲
Publisher ───▶ [Topic] ───┼───▶ Subscriber (Analytics Service)
                          ▼
                    ┌──────────────┐
                    │  Subscriber  │ (Audit Service)
                    └──────────────┘

Un mensaje = Múltiples consumidores
Ejemplo: Events, notifications, logs
```

```typescript
// Publisher
await topic.publish('user-events', {
  type: 'USER_REGISTERED',
  userId: '123',
  timestamp: Date.now()
});

// Subscribers (cada uno recibe el mensaje)
topic.subscribe('user-events', 'notification-service', async (event) => {
  await sendWelcomeNotification(event.userId);
});

topic.subscribe('user-events', 'analytics-service', async (event) => {
  await trackUserRegistration(event.userId);
});
```

### Message Delivery Guarantees

| Garantía | Descripción | Uso |
|----------|-------------|-----|
| **At-most-once** | Mensaje se entrega 0 o 1 vez | Logs, métricas no críticas |
| **At-least-once** | Mensaje se entrega 1 o más veces | Emails, notificaciones |
| **Exactly-once** | Mensaje se entrega exactamente 1 vez | Transacciones financieras |

```typescript
// At-least-once con idempotencia
class PaymentProcessor {
  async processPayment(message: PaymentMessage) {
    const { paymentId, amount } = message;
    
    // Verificar si ya se procesó (idempotencia)
    const existing = await db.findPayment(paymentId);
    if (existing) {
      console.log('Payment already processed, skipping');
      return;
    }
    
    // Procesar
    await db.createPayment({ id: paymentId, amount, status: 'completed' });
    message.ack();
  }
}
```

### Dead Letter Queue (DLQ)

```
┌──────────────┐     ┌────────────┐
│    Queue     │────▶│   Worker   │
└──────────────┘     └────────────┘
       │                   │
       │ Retry 3x failed   │
       ▼                   │
┌──────────────┐           │
│     DLQ      │◀──────────┘
│ (Dead Letter)│
└──────────────┘
       │
       ▼
  Manual review / Alert
```

```typescript
const queueConfig = {
  maxRetries: 3,
  retryDelay: [1000, 5000, 15000], // Exponential backoff
  deadLetterQueue: 'failed-messages-dlq'
};
```

### Herramientas de Message Queues

| Tool | Tipo | Características |
|------|------|-----------------|
| **RabbitMQ** | Message Broker | AMQP, routing flexible, plugins |
| **Apache Kafka** | Event Streaming | Alta throughput, replay de eventos |
| **AWS SQS** | Managed Queue | Simple, serverless, integración AWS |
| **AWS SNS** | Pub/Sub | Push notifications, multi-protocol |
| **Redis Pub/Sub** | In-memory | Ultra rápido, no persistencia |

### Kafka vs RabbitMQ

| Aspecto | Kafka | RabbitMQ |
|---------|-------|----------|
| **Modelo** | Log distribuido | Message broker |
| **Throughput** | Millones msg/seg | Miles msg/seg |
| **Retención** | Configurable (días/semanas) | Hasta consumir |
| **Replay** | ✅ Sí | ❌ No |
| **Orden** | Por partición | Por cola |
| **Complejidad** | Alta | Media |
| **Uso ideal** | Event sourcing, logs | Task queues, RPC |

---

## 🔌 API Design

### REST API

#### Principios REST

```typescript
// 1. Recursos como URLs
GET    /users          // Listar usuarios
GET    /users/123      // Obtener usuario específico
POST   /users          // Crear usuario
PUT    /users/123      // Actualizar usuario completo
PATCH  /users/123      // Actualizar parcialmente
DELETE /users/123      // Eliminar usuario

// 2. Recursos anidados
GET    /users/123/orders        // Órdenes del usuario
GET    /users/123/orders/456    // Orden específica
POST   /users/123/orders        // Crear orden para usuario
```

#### HTTP Status Codes

```typescript
// 2xx - Success
200 OK              // Request exitosa
201 Created         // Recurso creado
204 No Content      // Éxito sin body (DELETE)

// 4xx - Client Errors
400 Bad Request     // Request malformada
401 Unauthorized    // No autenticado
403 Forbidden       // No autorizado
404 Not Found       // Recurso no existe
409 Conflict        // Conflicto (duplicate)
422 Unprocessable   // Validación fallida
429 Too Many Requests // Rate limited

// 5xx - Server Errors
500 Internal Error  // Error del servidor
502 Bad Gateway     // Upstream server error
503 Service Unavailable // Servidor down
504 Gateway Timeout // Upstream timeout
```

#### Paginación

```typescript
// Offset-based (simple pero ineficiente para datasets grandes)
GET /users?page=2&limit=20

{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 150,
    "totalPages": 8
  }
}

// Cursor-based (eficiente para datos en tiempo real)
GET /users?cursor=abc123&limit=20

{
  "data": [...],
  "pagination": {
    "nextCursor": "xyz789",
    "hasMore": true
  }
}
```

#### Versionamiento

```typescript
// 1. URL Path (más común)
GET /api/v1/users
GET /api/v2/users

// 2. Header
GET /api/users
Headers: API-Version: 2

// 3. Query Parameter
GET /api/users?version=2
```

### GraphQL

```graphql
# Schema
type User {
  id: ID!
  name: String!
  email: String!
  orders: [Order!]!
}

type Order {
  id: ID!
  total: Float!
  status: String!
}

type Query {
  user(id: ID!): User
  users(limit: Int): [User!]!
}

type Mutation {
  createUser(name: String!, email: String!): User!
  updateUser(id: ID!, name: String): User!
}

# Query - Cliente pide exactamente lo que necesita
query {
  user(id: "123") {
    name
    email
    orders {
      total
    }
  }
}
```

**Ventajas GraphQL:**
- ✅ No over-fetching/under-fetching
- ✅ Un endpoint para todo
- ✅ Fuertemente tipado
- ✅ Excelente para mobile (reduce bandwidth)

**Desventajas GraphQL:**
- ❌ Caching más complejo (no HTTP cache)
- ❌ Queries N+1 fáciles de crear
- ❌ Curva de aprendizaje
- ❌ Difícil de rate-limit

### gRPC

```protobuf
// user.proto - Define el contrato
syntax = "proto3";

service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc CreateUser(CreateUserRequest) returns (User);
  rpc ListUsers(ListUsersRequest) returns (stream User); // Streaming
}

message User {
  string id = 1;
  string name = 2;
  string email = 3;
}

message GetUserRequest {
  string id = 1;
}
```

```typescript
// Cliente gRPC
const client = new UserServiceClient('localhost:50051');

const user = await client.getUser({ id: '123' });
console.log(user.name);
```

**Ventajas gRPC:**
- ✅ Muy rápido (Protocol Buffers binario)
- ✅ Streaming bidireccional
- ✅ Generación automática de código
- ✅ Ideal para microservicios internos

**Desventajas gRPC:**
- ❌ No funciona directo en browsers
- ❌ Más difícil de debuggear (binario)
- ❌ Requiere herramientas especiales

### Comparativa

| Aspecto | REST | GraphQL | gRPC |
|---------|------|---------|------|
| **Formato** | JSON | JSON | Protocol Buffers |
| **Velocidad** | Media | Media | Alta |
| **Caching** | Fácil (HTTP) | Complejo | Difícil |
| **Browser** | ✅ | ✅ | ❌ (necesita proxy) |
| **Uso** | APIs públicas | Apps con datos complejos | Microservicios |

---

## 🌍 CDN (Content Delivery Network)

### ¿Qué es un CDN?

Red de servidores distribuidos geográficamente que cachean contenido cerca de los usuarios.

### Arquitectura

```
Usuario en Madrid         Usuario en Tokyo
      │                         │
      ▼                         ▼
┌──────────────┐         ┌──────────────┐
│  Edge Server │         │  Edge Server │
│   (Madrid)   │         │   (Tokyo)    │
└──────────────┘         └──────────────┘
      │                         │
      └─────────┬───────────────┘
                │
                ▼
         ┌──────────────┐
         │ Origin Server│
         │   (US East)  │
         └──────────────┘

Sin CDN: Madrid → US East = 150ms
Con CDN: Madrid → Madrid Edge = 20ms
```

### Qué Cachear en CDN

```typescript
// ✅ IDEAL para CDN
- Imágenes, videos, audio
- CSS, JavaScript bundles
- Fuentes (fonts)
- PDFs, documentos estáticos
- HTML estático

// ⚠️ CON CUIDADO
- API responses (con headers correctos)
- HTML dinámico (con short TTL)

// ❌ NO CACHEAR
- Datos de usuario privados
- Contenido personalizado
- Transacciones en tiempo real
```

### Cache Headers

```typescript
// Cache por 1 año (assets con hash en nombre)
app.use('/static', (req, res, next) => {
  res.setHeader('Cache-Control', 'public, max-age=31536000, immutable');
  next();
});

// Cache por 5 minutos, revalidar después
res.setHeader('Cache-Control', 'public, max-age=300, stale-while-revalidate=60');

// No cachear
res.setHeader('Cache-Control', 'no-store, no-cache, must-revalidate');
```

### Cache Invalidation Strategies

```typescript
// 1. Cache Busting con hash en filename
// main.js → main.abc123.js
// Cuando cambias, nuevo hash = nuevo URL = nuevo cache

// 2. Purge manual via API
await cdnProvider.purge([
  'https://example.com/api/products',
  'https://example.com/images/*'
]);

// 3. Tag-based invalidation
// Cachear con tags, invalidar por tag
await cdnProvider.purgeByTag('product-123');
```

### CDN Providers

| Provider | Fortaleza |
|----------|-----------|
| **Cloudflare** | DDoS protection, Workers (edge computing) |
| **AWS CloudFront** | Integración AWS, Lambda@Edge |
| **Akamai** | Enterprise, red más grande |
| **Fastly** | Purge instantáneo, configuración flexible |
| **Vercel/Netlify** | Optimizado para JAMstack |

---

## 🔄 ETL Pipelines

### ¿Qué es ETL?

**E**xtract - **T**ransform - **L**oad: Proceso para mover datos entre sistemas.

### Arquitectura ETL

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│    EXTRACT      │    │   TRANSFORM     │    │     LOAD        │
│                 │    │                 │    │                 │
│ - Databases     │───▶│ - Clean data    │───▶│ - Data Warehouse│
│ - APIs          │    │ - Validate      │    │ - Data Lake     │
│ - Files (CSV)   │    │ - Aggregate     │    │ - Analytics DB  │
│ - Streams       │    │ - Join/Merge    │    │ - ML Systems    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Ejemplo Completo

```typescript
// Pipeline: Ventas diarias → Analytics

// 1. EXTRACT - Obtener datos de múltiples fuentes
async function extract(): Promise<RawData> {
  const [orders, users, products] = await Promise.all([
    salesDB.query('SELECT * FROM orders WHERE date = CURRENT_DATE'),
    userDB.query('SELECT id, name, country FROM users'),
    productAPI.fetchAll()
  ]);
  
  return { orders, users, products };
}

// 2. TRANSFORM - Limpiar y enriquecer datos
function transform(raw: RawData): TransformedData[] {
  return raw.orders.map(order => {
    const user = raw.users.find(u => u.id === order.userId);
    const product = raw.products.find(p => p.id === order.productId);
    
    return {
      orderId: order.id,
      date: order.date,
      // Enriquecer con datos de usuario
      userName: user?.name ?? 'Unknown',
      userCountry: user?.country ?? 'Unknown',
      // Enriquecer con datos de producto
      productName: product?.name,
      productCategory: product?.category,
      // Calcular métricas
      revenue: order.quantity * order.price,
      // Normalizar
      currency: 'USD',
      timestamp: new Date().toISOString()
    };
  });
}

// 3. LOAD - Cargar al data warehouse
async function load(data: TransformedData[]): Promise<void> {
  // Batch insert para eficiencia
  const batches = chunk(data, 1000);
  
  for (const batch of batches) {
    await dataWarehouse.batchInsert('sales_analytics', batch);
  }
  
  // Actualizar tabla de resumen
  await dataWarehouse.query(`
    INSERT INTO daily_sales_summary 
    SELECT 
      DATE(timestamp) as date,
      userCountry as country,
      productCategory as category,
      SUM(revenue) as total_revenue,
      COUNT(*) as order_count
    FROM sales_analytics
    WHERE DATE(timestamp) = CURRENT_DATE
    GROUP BY date, country, category
  `);
}

// Pipeline completo
async function runETLPipeline() {
  console.log('Starting ETL pipeline...');
  
  const rawData = await extract();
  console.log(`Extracted ${rawData.orders.length} orders`);
  
  const transformedData = transform(rawData);
  console.log(`Transformed ${transformedData.length} records`);
  
  await load(transformedData);
  console.log('Load complete!');
}
```

### ETL vs ELT

```
ETL (Traditional):
Source → Transform (ETL Server) → Load (Warehouse)
- Transformación antes de cargar
- Mejor para datos pequeños/medianos
- Herramientas: Informatica, Talend

ELT (Modern):
Source → Load (Data Lake) → Transform (in Warehouse)
- Cargar raw data primero
- Transformar usando poder del warehouse
- Mejor para Big Data
- Herramientas: dbt, Snowflake, BigQuery
```

### Orquestación de Pipelines

```python
# Apache Airflow - DAG definition
from airflow import DAG
from airflow.operators.python import PythonOperator

with DAG('sales_etl', schedule_interval='@daily') as dag:
    
    extract_task = PythonOperator(
        task_id='extract',
        python_callable=extract_sales_data
    )
    
    transform_task = PythonOperator(
        task_id='transform',
        python_callable=transform_sales_data
    )
    
    load_task = PythonOperator(
        task_id='load',
        python_callable=load_to_warehouse
    )
    
    # Define dependencies
    extract_task >> transform_task >> load_task
```

### Herramientas ETL

| Herramienta | Tipo | Uso |
|-------------|------|-----|
| **Apache Airflow** | Orquestación | Pipelines complejos, Python |
| **dbt** | Transform | SQL transformations in warehouse |
| **Apache Spark** | Processing | Big Data, ML pipelines |
| **AWS Glue** | Managed ETL | Serverless, integración AWS |
| **Fivetran** | ELT SaaS | Conectores pre-built |
| **Airbyte** | ELT Open Source | Alternativa open source a Fivetran |

---

## 🚦 Rate Limiting

### ¿Por Qué Rate Limiting?

- Proteger contra ataques DDoS
- Prevenir abuso de API
- Garantizar fair usage entre clientes
- Controlar costos de infraestructura

### Algoritmos de Rate Limiting

#### 1. Token Bucket

```typescript
class TokenBucket {
  private tokens: number;
  private lastRefill: number;

  constructor(
    private capacity: number,     // Máximo tokens
    private refillRate: number    // Tokens por segundo
  ) {
    this.tokens = capacity;
    this.lastRefill = Date.now();
  }

  tryConsume(): boolean {
    this.refill();
    
    if (this.tokens >= 1) {
      this.tokens -= 1;
      return true;  // Request permitida
    }
    return false;   // Rate limited
  }

  private refill() {
    const now = Date.now();
    const elapsed = (now - this.lastRefill) / 1000;
    this.tokens = Math.min(
      this.capacity,
      this.tokens + elapsed * this.refillRate
    );
    this.lastRefill = now;
  }
}

// Uso: 100 requests/minuto con burst de 10
const limiter = new TokenBucket(10, 100/60);
```

**Ventaja:** Permite bursts controlados.

#### 2. Sliding Window Log

```typescript
class SlidingWindowLog {
  private requests: number[] = []; // Timestamps

  constructor(
    private windowMs: number,  // Ventana en ms
    private maxRequests: number
  ) {}

  tryConsume(): boolean {
    const now = Date.now();
    const windowStart = now - this.windowMs;
    
    // Eliminar requests fuera de la ventana
    this.requests = this.requests.filter(t => t > windowStart);
    
    if (this.requests.length < this.maxRequests) {
      this.requests.push(now);
      return true;
    }
    return false;
  }
}

// 100 requests por minuto
const limiter = new SlidingWindowLog(60000, 100);
```

**Ventaja:** Muy preciso.
**Desventaja:** Usa más memoria.

#### 3. Fixed Window Counter

```typescript
class FixedWindowCounter {
  private count = 0;
  private windowStart: number;

  constructor(
    private windowMs: number,
    private maxRequests: number
  ) {
    this.windowStart = Date.now();
  }

  tryConsume(): boolean {
    const now = Date.now();
    
    // Nueva ventana
    if (now - this.windowStart >= this.windowMs) {
      this.count = 0;
      this.windowStart = now;
    }
    
    if (this.count < this.maxRequests) {
      this.count++;
      return true;
    }
    return false;
  }
}
```

**Ventaja:** Simple y eficiente en memoria.
**Desventaja:** Puede permitir 2x requests en boundary.

### Implementación con Redis

```typescript
// Rate limiting distribuido con Redis
class DistributedRateLimiter {
  constructor(private redis: Redis) {}

  async isAllowed(key: string, limit: number, windowSec: number): Promise<boolean> {
    const current = await this.redis.incr(key);
    
    if (current === 1) {
      await this.redis.expire(key, windowSec);
    }
    
    return current <= limit;
  }
}

// Middleware Express
const rateLimitMiddleware = async (req, res, next) => {
  const key = `ratelimit:${req.ip}`;
  const allowed = await limiter.isAllowed(key, 100, 60);
  
  if (!allowed) {
    res.setHeader('Retry-After', 60);
    return res.status(429).json({ error: 'Too many requests' });
  }
  
  next();
};
```

### Rate Limit Headers

```typescript
// Headers estándar para informar al cliente
res.setHeader('X-RateLimit-Limit', 100);       // Límite total
res.setHeader('X-RateLimit-Remaining', 95);    // Requests restantes
res.setHeader('X-RateLimit-Reset', 1609459200);// Unix timestamp reset
```

---

## 📝 Case Studies

### Case Study 1: URL Shortener (bit.ly)

**Requisitos:**
- Acortar URLs largas
- Redirigir a URL original
- Analytics de clicks

**Estimaciones:**
- 100M URLs creadas/mes
- Read:Write ratio = 100:1
- Latencia < 100ms

**High-Level Design:**

```
┌────────┐    ┌────────────┐    ┌─────────────┐
│ Client │───▶│ API Server │───▶│  Database   │
└────────┘    └────────────┘    │ (Key-Value) │
     │              │           └─────────────┘
     │              ▼                  │
     │        ┌──────────┐            │
     └───────▶│   Cache  │◀───────────┘
              │ (Redis)  │
              └──────────┘
```

**Generación de Short URL:**

```typescript
// Opción 1: Base62 encoding de ID auto-increment
function encodeBase62(id: number): string {
  const chars = '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz';
  let result = '';
  while (id > 0) {
    result = chars[id % 62] + result;
    id = Math.floor(id / 62);
  }
  return result;
}
// ID 1000000 → "4c92"

// Opción 2: Hash + collision handling
function generateShortUrl(longUrl: string): string {
  const hash = md5(longUrl).substring(0, 7);
  // Check collision, regenerate if needed
  return hash;
}
```

**Database Schema:**

```sql
CREATE TABLE urls (
    id BIGSERIAL PRIMARY KEY,
    short_code VARCHAR(10) UNIQUE,
    original_url TEXT NOT NULL,
    created_at TIMESTAMP,
    expires_at TIMESTAMP,
    click_count INT DEFAULT 0
);

CREATE INDEX idx_short_code ON urls(short_code);
```

---

### Case Study 2: Twitter Feed/Timeline

**Requisitos:**
- Ver tweets de quienes sigues
- Timeline ordenado por tiempo
- Manejar usuarios con millones de followers

**El Problema Fan-out:**

```
Opción 1: Fan-out on Read (Pull)
- Cuando user ve timeline, query todos los tweets de quienes sigue
- ❌ Lento para usuarios que siguen a muchos

Opción 2: Fan-out on Write (Push)
- Cuando user twittea, escribir a timeline cache de todos sus followers
- ❌ Costoso para celebridades (millones de followers)

Solución: Híbrido
- Usuarios normales: Fan-out on Write
- Celebridades (>1M followers): Fan-out on Read + merge en tiempo real
```

**Arquitectura:**

```
Tweet Write:
┌─────────┐    ┌─────────────┐    ┌──────────────┐
│  User   │───▶│ Tweet Store │───▶│ Fan-out      │
└─────────┘    └─────────────┘    │ Service      │
                                  └──────────────┘
                                        │
              ┌─────────────────────────┼─────────────────────────┐
              ▼                         ▼                         ▼
       ┌────────────┐           ┌────────────┐           ┌────────────┐
       │ Timeline   │           │ Timeline   │           │ Timeline   │
       │ Follower 1 │           │ Follower 2 │           │ Follower N │
       └────────────┘           └────────────┘           └────────────┘

Timeline Read:
┌─────────┐    ┌────────────┐    ┌─────────────────┐
│  User   │───▶│ Timeline   │───▶│ Merge Celebrity │
└─────────┘    │ Cache      │    │ Tweets          │
               └────────────┘    └─────────────────┘
```

---

### Case Study 3: Chat Application (WhatsApp)

**Requisitos:**
- Messaging 1:1 y grupos
- Delivery receipts (sent, delivered, read)
- Online/offline status
- Message history

**Arquitectura:**

```
┌─────────┐     ┌─────────────────┐     ┌─────────┐
│ User A  │◀═══▶│   WebSocket     │◀═══▶│ User B  │
└─────────┘     │   Gateway       │     └─────────┘
                └────────┬────────┘
                         │
         ┌───────────────┼───────────────┐
         ▼               ▼               ▼
   ┌──────────┐   ┌──────────┐   ┌──────────────┐
   │ Message  │   │ Presence │   │ Push         │
   │ Service  │   │ Service  │   │ Notification │
   └──────────┘   └──────────┘   └──────────────┘
         │               │
         ▼               ▼
   ┌──────────┐   ┌──────────┐
   │ Message  │   │  Redis   │
   │ Database │   │ (Online) │
   └──────────┘   └──────────┘
```

**Message Flow:**

```typescript
// 1. User A envía mensaje
const message = {
  id: uuid(),
  from: 'userA',
  to: 'userB',
  content: 'Hello!',
  timestamp: Date.now(),
  status: 'sent'
};

// 2. Guardar en DB
await messageDB.save(message);

// 3. Check si User B está online
const userBOnline = await presenceService.isOnline('userB');

if (userBOnline) {
  // 4a. Enviar via WebSocket
  await websocketGateway.send('userB', message);
  message.status = 'delivered';
} else {
  // 4b. Enviar push notification
  await pushService.notify('userB', message);
}

// 5. User B lee el mensaje
// Update status to 'read', notify User A
```

---

## 📊 Quick Reference Numbers

### Latency Numbers Every Programmer Should Know

| Operation | Time |
|-----------|------|
| L1 cache reference | 0.5 ns |
| L2 cache reference | 7 ns |
| Main memory reference | 100 ns |
| SSD random read | 150 μs |
| HDD seek | 10 ms |
| Network round trip (same datacenter) | 0.5 ms |
| Network round trip (cross-continent) | 150 ms |

### Scale Calculations

| Users | Requests/sec | Storage/year |
|-------|--------------|--------------|
| 1M MAU | ~400 RPS | ~10 TB |
| 10M MAU | ~4,000 RPS | ~100 TB |
| 100M MAU | ~40,000 RPS | ~1 PB |

### Availability

| Availability | Downtime/year | Downtime/month |
|--------------|---------------|----------------|
| 99% (2 nines) | 3.65 days | 7.3 hours |
| 99.9% (3 nines) | 8.76 hours | 43.8 min |
| 99.99% (4 nines) | 52.56 min | 4.38 min |
| 99.999% (5 nines) | 5.26 min | 26.3 sec |

---

## 📖 Resources Recomendados

### Libros
- 📕 "Designing Data-Intensive Applications" - Martin Kleppmann
- 📕 "System Design Interview" Vol 1 & 2 - Alex Xu
- 📕 "Building Microservices" - Sam Newman

### Online
- 🌐 [System Design Primer](https://github.com/donnemartin/system-design-primer)
- 🌐 [High Scalability Blog](http://highscalability.com/)
- 🌐 [AWS Architecture Center](https://aws.amazon.com/architecture/)

### Videos
- 🎥 Gaurav Sen (YouTube)
- 🎥 Tech Dummies (YouTube)
- 🎥 ByteByteGo (YouTube)

---

## ✅ System Design Checklist

```markdown
□ Requisitos clarificados (funcionales y no funcionales)
□ Estimaciones calculadas (traffic, storage, bandwidth)
□ API diseñada
□ High-level design dibujado
□ Database schema definido
□ Estrategia de caching
□ Load balancing considerado
□ Data partitioning/sharding si es necesario
□ Single points of failure identificados
□ Monitoring y alerting planificado
□ Trade-offs discutidos
```

---

← [[Programming/Software Engineering/Software Engineering Roadmap|Back to Software Engineering]]
