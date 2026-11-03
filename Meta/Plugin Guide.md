---
tags:
  - reference
  - meta
created: 2026-11-02
---
# 🔌 Guía de Plugins y Flujo de Trabajo Moderno

Esta guía resume las herramientas activas en tu vault optimizado, diseñadas para **cero mantenimiento y máxima velocidad de captura**.

---

## ⚡ 1. Funcionalidades Nativas (Cero Plugins, Cero Fallos)

Obsidian ha evolucionado enormemente. Las siguientes funciones ahora son **nativas**:

- **Slash Commands (`/`)**: Escribe `/` en cualquier línea en blanco para insertar encabezados, tablas, callouts, listas de tareas o plantillas instantáneamente. (Ya no necesitas barras de herramientas flotantes que ocupan espacio).
- **Tablas Nativas**: Crea tablas en Markdown sin plugins externos. Puedes arrastrar filas, columnas y alinearlas con clic derecho.
- **Properties (Metadatos Visuales)**: Obsidian maneja los tags, fechas y alias arriba de cada nota de forma gráfica, rápida y limpia.
- **Bandeja de Entrada (`Inbox/`)**: Todas las notas nuevas con `Ctrl + N` van automáticamente a la carpeta `Inbox/` para que no pienses en qué carpeta guardarlas mientras escribes.

---

## 🛠️ 2. Plugins de la Comunidad Activos

### 🔍 Omnisearch
- **¿Para qué sirve?** El mejor motor de búsqueda difusa (fuzzy search) para Obsidian. Encuentra contenido dentro de tus notas, PDFs y diagramas aunque cometas errores tipográficos.
- **Atajo:** `Ctrl + Shift + F` o mediante la paleta de comandos.

### 📊 Dataview
- **¿Para qué sirve?** Consultar y listar notas dinámicamente.
- **Uso recomendado:** Mantenlo para listas simples (como la sección de actividad reciente en `Home.md`). Evita consultas complejas de JavaScript (`dv.pages()`) inline que ralentizan el vault.

### ✅ Obsidian Tasks
- **¿Para qué sirve?** Gestión de tareas en cualquier parte de tus notas.
- **Cómo usarlo:** Agrega tareas con `- [ ] Mi tarea`. Puedes añadir fechas límite con `📅 2026-11-05` o prioridades.

### 📅 Calendar & Daily Notes
- **¿Para qué sirve?** Registro cronológico rápido. Haz clic en un día del calendario lateral para abrir tu nota de ese día con tu plantilla automática.

### 🏷️ Tag Wrangler
- **¿Para qué sirve?** Gestión rápida de etiquetas. Puedes hacer clic derecho en cualquier tag del panel de etiquetas y renombrarlo en todas las notas de un solo golpe.

### 💻 Code Styler
- **¿Para qué sirve?** Resaltado de sintaxis limpio, títulos en bloques de código (````typescript:index.ts`) y números de línea para tus notas de programación.

### 🌐 Natural Language Syntax Highlighting
- **¿Para qué sirve?** Excelente para tus notas y ensayos de inglés. Resalta sustantivos, verbos, adjetivos y conectores para mejorar tu redacción.

### 🎨 Excalidraw & Mermaid
- **¿Para qué sirve?** Creación de diagramas de arquitectura, flujos y bocetos visuales directamente enlazables con tus notas.

### 🔄 Obsidian Git
- **¿Para qué sirve?** Copias de seguridad automáticas y control de versiones a tu repositorio Git remoto sin tener que abrir la terminal.

---

## 🚫 Plugins Retirados (Por qué los eliminamos)

1. **Table Editor Obsidian:** Obsoleto. El editor de tablas nativo de Obsidian es más rápido y estable.
2. **Editing Toolbar:** Obsoleto. Interrumpía la lectura y agregaba peso visual; el comando `/` (Slash Command) lo reemplaza con creces.
3. **Obsidian Banners:** Inestable con las últimas versiones de Obsidian; obligaba a buscar imágenes de Unsplash para cada nota, causando lentitud de carga y fatiga.
4. **Colored Tags / File Color / Icon Folder / Note Count:** Sobrecarga visual y fatiga de mantenimiento ("parálisis por decoración"). Dedicabas más tiempo a elegir colores e iconos que a estudiar y escribir.
