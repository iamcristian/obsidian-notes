---
tags:
  - dashboard
  - programming
banner: "https://images.unsplash.com/photo-1555066931-4365d14bab8c?w=1200"
banner_y: 0.4
created: 2025-12-29
---
# 💻 Programming

## 🎯 Current Focus
| Priority | Area | Status |
|:--------:|------|--------|
| 1 | Frontend (React/Next.js) | 🟡 Learning |
| 2 | Algorithms | 🟡 Practicing |
| 3 | Backend (Node/Nest) | 🔴 To Start |

---

## 🗂️ Areas

> [!info]+ 🛠️ [[Programming/Programming Hub|Development]]
> Frontend & Backend technologies
> ```dataview
> LIST WITHOUT ID file.link
> FROM "Programming/Development"
> WHERE !contains(file.name, "_Index")
> LIMIT 3
> ```

> [!info]+ 📐 [[Programming/Software Engineering/Software Engineering Roadmap|Software Engineering]]
> Clean Code, SOLID, Design Patterns

> [!info]+ 🧮 [[Programming/Algorithms/Algorithms & LeetCode Roadmap|Algorithms]]
> LeetCode, Data Structures
> ```dataview
> TASK
> FROM "Programming/Algorithms"
> WHERE completed
> LIMIT 3
> ```

> [!info]+ 🗄️ [[Programming/Software Engineering/Database Design/Database Design Complete Guide|Databases]]
> PostgreSQL, MongoDB, Redis

> [!info]+ ⚙️ [[Programming/Software Engineering/DevOps/DevOps Complete Guide|DevOps]]
> Git, Docker, CI/CD
> ```dataview
> LIST WITHOUT ID file.link
> FROM "Programming/DevOps"
> WHERE !contains(file.name, "_Index")
> LIMIT 3
> ```

---

## 📝 Recent Notes
```dataview
TABLE WITHOUT ID
	file.link as "Note",
	file.folder as "Area"
FROM "Programming"
WHERE !contains(file.name, "_Index")
SORT file.cday desc
LIMIT 5
```

---

## 🎯 Goals
- [ ] Master React + Next.js
- [ ] Solve 100 LeetCode problems
- [ ] Build 3 portfolio projects
- [ ] Learn system design
- [ ] Get first dev job 🚀
