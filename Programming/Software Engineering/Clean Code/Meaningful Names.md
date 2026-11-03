---
tags:
  - software-engineering
  - clean-code
  - naming
created: 2026-01-02
status: 🔴
---
# 📛 Meaningful Names

> *"The name of a variable, function, or class should answer all the big questions."*

## 🎯 Core Rules

### 1. Use Intention-Revealing Names
```javascript
// ❌ Bad
const d; // elapsed time in days
const list1;

// ✅ Good
const elapsedTimeInDays;
const userAccounts;
```

### 2. Avoid Disinformation
```javascript
// ❌ Bad - No usar 'List' si no es una lista
const accountList = {}; // Es un objeto, no una lista

// ✅ Good
const accounts = {};
const accountGroup = {};
```

### 3. Make Meaningful Distinctions
```javascript
// ❌ Bad - Nombres sin distinción clara
function copyChars(a1, a2) { }
const productInfo;
const productData;

// ✅ Good
function copyChars(source, destination) { }
const product; // Uno basta si son lo mismo
```

### 4. Use Pronounceable Names
```javascript
// ❌ Bad
const genymdhms; // generation date, year, month, day, hour, minute, second

// ✅ Good
const generationTimestamp;
```

### 5. Use Searchable Names
```javascript
// ❌ Bad
if (status === 4) { } // ¿Qué significa 4?

// ✅ Good
const WORK_DAYS_PER_WEEK = 5;
const STATUS_COMPLETED = 4;
if (status === STATUS_COMPLETED) { }
```

---

## 📝 Naming Conventions

### Classes
```javascript
// Sustantivos o frases sustantivas
// ✅ Good
class Customer { }
class WikiPage { }
class AccountParser { }

// ❌ Avoid - verbos como nombres de clase
class Manager { }  // Muy vago
class Processor { } // Muy vago
class Data { } // Muy vago
```

### Methods/Functions
```javascript
// Verbos o frases verbales
// ✅ Good
function postPayment() { }
function deletePage() { }
function calculateTax() { }
user.getName();
user.setName("John");
user.isAdmin(); // Boolean prefijo 'is'
```

### Boolean Variables
```javascript
// ✅ Usar prefijos: is, has, can, should
const isActive = true;
const hasPermission = false;
const canEdit = user.role === 'admin';
const shouldUpdate = timestamp > lastUpdate;
```

---

## 🎨 Context Rules

### Add Meaningful Context
```javascript
// ❌ Bad - Variables sin contexto
const firstName, lastName, street, city, state, zipcode;

// ✅ Good - Agrupar en contexto
class Address {
  constructor(firstName, lastName, street, city, state, zipcode) { }
}

// O usar prefijos si no puedes crear clase
const addressFirstName;
const addressStreet;
```

### Don't Add Unnecessary Context
```javascript
// ❌ Bad - Prefijos redundantes en aplicación "Gas Station Deluxe"
class GSDAccountAddress { }

// ✅ Good
class AccountAddress { }
// O mejor aún si está claro del módulo
class Address { }
```

---

## 📋 Quick Reference

| Tipo | Convención | Ejemplo |
|------|------------|---------|
| Variables | camelCase | `userName`, `totalAmount` |
| Constantes | UPPER_SNAKE | `MAX_SIZE`, `API_URL` |
| Clases | PascalCase | `UserAccount`, `HttpClient` |
| Funciones | camelCase + verbo | `getUserById`, `calculateTotal` |
| Booleanos | is/has/can/should | `isValid`, `hasAccess` |
| Interfaces | IPascalCase (opcional) | `IRepository`, `IService` |

---

## ⚠️ Common Mistakes

> [!warning] Avoid These
> - **Single letter names**: `a`, `b`, `x` (excepto en loops: `i`, `j`)
> - **Mental mapping**: Forzar a recordar qué significa una variable
> - **Humor/inside jokes**: Código es comunicación, no entretenimiento
> - **Similar names**: `XYZControllerForHandlingStrings` vs `XYZControllerForStoringStrings`

---

## 💡 Tips

> [!tip] Remember
> - Si necesitas un comentario para explicar el nombre, el nombre es malo
> - El nombre debe revelar intención, contexto y tipo (cuando sea útil)
> - Usa el vocabulario del dominio del problema (Domain Language)

---

← [[Programming/Software Engineering/Clean Code/Clean Code Complete Guide|Back to Clean Code]]
