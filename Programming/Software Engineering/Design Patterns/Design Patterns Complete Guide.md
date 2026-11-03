---
tags:
  - software-engineering
  - design-patterns
  - dashboard
created: 2026-01-02
---
# 🎨 Design Patterns

> *"Design patterns are solutions to recurring problems; guidelines on how to tackle certain problems."*

## 📚 Pattern Categories

### 🏭 Creational Patterns
*Cómo se crean los objetos*

| Pattern | Description | Status |
|---------|-------------|--------|
| [[Programming/Software Engineering/Design Patterns/Creational/Singleton\|Singleton]] | Una sola instancia | 🔴 |
| [[Programming/Software Engineering/Design Patterns/Creational/Factory Method\|Factory Method]] | Crear sin especificar clase exacta | 🔴 |
| [[Programming/Software Engineering/Design Patterns/Creational/Abstract Factory\|Abstract Factory]] | Familias de objetos relacionados | 🔴 |
| [[Programming/Software Engineering/Design Patterns/Creational/Builder\|Builder]] | Construcción paso a paso | 🔴 |
| [[Programming/Software Engineering/Design Patterns/Creational/Prototype\|Prototype]] | Clonar objetos | 🔴 |

### 🏗️ Structural Patterns
*Cómo se componen los objetos*

| Pattern | Description | Status |
|---------|-------------|--------|
| [[Programming/Software Engineering/Design Patterns/Structural/Adapter\|Adapter]] | Interfaces incompatibles | 🔴 |
| [[Programming/Software Engineering/Design Patterns/Structural/Decorator\|Decorator]] | Añadir comportamiento | 🔴 |
| [[Programming/Software Engineering/Design Patterns/Structural/Facade\|Facade]] | Interfaz simplificada | 🔴 |
| [[Programming/Software Engineering/Design Patterns/Structural/Proxy\|Proxy]] | Placeholder para otro objeto | 🔴 |
| [[Programming/Software Engineering/Design Patterns/Structural/Composite\|Composite]] | Estructuras de árbol | 🔴 |
| [[Programming/Software Engineering/Design Patterns/Structural/Bridge\|Bridge]] | Separar abstracción de implementación | 🔴 |

### 🎭 Behavioral Patterns
*Cómo se comunican los objetos*

| Pattern | Description | Status |
|---------|-------------|--------|
| [[Programming/Software Engineering/Design Patterns/Behavioral/Strategy\|Strategy]] | Algoritmos intercambiables | 🔴 |
| [[Programming/Software Engineering/Design Patterns/Behavioral/Observer\|Observer]] | Suscripción a eventos | 🔴 |
| [[Programming/Software Engineering/Design Patterns/Behavioral/Command\|Command]] | Encapsular requests | 🔴 |
| [[Programming/Software Engineering/Design Patterns/Behavioral/State\|State]] | Cambiar comportamiento según estado | 🔴 |
| [[Programming/Software Engineering/Design Patterns/Behavioral/Template Method\|Template Method]] | Esqueleto de algoritmo | 🔴 |
| [[Programming/Software Engineering/Design Patterns/Behavioral/Iterator\|Iterator]] | Recorrer colecciones | 🔴 |
| [[Programming/Software Engineering/Design Patterns/Behavioral/Chain of Responsibility\|Chain of Responsibility]] | Cadena de handlers | 🔴 |

---

## 🎯 Quick Reference

```
CREATIONAL          STRUCTURAL          BEHAVIORAL
─────────────       ────────────        ────────────
Singleton           Adapter             Strategy
Factory Method      Decorator           Observer
Abstract Factory    Facade              Command
Builder             Proxy               State
Prototype           Composite           Template Method
                    Bridge              Iterator
                                        Chain of Resp.
```

---

## 📖 Resources

- 📕 **Book**: "Design Patterns" (Gang of Four)
- 📕 **Book**: "Head First Design Patterns"
- 🌐 **Website**: [Refactoring.Guru](https://refactoring.guru/design-patterns)

---

## 📝 Notes
```dataview
TABLE WITHOUT ID
	file.link as "Pattern",
	category as "Category"
FROM "Programming/Software Engineering/Design Patterns"
WHERE !contains(file.name, "_Index")
SORT category, file.name asc
```

---

← [[Programming/Software Engineering/Software Engineering Roadmap|Back to Software Engineering]]
