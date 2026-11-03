---
tags:
  - dashboard
  - books
banner: "https://images.unsplash.com/photo-1512820790803-83ca734da794?w=1200"
banner_y: 0.5
created: 2025-12-29
---
# 📚 Books

## 📖 Currently Reading
- [ ] **Book Title** by Author #reading
  - Started: YYYY-MM-DD
  - Progress: Chapter X/Y

## ✅ Completed 2025
- [x] Example Book by Author #completed ⭐⭐⭐⭐⭐

---

## 📋 Reading List

### 💻 Programming
- [ ] Clean Code - Robert C. Martin
- [ ] The Pragmatic Programmer - David Thomas
- [ ] Design Patterns - Gang of Four
- [ ] You Don't Know JS - Kyle Simpson
- [ ] Eloquent JavaScript - Marijn Haverbeke

### 📝 English
- [ ] Advanced Grammar in Use - Cambridge
- [ ] Word Power Made Easy - Norman Lewis
- [ ] English Vocabulary in Use (Advanced)

### 🧠 Personal Development
- [ ] Atomic Habits - James Clear
- [ ] Deep Work - Cal Newport
- [ ] The Psychology of Money - Morgan Housel

---

## 📝 Book Notes
```dataview
TABLE WITHOUT ID
	file.link as "Book",
	author as "Author",
	rating as "Rating"
FROM "Books"
WHERE !contains(file.name, "_Index") AND contains(tags, "book")
SORT file.cday desc
```

---

## ➕ Add New
Use template: [[Templates/Book Note Template|New Book Note]]
