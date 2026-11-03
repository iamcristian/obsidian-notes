---
tags:
  - software-engineering
  - clean-code
  - refactoring
created: 2026-01-02
status: 🔴
---
# 🦨 Code Smells

> *"A code smell is a surface indication that usually corresponds to a deeper problem in the system."* — Martin Fowler

## 🎯 What Are Code Smells?

Code smells no son bugs - el código funciona. Son **indicadores** de que algo puede mejorarse. Son síntomas de un diseño pobre o de deuda técnica.

---

## 📦 Bloaters

### Long Method
```javascript
// ❌ Métodos de más de 20-30 líneas
function processOrder(order) {
  // 100+ líneas de código
  // validación, cálculos, guardado, emails, logs...
}

// ✅ Extraer a métodos más pequeños
function processOrder(order) {
  validateOrder(order);
  const total = calculateTotal(order);
  saveOrder(order, total);
  notifyCustomer(order);
}
```

### Large Class (God Class)
```javascript
// ❌ Clase que hace demasiado
class User {
  // Datos del usuario
  // Autenticación
  // Validación de emails
  // Generación de reportes
  // Envío de notificaciones
  // Manejo de pagos
  // 500+ líneas
}

// ✅ Single Responsibility - dividir en clases
class User { } // Solo datos
class AuthenticationService { }
class NotificationService { }
class PaymentService { }
```

### Primitive Obsession
```javascript
// ❌ Usar primitivos para conceptos de dominio
const phone = "1234567890";
const email = "test@test.com";
const money = 100.50;
const coordinates = [40.7128, -74.0060];

// ✅ Crear tipos de dominio
class PhoneNumber {
  constructor(value) {
    if (!this.isValid(value)) throw new Error("Invalid phone");
    this.value = value;
  }
}

class Money {
  constructor(amount, currency) {
    this.amount = amount;
    this.currency = currency;
  }
  add(other) { /* ... */ }
}

class Coordinates {
  constructor(lat, lng) {
    this.lat = lat;
    this.lng = lng;
  }
  distanceTo(other) { /* ... */ }
}
```

### Long Parameter List
```javascript
// ❌ Demasiados parámetros
function createUser(name, email, age, address, city, country, phone, role) { }

// ✅ Agrupar en objetos
function createUser({ name, email, age, address, role }) { }

// O mejor, usar un objeto de dominio
function createUser(userData) { }
```

### Data Clumps
```javascript
// ❌ Datos que siempre van juntos
function calculateDistance(x1, y1, x2, y2) { }
function drawLine(x1, y1, x2, y2, color) { }
function movePoint(x, y, deltaX, deltaY) { }

// ✅ Extraer a una clase
class Point {
  constructor(x, y) { this.x = x; this.y = y; }
}

function calculateDistance(point1, point2) { }
function drawLine(start, end, color) { }
function movePoint(point, delta) { }
```

---

## 🔧 Object-Orientation Abusers

### Switch Statements
```javascript
// ❌ Switch que se repite en múltiples lugares
function calculatePay(employee) {
  switch (employee.type) {
    case 'HOURLY': return calculateHourlyPay(employee);
    case 'SALARIED': return calculateSalariedPay(employee);
    case 'COMMISSIONED': return calculateCommissionedPay(employee);
  }
}

// ✅ Usar polimorfismo
class Employee {
  calculatePay() { throw new Error("Must implement"); }
}

class HourlyEmployee extends Employee {
  calculatePay() { /* ... */ }
}

class SalariedEmployee extends Employee {
  calculatePay() { /* ... */ }
}
```

### Feature Envy
```javascript
// ❌ Método usa más datos de otra clase que de la suya
class Order {
  calculateDiscount() {
    // Usa datos de Customer, no de Order
    if (this.customer.loyaltyPoints > 100 &&
        this.customer.memberSince.year < 2020 &&
        this.customer.purchases.length > 10) {
      return this.customer.preferredDiscount;
    }
  }
}

// ✅ Mover el método a donde pertenece
class Customer {
  isEligibleForDiscount() {
    return this.loyaltyPoints > 100 &&
           this.memberSince.year < 2020 &&
           this.purchases.length > 10;
  }
  getDiscount() {
    return this.isEligibleForDiscount() ? this.preferredDiscount : 0;
  }
}

class Order {
  calculateDiscount() {
    return this.customer.getDiscount();
  }
}
```

