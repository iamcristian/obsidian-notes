---
tags:
  - software-engineering
  - clean-architecture
  - dashboard
created: 2026-01-02
---
# 🏛️ Clean Architecture

> *"The goal of software architecture is to minimize the human resources required to build and maintain the required system."* — Robert C. Martin

## 📚 Contents

| Topic | Description | Status |
|-------|-------------|--------|
| [[Programming/Software Engineering/Clean Architecture/Overview\|Overview]] | Principios fundamentales | 🔴 |
| [[Programming/Software Engineering/Clean Architecture/Dependency Rule\|Dependency Rule]] | La regla de dependencia | 🔴 |
| [[Programming/Software Engineering/Clean Architecture/Entities\|Entities]] | Reglas de negocio empresariales | 🔴 |
| [[Programming/Software Engineering/Clean Architecture/Use Cases\|Use Cases]] | Reglas de negocio de aplicación | 🔴 |
| [[Programming/Software Engineering/Clean Architecture/Interface Adapters\|Interface Adapters]] | Controladores, presentadores, gateways | 🔴 |
| [[Programming/Software Engineering/Clean Architecture/Frameworks and Drivers\|Frameworks and Drivers]] | Capa externa | 🔴 |

---

## 🎯 The Clean Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Frameworks & Drivers                      │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                  Interface Adapters                    │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │              Application Business               │  │  │
│  │  │                  (Use Cases)                    │  │  │
│  │  │  ┌───────────────────────────────────────────┐  │  │  │
│  │  │  │         Enterprise Business Rules          │  │  │  │
│  │  │  │              (Entities)                    │  │  │  │
│  │  │  └───────────────────────────────────────────┘  │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘

Dependencies point INWARD →
```

---

## 🔑 Key Principles

> [!abstract] Core Concepts
> 1. **Independent of Frameworks**: La arquitectura no depende de frameworks
> 2. **Testable**: Las reglas de negocio se prueban sin UI, DB, o servicios externos
> 3. **Independent of UI**: La UI puede cambiar sin cambiar el sistema
> 4. **Independent of Database**: Las reglas de negocio no están atadas a la DB
> 5. **Independent of External Agency**: Las reglas de negocio no saben del mundo exterior

---

## 📂 Typical Project Structure

```
src/
├── domain/                  # Entities (innermost circle)
│   ├── entities/
│   │   ├── User.ts
│   │   └── Order.ts
│   └── value-objects/
│       ├── Email.ts
│       └── Money.ts
│
├── application/             # Use Cases (second circle)
│   ├── use-cases/
│   │   ├── CreateOrder.ts
│   │   └── GetUserProfile.ts
│   ├── ports/               # Interfaces for external services
│   │   ├── IUserRepository.ts
│   │   └── IEmailService.ts
│   └── dto/
│       └── CreateOrderDTO.ts
│
├── infrastructure/          # Interface Adapters + Frameworks
│   ├── repositories/
│   │   └── PostgresUserRepository.ts
│   ├── services/
│   │   └── SendGridEmailService.ts
│   └── database/
│       └── prisma/
│
└── presentation/            # Controllers, Views
    ├── http/
    │   ├── controllers/
    │   └── routes/
    └── graphql/
        └── resolvers/
```

---

## 📖 Resources

- 📕 **Book**: "Clean Architecture" by Robert C. Martin
- 📕 **Book**: "Get Your Hands Dirty on Clean Architecture" by Tom Hombergs
- 🎥 **Talk**: "Clean Architecture" by Uncle Bob

---

## 🔗 Related Topics

- [[Programming/Software Engineering/Architectural Patterns/Hexagonal Architecture\|Hexagonal Architecture]]
- [[Programming/Software Engineering/Architectural Patterns/Onion Architecture\|Onion Architecture]]
- [[Programming/Software Engineering/SOLID/SOLID Principles Complete Guide|SOLID Principles]]
- [[Programming/Software Engineering/Design Patterns/Design Patterns Complete Guide|Design Patterns]]

---

## 📝 Notes
```dataview
TABLE WITHOUT ID
	file.link as "Note",
	status as "Status"
FROM "Programming/Software Engineering/Clean Architecture"
WHERE !contains(file.name, "_Index")
SORT file.name asc
```

---

← [[Programming/Software Engineering/Software Engineering Roadmap|Back to Software Engineering]]
