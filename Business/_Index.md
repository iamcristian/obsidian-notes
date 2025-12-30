---
tags:
  - dashboard
  - business
created: 2025-12-29
---
# 💼 Business

## 💡 Ideas
- [ ] Idea 1
- [ ] Idea 2
- [ ] Idea 3

---

## 🎯 Projects
```dataview
TABLE WITHOUT ID
	file.link as "Project",
	status as "Status"
FROM "Business"
WHERE !contains(file.name, "_Index") AND contains(tags, "project")
SORT file.cday desc
```

---

## 📝 Notes
```dataview
TABLE WITHOUT ID
	file.link as "Note",
	dateformat(file.cday, "MMM dd") as "Added"
FROM "Business"
WHERE !contains(file.name, "_Index")
SORT file.cday desc
```

---

## ➕ Add New
Use template: [[Templates/Project Template|New Project]] or [[Templates/Fleeting Note Template|New Idea]]
