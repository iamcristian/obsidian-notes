---
tags:
  - dashboard
  - software-engineering
created: 2025-12-29
banner: "https://images.unsplash.com/photo-1555066931-4365d14bab8c?w=1200"
banner_y: 0.5
---
# 📐 Software Engineering

## 🎯 Learning Path

```
Fundamentals → Design → Architecture → System Design
    │              │           │              │
    ▼              ▼           ▼              ▼
Clean Code    Patterns    Clean Arch    Distributed
  SOLID       Factory     Hexagonal     Scalability
             Strategy      CQRS          Caching
```

---

## 📚 Topics

> [!info]+ 🧹 [[Programming/Software Engineering/Clean Code/Clean Code Complete Guide|Clean Code]]
> Nombres, funciones, comments, error handling
> ```dataview
> LIST WITHOUT ID file.link
> FROM "Programming/Software Engineering/Clean Code"
> WHERE !contains(file.name, "_Index")
> LIMIT 3
> ```

> [!info]+ 🎯 [[Programming/Software Engineering/SOLID/SOLID Principles Complete Guide|SOLID Principles]]
> Single Responsibility, Open/Closed, Liskov, Interface Segregation, Dependency Inversion

> [!info]+ 🎨 [[Programming/Software Engineering/Design Patterns/Design Patterns Complete Guide|Design Patterns]]
> Creational, Structural, Behavioral patterns
> ```dataview
> LIST WITHOUT ID file.link
> FROM "Programming/Software Engineering/Design Patterns"
> WHERE !contains(file.name, "_Index")
> LIMIT 4
> ```

> [!info]+ 🏛️ [[Programming/Software Engineering/Clean Architecture/Clean Architecture Complete Guide|Clean Architecture]]
> Entities, Use Cases, Dependency Rule

> [!info]+ 🏗️ [[Programming/Software Engineering/Architectural Patterns/Architectural Patterns Complete Guide|Architectural Patterns]]
> Hexagonal, Microservices, CQRS, Event Sourcing

> [!info]+ 🌐 [[Programming/Software Engineering/System Design/System Design Overview|System Design]]
> Scalability, Caching, Load Balancing, API Design

> [!info]+ 🎯 [[Programming/Software Engineering/DDD/Domain-Driven Design Complete Guide|Domain-Driven Design]]
> Bounded Contexts, Aggregates, Value Objects

> [!info]+ 🧪 [[Programming/Software Engineering/Testing/Testing Complete Guide|Testing]]
> TDD, Unit Testing, Integration Testing, Mocking

> [!info]+ 🛠️ [[Programming/Software Engineering/DevOps/DevOps Complete Guide|DevOps]]
> Docker, CI/CD, GitHub Actions, Cloud, Serverless

> [!info]+ 🗄️ [[Programming/Software Engineering/Database Design/Database Design Complete Guide|Database Design]]
> Relational, NoSQL, Normalization, Indexing

> [!info]+ 🔧 [[Programming/Software Engineering/Refactoring/Refactoring Complete Guide|Refactoring]]
> Extract Method, Replace Conditional, Code Smells

---

## 📊 Progress Overview

| Area | Status | Priority |
|------|--------|----------|
| Clean Code | � Complete | High |
| SOLID | 🟡 Learning | High |
| Design Patterns | 🟢 Complete | High |
| Clean Architecture | 🟢 Complete | High |
| System Design | 🟢 Complete | High |
| DevOps | 🟢 Complete | High |
| Database Design | 🟢 Complete | High |
| Testing | 🟢 Complete | High |
| Architectural Patterns | 🟢 Complete | Medium |
| DDD | 🟡 Learning | Medium |
| Refactoring | 🔴 To Start | Low |

---

## 📖 Recommended Books

| Book | Author | Topic |
|------|--------|-------|
| Clean Code | Robert C. Martin | Code Quality |
| Clean Architecture | Robert C. Martin | Architecture |
| Design Patterns | Gang of Four | Patterns |
| Refactoring | Martin Fowler | Code Improvement |
| DDD | Eric Evans | Domain Modeling |
| DDIA | Martin Kleppmann | System Design |

---

## 📝 Recent Notes
```dataview
TABLE WITHOUT ID
	file.link as "Note",
	dateformat(file.cday, "MMM dd") as "Added"
FROM "Programming/Software Engineering"
WHERE !contains(file.name, "_Index")
SORT file.cday desc
LIMIT 10
```

---

## ➕ Add New
Use template: [[Templates/Programming Concept Template|New Concept]]

← [[Programming/Programming Hub|Back to Programming]]
