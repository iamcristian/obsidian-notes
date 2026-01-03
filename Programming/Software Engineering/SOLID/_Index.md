---
tags:
  - software-engineering
  - solid
  - dashboard
created: 2026-01-02
---
# 🎯 SOLID Principles

> *"The SOLID principles are a set of five design principles intended to make software designs more understandable, flexible, and maintainable."*

## 📚 The Five Principles

| Letter | Principle | Summary | Status |
|--------|-----------|---------|--------|
| **S** | [[Programming/Software Engineering/SOLID/Single Responsibility\|Single Responsibility]] | Una clase = Una razón para cambiar | ✅ |
| **O** | [[Programming/Software Engineering/SOLID/Open-Closed Principle\|Open/Closed]] | Abierto a extensión, cerrado a modificación | ✅ |
| **L** | [[Programming/Software Engineering/SOLID/Liskov Substitution Principle\|Liskov Substitution]] | Subtipos deben ser sustituibles | ✅ |
| **I** | [[Programming/Software Engineering/SOLID/Interface Segregation Principle\|Interface Segregation]] | Interfaces pequeñas y específicas | ✅ |
| **D** | [[Programming/Software Engineering/SOLID/Dependency Inversion\|Dependency Inversion]] | Depender de abstracciones | ✅ |

---

## 🎯 Quick Reference

### S - Single Responsibility
```typescript
// ❌ Una clase haciendo todo
class User {
  save() { }
  sendEmail() { }
  generateReport() { }
}

// ✅ Responsabilidades separadas
class User { }
class UserRepository { save() { } }
class EmailService { sendEmail() { } }
class ReportGenerator { generate() { } }
```

### O - Open/Closed
```typescript
// ❌ Modificar existente para añadir nuevo
class AreaCalculator {
  calculate(shape) {
    if (shape.type === 'circle') { }
    else if (shape.type === 'square') { }
    // Añadir nuevo = modificar aquí
  }
}

// ✅ Extender sin modificar
interface Shape { area(): number; }
class Circle implements Shape { area() { } }
class Square implements Shape { area() { } }
// Añadir nuevo = crear nueva clase
```

### L - Liskov Substitution
```typescript
// ❌ Subclase rompe comportamiento esperado
class Bird { fly() { } }
class Penguin extends Bird { 
  fly() { throw new Error("Can't fly!"); } 
}

// ✅ Jerarquía correcta
interface Bird { }
interface FlyingBird extends Bird { fly(): void; }
class Sparrow implements FlyingBird { fly() { } }
class Penguin implements Bird { swim() { } }
```

### I - Interface Segregation
```typescript
// ❌ Interfaz gorda
interface Worker {
  work(): void;
  eat(): void;
  sleep(): void;
}

// ✅ Interfaces específicas
interface Workable { work(): void; }
interface Eatable { eat(): void; }
interface Sleepable { sleep(): void; }
```

### D - Dependency Inversion
```typescript
// ❌ Dependencia de concreto
class UserService {
  private db = new MySQLDatabase(); // Acoplado
}

// ✅ Dependencia de abstracción
interface Database { query(): void; }
class UserService {
  constructor(private db: Database) { } // Inyectado
}
```

---

## 📖 Resources

- 📕 **Book**: "Clean Architecture" - Robert C. Martin
- 📕 **Book**: "Agile Principles, Patterns, and Practices" - Robert C. Martin

---

## 📝 Notes
```dataview
TABLE WITHOUT ID
	file.link as "Principle",
	status as "Status"
FROM "Programming/Software Engineering/SOLID"
WHERE !contains(file.name, "_Index")
SORT file.name asc
```

---

← [[Programming/Software Engineering/_Index|Back to Software Engineering]]
