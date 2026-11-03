---
tags:
  - software-engineering
  - databases
  - normalization
created: 2026-01-02
status: 🔴
---
# 📏 Database Normalization

> *"Normalization is the process of organizing data to minimize redundancy."*

## 🎯 What is Normalization?

Normalización es el proceso de estructurar una base de datos relacional para **reducir redundancia** y **mejorar integridad de datos** mediante reglas progresivas llamadas "formas normales".

---

## 🎯 Goals of Normalization

1. **Eliminate redundant data** - No duplicar información
2. **Ensure data dependencies make sense** - Relaciones lógicas
3. **Reduce storage space** - Menos datos duplicados
4. **Prevent anomalies** - Insert, update, delete anomalies

---

## ⚠️ Anomalies Without Normalization

### Example: Denormalized Table
```
┌────────────────────────────────────────────────────────────────┐
│ student_courses (NOT NORMALIZED)                                │
├──────────┬─────────────┬────────────┬───────────┬─────────────┤
│ StudentId│ StudentName │ CourseId   │ CourseName│ Instructor  │
├──────────┼─────────────┼────────────┼───────────┼─────────────┤
│ 1        │ John Doe    │ CS101      │ Intro CS  │ Dr. Smith   │
│ 1        │ John Doe    │ MATH201    │ Calculus  │ Dr. Jones   │
│ 2        │ Jane Smith  │ CS101      │ Intro CS  │ Dr. Smith   │
│ 2        │ Jane Smith  │ ENG101     │ English   │ Dr. Brown   │
└──────────┴─────────────┴────────────┴───────────┴─────────────┘
```

### Problems:

**Insert Anomaly**: Can't add a new course without a student
```
❌ INSERT INTO student_courses (CourseId, CourseName, Instructor)
   VALUES ('PHYS101', 'Physics', 'Dr. Newton');
   -- ERROR: StudentId cannot be null
```

**Update Anomaly**: Changing course name requires multiple updates
```
❌ UPDATE student_courses SET CourseName = 'Computer Science 101'
   WHERE CourseId = 'CS101';
   -- Must update EVERY row with CS101
   -- Risk of inconsistency if one is missed
```

**Delete Anomaly**: Deleting a student might delete course info
```
❌ DELETE FROM student_courses WHERE StudentId = 2 AND CourseId = 'ENG101';
   -- If Jane was the only student in ENG101, we lose course info
```

---

## 📊 Normal Forms Progression

```
┌─────────────────────────────────────────────────────────────────┐
│                    NORMAL FORMS HIERARCHY                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                         BCNF                             │   │
│   │   ┌─────────────────────────────────────────────────┐   │   │
│   │   │                       3NF                        │   │   │
│   │   │   ┌─────────────────────────────────────────┐   │   │   │
│   │   │   │                   2NF                    │   │   │   │
│   │   │   │   ┌─────────────────────────────────┐   │   │   │   │
│   │   │   │   │               1NF                │   │   │   │   │
│   │   │   │   │                                  │   │   │   │   │
│   │   │   │   │   Atomic values                  │   │   │   │   │
│   │   │   │   │   No repeating groups            │   │   │   │   │
│   │   │   │   └─────────────────────────────────┘   │   │   │   │
│   │   │   │   + No partial dependencies             │   │   │   │
│   │   │   └─────────────────────────────────────────┘   │   │   │
│   │   │   + No transitive dependencies                  │   │   │
│   │   └─────────────────────────────────────────────────┘   │   │
│   │   + Every determinant is a candidate key                │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ First Normal Form (1NF)

### Rules:
1. Each column contains **atomic** (indivisible) values
2. No **repeating groups** or arrays
3. Each row is unique (has a primary key)

### ❌ Not in 1NF
```
┌──────────┬─────────────┬────────────────────────────────┐
│ StudentId│ StudentName │ Courses                        │
├──────────┼─────────────┼────────────────────────────────┤
│ 1        │ John Doe    │ CS101, MATH201, ENG101         │ ◄─ Not atomic
│ 2        │ Jane Smith  │ CS101                          │
└──────────┴─────────────┴────────────────────────────────┘
```

### ✅ In 1NF
```
┌──────────┬─────────────┬──────────┐
│ StudentId│ StudentName │ CourseId │
├──────────┼─────────────┼──────────┤
│ 1        │ John Doe    │ CS101    │
│ 1        │ John Doe    │ MATH201  │
│ 1        │ John Doe    │ ENG101   │
│ 2        │ Jane Smith  │ CS101    │
└──────────┴─────────────┴──────────┘
```

---

## 2️⃣ Second Normal Form (2NF)

### Rules:
1. Must be in 1NF
2. No **partial dependencies** (non-key attribute depends on part of composite key)

### ❌ Not in 2NF
```
┌──────────┬──────────┬─────────────┬────────────┐
│ StudentId│ CourseId │ StudentName │ CourseName │
├──────────┼──────────┼─────────────┼────────────┤
│ 1        │ CS101    │ John Doe    │ Intro CS   │
│ 1        │ MATH201  │ John Doe    │ Calculus   │
│ 2        │ CS101    │ Jane Smith  │ Intro CS   │
└──────────┴──────────┴─────────────┴────────────┘

PK: (StudentId, CourseId)

