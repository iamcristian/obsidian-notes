---
tags:
  - dashboard
  - vocabulary
banner: "https://images.unsplash.com/photo-1457369804613-52c61a468e7d?w=1200"
banner_y: 0.5
created: 2025-12-29
---
# 📖 Vocabulary

## 📊 Progress

```chart
type: doughnut
labels: [Words Learned, Remaining]
series:
  - title: Progress
    data: [2, 198]
width: 50%
labelColors: true
```

| Category      |                                            Learned                                            | Target |                                                              %                                                              |
| ------------- | :-------------------------------------------------------------------------------------------: | :----: | :-------------------------------------------------------------------------------------------------------------------------: |
| General       |  `$= dv.pages('"English/Vocabulary"').where(p => p.tags && p.tags.includes("word")).length`   |  200   |  `$= Math.round(dv.pages('"English/Vocabulary"').where(p => p.tags && p.tags.includes("word")).length / 200 * 100) + "%"`   |
| Phrasal Verbs | `$= dv.pages('"English/Vocabulary"').where(p => p.tags && p.tags.includes("phrasal")).length` |  100   | `$= Math.round(dv.pages('"English/Vocabulary"').where(p => p.tags && p.tags.includes("phrasal")).length / 100 * 100) + "%"` |

---

## 📚 Categorías Organizadas

### 📖 C1 Vocabulary
**10 categorías temáticas** (estilo "English Vocabulary in Use Advanced")

📂 [[C1 Vocabulary/_Index|C1 Vocabulary Index]]

Incluye: Work & Study, People & Relationships, Leisure, Travel, Environment, Society, Media, Health, Technology, Words & Meanings

---

### 🚀 Phrasal Verbs
**200+ phrasal verbs** organizados por partícula para memorización eficiente

📂 [[Phrasal Verbs/_Index|Phrasal Verbs Index]]

Partículas: UP, DOWN, OUT, ON, OFF, AWAY, THROUGH, OVER, BACK, IN

---

### 🎤 Debate Vocabulary
**Vocabulario especializado** para debates formales y discusión académica

📂 [[Debate/_Index|Debate Vocabulary Index]]

Incluye: Expressing opinions, agreeing/disagreeing, argumentación avanzada

---

### 🔧 Tools
- 📖 Use **Dictionary** plugin: Select any word → Cmd/Ctrl+Shift+D

---

## 🔥 Review Schedule (Spaced Repetition)

> [!tip]+ 📅 Today (review yesterday's words)
> ```dataview
> LIST WITHOUT ID file.link + " — " + spanish
> FROM "English/Vocabulary"
> WHERE (contains(tags, "word") OR contains(tags, "phrasal")) 
> 	AND file.cday = date(today) - dur(1 day)
> ```

> [!note]+ 📆 This Week
> ```dataview
> LIST WITHOUT ID file.link + " — " + spanish
> FROM "English/Vocabulary"
> WHERE (contains(tags, "word") OR contains(tags, "phrasal"))
> 	AND file.cday >= date(today) - dur(7 days)
> SORT file.cday desc
> LIMIT 10
> ```

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
