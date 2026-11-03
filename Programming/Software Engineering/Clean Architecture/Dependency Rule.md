---
tags:
  - software-engineering
  - clean-architecture
  - dependency-rule
created: 2026-01-02
status: 🔴
---
# ⬅️ The Dependency Rule

> *"Source code dependencies must point only inward, toward higher-level policies."*

## 🎯 The Rule

> [!important] Core Principle
> **Las dependencias del código fuente solo pueden apuntar hacia adentro.**
> 
> Un círculo interno NO PUEDE saber NADA sobre un círculo externo.
> Esto incluye funciones, clases, variables, o cualquier entidad de software.

---

## 📊 Visual Representation

```
┌─────────────────────────────────────────────────────────────┐
│  FRAMEWORKS & DRIVERS                                        │
│  (Web, DB, Devices, External Interfaces)                    │
│                              ↓                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  INTERFACE ADAPTERS                                    │  │
│  │  (Controllers, Gateways, Presenters)                  │  │
│  │                         ↓                              │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │  APPLICATION BUSINESS RULES                      │  │  │
│  │  │  (Use Cases)                                     │  │  │
│  │  │                    ↓                             │  │  │
│  │  │  ┌───────────────────────────────────────────┐  │  │  │
│  │  │  │  ENTERPRISE BUSINESS RULES                 │  │  │  │
│  │  │  │  (Entities)                                │  │  │  │
│  │  │  │                                            │  │  │  │
│  │  │  │        ★ MOST STABLE ★                     │  │  │  │
│  │  │  │                                            │  │  │  │
│  │  │  └───────────────────────────────────────────┘  │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘

ALL ARROWS POINT INWARD ⬅️
```

---

## ❌ Violations

### Bad: Use Case Knowing About Framework
```javascript
// ❌ VIOLACIÓN: Use Case importa de Express
import { Request, Response } from 'express';

class CreateUserUseCase {
  execute(req: Request, res: Response) {  // ❌ Conoce Express
    const user = new User(req.body.name);
    // ...
    res.json(user);  // ❌ Conoce Response
  }
}
```

### Bad: Entity Knowing About Database
```javascript
// ❌ VIOLACIÓN: Entidad sabe sobre Prisma
import { PrismaClient } from '@prisma/client';

class User {
  async save() {
    const prisma = new PrismaClient();  // ❌ Conoce Prisma
    await prisma.user.create({ data: this });
  }
}
```

### Bad: Use Case Knowing About SQL
```javascript
// ❌ VIOLACIÓN: Use Case sabe sobre SQL
class GetUsersUseCase {
  execute() {
    return db.query('SELECT * FROM users');  // ❌ SQL específico
  }
}
```

---

## ✅ Correct Implementation

### Using Dependency Inversion

```typescript
// 1. ENTITIES (innermost - no dependencies)
class User {
  constructor(
    public readonly id: string,
    public name: string,
    public email: string
  ) {}

  updateName(newName: string) {
    if (newName.length < 2) throw new Error("Name too short");
    this.name = newName;
  }
}

// 2. USE CASES (depends only on entities and abstractions)
// Define interfaces (ports) in this layer
interface IUserRepository {
  save(user: User): Promise<void>;
  findById(id: string): Promise<User | null>;
}

class CreateUserUseCase {
  // Depends on INTERFACE, not implementation
  constructor(private userRepository: IUserRepository) {}

  async execute(input: { name: string; email: string }): Promise<User> {
    const user = new User(generateId(), input.name, input.email);
    await this.userRepository.save(user);
    return user;
  }
}

// 3. INTERFACE ADAPTERS (implements interfaces from use cases)
class PostgresUserRepository implements IUserRepository {
  constructor(private prisma: PrismaClient) {}

  async save(user: User): Promise<void> {
    await this.prisma.user.create({
      data: { id: user.id, name: user.name, email: user.email }
    });
  }

  async findById(id: string): Promise<User | null> {
    const data = await this.prisma.user.findUnique({ where: { id } });
    return data ? new User(data.id, data.name, data.email) : null;
  }
}

// 4. FRAMEWORKS & DRIVERS (wires everything together)
// main.ts or dependency injection container
const prisma = new PrismaClient();
const userRepository = new PostgresUserRepository(prisma);
const createUserUseCase = new CreateUserUseCase(userRepository);

// Express controller (adapter)
app.post('/users', async (req, res) => {
  const user = await createUserUseCase.execute(req.body);
  res.json(user);
});
```

---

## 🔄 Data Crossing Boundaries

> [!info] Data Format Rule
> Cuando los datos cruzan boundaries, siempre están en el formato más conveniente para el círculo **interno**.

```typescript
// ❌ Bad: Use Case returns database-specific format
class GetUserUseCase {
  async execute(id: string) {
    return await this.prisma.user.findUnique({ where: { id } });
    // Returns Prisma-specific format
  }
}

// ✅ Good: Use Case returns domain format
class GetUserUseCase {
  async execute(id: string): Promise<UserDTO | null> {
    const user = await this.userRepository.findById(id);
    if (!user) return null;
    
    return {
      id: user.id,
      name: user.name,
      email: user.email
    };
  }
}
```

---

## 🔌 Dependency Inversion at Boundaries

```
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Use Case Layer                                          │
│  ┌──────────────┐      ┌─────────────────────┐          │
│  │ CreateOrder  │─────→│ <<interface>>       │          │
│  │  UseCase     │      │ IOrderRepository    │          │
│  └──────────────┘      └─────────────────────┘          │
│                                   △                      │
│                                   │                      │
├───────────────────────────────────┼──────────────────────┤
│                                   │                      │
│  Infrastructure Layer             │                      │
│                        ┌──────────┴──────────┐          │
│                        │ PostgresOrder       │          │
│                        │ Repository          │          │
│                        └─────────────────────┘          │
│                                                          │
└─────────────────────────────────────────────────────────┘

La flecha de dependencia SUBE (hacia Use Case)
aunque el flujo de datos BAJA (hacia Database)
```

---

## 📋 Checklist

> [!check] Verify Your Dependencies
> - [ ] ¿Entities importan de frameworks? → ❌
> - [ ] ¿Use Cases importan de Express/Fastify/etc? → ❌
> - [ ] ¿Use Cases conocen SQL o queries específicos? → ❌
> - [ ] ¿Entities saben cómo persistirse? → ❌
> - [ ] ¿Use Cases definen interfaces para dependencias externas? → ✅
> - [ ] ¿Infrastructure implementa interfaces de Use Cases? → ✅

---

## 💡 Benefits

| Benefit | Description |
|---------|-------------|
| **Testability** | Puedes mockear interfaces fácilmente |
| **Flexibility** | Cambiar DB = cambiar solo el adapter |
| **Stability** | El core no cambia cuando cambian detalles |
| **Parallel work** | Equipos trabajan en capas independientes |

---

← [[Programming/Software Engineering/Clean Architecture/Clean Architecture Complete Guide|Back to Clean Architecture]]
