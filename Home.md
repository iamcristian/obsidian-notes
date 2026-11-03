---
tags:
  - home
  - hub
created: 2026-11-02
---
# 🧭 Centro de Control

> [!quote]+ 🎯 Tu Propósito ("The Why")
> **Independencia** · **Libertad Financiera** · **Fluidez en Inglés (C1)** · **Software Engineer de Alto Nivel**

> [!tip]+ 📖 Manual de Referencia
> Consulta en cualquier momento la [[Guia de Uso|Guía Completa de Uso y Atajos de Obsidian]] para recordar atajos, comandos slash, formato y repetición espaciada.

---

## 🗂️ Pilares Principales

> [!column|3]
> 
> > [!info] 💻 [[Programming/Programming Hub|Programación]]
> > - [[Programming/Frontend/React Complete Guide|React Complete Guide]]
> > - [[Programming/Frontend/Next.js Complete Guide|Next.js Complete Guide]]
> > - [[Programming/Backend/NestJS Complete Guide|NestJS Complete Guide]]
> > - [[Programming/Software Engineering/System Design/System Design Complete Guide|System Design Complete Guide]]
> > - [[Programming/Software Engineering/Software Engineering Roadmap|Roadmap de Ingeniería]]
> 
> > [!success] 📝 [[English/English Hub|Inglés (B2 → C1)]]
> > - [[English/Cambridge Exam Suite/Cambridge Exam Suite Hub|🏛️ Cambridge Exam Suite (1,000 Tests)]]
> > - [[English/Vocabulary/Vocabulary Hub|📖 Vocabulario & Mazos Anki]]
> > - [[English/Speaking/B2 Speaking Exam Scenarios - Units 9 & 10|B2 Speaking Exam Scenarios]]
> > - [[English/Speaking/B2 Exam Prep - Vocabulary & Grammar|B2 Vocabulary & Grammar]]
> > - [[English/Vocabulary/C1 Vocabulary/C1 Vocabulary Guide & Index|C1 Vocabulary Overview]]
> > - [[English/Writing/Writing Hub & Resources|Writing & Essays]]
> > - [[English/Grammar/C1 Grammar Complete Guide|C1 Grammar Guide]]
> 
> > [!warning] 🧘 [[Personal/Personal Goals & Vision|Personal & Objetivos]]
> > - [[Personal/Weekly Routine|Rutina Semanal]]
> > - [[Personal/Personal Goals & Vision|Metas y Visión Personal]]
> > - [[Books/Reading Tracker.base|📚 Tracker de Libros (Base)]]
> > - [[English/Writing/Essays Tracker.base|✍️ Tracker de Ensayos (Base)]]
> > - [[Blog/Blog Ideas & Writing Pipeline|Ideas para Blog]]
> > - [[Inbox/README - Quick Capture|📥 Bandeja de Entrada]]

---

## ⚡ Atajos & Captura Rápida

| Acción | Atajo / Cómo usarlo |
| :--- | :--- |
| **Buscar cualquier nota** | `Ctrl + O` (o `Ctrl + Shift + F` con Omnisearch) |
| **Nueva nota rápida** | `Ctrl + N` (cae directo en tu carpeta `Inbox/`) |
| **Comandos & Formato** | Escribe `/` en cualquier línea (Slash commands nativo) |
| **Nota Diaria** | Clic en el icono del calendario o comando *Daily notes* |

---

## ✅ Tareas Pendientes

```tasks
not done
due before tomorrow
sort by priority
limit 8
```

---

## 📝 Actividad Reciente

```dataview
TABLE WITHOUT ID
	file.link as "Nota",
	replace(replace(file.folder, "Programming/", "💻 "), "English/", "📝 ") as "Área",
	dateformat(file.mtime, "yyyy-MM-dd HH:mm") as "Modificado"
FROM ""
WHERE !contains(file.path, "Templates") 
	AND !contains(file.path, "Assets") 
	AND !contains(file.path, "Meta")
	AND file.name != "Home"
SORT file.mtime desc
LIMIT 8
```
