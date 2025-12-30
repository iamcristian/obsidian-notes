---
tags:
  - dashboard
  - databases
created: 2025-12-29
---
# 🗄️ Databases

## 📊 Technologies

### SQL
| Database | Status |
|----------|--------|
| PostgreSQL | 🔴 To Learn |
| MySQL | 🟡 Basic |

### NoSQL
| Database | Status | Use Case |
|----------|--------|----------|
| MongoDB | 🔴 To Learn | Documents |
| Redis | 🔴 To Learn | Cache |

---

## 📚 Topics to Learn
- [ ] SQL fundamentals (SELECT, JOIN, GROUP BY)
- [ ] Database design & normalization
- [ ] Indexing & query optimization
- [ ] Transactions & ACID
- [ ] SQL vs NoSQL - when to use each
- [ ] ORMs (Prisma, TypeORM, Sequelize)

---

## 📝 My Notes
```dataview
TABLE WITHOUT ID
	file.link as "Note",
	dateformat(file.cday, "MMM dd") as "Added"
FROM "Programming/Databases"
WHERE !contains(file.name, "_Index")
SORT file.cday desc
```

---

## ➕ Add New
Use template: [[Templates/Programming Concept Template|New Concept]]
