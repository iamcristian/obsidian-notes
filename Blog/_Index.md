---
tags:
  - dashboard
  - blog
created: 2025-12-29
---
# 📰 Blog

## 📊 Status Overview
| Status | Count |
|--------|:-----:|
| 💡 Ideas | `$= dv.pages('"Blog"').where(p => p.status == "idea").length` |
| 📝 Drafts | `$= dv.pages('"Blog"').where(p => p.status == "draft").length` |
| ✅ Published | `$= dv.pages('"Blog"').where(p => p.status == "published").length` |

---

## 📝 Drafts
```dataview
TABLE WITHOUT ID
	file.link as "Post",
	category as "Category",
	dateformat(file.cday, "MMM dd") as "Started"
FROM "Blog"
WHERE !contains(file.name, "_Index") AND status = "draft"
SORT file.cday desc
```

## ✅ Published
```dataview
TABLE WITHOUT ID
	file.link as "Post",
	category as "Category",
	dateformat(file.cday, "MMM dd") as "Date"
FROM "Blog"
WHERE !contains(file.name, "_Index") AND status = "published"
SORT file.cday desc
```

---

## 💡 Post Ideas
- [ ] How I'm learning to code
- [ ] My journey learning English
- [ ] Useful Git commands for beginners
- [ ] React hooks explained simply

---

## ➕ Add New
Use template: [[Templates/Blog Post Template|New Blog Post]]