Partial dependencies:
- StudentName depends only on StudentId ❌
- CourseName depends only on CourseId ❌
```

### ✅ In 2NF (Split into multiple tables)
```
students:                   courses:
┌──────────┬─────────────┐  ┌──────────┬────────────┐
│ StudentId│ StudentName │  │ CourseId │ CourseName │
├──────────┼─────────────┤  ├──────────┼────────────┤
│ 1        │ John Doe    │  │ CS101    │ Intro CS   │
│ 2        │ Jane Smith  │  │ MATH201  │ Calculus   │
└──────────┴─────────────┘  └──────────┴────────────┘

enrollments:
┌──────────┬──────────┐
│ StudentId│ CourseId │
├──────────┼──────────┤
│ 1        │ CS101    │
│ 1        │ MATH201  │
│ 2        │ CS101    │
└──────────┴──────────┘
```

---

## 3️⃣ Third Normal Form (3NF)

### Rules:
1. Must be in 2NF
2. No **transitive dependencies** (non-key depends on another non-key)

### ❌ Not in 3NF
```
┌──────────┬─────────────┬──────────────┬───────────────┐
│ StudentId│ StudentName │ DepartmentId │DepartmentName │
├──────────┼─────────────┼──────────────┼───────────────┤
│ 1        │ John Doe    │ CS           │ Computer Sci  │
│ 2        │ Jane Smith  │ MATH         │ Mathematics   │
│ 3        │ Bob Brown   │ CS           │ Computer Sci  │
└──────────┴─────────────┴──────────────┴───────────────┘

Transitive dependency:
StudentId → DepartmentId → DepartmentName
(DepartmentName depends on DepartmentId, not directly on StudentId)
```

### ✅ In 3NF
```
students:                      departments:
┌──────────┬─────────────┬────────────┐  ┌──────────────┬───────────────┐
│ StudentId│ StudentName │DepartmentId│  │ DepartmentId │DepartmentName │
├──────────┼─────────────┼────────────┤  ├──────────────┼───────────────┤
│ 1        │ John Doe    │ CS         │  │ CS           │ Computer Sci  │
│ 2        │ Jane Smith  │ MATH       │  │ MATH         │ Mathematics   │
│ 3        │ Bob Brown   │ CS         │  └──────────────┴───────────────┘
└──────────┴─────────────┴────────────┘
```

---

## 🅱️ Boyce-Codd Normal Form (BCNF)

### Rules:
1. Must be in 3NF
2. For every functional dependency X → Y, X must be a **superkey**

### ❌ Not in BCNF
```
┌──────────┬──────────┬────────────┐
│ StudentId│ Subject  │ Teacher    │
├──────────┼──────────┼────────────┤
│ 1        │ Math     │ Dr. Smith  │
│ 2        │ Math     │ Dr. Smith  │
│ 3        │ Physics  │ Dr. Jones  │
│ 1        │ Physics  │ Dr. Brown  │
└──────────┴──────────┴────────────┘

PK: (StudentId, Subject)

But: Teacher → Subject (each teacher teaches one subject)
Teacher is NOT a superkey, violating BCNF
```

### ✅ In BCNF
```
teachers:                      student_teachers:
┌──────────┬──────────┐        ┌──────────┬──────────┐
│ Teacher  │ Subject  │        │ StudentId│ Teacher  │
├──────────┼──────────┤        ├──────────┼──────────┤
│ Dr. Smith│ Math     │        │ 1        │ Dr. Smith│
│ Dr. Jones│ Physics  │        │ 2        │ Dr. Smith│
│ Dr. Brown│ Physics  │        │ 3        │ Dr. Jones│
└──────────┴──────────┘        │ 1        │ Dr. Brown│
                               └──────────┴──────────┘
```

---

## ⚖️ When to Denormalize

```
┌─────────────────────────────────────────────────────────────────┐
│                 NORMALIZE vs DENORMALIZE                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  NORMALIZE when:                                                │
│  • Data integrity is critical                                   │
│  • Storage is a concern                                         │
│  • Writes are more frequent than reads                          │
│  • Data changes frequently                                      │
│                                                                 │
│  DENORMALIZE when:                                              │
│  • Read performance is critical                                 │
│  • Data rarely changes                                          │
│  • Complex joins hurt performance                               │
│  • Reporting/analytics use cases                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Denormalization Strategies
```sql
-- Computed/derived columns
ALTER TABLE orders ADD COLUMN item_count INT;
-- Update with trigger or application logic

-- Caching aggregates
CREATE TABLE user_stats (
    user_id INT PRIMARY KEY,
    total_orders INT DEFAULT 0,
    total_spent DECIMAL(10,2) DEFAULT 0,
    last_order_date TIMESTAMP
);

-- Materialized views
CREATE MATERIALIZED VIEW sales_summary AS
SELECT 
    DATE_TRUNC('month', created_at) AS month,
    COUNT(*) AS orders,
    SUM(total) AS revenue
FROM orders
GROUP BY 1;
```

---

## 📋 Normalization Quick Reference

| Form | Requirement | Eliminates |
|------|-------------|------------|
| **1NF** | Atomic values, no repeating groups | Repeating groups |
| **2NF** | 1NF + No partial dependencies | Partial dependencies |
| **3NF** | 2NF + No transitive dependencies | Transitive dependencies |
| **BCNF** | 3NF + Every determinant is a key | Non-key determinants |

---

## 📚 Related

- [[Programming/Software Engineering/Database Design/Relational Databases|Relational Databases]]
- [[Programming/Software Engineering/Database Design/Indexing|Indexing]]

---

← [[Programming/Software Engineering/Database Design/Database Design Complete Guide|Back to Database Design]]
