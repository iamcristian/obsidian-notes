---
tags:
  - dashboard
  - personal
banner: "https://images.unsplash.com/photo-1499750310107-5fef28a66643?w=1200"
banner_y: 0.4
created: 2025-12-29
---
# 🧘 Personal

## 🎯 2025 Goals

### 💻 Career
- [ ] Get first developer job
- [ ] Master React + Next.js
- [ ] Build portfolio with 3+ projects

### 📝 English
- [ ] Reach C1 level
- [ ] Pass Cambridge exam
- [ ] Write 20 essays

### 💪 Habits
- [ ] Exercise 3x per week
- [ ] Read 30 min daily
- [ ] Code every day

---

## 📅 Weekly Routine
![[Weekly Routine#📆 Weekly Schedule]]

---

## 💪 Why I'm Doing This

> - 💰 **Financial freedom** for my family
> - 🔓 **Independence** - not depending on others
> - 🚀 **Career path**: Software Engineer → Cybersecurity → AI
> - 🗣️ **English fluency** - open more opportunities

---

## 📝 Notes
```dataview
TABLE WITHOUT ID
	file.link as "Note",
	dateformat(file.cday, "MMM dd") as "Added"
FROM "Personal"
WHERE !contains(file.name, "_Index") 
	AND !contains(file.name, "Weekly Routine")
	AND !contains(file.name, "Learning Goals")
SORT file.cday desc
```
