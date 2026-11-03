---
tags:
  - software-engineering
  - system-design
  - dashboard
created: 2026-01-02
---
# 🌐 System Design

> *"The art of designing distributed systems that are scalable, reliable, and maintainable."*

## 📚 Contents

### 🔧 Core Concepts
| Topic | Description | Status |
|-------|-------------|--------|
| [[Programming/Software Engineering/System Design/Scalability\|Scalability]] | Horizontal vs Vertical | 🔴 |
| [[Programming/Software Engineering/System Design/Load Balancing\|Load Balancing]] | Distribuir carga | 🔴 |
| [[Programming/Software Engineering/System Design/Caching\|Caching]] | Estrategias de cache | 🔴 |
| [[Programming/Software Engineering/System Design/Database Design\|Database Design]] | SQL, NoSQL, Sharding | 🔴 |
| [[Programming/Software Engineering/System Design/CAP Theorem\|CAP Theorem]] | Consistency, Availability, Partition | 🔴 |

### 🏗️ Distributed Systems
| Topic | Description | Status |
|-------|-------------|--------|
| [[Programming/Software Engineering/System Design/Message Queues\|Message Queues]] | RabbitMQ, Kafka | 🔴 |
| [[Programming/Software Engineering/System Design/API Design\|API Design]] | REST, GraphQL, gRPC | 🔴 |
| [[Programming/Software Engineering/System Design/Rate Limiting\|Rate Limiting]] | Throttling requests | 🔴 |
| [[Programming/Software Engineering/System Design/CDN\|CDN]] | Content Delivery Networks | 🔴 |
| [[Programming/Software Engineering/System Design/ETL\|ETL]] | Extract, Transform, Load pipelines | 🔴 |

### 📝 Case Studies
| System | Key Concepts | Status |
|--------|--------------|--------|
| [[Programming/Software Engineering/System Design/Cases/URL Shortener\|URL Shortener]] | Hashing, DB design | 🔴 |
| [[Programming/Software Engineering/System Design/Cases/Twitter Timeline\|Twitter Timeline]] | Fan-out, caching | 🔴 |
| [[Programming/Software Engineering/System Design/Cases/Chat Application\|Chat Application]] | WebSockets, presence | 🔴 |

---

## 🎯 System Design Framework

### 1. Requirements Clarification
```markdown
- ¿Cuántos usuarios?
- ¿Read-heavy o write-heavy?
- ¿Consistencia o disponibilidad?
- ¿Latencia aceptable?
```

### 2. Back-of-the-envelope Estimation
```markdown
- Requests per second
- Storage requirements
- Bandwidth needs
- Memory for caching
```

### 3. System Interface
```markdown
- API endpoints
- Input/Output
- Error handling
```

### 4. High-Level Design
```markdown
- Components principales
- Flujo de datos
- Database schema
```

### 5. Deep Dive
```markdown
- Bottlenecks
- Scaling strategies
- Trade-offs
```

---

## 📊 Quick Reference Numbers

| Metric | Value |
|--------|-------|
| **L1 cache** | 0.5 ns |
| **L2 cache** | 7 ns |
| **RAM** | 100 ns |
| **SSD** | 150 μs |
| **HDD** | 10 ms |
| **Network (same DC)** | 0.5 ms |
| **Network (cross-continent)** | 150 ms |

### Scale Numbers
| Users | Requests/day | DB Rows/year |
|-------|--------------|--------------|
| 1M | 86M | 365M |
| 10M | 860M | 3.65B |
| 100M | 8.6B | 36.5B |

---

## 📖 Resources

- 📕 **Book**: "Designing Data-Intensive Applications" - Martin Kleppmann
- 📕 **Book**: "System Design Interview" - Alex Xu
- 🌐 **Website**: [System Design Primer](https://github.com/donnemartin/system-design-primer)

---

## 📝 Notes
```dataview
TABLE WITHOUT ID
	file.link as "Topic",
	status as "Status"
FROM "Programming/Software Engineering/System Design"
WHERE !contains(file.name, "_Index")
SORT file.name asc
```

---

← [[Programming/Software Engineering/Software Engineering Roadmap|Back to Software Engineering]]
