---
tags:
  - software-engineering
  - design-patterns
  - behavioral
created: 2026-01-02
status: 🔴
category: Behavioral
---
# 🎯 Strategy Pattern

> *"Define a family of algorithms, encapsulate each one, and make them interchangeable."*

## 🎯 Intent

- Definir una **familia de algoritmos**
- Encapsular cada uno en una clase separada
- Hacerlos **intercambiables** en runtime

---

## 📊 Structure

```
┌─────────────────┐        ┌───────────────────┐
│    Context      │───────→│   <<interface>>   │
├─────────────────┤        │     Strategy      │
│ - strategy      │        ├───────────────────┤
│ + setStrategy() │        │ + execute()       │
│ + doSomething() │        └───────────────────┘
└─────────────────┘                 △
                                   │
                    ┌──────────────┼──────────────┐
                    │              │              │
            ┌───────┴───────┐ ┌────┴────┐ ┌──────┴──────┐
            │ StrategyA     │ │StrategyB│ │ StrategyC  │
            └───────────────┘ └─────────┘ └────────────┘
```

---

## 💻 Implementation

### Problem: Multiple Algorithms
```typescript
// ❌ Bad - múltiples if/else o switch
class PaymentProcessor {
  process(amount: number, method: string) {
    if (method === 'credit') {
      // 50 líneas de lógica de tarjeta de crédito
    } else if (method === 'paypal') {
      // 50 líneas de lógica de PayPal
    } else if (method === 'crypto') {
      // 50 líneas de lógica de crypto
    }
    // Añadir un nuevo método = modificar esta clase
  }
}
```

### Solution: Strategy Pattern
```typescript
// Strategy Interface
interface PaymentStrategy {
  pay(amount: number): PaymentResult;
  validate(): boolean;
}

// Concrete Strategies
class CreditCardStrategy implements PaymentStrategy {
  constructor(
    private cardNumber: string,
    private cvv: string,
    private expiry: string
  ) {}

  validate(): boolean {
    // Validación de tarjeta
    return this.cardNumber.length === 16;
  }

  pay(amount: number): PaymentResult {
    console.log(`Paying ${amount} with Credit Card`);
    return { success: true, transactionId: `CC-${Date.now()}` };
  }
}

class PayPalStrategy implements PaymentStrategy {
  constructor(private email: string) {}

  validate(): boolean {
    return this.email.includes('@');
  }

  pay(amount: number): PaymentResult {
    console.log(`Paying ${amount} with PayPal (${this.email})`);
    return { success: true, transactionId: `PP-${Date.now()}` };
  }
}

class CryptoStrategy implements PaymentStrategy {
  constructor(private walletAddress: string) {}

  validate(): boolean {
    return this.walletAddress.startsWith('0x');
  }

  pay(amount: number): PaymentResult {
    console.log(`Paying ${amount} with Crypto`);
    return { success: true, transactionId: `CRYPTO-${Date.now()}` };
  }
}

// Context
class PaymentProcessor {
  private strategy: PaymentStrategy;

  setStrategy(strategy: PaymentStrategy) {
    this.strategy = strategy;
  }

  checkout(amount: number): PaymentResult {
    if (!this.strategy.validate()) {
      throw new Error('Invalid payment method');
    }
    return this.strategy.pay(amount);
  }
}

// Usage
const processor = new PaymentProcessor();

// Pay with credit card
processor.setStrategy(new CreditCardStrategy('1234567890123456', '123', '12/25'));
processor.checkout(100);

// Change to PayPal at runtime
processor.setStrategy(new PayPalStrategy('user@email.com'));
processor.checkout(50);
```

