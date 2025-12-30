---
tags:
  - dashboard
  - vocabulary
created: 2025-12-29
---
# 📖 Vocabulary

## 📊 Progress
| Category | Learned | Target |
|----------|---------|--------|
| General | `$= dv.pages('"English/Vocabulary"').where(p => p.tags && p.tags.includes("word")).length` | 200 |
| Phrasal Verbs | `$= dv.pages('"English/Vocabulary"').where(p => p.tags && p.tags.includes("phrasal")).length` | 100 |

---

## 📚 Resources
- [[Debate Dictionary]] - Formal vocabulary for arguments
- [[Debate Quick Reference]] - Quick lookup

---

## 🔥 Review Schedule

### Today (learned yesterday)
```dataview
LIST WITHOUT ID file.link + " — " + spanish
FROM "English/Vocabulary"
WHERE (contains(tags, "word") OR contains(tags, "phrasal")) 
	AND file.cday = date(today) - dur(1 day)
```

### This Week
```dataview
LIST WITHOUT ID file.link + " — " + spanish
FROM "English/Vocabulary"
WHERE (contains(tags, "word") OR contains(tags, "phrasal"))
	AND file.cday >= date(today) - dur(7 days)
SORT file.cday desc
LIMIT 10
```

---

## 📝 All Words
```dataview
TABLE WITHOUT ID
	file.link as "Word",
	spanish as "Spanish",
	part_of_speech as "Type"
FROM "English/Vocabulary"
WHERE contains(tags, "word") OR contains(tags, "phrasal")
SORT file.cday desc
```

---

## ➕ Add New
Use template: [[Templates/Vocabulary Template|New Word]] or [[Templates/Phrasal Verb Template|New Phrasal Verb]]
