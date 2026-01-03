---
tags:
  - software-engineering
  - system-design
  - caching
created: 2026-01-02
status: 🔴
---
# 🚀 Caching

> *"The fastest request is the one you don't make."*

## 🎯 What is Caching?

Almacenar datos temporalmente en una capa de acceso más rápido para reducir latencia y carga en sistemas downstream.

---

## 📊 Cache Hierarchy

```
                 Speed          Size         Cost
                   ↑              ↓            ↓
┌────────────────────────────────────────────────────┐
│              L1 Cache (CPU)                         │  0.5 ns
├────────────────────────────────────────────────────┤
│              L2 Cache (CPU)                         │  7 ns
├────────────────────────────────────────────────────┤
│              L3 Cache (CPU)                         │  20 ns
├────────────────────────────────────────────────────┤
│              RAM                                    │  100 ns
├────────────────────────────────────────────────────┤
│              In-Memory Cache (Redis)               │  0.5 ms
├────────────────────────────────────────────────────┤
│              SSD                                    │  150 μs
├────────────────────────────────────────────────────┤
│              Database                               │  1-10 ms
├────────────────────────────────────────────────────┤
│              Network Request                        │  50-150 ms
└────────────────────────────────────────────────────┘
```

---

## 🔄 Caching Strategies

### 1. Cache-Aside (Lazy Loading)
```typescript
// La aplicación maneja el cache directamente
async function getUser(id: string): Promise<User> {
  // 1. Intentar obtener del cache
  const cached = await redis.get(`user:${id}`);
  if (cached) {
    return JSON.parse(cached);
  }
  
  // 2. Si no está, ir a la DB
  const user = await db.users.findById(id);
  
  // 3. Guardar en cache para próximas requests
  await redis.setex(`user:${id}`, 3600, JSON.stringify(user));
  
  return user;
}
```

**Pros:** Simple, solo cachea lo que se usa
**Cons:** Cache miss = latencia extra, posible inconsistencia

### 2. Write-Through
```typescript
// Escribe en cache Y en DB simultáneamente
async function updateUser(id: string, data: UserData): Promise<User> {
  // 1. Escribir en DB
  const user = await db.users.update(id, data);
  
  // 2. Actualizar cache inmediatamente
  await redis.setex(`user:${id}`, 3600, JSON.stringify(user));
  
  return user;
}
```

**Pros:** Cache siempre consistente
**Cons:** Latencia de escritura mayor

### 3. Write-Behind (Write-Back)
```typescript
// Escribe en cache primero, DB después (async)
async function updateUser(id: string, data: UserData): Promise<User> {
  const user = { id, ...data, updatedAt: new Date() };
  
  // 1. Actualizar cache (rápido)
  await redis.setex(`user:${id}`, 3600, JSON.stringify(user));
  
  // 2. Encolar escritura a DB (async)
  await queue.publish('user-updates', { id, data });
  
  return user;
}

// Worker procesa la cola
async function processUserUpdate(message: Message) {
  const { id, data } = message;
  await db.users.update(id, data);
}
```

**Pros:** Escrituras muy rápidas
**Cons:** Riesgo de pérdida de datos, complejidad

### 4. Read-Through
```typescript
// El cache maneja la carga de datos automáticamente
class CacheService {
  async get<T>(key: string, loader: () => Promise<T>, ttl: number): Promise<T> {
    const cached = await this.redis.get(key);
    if (cached) {
      return JSON.parse(cached);
    }
    
    // Automáticamente carga y cachea
    const data = await loader();
    await this.redis.setex(key, ttl, JSON.stringify(data));
    return data;
  }
}

// Uso
const user = await cache.get(
  `user:${id}`,
  () => db.users.findById(id),
  3600
);
```

---

## 🗑️ Cache Eviction Policies

### LRU (Least Recently Used)
```
Access pattern: A B C D E A B C F
Cache size: 4

[A] → [B,A] → [C,B,A] → [D,C,B,A] → [E,D,C,B] → [A,E,D,C] → [B,A,E,D]
                                      ↑
                                   A evicted
```

### LFU (Least Frequently Used)
```
Tracks access count, evicts least accessed
```

### TTL (Time To Live)
```typescript
// Expiración por tiempo
await redis.setex('session:abc', 3600, data); // Expira en 1 hora

// Con refresh on access
async function getSession(id: string) {
  const data = await redis.get(`session:${id}`);
  if (data) {
    // Refresh TTL on each access
    await redis.expire(`session:${id}`, 3600);
  }
  return data;
}
```

---

## 💻 Common Cache Patterns

