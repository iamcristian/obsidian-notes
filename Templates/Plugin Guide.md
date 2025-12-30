---
tags:
  - reference
  - guide
banner: "https://images.unsplash.com/photo-1618401471353-b98afee0b2eb?w=1200"
banner_y: 0.5
created: 2025-12-29
---
# 🔌 Plugin Guide

Quick reference for your installed Obsidian plugins.

---

## 🖼️ Banners

Adds beautiful header images to notes.

**How to use:**
```yaml
---
banner: "https://images.unsplash.com/photo-ID?w=1200"
banner_y: 0.5  # vertical position (0-1)
---
```

**Tips:**
- Use Unsplash for free images: `https://unsplash.com`
- Add `?w=1200` for optimized loading
- `banner_y` controls vertical crop (0 = top, 1 = bottom)

---

## 📊 Charts

Create visual charts from data.

**Bar Chart:**
```
​```chart
type: bar
labels: [Mon, Tue, Wed, Thu, Fri]
series:
  - title: Hours Studied
    data: [2, 3, 1, 4, 2]
width: 80%
labelColors: true
beginAtZero: true
​```
```

**Doughnut/Pie:**
```
​```chart
type: doughnut
labels: [Done, Remaining]
series:
  - title: Progress
    data: [30, 70]
width: 50%
​```
```

**Types available:** `bar`, `line`, `pie`, `doughnut`, `radar`, `polarArea`

---

## ✅ Tasks

Advanced task management with dates and priorities.

**Priority levels:**
- `⏫` = Highest priority
- `🔼` = High priority  
- `🔽` = Low priority

**Due dates:**
- `📅 2025-01-15` = Due date
- `⏳ 2025-01-10` = Scheduled date
- `🛫 2025-01-01` = Start date

**Recurring:**
- `🔁 every day`
- `🔁 every week`
- `🔁 every month`

**Examples:**
```markdown
- [ ] Study React ⏫ 📅 2025-01-05
- [ ] Write essay 🔼 🔁 every week
- [ ] Exercise 🔁 every day
```

**Query tasks:**
```
​```tasks
not done
due before tomorrow
sort by priority
​```
```

---

## 📖 Dictionary

Look up word definitions instantly.

**How to use:**
1. Select any word
2. Press `Ctrl+Shift+D` (or `Cmd+Shift+D` on Mac)
3. Definition appears in sidebar

**Pro tip:** Great for vocabulary learning!

---

## 📅 Calendar

Shows calendar with daily notes.

**How to use:**
1. Click calendar icon in sidebar
2. Click any date to create/open daily note
3. Dots show days with notes

---

## 🎨 Advanced Tables

Makes editing tables super easy.

**Shortcuts:**
| Action | Shortcut |
|--------|----------|
| Next cell | `Tab` |
| Previous cell | `Shift+Tab` |
| New row | `Enter` (at end) |
| Format table | `Ctrl+Shift+F` |
| Sort column | Right-click header |

---

## 🏷️ Colored Tags

Your tags will have colors automatically.

**Tag colors in your vault:**
| Tag | Meaning |
|-----|---------|
| `#word` | Vocabulary |
| `#phrasal` | Phrasal verbs |
| `#essay` | Essays |
| `#concept` | Programming concepts |
| `#draft` | Work in progress |
| `#done` | Completed |
| `#C1` | Advanced level |

---

## 📁 Iconize

Add icons to folders and files.

**How to use:**
1. Right-click any folder/file in sidebar
2. Select "Change icon"
3. Choose from emoji or icon library

**Suggested icons:**
| Folder | Icon |
|--------|------|
| Programming | 💻 |
| English | 📝 |
| Books | 📚 |
| Personal | 🧘 |
| Business | 💼 |
| Blog | 📰 |

---

## 🎨 Excalidraw

Create diagrams and drawings.

**How to use:**
1. Create new file: `Ctrl+Shift+D`
2. Or use command: "Create new Excalidraw drawing"

**Great for:**
- 🗺️ Mind maps
- 📊 Flowcharts
- 🏗️ Architecture diagrams
- 📐 Algorithm visualizations

---

## 🔍 MiniSearch

Quick file search.

**Shortcut:** `Ctrl+Shift+O` (or `Cmd+Shift+O`)

---

## 💡 Quick Tips

1. **Editing Toolbar** - Format text without remembering shortcuts
2. **File Color** - Right-click files to add colors
3. **File Explorer++** - Enhanced folder navigation
4. **Note Count** - Shows count of notes in each folder
5. **Code Styler** - Better syntax highlighting in code blocks
6. **Commander** - Add custom buttons to your toolbar

---

## 🚀 Workflow Recommendations

### Daily Routine
1. Open **Calendar** → Click today
2. Check **Tasks** in Home dashboard
3. Use **Dictionary** when reading English

### When Adding Vocabulary
1. Create note from template
2. Use **Dictionary** plugin to get definition
3. Tags auto-color with **Colored Tags**

### When Planning
1. Use **Excalidraw** for mind maps
2. **Charts** for progress visualization
3. **Tasks** for deadlines

