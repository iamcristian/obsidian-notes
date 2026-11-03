---
tags:
  - software-engineering
  - databases
created: 2026-01-02
status: 🟡
---
# 🗄️ Database Design

> *"Data is a precious thing and will last longer than the systems themselves."* — Tim Berners-Lee

## 🗺️ Overview

Database Design es el proceso de crear un modelo de datos optimizado para almacenar, organizar y recuperar información de manera eficiente.

---

## 📚 Topics

### 📐 Relational Databases
- [[Programming/Software Engineering/Database Design/Relational Databases|Relational Databases]] - SQL fundamentals
- [[Programming/Software Engineering/Database Design/Normalization|Normalization]] - 1NF to BCNF
- [[Programming/Software Engineering/Database Design/Indexing|Indexing]] - Query optimization

### 📦 NoSQL Databases
- [[Programming/Software Engineering/Database Design/NoSQL|NoSQL Databases]] - Document, Key-Value, Graph
- [[Programming/Software Engineering/Database Design/Data Modeling NoSQL|Data Modeling for NoSQL]] - Denormalization patterns

### 🔧 Advanced Topics
- [[Programming/Software Engineering/Database Design/Transactions|Transactions]] - ACID properties
- [[Programming/Software Engineering/Database Design/Sharding|Sharding]] - Horizontal scaling
- [[Programming/Software Engineering/Database Design/Replication|Replication]] - High availability

---

## 📊 SQL vs NoSQL Decision Tree

```
                    ┌─────────────────────────┐
                    │ What type of data?      │
                    └───────────┬─────────────┘
                                │
            ┌───────────────────┼───────────────────┐
            ▼                   ▼                   ▼
    ┌───────────────┐  ┌───────────────┐   ┌───────────────┐
    │  Structured   │  │Semi-structured│   │ Relationships │
    │  (tables)     │  │   (JSON)      │   │   (graphs)    │
    └───────┬───────┘  └───────┬───────┘   └───────┬───────┘
            │                  │                   │
            ▼                  ▼                   ▼
    ┌───────────────┐  ┌───────────────┐   ┌───────────────┐
    │  SQL Database │  │   Document    │   │    Graph      │
    │  PostgreSQL   │  │   MongoDB     │   │    Neo4j      │
    │  MySQL        │  │   Firestore   │   │               │
    └───────────────┘  └───────────────┘   └───────────────┘
```

---

## 🎯 Database Selection Guide

| Requirement | Best Choice | Examples |
|-------------|-------------|----------|
| Complex queries, joins | Relational | PostgreSQL, MySQL |
| Flexible schema | Document | MongoDB, Firestore |
| High write throughput | Wide-column | Cassandra, ScyllaDB |
| Key-value lookups | Key-Value | Redis, DynamoDB |
| Relationship traversal | Graph | Neo4j, Neptune |
| Full-text search | Search engine | Elasticsearch |
| Time series data | Time series | InfluxDB, TimescaleDB |

---

## 📈 Learning Path

```dataview
TABLE status as "Status", created as "Created"
FROM "Programming/Software Engineering/Database Design"
WHERE file.name != "_Index"
SORT created ASC
```

---

← [[Programming/Software Engineering/Software Engineering Roadmap|Back to Software Engineering]]