### Pattern 1: Cache User Sessions
```typescript
class SessionCache {
  private readonly TTL = 24 * 60 * 60; // 24 hours

  async get(sessionId: string): Promise<Session | null> {
    const data = await redis.get(`session:${sessionId}`);
    if (!data) return null;
    
    // Extend TTL on access (sliding expiration)
    await redis.expire(`session:${sessionId}`, this.TTL);
    return JSON.parse(data);
  }

  async set(session: Session): Promise<void> {
    await redis.setex(
      `session:${session.id}`,
      this.TTL,
      JSON.stringify(session)
    );
  }

  async delete(sessionId: string): Promise<void> {
    await redis.del(`session:${sessionId}`);
  }
}
```

### Pattern 2: Cache with Tags (Invalidation Groups)
```typescript
class TaggedCache {
  async set(key: string, value: any, tags: string[], ttl: number) {
    await redis.setex(key, ttl, JSON.stringify(value));
    
    // Associate key with tags
    for (const tag of tags) {
      await redis.sadd(`tag:${tag}`, key);
    }
  }

  async invalidateByTag(tag: string) {
    const keys = await redis.smembers(`tag:${tag}`);
    if (keys.length > 0) {
      await redis.del(...keys);
      await redis.del(`tag:${tag}`);
    }
  }
}

// Usage
await cache.set('user:123', user, ['users', 'user:123'], 3600);
await cache.set('user:456', user, ['users', 'user:456'], 3600);

// Invalidate all users
await cache.invalidateByTag('users');
```

### Pattern 3: Request Deduplication
```typescript
class DeduplicatedCache {
  private pending = new Map<string, Promise<any>>();

  async getOrFetch<T>(key: string, fetcher: () => Promise<T>): Promise<T> {
    // Check cache first
    const cached = await redis.get(key);
    if (cached) return JSON.parse(cached);

    // Check if request is already in flight
    if (this.pending.has(key)) {
      return this.pending.get(key);
    }

    // Create new request
    const promise = fetcher().then(async (data) => {
      await redis.setex(key, 3600, JSON.stringify(data));
      this.pending.delete(key);
      return data;
    });

    this.pending.set(key, promise);
    return promise;
  }
}
```

---

## ⚠️ Cache Problems

### Cache Stampede (Thundering Herd)
```typescript
// ❌ Problem: Many requests hit expired cache simultaneously
// All go to DB at once

// ✅ Solution 1: Lock
async function getWithLock(key: string, fetcher: () => Promise<any>) {
  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached);

  const lockKey = `lock:${key}`;
  const acquired = await redis.set(lockKey, '1', 'NX', 'EX', 10);
  
  if (acquired) {
    const data = await fetcher();
    await redis.setex(key, 3600, JSON.stringify(data));
    await redis.del(lockKey);
    return data;
  } else {
    // Wait and retry
    await sleep(100);
    return getWithLock(key, fetcher);
  }
}

// ✅ Solution 2: Stale-while-revalidate
async function getStaleWhileRevalidate(key: string, fetcher: () => Promise<any>) {
  const cached = await redis.get(key);
  const ttl = await redis.ttl(key);
  
  if (cached) {
    // If TTL is low, trigger background refresh
    if (ttl < 60) {
      setImmediate(() => refreshCache(key, fetcher));
    }
    return JSON.parse(cached);
  }
  
  return fetcher();
}
```

### Cache Invalidation
```typescript
// "There are only two hard things in Computer Science:
// cache invalidation and naming things." - Phil Karlton

// Strategy 1: TTL-based (eventual consistency)
await redis.setex('user:123', 300, data); // 5 min TTL

// Strategy 2: Event-based invalidation
eventBus.on('user:updated', async (userId) => {
  await redis.del(`user:${userId}`);
  await redis.del(`user:${userId}:profile`);
  await redis.del(`user:${userId}:settings`);
});

// Strategy 3: Versioned keys
const version = await redis.incr('users:version');
await redis.setex(`user:123:v${version}`, 3600, data);
```

---

## 📋 Summary

| Strategy | Use Case | Trade-off |
|----------|----------|-----------|
| **Cache-Aside** | Read-heavy, tolerant to stale | Simple pero posible inconsistencia |
| **Write-Through** | Consistencia importante | Mayor latencia de escritura |
| **Write-Behind** | Write-heavy | Riesgo pérdida de datos |
| **Read-Through** | Simplificar código | Abstracción puede ocultar problemas |

---

← [[Programming/Software Engineering/System Design/_Index|Back to System Design]]
