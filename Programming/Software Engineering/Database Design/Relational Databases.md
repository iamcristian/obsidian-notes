---
tags:
  - software-engineering
  - databases
  - sql
  - relational
created: 2026-01-02
status: 🔴
---
# 📐 Relational Databases

> *"The relational model provides the only means for expressing data constraints that preserve data integrity."*

## 🎯 What is a Relational Database?

Una base de datos relacional organiza datos en **tablas** (relaciones) con **filas** (tuplas) y **columnas** (atributos), conectadas mediante **claves**.

---

## 🏗️ Core Concepts

### Tables, Rows, and Columns
```sql
-- Table: users
-- Columns: id, name, email, created_at
-- Each row is a record/tuple

CREATE TABLE users (
    id          SERIAL PRIMARY KEY,
    name        VARCHAR(100) NOT NULL,
    email       VARCHAR(255) UNIQUE NOT NULL,
    created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Keys

```sql
-- PRIMARY KEY: Unique identifier for each row
-- FOREIGN KEY: Reference to another table
-- UNIQUE KEY: Ensures uniqueness (can have nulls)
-- COMPOSITE KEY: Multiple columns as key

CREATE TABLE orders (
    id          SERIAL PRIMARY KEY,          -- Primary key
    user_id     INT NOT NULL,                -- Will be foreign key
    total       DECIMAL(10,2) NOT NULL,
    
    FOREIGN KEY (user_id) REFERENCES users(id)  -- Foreign key
);

-- Composite primary key
CREATE TABLE order_items (
    order_id    INT,
    product_id  INT,
    quantity    INT NOT NULL,
    
    PRIMARY KEY (order_id, product_id)  -- Composite key
);
```

---

## 🔗 Relationships

### One-to-One (1:1)
```sql
-- User has one profile
CREATE TABLE users (
    id    SERIAL PRIMARY KEY,
    name  VARCHAR(100)
);

