---
tags:
  - home
cssclass: dashboard
banner: "![[home.jpg]]"
banner_y: 0.5
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
> ```tasks
> not done
> path includes Programming
> sort by priority
> limit 5
> ```

> [!todo]+ 📝 English
> ```tasks
> not done
> path includes English
> sort by priority
> limit 5
> ```

> [!todo]+ 🎯 Personal Goals
> ```tasks
> not done
> path includes Personal
> limit 5
> ```

---

## 📊 Progress This Month

```chart
type: bar
labels: [Programming, English, Books]
series:
  - title: Notes Created
    data: [5, 8, 2]
width: 80%
labelColors: true
beginAtZero: true
```

| Area | Notes | Tasks Done |
|------|:-----:|:----------:|
| 💻 Programming | `$= dv.pages('"Programming"').where(p => !p.file.name.includes("_Index")).length` | `$= dv.pages('"Programming"').file.tasks.where(t => t.completed).length` |
| 📝 English | `$= dv.pages('"English"').where(p => !p.file.name.includes("_Index") && !p.file.name.includes("Dictionary") && !p.file.name.includes("Reference") && !p.file.name.includes("Guide")).length` | `$= dv.pages('"English"').file.tasks.where(t => t.completed).length` |
| 📚 Books | `$= dv.pages('"Books"').where(p => !p.file.name.includes("_Index")).length` | `$= dv.pages('"Books"').file.tasks.where(t => t.completed).length` |

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
| Plugin Guide | [[Templates/Plugin Guide\|Open]] |

