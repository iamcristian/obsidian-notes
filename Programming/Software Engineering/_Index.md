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

> [!info]+ 🧹 [[Programming/Software Engineering/Clean Code/_Index|Clean Code]]
> Nombres, funciones, comments, error handling
> ```dataview
> LIST WITHOUT ID file.link
> FROM "Programming/Software Engineering/Clean Code"
> WHERE !contains(file.name, "_Index")
> LIMIT 3
> ```

> [!info]+ 🎯 [[Programming/Software Engineering/SOLID/_Index|SOLID Principles]]
> Single Responsibility, Open/Closed, Liskov, Interface Segregation, Dependency Inversion

> [!info]+ 🎨 [[Programming/Software Engineering/Design Patterns/_Index|Design Patterns]]
> Creational, Structural, Behavioral patterns
> ```dataview
> LIST WITHOUT ID file.link
> FROM "Programming/Software Engineering/Design Patterns"
> WHERE !contains(file.name, "_Index")
> LIMIT 4
> ```

> [!info]+ 🏛️ [[Programming/Software Engineering/Clean Architecture/_Index|Clean Architecture]]
> Entities, Use Cases, Dependency Rule

> [!info]+ 🏗️ [[Programming/Software Engineering/Architectural Patterns/_Index|Architectural Patterns]]
> Hexagonal, Microservices, CQRS, Event Sourcing

> [!info]+ 🌐 [[Programming/Software Engineering/System Design/_Index|System Design]]
> Scalability, Caching, Load Balancing, API Design

> [!info]+ 🎯 [[Programming/Software Engineering/DDD/_Index|Domain-Driven Design]]
> Bounded Contexts, Aggregates, Value Objects

> [!info]+ 🧪 [[Programming/Software Engineering/Testing/_Index|Testing]]
> TDD, Unit Testing, Integration Testing, Mocking

> [!info]+ 🛠️ [[Programming/Software Engineering/DevOps/_Index|DevOps]]
> Docker, CI/CD, GitHub Actions, Cloud, Serverless

> [!info]+ 🗄️ [[Programming/Software Engineering/Database Design/_Index|Database Design]]
> Relational, NoSQL, Normalization, Indexing

> [!info]+ 🔧 [[Programming/Software Engineering/Refactoring/_Index|Refactoring]]
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

← [[Programming/_Index|Back to Programming]]
