---
tags:
  - software-engineering
  - design-patterns
  - creational
created: 2026-01-02
status: 🔴
category: Creational
---
# 🔨 Builder Pattern

> *"Separate the construction of a complex object from its representation."*

## 🎯 Intent

- Construir objetos complejos **paso a paso**
- Permitir diferentes representaciones del mismo proceso
- Evitar constructores con muchos parámetros (telescoping constructor)

---

## 📊 Structure

```
                           ┌─────────────────┐
                           │    Director     │
                           ├─────────────────┤
              ┌───────────→│ + construct()   │
              │            └─────────────────┘
              │                    │
              │                    ▼
       ┌──────┴──────┐     ┌─────────────────┐
       │   Client    │     │ <<interface>>   │
       │             │     │    Builder      │
       └─────────────┘     ├─────────────────┤
                           │ + buildStepA() │
                           │ + buildStepB() │
                           │ + getResult()  │
                           └─────────────────┘
                                   △
                    ┌──────────────┼──────────────┐
                    │              │              │
           ┌────────┴────┐  ┌─────┴─────┐  ┌────┴────────┐
           │ BuilderA    │  │ BuilderB  │  │ BuilderC    │
           └─────────────┘  └───────────┘  └─────────────┘
```

---

## 💻 Implementation

### Problem: Telescoping Constructor
```typescript
// ❌ Bad - Demasiados parámetros, difícil de usar
class User {
  constructor(
    firstName: string,
    lastName: string,
    email: string,
    age?: number,
    phone?: string,
    address?: string,
    city?: string,
    country?: string,
    role?: string,
    isActive?: boolean
  ) { }
}

// Confuso al usar
const user = new User('John', 'Doe', 'john@email.com', undefined, undefined, '123 Main St');
```

### Solution: Builder Pattern
```typescript
class User {
  firstName: string;
  lastName: string;
  email: string;
  age?: number;
  phone?: string;
  address?: string;
  role: string = 'user';
  isActive: boolean = true;

  constructor(builder: UserBuilder) {
    this.firstName = builder.firstName;
    this.lastName = builder.lastName;
    this.email = builder.email;
    this.age = builder.age;
    this.phone = builder.phone;
    this.address = builder.address;
    this.role = builder.role;
    this.isActive = builder.isActive;
  }
}

class UserBuilder {
  firstName: string;
  lastName: string;
  email: string;
  age?: number;
  phone?: string;
  address?: string;
  role: string = 'user';
  isActive: boolean = true;

  constructor(firstName: string, lastName: string, email: string) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.email = email;
  }

  setAge(age: number): UserBuilder {
    this.age = age;
    return this;
  }

  setPhone(phone: string): UserBuilder {
    this.phone = phone;
    return this;
  }

  setAddress(address: string): UserBuilder {
    this.address = address;
    return this;
  }

  setRole(role: string): UserBuilder {
    this.role = role;
    return this;
  }

  setActive(isActive: boolean): UserBuilder {
    this.isActive = isActive;
    return this;
  }

  build(): User {
    return new User(this);
  }
}

// ✅ Usage - claro y legible
const user = new UserBuilder('John', 'Doe', 'john@email.com')
  .setAge(30)
  .setPhone('555-1234')
  .setAddress('123 Main St')
  .setRole('admin')
  .build();
```

### Fluent Builder (Modern Style)
```typescript
class QueryBuilder {
  private query: string = '';
  private params: any[] = [];

  select(...columns: string[]): QueryBuilder {
    this.query = `SELECT ${columns.join(', ')}`;
    return this;
  }

  from(table: string): QueryBuilder {
    this.query += ` FROM ${table}`;
    return this;
  }

  where(condition: string, value: any): QueryBuilder {
    this.query += ` WHERE ${condition}`;
    this.params.push(value);
    return this;
  }

  orderBy(column: string, direction: 'ASC' | 'DESC' = 'ASC'): QueryBuilder {
    this.query += ` ORDER BY ${column} ${direction}`;
    return this;
  }

  limit(count: number): QueryBuilder {
    this.query += ` LIMIT ${count}`;
    return this;
  }

  build(): { query: string; params: any[] } {
    return { query: this.query, params: this.params };
  }
}

// Usage
const { query, params } = new QueryBuilder()
  .select('id', 'name', 'email')
  .from('users')
  .where('status = ?', 'active')
  .orderBy('created_at', 'DESC')
  .limit(10)
  .build();
```

