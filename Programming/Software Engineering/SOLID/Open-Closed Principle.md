---
tags:
  - software-engineering
  - solid
  - ocp
created: 2026-01-02
status: 🔴
---
# 🔓 Open/Closed Principle (OCP)

> *"Software entities should be open for extension, but closed for modification."* — Bertrand Meyer

## 🎯 The Principle

El código debe estar:
- **OPEN** para extensión (añadir nuevo comportamiento)
- **CLOSED** para modificación (no cambiar código existente)

---

## ❌ Violating OCP

```typescript
// ❌ BAD: Must modify this class every time we add a new shape
class AreaCalculator {
  calculateArea(shape: any): number {
    if (shape.type === 'circle') {
      return Math.PI * shape.radius ** 2;
    } else if (shape.type === 'rectangle') {
      return shape.width * shape.height;
    } else if (shape.type === 'triangle') {
      // Added later - HAD TO MODIFY existing class!
      return (shape.base * shape.height) / 2;
    }
    // What about pentagon? hexagon? We keep modifying!
    throw new Error('Unknown shape');
  }
}
```

**Problems:**
- Each new shape requires modifying `AreaCalculator`
- Risk of breaking existing functionality
- Growing if/else chain
- Violates Single Responsibility too

---

## ✅ Following OCP

```typescript
// ✅ GOOD: Use abstraction and polymorphism

// Define the contract
interface Shape {
  calculateArea(): number;
}

// Each shape implements its own logic
class Circle implements Shape {
  constructor(private radius: number) {}
  
  calculateArea(): number {
    return Math.PI * this.radius ** 2;
  }
}

class Rectangle implements Shape {
  constructor(private width: number, private height: number) {}
  
  calculateArea(): number {
    return this.width * this.height;
  }
}

class Triangle implements Shape {
  constructor(private base: number, private height: number) {}
  
  calculateArea(): number {
    return (this.base * this.height) / 2;
  }
}

// Calculator is CLOSED for modification
// but OPEN for extension (new shapes)
class AreaCalculator {
  calculateArea(shape: Shape): number {
    return shape.calculateArea();
  }
  
  calculateTotalArea(shapes: Shape[]): number {
    return shapes.reduce((sum, shape) => sum + shape.calculateArea(), 0);
  }
}

// Adding new shape - NO modification to AreaCalculator!
class Pentagon implements Shape {
  constructor(private side: number) {}
  
  calculateArea(): number {
    return (Math.sqrt(5 * (5 + 2 * Math.sqrt(5))) / 4) * this.side ** 2;
  }
}
```

---

## 🔄 Real-World Example: Payment Processing

### ❌ Before OCP
```typescript
class PaymentProcessor {
  processPayment(amount: number, method: string, details: any) {
    if (method === 'credit_card') {
      // Process credit card
      return this.chargeCreditCard(details.cardNumber, amount);
    } else if (method === 'paypal') {
      // Process PayPal
      return this.chargePayPal(details.email, amount);
    } else if (method === 'crypto') {
      // Added later - MODIFIED existing class
      return this.chargeCrypto(details.walletAddress, amount);
    }
    // Apple Pay? Google Pay? Keep modifying...
  }
}
```

### ✅ After OCP
```typescript
// Payment strategy interface
interface PaymentMethod {
  pay(amount: number): Promise<PaymentResult>;
  validate(): boolean;
}

// Implementations - each in its own class
class CreditCardPayment implements PaymentMethod {
  constructor(private cardNumber: string, private cvv: string) {}
  
  validate(): boolean {
    return this.cardNumber.length === 16 && this.cvv.length === 3;
  }
  
  async pay(amount: number): Promise<PaymentResult> {
    // Credit card processing logic
    return { success: true, transactionId: 'cc_123' };
  }
}

class PayPalPayment implements PaymentMethod {
  constructor(private email: string) {}
  
  validate(): boolean {
    return this.email.includes('@');
  }
  
  async pay(amount: number): Promise<PaymentResult> {
    // PayPal processing logic
    return { success: true, transactionId: 'pp_456' };
  }
}

// Processor is CLOSED - never needs modification
class PaymentProcessor {
  async processPayment(amount: number, method: PaymentMethod): Promise<PaymentResult> {
    if (!method.validate()) {
      throw new Error('Invalid payment method');
    }
    return method.pay(amount);
  }
}

// Adding Apple Pay - NO modification to PaymentProcessor!
class ApplePayPayment implements PaymentMethod {
  constructor(private token: string) {}
  
  validate(): boolean {
    return this.token.length > 0;
  }
  
  async pay(amount: number): Promise<PaymentResult> {
    return { success: true, transactionId: 'ap_789' };
  }
}
```

---

## 🎯 Techniques for OCP

### 1. Strategy Pattern
```typescript
interface SortStrategy {
  sort<T>(items: T[]): T[];
}

class QuickSort implements SortStrategy {
  sort<T>(items: T[]): T[] { /* ... */ }
}

class MergeSort implements SortStrategy {
  sort<T>(items: T[]): T[] { /* ... */ }
}

class Sorter {
  constructor(private strategy: SortStrategy) {}
  
  sort<T>(items: T[]): T[] {
    return this.strategy.sort(items);
  }
}
```

### 2. Template Method
```typescript
abstract class DataExporter {
  // Template method - closed for modification
  export(data: any[]): void {
    const formatted = this.format(data);
    const validated = this.validate(formatted);
    this.write(validated);
  }
  
  // Open for extension
  protected abstract format(data: any[]): string;
  protected abstract validate(content: string): string;
  protected abstract write(content: string): void;
}

class CSVExporter extends DataExporter {
  protected format(data: any[]): string { /* CSV formatting */ }
  protected validate(content: string): string { /* CSV validation */ }
  protected write(content: string): void { /* Write to file */ }
}
```

### 3. Decorator Pattern
```typescript
interface Coffee {
  cost(): number;
  description(): string;
}

class SimpleCoffee implements Coffee {
  cost() { return 2; }
  description() { return 'Coffee'; }
}

// Decorators extend without modifying
class MilkDecorator implements Coffee {
  constructor(private coffee: Coffee) {}
  
  cost() { return this.coffee.cost() + 0.5; }
  description() { return this.coffee.description() + ', Milk'; }
}

class SugarDecorator implements Coffee {
  constructor(private coffee: Coffee) {}
  
  cost() { return this.coffee.cost() + 0.2; }
  description() { return this.coffee.description() + ', Sugar'; }
}

// Usage - extend behavior without modification
let coffee: Coffee = new SimpleCoffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);
// coffee.cost() = 2.7, description = "Coffee, Milk, Sugar"
```

---

## 📋 OCP Checklist

> [!check] Signs You're Following OCP
> - [ ] Adding features doesn't require changing existing classes
> - [ ] Using interfaces/abstract classes for extension points
> - [ ] New behavior = new class implementing existing interface
> - [ ] Core logic remains stable over time

> [!warning] Signs You're Violating OCP
> - Growing if/else or switch statements
> - Frequently modifying the same class for new features
> - Type checking with `instanceof` or `typeof`
> - Fear of breaking existing code when adding features

---

← [[Programming/Software Engineering/SOLID/_Index|Back to SOLID]]
