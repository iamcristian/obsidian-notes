---
tags:
  - home
cssclass: dashboard
created: 2025-12-29
---
# 🏠 Dashboard

> [!quote]+ 💪 Remember Your Why
> **Financial freedom** · **Independence** · **Career growth** · **English fluency**

---

## 🎯 Quick Navigation

| | | |
|:---:|:---:|:---:|
| [[Programming/_Index\|💻 Programming]] | [[English/_Index\|📝 English]] | [[Books/_Index\|📚 Books]] |
| [[Business/_Index\|💼 Business]] | [[Blog/_Index\|📰 Blog]] | [[Personal/_Index\|🧘 Personal]] |

---

## 📅 Today's Schedule

![[Weekly Routine#Monday - Friday]]

---

## ✅ Priority Tasks

> [!todo]+ 💻 Programming
> ```dataview
> TASK
> FROM "Programming"
> WHERE !completed
> SORT file.cday desc
> LIMIT 5
> ```

> [!todo]+ 📝 English
> ```dataview
> TASK
> FROM "English"
> WHERE !completed
> SORT file.cday desc
> LIMIT 5
> ```

> [!todo]+ 🎯 Personal Goals
> ```dataview
> TASK
> FROM "Personal"
> WHERE !completed
> LIMIT 5
> ```

---

## 📊 Progress This Month

### 💻 Programming
| Metric | Count |
|--------|:-----:|
| Concepts Learned | `$= dv.pages('"Programming"').where(p => !p.file.name.includes("_Index")).length` |
| Algorithms Solved | `$= dv.pages('"Programming/Algorithms"').file.tasks.where(t => t.completed).length` |

### 📝 English
| Metric | Count |
|--------|:-----:|
| Words Learned | `$= dv.pages('"English/Vocabulary"').where(p => p.tags && (p.tags.includes("word") || p.tags.includes("phrasal"))).length` |
| Essays Written | `$= dv.pages('"English/Writing/Essays"').where(p => !p.file.name.includes("_Index")).length` |

### 📚 Books
| Status | Count |
|--------|:-----:|
| Reading | `$= dv.pages('"Books"').file.tasks.where(t => t.text.includes("#reading") && !t.completed).length` |
| Completed | `$= dv.pages('"Books"').file.tasks.where(t => t.text.includes("#completed")).length` |

---

## 📝 Recent Activity

```dataview
TABLE WITHOUT ID
	file.link as "Note",
	replace(file.folder, "English/", "📝 ") as "Area",
	dateformat(file.mday, "MMM dd") as "Updated"
FROM ""
WHERE !contains(file.name, "_Index") 
	AND !contains(file.name, "Home")
	AND !contains(file.path, "Templates")
	AND !contains(file.path, "Assets")
	AND !contains(file.name, "Weekly Routine")
SORT file.mday desc
LIMIT 8
```

---

## ⚡ Quick Add

| Type | Template |
|------|:--------:|
| 📖 New Word | [[Templates/Vocabulary Template\|Create]] |
| 💻 New Concept | [[Templates/Programming Concept Template\|Create]] |
| ✍️ New Essay | [[Templates/Essay Template\|Create]] |
| 📚 New Book Note | [[Templates/Book Note Template\|Create]] |
| 💡 New Idea | [[Templates/Fleeting Note Template\|Create]] |

---

## 🔗 Quick Links

| Resource | Link |
|----------|------|
| Essay Guide | [[English/Writing/Essay Guide\|Open]] |
| Essay Dictionary | [[English/Writing/Essay Dictionary\|Open]] |
| Debate Vocabulary | [[English/Vocabulary/Debate Dictionary\|Open]] |
| Git Commands | [[Programming/DevOps/Git\|Open]] |
| Weekly Routine | [[Personal/Weekly Routine\|Open]] |