### Inappropriate Intimacy
```javascript
// ❌ Clases que saben demasiado una de otra
class Order {
  process() {
    // Accede a internals del Customer
    this.customer._balance -= this.total;
    this.customer._orders.push(this);
    this.customer._lastOrderDate = new Date();
  }
}

// ✅ Usar métodos públicos, respetar encapsulación
class Customer {
  chargeOrder(order) {
    this.balance -= order.total;
    this.orders.push(order);
    this.lastOrderDate = new Date();
  }
}

class Order {
  process() {
    this.customer.chargeOrder(this);
  }
}
```

---

## 🚫 Change Preventers

### Divergent Change
```javascript
// ❌ Una clase que cambia por múltiples razones diferentes
class Report {
  // Cambia si cambian las reglas de negocio
  calculateData() { }
  
  // Cambia si cambia el formato de salida
  formatAsHTML() { }
  formatAsPDF() { }
  formatAsExcel() { }
  
  // Cambia si cambia cómo se guarda
  saveToDatabase() { }
  saveToFile() { }
}

// ✅ Single Responsibility
class ReportCalculator { calculateData() { } }
class HTMLReportFormatter { format(data) { } }
class PDFReportFormatter { format(data) { } }
class ReportRepository { save(report) { } }
```

### Shotgun Surgery
```javascript
// ❌ Un cambio requiere modificar muchos lugares
// Cambiar el formato de fecha requiere cambiar:
// - UserController.js
// - OrderService.js
// - ReportGenerator.js
// - EmailTemplates.js
// ... y 10 archivos más

// ✅ Centralizar
class DateFormatter {
  static format(date, style = 'default') {
    // Un solo lugar para cambiar
  }
}
```

---

## 🗑️ Dispensables

### Comments (Excessive)
```javascript
// ❌ Comentarios que compensan código poco claro
// This function calculates the total price with tax
function calc(p, t) {
  return p + (p * t); // Add tax to price
}

// ✅ Código auto-documentado
function calculateTotalWithTax(price, taxRate) {
  return price + (price * taxRate);
}
```

### Duplicate Code
```javascript
// ❌ Código duplicado
class UserValidator {
  validate(user) {
    if (!user.email.includes('@')) throw new Error('Invalid email');
    if (user.name.length < 2) throw new Error('Name too short');
  }
}

class AdminValidator {
  validate(admin) {
    if (!admin.email.includes('@')) throw new Error('Invalid email');
    if (admin.name.length < 2) throw new Error('Name too short');
    // ... plus admin-specific validation
  }
}

// ✅ Extraer duplicación
class BaseValidator {
  validateCommonFields(entity) {
    if (!entity.email.includes('@')) throw new Error('Invalid email');
    if (entity.name.length < 2) throw new Error('Name too short');
  }
}
```

### Dead Code
```javascript
// ❌ Código que nunca se ejecuta
function process(type) {
  if (type === 'A') { /* ... */ }
  else if (type === 'B') { /* ... */ }
  else if (type === 'C') { /* ... */ }
  else if (type === 'X') { 
    // Nadie usa type 'X' desde 2020
  }
}

// ❌ Funciones que nadie llama
function oldCalculation() { } // Ya no se usa

// ✅ Eliminar código muerto - Git lo tiene si lo necesitas
```

### Speculative Generality
```javascript
// ❌ Abstracción innecesaria "por si acaso"
class AbstractUserFactoryBuilder {
  createAbstractFactory() {
    return new AbstractUserFactory();
  }
}
// Solo hay un tipo de User...

// ✅ YAGNI - You Aren't Gonna Need It
class User {
  constructor(data) { /* ... */ }
}
```

---

## 📋 Code Smell Quick Reference

| Category | Smell | Solution |
|----------|-------|----------|
| **Bloaters** | Long Method | Extract Method |
| | Large Class | Extract Class |
| | Primitive Obsession | Replace with Object |
| | Long Parameter List | Parameter Object |
| **OO Abusers** | Switch Statements | Polymorphism |
| | Feature Envy | Move Method |
| | Inappropriate Intimacy | Move/Hide Methods |
| **Preventers** | Divergent Change | Split Class |
| | Shotgun Surgery | Consolidate |
| **Dispensables** | Duplicate Code | Extract/Template |
| | Dead Code | Remove |
| | Comments | Refactor to clarity |

---

## 💡 Tips

> [!tip] Detecting Smells
> - **Code reviews**: Fresh eyes catch smells easily
> - **Unit tests**: Hard-to-test code usually smells
> - **"And" in names**: `UserAndOrderManager` = too many responsibilities
> - **Changes touching many files**: Possible Shotgun Surgery

---

← [[Programming/Software Engineering/Clean Code/Clean Code Complete Guide|Back to Clean Code]]
