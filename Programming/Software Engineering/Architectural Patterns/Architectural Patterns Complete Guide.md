---
tags:
  - software-engineering
  - architectural-patterns
  - dashboard
created: 2026-01-02
---
# 🏗️ Architectural Patterns

> *"Architecture is about the important stuff. Whatever that is."* — Ralph Johnson

## 📚 Contents

### 🎯 Application Architecture
| Pattern | Description | Status |
|---------|-------------|--------|
| [[Programming/Software Engineering/Architectural Patterns/Hexagonal Architecture\|Hexagonal Architecture]] | Ports & Adapters | 🔴 |
| [[Programming/Software Engineering/Architectural Patterns/Onion Architecture\|Onion Architecture]] | Capas concéntricas | 🔴 |
| [[Programming/Software Engineering/Architectural Patterns/Layered Architecture\|Layered Architecture]] | N-Tier tradicional | 🔴 |
| [[Programming/Software Engineering/Architectural Patterns/Vertical Slice\|Vertical Slice]] | Organización por feature | 🔴 |

### 🌐 Distributed Systems
| Pattern | Description | Status |
|---------|-------------|--------|
| [[Programming/Software Engineering/Architectural Patterns/Microservices\|Microservices]] | Servicios independientes | 🔴 |
| [[Programming/Software Engineering/Architectural Patterns/Event-Driven Architecture\|Event-Driven Architecture]] | Comunicación por eventos | 🔴 |
| [[Programming/Software Engineering/Architectural Patterns/CQRS\|CQRS]] | Command Query Responsibility Segregation | 🔴 |
| [[Programming/Software Engineering/Architectural Patterns/Event Sourcing\|Event Sourcing]] | Estado como secuencia de eventos | 🔴 |
| [[Programming/Software Engineering/Architectural Patterns/Microfrontends\|Microfrontends]] | Frontend microservice architecture | 🔴 |

### 📱 Presentation Patterns
| Pattern | Description | Status |
|---------|-------------|--------|
| [[Programming/Software Engineering/Architectural Patterns/MVC\|MVC]] | Model-View-Controller | 🔴 |
| [[Programming/Software Engineering/Architectural Patterns/MVP\|MVP]] | Model-View-Presenter | 🔴 |
| [[Programming/Software Engineering/Architectural Patterns/MVVM\|MVVM]] | Model-View-ViewModel | 🔴 |

---

## 🎯 Quick Comparison

```
┌─────────────────┬─────────────────┬─────────────────┐
│   Hexagonal     │     Onion       │  Clean Arch     │
├─────────────────┼─────────────────┼─────────────────┤
│  Ports/Adapters │  Infrastructure │  Frameworks     │
│                 │  Services       │  Interface Adap │
│  Application    │  Application    │  Use Cases      │
│  Domain         │  Domain         │  Entities       │
└─────────────────┴─────────────────┴─────────────────┘
        ↑ Todas comparten: Domain en el centro ↑
```

---

## 📖 Resources

- 📕 **Book**: "Patterns of Enterprise Application Architecture" - Martin Fowler
- 📕 **Book**: "Building Microservices" - Sam Newman
- 📕 **Book**: "Domain-Driven Design" - Eric Evans

---

## 📝 Notes
```dataview
TABLE WITHOUT ID
	file.link as "Pattern",
	status as "Status"
FROM "Programming/Software Engineering/Architectural Patterns"
WHERE !contains(file.name, "_Index")
SORT file.name asc
```

---

← [[Programming/Software Engineering/Software Engineering Roadmap|Back to Software Engineering]]
