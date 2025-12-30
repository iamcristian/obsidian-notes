---
tags:
  - dashboard
  - development
created: 2025-12-29
---
# 🛠️ Development

## 🎨 Frontend
| Technology | Status | Priority |
|------------|--------|:--------:|
| React | 🟡 Learning | 1 |
| Next.js | 🔴 To Learn | 2 |
| TypeScript | 🟡 Learning | 1 |
| Angular | 🟠 Basic | 3 |
| Tailwind CSS | 🟡 Learning | 2 |

## ⚙️ Backend
| Technology | Status | Priority |
|------------|--------|:--------:|
| Node.js | 🟡 Learning | 1 |
| NestJS | 🔴 To Learn | 2 |
| Express | 🟡 Basic | 3 |

---

## 📝 My Notes
```dataview
TABLE WITHOUT ID
	file.link as "Note",
	dateformat(file.cday, "MMM dd") as "Added"
FROM "Programming/Development"
WHERE !contains(file.name, "_Index")
SORT file.cday desc
```

---

## ➕ Add New
Use template: [[Templates/Programming Concept Template|New Concept]] or [[Templates/Code Snippet Template|New Snippet]]
