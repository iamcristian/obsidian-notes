---
tags:
  - software-engineering
  - refactoring
  - dashboard
created: 2026-01-02
---
# 🔧 Refactoring

> *"Refactoring is the process of changing a software system in such a way that it does not alter the external behavior of the code yet improves its internal structure."* — Martin Fowler

## 📚 Contents

| Topic | Description | Status |
|-------|-------------|--------|
| [[Programming/Software Engineering/Refactoring/Extract Method\|Extract Method]] | Extraer código a nuevo método | 🔴 |
| [[Programming/Software Engineering/Refactoring/Extract Class\|Extract Class]] | Separar responsabilidades | 🔴 |
| [[Programming/Software Engineering/Refactoring/Rename\|Rename]] | Mejorar nombres | 🔴 |
| [[Programming/Software Engineering/Refactoring/Move Method\|Move Method]] | Mover a donde pertenece | 🔴 |

---

## 🎯 Common Refactorings

### Extract Method
```typescript
// Before
function printOwing(invoice: Invoice) {
  let outstanding = 0;
  
  // Print banner
  console.log("***********************");
  console.log("**** Customer Owes ****");
  console.log("***********************");
  
  // Calculate outstanding
  for (const order of invoice.orders) {
    outstanding += order.amount;
  }
  
  // Print details
  console.log(`name: ${invoice.customer}`);
  console.log(`amount: ${outstanding}`);
}

// After
function printOwing(invoice: Invoice) {
  printBanner();
  const outstanding = calculateOutstanding(invoice);
  printDetails(invoice, outstanding);
}

function printBanner() {
  console.log("***********************");
  console.log("**** Customer Owes ****");
  console.log("***********************");
}

function calculateOutstanding(invoice: Invoice): number {
  return invoice.orders.reduce((sum, order) => sum + order.amount, 0);
}

function printDetails(invoice: Invoice, outstanding: number) {
  console.log(`name: ${invoice.customer}`);
  console.log(`amount: ${outstanding}`);
}
```

### Replace Conditional with Polymorphism
```typescript
// Before
function getPayAmount(employee: Employee): number {
  switch (employee.type) {
    case 'engineer':
      return employee.monthlySalary;
    case 'salesman':
      return employee.monthlySalary + employee.commission;
    case 'manager':
      return employee.monthlySalary + employee.bonus;
    default:
      throw new Error('Unknown employee type');
  }
}

// After
abstract class Employee {
  abstract getPayAmount(): number;
}

class Engineer extends Employee {
  getPayAmount(): number {
    return this.monthlySalary;
  }
}

class Salesman extends Employee {
  getPayAmount(): number {
    return this.monthlySalary + this.commission;
  }
}

class Manager extends Employee {
  getPayAmount(): number {
    return this.monthlySalary + this.bonus;
  }
}
```

### Replace Magic Number with Constant
```typescript
// Before
if (user.age >= 18) { }
const discount = price * 0.1;

// After
const LEGAL_AGE = 18;
const STANDARD_DISCOUNT_RATE = 0.1;

if (user.age >= LEGAL_AGE) { }
const discount = price * STANDARD_DISCOUNT_RATE;
```

---

## 🔄 Refactoring Workflow

```
1. Asegurar tests existen ✓
2. Hacer cambio pequeño
3. Ejecutar tests
4. Commit si pasa
5. Repetir
```

---

## 📖 Resources

- 📕 **Book**: "Refactoring" - Martin Fowler
- 🌐 **Website**: [Refactoring.Guru](https://refactoring.guru/refactoring)

---

← [[Programming/Software Engineering/_Index|Back to Software Engineering]]
