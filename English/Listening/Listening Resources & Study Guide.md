---
tags:
  - dashboard
  - listening
created: 2025-12-29
---
# 👂 Listening

## 🎯 Weekly Goal
- [ ] 30 min daily practice
- [ ] 1 TED Talk per week
- [ ] 2 podcast episodes

---

## 📊 Practice Log
| Date | Source | Duration | Topic |
|------|--------|----------|-------|
| | | | |

---

## 📚 Resources

### Podcasts
- 6 Minute English (BBC)
- All Ears English
- Luke's English Podcast
- The English We Speak

### YouTube
- TED Talks
- Vox
- Kurzgesagt
- Real English

### Apps
- BBC Learning English
- Spotify (English podcasts)

---

## 📝 Notes
```dataview
TABLE WITHOUT ID
	file.link as "Note",
	dateformat(file.cday, "MMM dd") as "Date"
FROM "English/Listening"
WHERE !contains(file.name, "_Index")
SORT file.cday desc
```
