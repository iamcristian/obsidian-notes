---
tags:
  - software-engineering
  - clean-code
  - comments
created: 2026-01-02
status: 🔴
---
# 💬 Comments

> *"Don't comment bad code — rewrite it."* — Brian W. Kernighan

## 🎯 The Truth About Comments

> [!warning] Core Principle
> Los comentarios son un **fracaso** en expresar tu intención en código.
> El código debería ser tan claro que no necesite comentarios.

---

## ✅ Good Comments

### 1. Legal Comments
```javascript
// Copyright (c) 2026 Company Inc. All rights reserved.
// Licensed under Apache 2.0 License
```

### 2. Informative Comments
```javascript
// Format: kk:mm:ss EEE, MMM dd, yyyy
const pattern = /\d{2}:\d{2}:\d{2} \w{3}, \w{3} \d{2}, \d{4}/;

// Returns an instance of the responder being tested
function responderInstance() { }
```

### 3. Explanation of Intent
```javascript
// We sort by priority because users expect most important items first
items.sort((a, b) => b.priority - a.priority);

// Using insertion sort because our data is almost always nearly sorted
function insertionSort(items) { }
```

### 4. Warning of Consequences
```javascript
// Don't run unless you have 30+ minutes to spare
function testWithRealNetworkCalls() { }

// This is not thread-safe, use only in single-threaded context
class SimpleCache { }
```

### 5. TODO Comments
```javascript
// TODO: We need to optimize this once we have benchmarks
// TODO(john): Remove after v2.0 release
// FIXME: This breaks when user has no address

// Use consistent format: TODO, FIXME, HACK, XXX
```

### 6. Amplification
```javascript
// The trim is important. Users often paste with leading spaces
// which causes the validator to fail silently.
const trimmedInput = userInput.trim();
```

### 7. JSDoc/TSDoc for Public APIs
```typescript
/**
 * Calculates the total price including tax.
 * @param items - Array of items in cart
 * @param taxRate - Tax rate as decimal (e.g., 0.21 for 21%)
 * @returns Total price with tax applied
 * @throws {InvalidTaxRateError} If tax rate is negative
 * @example
 * calculateTotal([{price: 100}], 0.21) // Returns 121
 */
function calculateTotal(items: Item[], taxRate: number): number { }
```

---

## ❌ Bad Comments

### 1. Mumbling / Redundant Comments
```javascript
// ❌ Completamente inútil
// The constructor
constructor() { }

// ❌ Repite lo que el código ya dice
// Increment i by 1
i++;

// ❌ No dice nada útil
// This is important
doSomething();
```

### 2. Mandated Comments
```javascript
// ❌ Comentarios obligatorios sin valor
/**
 * @param title The title
 * @param author The author  
 * @returns Returns the result
 */
function addBook(title, author) { }
```

### 3. Journal/Changelog Comments
```javascript
// ❌ Esto es para Git, no para el código
// Modified by John on 2026-01-01 - Added validation
// Modified by Jane on 2026-01-02 - Fixed bug in validation
// Modified by John on 2026-01-03 - Removed validation
function process() { }
```

### 4. Noise Comments
```javascript
// ❌ Solo añade ruido visual
// Default constructor
constructor() { }

// ❌ Closing brace comments
} // end for
} // end while  
} // end if
```

### 5. Commented-Out Code
```javascript
// ❌ NUNCA dejes código comentado
function calculate() {
  // const oldResult = oldCalculation();
  // return oldResult * 1.5;
  // const temp = something();
  return newCalculation();
}
// Si lo necesitas después, está en Git
```

### 6. Position Markers
```javascript
// ❌ Separadores innecesarios
// /////////////////// ACTIONS ///////////////////
function doThis() { }
function doThat() { }

// /////////////////// HELPERS ///////////////////
function helper1() { }
```

### 7. Attributions
```javascript
// ❌ Git blame existe para esto
// Added by Richard - October 2025
function newFeature() { }
```

---

## 🔄 Comment vs Better Code

### Example 1: Check if Employee is Eligible
```javascript
// ❌ Comment explaining complex condition
// Check to see if employee is eligible for full benefits
if ((employee.flags & HOURLY_FLAG) && 
    (employee.age > 65)) { }

// ✅ Extract to well-named function
if (employee.isEligibleForFullBenefits()) { }
```

### Example 2: Complex Logic
```javascript
// ❌ Explaining what code does
// If we have a valid response and the user is active,
// and we haven't exceeded rate limits, process the request
if (response !== null && user.status === 'active' && 
    rateLimiter.check(user.id)) {
  processRequest(response);
}

// ✅ Self-documenting code
const hasValidResponse = response !== null;
const isUserActive = user.status === 'active';
const withinRateLimit = rateLimiter.check(user.id);

if (hasValidResponse && isUserActive && withinRateLimit) {
  processRequest(response);
}
```

### Example 3: Magic Numbers
```javascript
// ❌ Comment explaining magic number
if (age > 18) { } // Legal adult age

// ✅ Named constant
const LEGAL_ADULT_AGE = 18;
if (age > LEGAL_ADULT_AGE) { }
```

---

## 📋 Quick Reference

| Type | Keep? | Why |
|------|-------|-----|
| Legal | ✅ | Required |
| Intent explanation | ✅ | Adds context |
| Warning | ✅ | Prevents mistakes |
| TODO/FIXME | ✅ | Track work |
| JSDoc (public API) | ✅ | Documentation |
| Redundant | ❌ | Noise |
| Commented code | ❌ | Use Git |
| Journal/Changelog | ❌ | Use Git |
| Closing brace | ❌ | Noise |
| Attribution | ❌ | Use Git blame |

---

## 💡 Rule of Thumb

> [!tip] Before Writing a Comment
> 1. ¿Puedo renombrar la variable/función para que sea más claro?
> 2. ¿Puedo extraer a una función con nombre descriptivo?
> 3. ¿Puedo usar una constante con nombre significativo?
> 4. Si no → entonces escribe el comentario

---

← [[Programming/Software Engineering/Clean Code/Clean Code Complete Guide|Back to Clean Code]]
