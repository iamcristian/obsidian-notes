---
tags:
  - dashboard
  - writing
created: 2025-12-29
---
# ✍️ Writing

## 📚 Resources
- [[Essay Guide]] - How to write essays
- [[Essay Dictionary]] - Useful phrases
- [[Essay Quick Reference]] - Quick lookup

---

## ✍️ My Essays

### 📝 In Progress
```dataview
TABLE WITHOUT ID
	file.link as "Essay",
	essay_type as "Type",
	dateformat(file.cday, "MMM dd") as "Started"
FROM "English/Writing/Essays"
WHERE !contains(file.name, "_Index") AND (!status OR status = "draft")
SORT file.cday desc
```

### ✅ Completed
```dataview
TABLE WITHOUT ID
	file.link as "Essay",
	essay_type as "Type",
	word_count as "Words"
FROM "English/Writing/Essays"
WHERE !contains(file.name, "_Index") AND status = "done"
SORT file.cday desc
```

---

## 📊 Stats
| Type | Written | Target |
|------|---------|--------|
| Argumentative | `$= dv.pages('"English/Writing/Essays"').where(p => p.essay_type == "argumentative").length` | 5 |
| Persuasive | `$= dv.pages('"English/Writing/Essays"').where(p => p.essay_type == "persuasive").length` | 5 |
| Expository | `$= dv.pages('"English/Writing/Essays"').where(p => p.essay_type == "expository").length` | 3 |

---

## 💡 Essay Ideas
- [ ] Technology: benefit or harm?
- [ ] Should coding be mandatory in schools?
- [ ] Remote work vs office work
- [ ] AI impact on jobs
- [ ] Social media and mental health

---

## ➕ Add New
Use template: [[Templates/Essay Template|New Essay]]