### With Director
```typescript
interface HouseBuilder {
  reset(): void;
  buildWalls(): void;
  buildDoors(): void;
  buildWindows(): void;
  buildRoof(): void;
  buildGarage(): void;
  getResult(): House;
}

class ModernHouseBuilder implements HouseBuilder {
  private house: House;

  reset() { this.house = new House(); }
  buildWalls() { this.house.walls = 'glass walls'; }
  buildDoors() { this.house.doors = 'sliding doors'; }
  buildWindows() { this.house.windows = 'floor-to-ceiling windows'; }
  buildRoof() { this.house.roof = 'flat roof'; }
  buildGarage() { this.house.garage = 'underground garage'; }
  getResult(): House { return this.house; }
}

// Director knows HOW to build
class HouseDirector {
  private builder: HouseBuilder;

  setBuilder(builder: HouseBuilder) {
    this.builder = builder;
  }

  // Diferentes "recetas" de construcción
  buildMinimalHouse() {
    this.builder.reset();
    this.builder.buildWalls();
    this.builder.buildDoors();
    this.builder.buildRoof();
  }

  buildFullHouse() {
    this.builder.reset();
    this.builder.buildWalls();
    this.builder.buildDoors();
    this.builder.buildWindows();
    this.builder.buildRoof();
    this.builder.buildGarage();
  }
}

// Usage
const director = new HouseDirector();
const builder = new ModernHouseBuilder();
director.setBuilder(builder);

director.buildMinimalHouse();
const simpleHouse = builder.getResult();

director.buildFullHouse();
const fullHouse = builder.getResult();
```

---

## 🔄 Real World Examples

### HTTP Request Builder
```typescript
class HttpRequestBuilder {
  private method: string = 'GET';
  private url: string = '';
  private headers: Map<string, string> = new Map();
  private body: any = null;
  private timeout: number = 30000;

  setMethod(method: 'GET' | 'POST' | 'PUT' | 'DELETE'): this {
    this.method = method;
    return this;
  }

  setUrl(url: string): this {
    this.url = url;
    return this;
  }

  addHeader(key: string, value: string): this {
    this.headers.set(key, value);
    return this;
  }

  setBody(body: any): this {
    this.body = body;
    return this;
  }

  setTimeout(ms: number): this {
    this.timeout = ms;
    return this;
  }

  build(): RequestConfig {
    return {
      method: this.method,
      url: this.url,
      headers: Object.fromEntries(this.headers),
      body: this.body,
      timeout: this.timeout
    };
  }
}

// Usage
const request = new HttpRequestBuilder()
  .setMethod('POST')
  .setUrl('/api/users')
  .addHeader('Content-Type', 'application/json')
  .addHeader('Authorization', 'Bearer token123')
  .setBody({ name: 'John' })
  .setTimeout(5000)
  .build();
```

### Email Builder
```typescript
class EmailBuilder {
  private from: string = '';
  private to: string[] = [];
  private cc: string[] = [];
  private subject: string = '';
  private body: string = '';
  private attachments: File[] = [];
  private isHtml: boolean = false;

  setFrom(email: string): this { this.from = email; return this; }
  addTo(email: string): this { this.to.push(email); return this; }
  addCc(email: string): this { this.cc.push(email); return this; }
  setSubject(subject: string): this { this.subject = subject; return this; }
  setBody(body: string, isHtml = false): this { 
    this.body = body; 
    this.isHtml = isHtml;
    return this; 
  }
  addAttachment(file: File): this { this.attachments.push(file); return this; }

  build(): Email {
    if (!this.from) throw new Error('From is required');
    if (this.to.length === 0) throw new Error('At least one recipient required');
    return new Email(this);
  }
}
```

---

## ✅ When to Use

| Scenario | Example |
|----------|---------|
| Muchos parámetros opcionales | User, Config objects |
| Construcción paso a paso | Queries, Documents |
| Mismo proceso, diferentes representaciones | House builder |
| Objetos inmutables complejos | Request/Response objects |

---

## 📋 Summary

| Aspect | Details |
|--------|---------|
| **Type** | Creational |
| **Intent** | Construcción paso a paso |
| **Key Benefit** | Código legible, objetos inmutables |
| **Trade-off** | Más código (pero más claro) |
| **Common Use** | Fluent APIs, configs, queries |

---

← [[Programming/Software Engineering/Design Patterns/_Index|Back to Design Patterns]]
