---
tags:
  - meta
  - guia
  - obsidian
created: 2026-11-02
---
# 🧭 Guía Definitiva de Obsidian (Atajos, Flujo y Comandos)

> Esta guía reúne todo lo que necesitas saber para usar tu Obsidian optimizado sin esfuerzo. Mantenla a mano o ábrela en un panel lateral cuando tengas dudas.

---

## ⚡ 1. Atajos de Teclado Imprescindibles

### 🔍 Navegación y Búsqueda
| Atajo | Acción | ¿Para qué sirve? |
| :--- | :--- | :--- |
| **`Ctrl + O`** | **Quick Switcher** | Busca cualquier nota por su título en menos de un segundo. |
| **`Ctrl + Shift + F`** | **Omnisearch** | Búsqueda profunda difusa (encuentra texto en notas, código `camelCase`, errores tipográficos). |
| **`Ctrl + P`** | **Command Palette** | Abre el menú de todos los comandos de Obsidian. |
| **`Alt + D`** | **Daily Note** | Abre o crea la nota del día de hoy en la carpeta `Daily/`. |
| **`Ctrl + G`** | **Graph View** | Abre el grafo de conocimiento (filtrado sin ruido de adjuntos o plantillas). |

### ✍️ Edición y Redacción
| Atajo | Acción | ¿Para qué sirve? |
| :--- | :--- | :--- |
| **`Ctrl + N`** | **Nueva Nota** | Crea una nota limpia que cae directo a tu `Inbox/`. |
| **`/`** | **Slash Commands** | Escribe `/` en cualquier línea vacía para insertar tablas, callouts, plantillas o títulos. |
| **`Ctrl + S`** | **Guardar + Linter** | Guarda y **auto-formatea** la nota (corrige espacios, alinea tablas y listas). |
| **`Ctrl + E`** | **Modo Lectura / Edición** | Alterna entre ver la nota formateada o editar el texto Markdown. |
| **`Ctrl + K`** | **Insertar Enlace** | Convierte el texto seleccionado en un enlace `[[...]]`. |

### 🧠 Estudio y Repaso Activo (Spaced Repetition)
| Atajo | Acción | ¿Para qué sirve? |
| :--- | :--- | :--- |
| **`Alt + R`** | **Repasar Flashcards** | Abre la sesión de repaso tipo Anki con las tarjetas programadas para hoy. |
| **`Alt + Shift + R`** | **Cramming (Repaso Total)** | Repasa todas las tarjetas del vault de golpe, sin alterar el algoritmo. |
| **`Espacio`** | **Mostrar Respuesta** | Dentro del visor de tarjetas, voltea la tarjeta. |
| **`1`, `2`, `3`** | **Calificar Tarjeta** | `1` = Hard (difícil), `2` = Good (bien), `3` = Easy (fácil). |

---

## 🌊 2. El Flujo de Trabajo Recomendado (Cero Fricción)

Para que el vault **nunca vuelva a ser tedioso**, sigue estas 3 reglas de oro:

1. **Captura primero, organiza después:**
   - Presiona `Ctrl + N` y escribe lo que tengas en mente. 
   - No pierdas tiempo decidiendo carpetas, colores ni tags al momento de escribir. La nota irá a `Inbox/`.
2. **Usa enlaces `[[...]]` en lugar de carpetas profundas:**
   - Si estás escribiendo sobre React y quieres mencionar NestJS, solo escribe `[[NestJS Complete Guide]]`.
   - Puedes usar alias: `[[React Complete Guide|mi guía de React]]`.
3. **Busca con `Ctrl + O` o `Ctrl + Shift + F`:**
   - No navegues manualmente por el árbol de carpetas con el ratón. Es 10 veces más rápido presionar `Ctrl + O` y teclear las primeras 3 letras de la nota.

---

## 🗂️ 3. Menú Slash (`/`) y Bloques Nativos

Al escribir una barra inclinada **`/`** al inicio de una línea, aparece un menú contextual para insertar:

### Callouts (Cajas destacadas)
Se escriben con la sintaxis `> [!tipo]`:
```markdown
> [!note] Nota informativa
> [!tip] Consejo útil o truco
> [!warning] Advertencia o punto crítico
> [!quote] Cita o recordatorio motivacional
> [!example] Ejemplo práctico de código o gramática
```

### Tablas Nativas
- Ya no necesitas plugins externos ni escribir barras `| | |` a mano.
- Escribe `/tabla` y Obsidian insertará una tabla visual con controles para:
  - Añadir columnas o filas con un clic.
  - Arrastrar encabezados para cambiar el orden.
  - Alinear texto a la izquierda, centro o derecha.

### Bloques de Código con Resaltado
Gracias al plugin `Code Styler`, puedes añadir títulos y rutas a tus bloques:
````markdown
```typescript:src/auth/auth.service.ts
export class AuthService {
  async validateUser() { ... }
}
```
````

---

## 🧠 4. Cómo Crear Tarjetas de Repetición Espaciada

Puedes convertir cualquier nota en una tarjeta de estudio usando estos formatos:

### A. Tarjeta de una línea (Vocabulario / Pregunta directa)
```markdown
Eloquent:: Fluent or persuasive in speaking or writing (Elocuente).
DIP (Clean Code):: High-level modules should not depend on low-level modules.
```

### B. Tarjeta bidireccional (De ida y vuelta)
```markdown
Substantiate::: Probar o fundamentar con evidencia
```
*(Obsidian te preguntará tanto por la palabra en inglés como por su traducción en español en días distintos).*

### C. Tarjeta multilínea (Preguntas complejas o código)
```markdown
¿Cómo se resuelve Two Sum en tiempo O(n)?
?
Usando un HashMap donde guardamos cada número recorrido y buscamos `target - num`.
```

---

## 📊 5. Bases Nativas de Obsidian (`.base`)

Obsidian ahora incluye **Bases**, una alternativa nativa a Dataview que te permite ver tus notas como bases de datos interactivas tipo Notion:

- Puedes abrir [Reading Tracker.base](file:///c:/Users/crisa/Documentos/Zettelkasten/Books/Reading%20Tracker.base) para ver tus libros en tabla interactiva.
- Puedes abrir [Essays Tracker.base](file:///c:/Users/crisa/Documentos/Zettelkasten/English/Writing/Essays%20Tracker.base) para gestionar el estado de tus ensayos.
- Puedes cambiar valores directamente en las celdas de la tabla sin tener que abrir la nota.

---

## 🔄 6. Copia de Seguridad Automática (Git)

- Tu vault tiene instalado y activo **Obsidian Git**.
- Si quieres hacer un respaldo manual forzado a tu repositorio remoto en cualquier momento:
  1. Presiona `Ctrl + P`.
  2. Escribe `Git: Backup and sync`.
  3. Pulsa `Enter`. Obsidian hará `add`, `commit` y `push` automáticamente.

---

## 📌 Resumen de Rutina Rápida

```text
🌅 Mañana:
   1. Alt + D   --> Revisa tu nota diaria y tus 2-3 prioridades.
   2. Alt + R   --> Haz tu sesión de 3 a 5 minutos de flashcards C1 y conceptos.

 Durante el día:
   1. Ctrl + N  --> Captura rápida al Inbox de snippets, ideas o dudas.
   2. Ctrl + S  --> Guarda y auto-formatea con Linter.

🔍 En cualquier momento:
   1. Ctrl + O  --> Salta a cualquier guía técnica o de inglés.
```