### Sorting Strategy Example
```typescript
interface SortStrategy<T> {
  sort(items: T[]): T[];
}

class QuickSort<T> implements SortStrategy<T> {
  sort(items: T[]): T[] {
    console.log('Using QuickSort');
    // QuickSort implementation
    return [...items].sort();
  }
}

class MergeSort<T> implements SortStrategy<T> {
  sort(items: T[]): T[] {
    console.log('Using MergeSort');
    // MergeSort implementation
    return [...items].sort();
  }
}

class BubbleSort<T> implements SortStrategy<T> {
  sort(items: T[]): T[] {
    console.log('Using BubbleSort');
    // BubbleSort implementation
    return [...items].sort();
  }
}

class Sorter<T> {
  constructor(private strategy: SortStrategy<T>) {}

  setStrategy(strategy: SortStrategy<T>) {
    this.strategy = strategy;
  }

  sort(items: T[]): T[] {
    return this.strategy.sort(items);
  }
}

// Usage - elegir estrategia según el tamaño
const numbers = [5, 2, 8, 1, 9];
const sorter = new Sorter<number>(
  numbers.length < 10 
    ? new BubbleSort() 
    : new QuickSort()
);
```

### Validation Strategy
```typescript
interface ValidationStrategy {
  validate(value: string): ValidationResult;
}

interface ValidationResult {
  isValid: boolean;
  error?: string;
}

class EmailValidation implements ValidationStrategy {
  validate(value: string): ValidationResult {
    const isValid = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value);
    return {
      isValid,
      error: isValid ? undefined : 'Invalid email format'
    };
  }
}

class PhoneValidation implements ValidationStrategy {
  validate(value: string): ValidationResult {
    const isValid = /^\+?[\d\s-]{10,}$/.test(value);
    return {
      isValid,
      error: isValid ? undefined : 'Invalid phone number'
    };
  }
}

class PasswordValidation implements ValidationStrategy {
  validate(value: string): ValidationResult {
    const checks = [
      { test: value.length >= 8, error: 'At least 8 characters' },
      { test: /[A-Z]/.test(value), error: 'At least one uppercase' },
      { test: /[0-9]/.test(value), error: 'At least one number' },
    ];
    
    const failed = checks.find(c => !c.test);
    return {
      isValid: !failed,
      error: failed?.error
    };
  }
}

// Form field with dynamic validation
class FormField {
  constructor(
    private value: string,
    private validator: ValidationStrategy
  ) {}

  validate(): ValidationResult {
    return this.validator.validate(this.value);
  }
}
```

---

## 🔄 Strategy with Functional Approach

```typescript
// En JavaScript moderno, puedes usar funciones directamente
type DiscountStrategy = (price: number) => number;

const noDiscount: DiscountStrategy = (price) => price;
const percentageDiscount = (percent: number): DiscountStrategy => 
  (price) => price * (1 - percent / 100);
const fixedDiscount = (amount: number): DiscountStrategy =>
  (price) => Math.max(0, price - amount);

class ShoppingCart {
  private items: { price: number }[] = [];
  private discountStrategy: DiscountStrategy = noDiscount;

  setDiscount(strategy: DiscountStrategy) {
    this.discountStrategy = strategy;
  }

  getTotal(): number {
    const subtotal = this.items.reduce((sum, item) => sum + item.price, 0);
    return this.discountStrategy(subtotal);
  }
}

// Usage
const cart = new ShoppingCart();
cart.setDiscount(percentageDiscount(20)); // 20% off
cart.setDiscount(fixedDiscount(50));      // $50 off
```

---

## ✅ When to Use

| Scenario | Example |
|----------|---------|
| Múltiples algoritmos para la misma tarea | Sorting, compression |
| Cambiar comportamiento en runtime | Payment methods |
| Evitar muchos condicionales | Validation rules |
| Aislar lógica de algoritmos | Tax calculations |

---

## 📊 Strategy vs State

| Aspect | Strategy | State |
|--------|----------|-------|
| **Purpose** | Algoritmos intercambiables | Comportamiento según estado |
| **Who changes** | El cliente elige | El contexto cambia automáticamente |
| **Awareness** | Strategies no se conocen | States pueden conocerse |
| **Example** | Sorting algorithms | Order status |

---

## 📋 Summary

| Aspect | Details |
|--------|---------|
| **Type** | Behavioral |
| **Intent** | Algoritmos intercambiables |
| **Key Benefit** | Open/Closed, elimina condicionales |
| **Trade-off** | Más clases |
| **Related** | State, Template Method |

---

← [[Programming/Software Engineering/Design Patterns/_Index|Back to Design Patterns]]
