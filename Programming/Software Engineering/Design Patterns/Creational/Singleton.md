---
tags:
  - software-engineering
  - design-patterns
  - creational
created: 2026-01-02
status: 🔴
category: Creational
---
# 🔒 Singleton Pattern

> *"Ensure a class has only one instance and provide a global point of access to it."*

## 🎯 Intent

- Garantizar que una clase tenga **exactamente una instancia**
- Proveer un punto de acceso **global** a esa instancia

---

## 📊 Structure

```
┌─────────────────────────────────────┐
│           Singleton                 │
├─────────────────────────────────────┤
│ - instance: Singleton               │
├─────────────────────────────────────┤
│ - constructor()                     │
│ + getInstance(): Singleton          │
│ + businessLogic()                   │
└─────────────────────────────────────┘
```

---

## 💻 Implementation

### Basic Singleton (JavaScript/TypeScript)
```typescript
class Database {
  private static instance: Database;
  private connection: any;

  // Constructor privado
  private constructor() {
    this.connection = this.createConnection();
  }

  // Punto de acceso global
  public static getInstance(): Database {
    if (!Database.instance) {
      Database.instance = new Database();
    }
    return Database.instance;
  }

  private createConnection() {
    console.log('Creating database connection...');
    return { connected: true };
  }

  public query(sql: string) {
    return `Executing: ${sql}`;
  }
}

// Usage
const db1 = Database.getInstance();
const db2 = Database.getInstance();
console.log(db1 === db2); // true - misma instancia
```

### Modern JavaScript (Module Pattern)
```javascript
// database.js
// En JavaScript moderno, los módulos ya son singletons

let connection = null;

function createConnection() {
  console.log('Creating database connection...');
  return { connected: true, timestamp: Date.now() };
}

export function getConnection() {
  if (!connection) {
    connection = createConnection();
  }
  return connection;
}

export function query(sql) {
  return `Executing: ${sql}`;
}

// Usage en otros archivos
import { getConnection } from './database.js';
const conn1 = getConnection();
const conn2 = getConnection();
// conn1 === conn2 → true
```

### Thread-Safe Singleton (para referencia)
```typescript
// En JavaScript single-threaded no es necesario, pero útil para entender el concepto
class ThreadSafeSingleton {
  private static instance: ThreadSafeSingleton;
  private static lock = false;

  private constructor() {}

  public static getInstance(): ThreadSafeSingleton {
    if (!ThreadSafeSingleton.instance) {
      if (!ThreadSafeSingleton.lock) {
        ThreadSafeSingleton.lock = true;
        ThreadSafeSingleton.instance = new ThreadSafeSingleton();
        ThreadSafeSingleton.lock = false;
      }
    }
    return ThreadSafeSingleton.instance;
  }
}
```

---

## ✅ When to Use

| Scenario | Example |
|----------|---------|
| Database connections | Un pool de conexiones compartido |
| Configuration | Settings de aplicación |
| Logging | Logger centralizado |
| Caching | Cache compartido |
| Thread pools | Pool de workers |

---

## ⚠️ Problems with Singleton

> [!warning] Anti-Pattern Alert
> Singleton es considerado frecuentemente un **anti-pattern**. Úsalo con precaución.

### 1. Global State (Hidden Dependencies)
```javascript
// ❌ Bad - dependencia oculta
class UserService {
  getUser(id) {
    const db = Database.getInstance(); // Dependencia oculta
    return db.query(`SELECT * FROM users WHERE id = ${id}`);
  }
}

// ✅ Better - Dependency Injection
class UserService {
  constructor(database) {
    this.database = database; // Dependencia explícita
  }
  
  getUser(id) {
    return this.database.query(`SELECT * FROM users WHERE id = ${id}`);
  }
}
```

### 2. Testing Difficulty
```javascript
// ❌ Hard to test - no puedes inyectar mock
class OrderService {
  process(order) {
    const db = Database.getInstance(); // Imposible mockear fácilmente
    db.query('INSERT INTO orders...');
  }
}

// ✅ Testable
class OrderService {
  constructor(database) {
    this.database = database;
  }
  
  process(order) {
    this.database.query('INSERT INTO orders...');
  }
}

// Test
const mockDb = { query: jest.fn() };
const service = new OrderService(mockDb);
```

### 3. Violates Single Responsibility
```javascript
// Singleton hace DOS cosas:
// 1. Manejar su ciclo de vida (creación única)
// 2. Su lógica de negocio real
```

---

## 🔄 Better Alternatives

### 1. Dependency Injection Container
```typescript
// Usar un DI container en lugar de Singleton
import { Container } from 'inversify';

const container = new Container();
container.bind<Database>('Database').to(PostgresDatabase).inSingletonScope();

// El container maneja el singleton, no la clase
const db = container.get<Database>('Database');
```

### 2. Module Scope (JavaScript)
```javascript
// services/database.js
const connection = createConnection();

export function query(sql) {
  return connection.execute(sql);
}

// El módulo es singleton por naturaleza
```

### 3. Factory with Caching
```typescript
class DatabaseFactory {
  private static cache = new Map<string, Database>();

  static create(config: Config): Database {
    const key = JSON.stringify(config);
    
    if (!this.cache.has(key)) {
      this.cache.set(key, new Database(config));
    }
    
    return this.cache.get(key)!;
  }
}
```

---

## 📋 Summary

| Aspect | Details |
|--------|---------|
| **Type** | Creational |
| **Intent** | Una instancia, acceso global |
| **Pros** | Control de instancia, lazy loading |
| **Cons** | Global state, hard to test, hidden deps |
| **Alternative** | Dependency Injection |

---

## 💡 Rule of Thumb

> [!tip] When to Actually Use
> - Cuando REALMENTE necesitas exactamente una instancia
> - El costo de crear múltiples instancias es prohibitivo
> - Prefieres Dependency Injection sobre getInstance()

---

← [[Programming/Software Engineering/Design Patterns/Design Patterns Complete Guide|Back to Design Patterns]]