CREATE TABLE profiles (
    id       SERIAL PRIMARY KEY,
    user_id  INT UNIQUE NOT NULL,  -- UNIQUE ensures 1:1
    bio      TEXT,
    avatar   VARCHAR(255),
    
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

### One-to-Many (1:N)
```sql
-- Author has many books
CREATE TABLE authors (
    id    SERIAL PRIMARY KEY,
    name  VARCHAR(100)
);

CREATE TABLE books (
    id         SERIAL PRIMARY KEY,
    author_id  INT NOT NULL,  -- Many books per author
    title      VARCHAR(255),
    
    FOREIGN KEY (author_id) REFERENCES authors(id)
);
```

### Many-to-Many (M:N)
```sql
-- Students enrolled in many courses
-- Courses have many students
-- Junction/Bridge table needed

CREATE TABLE students (
    id    SERIAL PRIMARY KEY,
    name  VARCHAR(100)
);

CREATE TABLE courses (
    id    SERIAL PRIMARY KEY,
    name  VARCHAR(100)
);

-- Junction table
CREATE TABLE enrollments (
    student_id  INT,
    course_id   INT,
    enrolled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    grade       CHAR(2),
    
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(id),
    FOREIGN KEY (course_id) REFERENCES courses(id)
);
```

---

## 📝 Essential SQL Operations

### CRUD Operations
```sql
-- CREATE (Insert)
INSERT INTO users (name, email) 
VALUES ('John Doe', 'john@example.com');

INSERT INTO users (name, email) VALUES
    ('Jane Doe', 'jane@example.com'),
    ('Bob Smith', 'bob@example.com');

-- READ (Select)
SELECT * FROM users;
SELECT name, email FROM users WHERE id = 1;
SELECT * FROM users WHERE name LIKE 'John%';

-- UPDATE
UPDATE users 
SET email = 'newemail@example.com' 
WHERE id = 1;

-- DELETE
DELETE FROM users WHERE id = 1;

-- UPSERT (Insert or Update)
INSERT INTO users (id, name, email)
VALUES (1, 'John', 'john@example.com')
ON CONFLICT (id) 
DO UPDATE SET name = EXCLUDED.name, email = EXCLUDED.email;
```

### JOINs
```sql
-- INNER JOIN: Only matching rows
SELECT users.name, orders.total
FROM users
INNER JOIN orders ON users.id = orders.user_id;

-- LEFT JOIN: All from left, matching from right
SELECT users.name, orders.total
FROM users
LEFT JOIN orders ON users.id = orders.user_id;

-- RIGHT JOIN: All from right, matching from left
SELECT users.name, orders.total
FROM users
RIGHT JOIN orders ON users.id = orders.user_id;

-- FULL OUTER JOIN: All rows from both
SELECT users.name, orders.total
FROM users
FULL OUTER JOIN orders ON users.id = orders.user_id;

-- Multiple JOINs
SELECT 
    users.name,
    orders.id AS order_id,
    products.name AS product_name,
    order_items.quantity
FROM users
JOIN orders ON users.id = orders.user_id
JOIN order_items ON orders.id = order_items.order_id
JOIN products ON order_items.product_id = products.id;
```

### Aggregations
```sql
-- COUNT, SUM, AVG, MIN, MAX
SELECT 
    COUNT(*) AS total_orders,
    SUM(total) AS revenue,
    AVG(total) AS avg_order,
    MIN(total) AS smallest_order,
    MAX(total) AS largest_order
FROM orders;

-- GROUP BY
SELECT 
    user_id,
    COUNT(*) AS order_count,
    SUM(total) AS total_spent
FROM orders
GROUP BY user_id;

-- HAVING (filter after GROUP BY)
SELECT 
    user_id,
    COUNT(*) AS order_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 5;

-- Window functions
SELECT 
    id,
    user_id,
    total,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY total DESC) AS rank,
    SUM(total) OVER (PARTITION BY user_id) AS user_total,
    AVG(total) OVER () AS overall_avg
FROM orders;
```

### Subqueries
```sql
-- Scalar subquery
SELECT name, 
    (SELECT COUNT(*) FROM orders WHERE orders.user_id = users.id) AS order_count
FROM users;

-- IN subquery
SELECT * FROM users
WHERE id IN (SELECT user_id FROM orders WHERE total > 100);

-- EXISTS subquery
SELECT * FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);

-- FROM subquery (derived table)
SELECT *
FROM (
    SELECT user_id, SUM(total) AS total_spent
    FROM orders
    GROUP BY user_id
) AS user_totals
WHERE total_spent > 1000;

-- CTE (Common Table Expression)
WITH high_spenders AS (
    SELECT user_id, SUM(total) AS total_spent
    FROM orders
    GROUP BY user_id
    HAVING SUM(total) > 1000
)
SELECT users.*, high_spenders.total_spent
FROM users
JOIN high_spenders ON users.id = high_spenders.user_id;
```

---

## 🔒 Constraints

```sql
CREATE TABLE products (
    id           SERIAL PRIMARY KEY,
    name         VARCHAR(100) NOT NULL,           -- Cannot be NULL
    sku          VARCHAR(50) UNIQUE,              -- Must be unique
    price        DECIMAL(10,2) CHECK (price > 0), -- Custom validation
    category_id  INT REFERENCES categories(id),   -- Foreign key
    created_at   TIMESTAMP DEFAULT NOW()          -- Default value
);

-- Add constraints later
ALTER TABLE products 
ADD CONSTRAINT positive_price CHECK (price > 0);

ALTER TABLE products 
ADD CONSTRAINT fk_category 
FOREIGN KEY (category_id) REFERENCES categories(id) 
ON DELETE SET NULL 
ON UPDATE CASCADE;
```

### Referential Actions
```sql
-- ON DELETE options:
-- CASCADE    - Delete child rows
-- SET NULL   - Set FK to NULL
-- SET DEFAULT - Set FK to default
-- RESTRICT   - Prevent deletion
-- NO ACTION  - Similar to RESTRICT

FOREIGN KEY (user_id) REFERENCES users(id) 
    ON DELETE CASCADE 
    ON UPDATE CASCADE;
```

---

## 📊 Data Types (PostgreSQL)

| Category | Types |
|----------|-------|
| **Numeric** | INTEGER, BIGINT, DECIMAL, NUMERIC, REAL, DOUBLE PRECISION |
| **String** | VARCHAR(n), CHAR(n), TEXT |
| **Boolean** | BOOLEAN |
| **Date/Time** | DATE, TIME, TIMESTAMP, TIMESTAMPTZ, INTERVAL |
| **Binary** | BYTEA |
| **JSON** | JSON, JSONB |
| **Array** | INTEGER[], TEXT[], etc. |
| **UUID** | UUID |

---

## 🎯 Best Practices

> [!check] Schema Design
> - [ ] Use appropriate data types
> - [ ] Add NOT NULL where appropriate
> - [ ] Use meaningful column names
> - [ ] Add indexes for frequently queried columns
> - [ ] Use foreign keys for referential integrity

> [!check] Query Writing
> - [ ] Select only needed columns (avoid SELECT *)
> - [ ] Use parameterized queries (prevent SQL injection)
> - [ ] Use EXPLAIN ANALYZE for optimization
> - [ ] Use transactions for multiple operations

---

## 📚 Related

- [[Programming/Software Engineering/Database Design/Normalization|Normalization]]
- [[Programming/Software Engineering/Database Design/Indexing|Indexing]]
- [[Programming/Software Engineering/Database Design/Transactions|Transactions]]

---

← [[Programming/Software Engineering/Database Design/_Index|Back to Database Design]]
