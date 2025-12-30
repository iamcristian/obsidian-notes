---
tags:
  - dashboard
  - algorithms
banner: "https://images.unsplash.com/photo-1509228468518-180dd4864904?w=1200"
banner_y: 0.5
created: 2025-12-29
---
# 🧮 Algorithms

## 📊 Progress

```chart
type: bar
labels: [Easy, Medium, Hard]
series:
  - title: Solved
    data: [0, 0, 0]
  - title: Target
    data: [50, 30, 10]
width: 80%
labelColors: true
beginAtZero: true
```

| Difficulty | Solved | Target |
|------------|:------:|:------:|
| 🟢 Easy | `$= dv.pages('"Programming/Algorithms"').file.tasks.where(t => t.text.includes("🟢") && t.completed).length` | 50 |
| 🟡 Medium | `$= dv.pages('"Programming/Algorithms"').file.tasks.where(t => t.text.includes("🟡") && t.completed).length` | 30 |
| 🔴 Hard | `$= dv.pages('"Programming/Algorithms"').file.tasks.where(t => t.text.includes("🔴") && t.completed).length` | 10 |

---

## 📚 Problems by Topic

### Arrays & Strings
- [ ] Two Sum 🟢 #arrays
- [ ] Valid Palindrome 🟢 #strings
- [ ] Three Sum 🟡 #arrays
- [ ] Container With Most Water 🟡 #arrays
- [ ] Longest Substring Without Repeating 🟡 #strings

### Linked Lists
- [ ] Reverse Linked List 🟢 #linkedlist
- [ ] Merge Two Sorted Lists 🟢 #linkedlist
- [ ] Linked List Cycle 🟢 #linkedlist
- [ ] Remove Nth Node From End 🟡 #linkedlist

### Trees
- [ ] Max Depth of Binary Tree 🟢 #trees
- [ ] Invert Binary Tree 🟢 #trees
- [ ] Validate BST 🟡 #trees
- [ ] Level Order Traversal 🟡 #trees

### Dynamic Programming
- [ ] Climbing Stairs 🟢 #dp
- [ ] House Robber 🟡 #dp
- [ ] Coin Change 🟡 #dp
- [ ] Longest Common Subsequence 🟡 #dp

### Graphs
- [ ] Number of Islands 🟡 #graphs
- [ ] Clone Graph 🟡 #graphs
- [ ] Course Schedule 🟡 #graphs

---

## ✅ Completed Problems

```tasks
done
path includes Programming/Algorithms
```

---

## 📝 Pattern Notes
```dataview
TABLE WITHOUT ID
	file.link as "Pattern",
	dateformat(file.cday, "MMM dd") as "Added"
FROM "Programming/Algorithms"
WHERE !contains(file.name, "_Index")
SORT file.cday desc
```

---

## 🔗 Resources
- [LeetCode](https://leetcode.com)
- [NeetCode Roadmap](https://neetcode.io/roadmap)
- [Blind 75](https://leetcode.com/discuss/general-discussion/460599/)
