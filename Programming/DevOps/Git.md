---
tags:
  - devops
  - git
  - programming/git
difficulty: beginner
created: 2025-12-29
---
# Git

## 📖 What is it?
> Git is a distributed version control system that tracks changes in source code during software development.

## 🔧 Essential Commands

### Setup
```bash
# Configure user
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

# Initialize repository
git init

# Clone repository
git clone <url>
```

### Daily Workflow
```bash
# Check status
git status

# Add files to staging
git add <file>          # Specific file
git add .               # All files

# Commit changes
git commit -m "message"

# Push to remote
git push origin <branch>

# Pull latest changes
git pull origin <branch>
```

### Branching
```bash
# Create new branch
git branch <name>

# Switch branch
git checkout <name>

# Create and switch
git checkout -b <name>

# List branches
git branch -a

# Delete branch
git branch -d <name>

# Merge branch
git merge <branch>
```

### Viewing History
```bash
# View commit history
git log
git log --oneline

# View changes
git diff
git diff --staged
```

### Undoing Changes
```bash
# Unstage file
git reset <file>

# Discard changes in working directory
git checkout -- <file>

# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo last commit (discard changes)
git reset --hard HEAD~1
```

### Stashing
```bash
# Save changes temporarily
git stash

# List stashes
git stash list

# Apply stash
git stash pop

# Apply specific stash
git stash apply stash@{n}
```

---

## 🔄 Common Workflows

### Feature Branch Workflow
```bash
# 1. Create feature branch
git checkout -b feature/new-feature

# 2. Make changes and commit
git add .
git commit -m "Add new feature"

# 3. Push to remote
git push origin feature/new-feature

# 4. Create Pull Request on GitHub

# 5. After merge, delete branch
git branch -d feature/new-feature
```

### Fixing Mistakes
```bash
# Amend last commit message
git commit --amend -m "New message"

# Add forgotten file to last commit
git add forgotten-file
git commit --amend --no-edit
```

---

## ⚠️ Common Mistakes
1. **Committing to wrong branch** → Use `git stash`, switch branch, `git stash pop`
2. **Forgetting to pull before push** → Always `git pull` first
3. **Large commits** → Commit small, commit often

---

## 🔗 Related
- [[GitHub]]
- [[CI/CD]]

## 📚 Resources
- [Pro Git Book](https://git-scm.com/book/en/v2)
- [GitHub Docs](https://docs.github.com)
