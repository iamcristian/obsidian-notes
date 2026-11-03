---
tags:
  - software-engineering
  - solid
  - srp
created: 2026-01-02
status: 🔴
---
# 📦 Single Responsibility Principle (SRP)

> *"A class should have one, and only one, reason to change."* — Robert C. Martin

## 🎯 The Principle

Una clase debe tener **una sola responsabilidad** - una sola razón para cambiar. Esto significa que debe encapsular solo un aspecto del software.

---

## ❌ Bad Example - Multiple Responsibilities

```typescript
class User {
  id: string;
  name: string;
  email: string;

  constructor(name: string, email: string) {
    this.id = generateId();
    this.name = name;
    this.email = email;
  }

  // Responsabilidad 1: Lógica de dominio
  validate(): boolean {
    return this.email.includes('@') && this.name.length > 0;
  }

  // Responsabilidad 2: Persistencia
  save(): void {
    const sql = `INSERT INTO users VALUES (${this.id}, ${this.name}, ${this.email})`;
    database.execute(sql);
  }

  // Responsabilidad 3: Presentación
  toJSON(): object {
    return { id: this.id, name: this.name, email: this.email };
  }

  toHTML(): string {
    return `<div class="user">${this.name}</div>`;
  }

  // Responsabilidad 4: Notificaciones
  sendWelcomeEmail(): void {
    const message = `Welcome ${this.name}!`;
    emailService.send(this.email, message);
  }

  // Responsabilidad 5: Logging
  logActivity(action: string): void {
    console.log(`[${new Date()}] User ${this.id}: ${action}`);
  }
}
```

**Problemas:**
- Cambiar la DB → modificar User
- Cambiar formato de email → modificar User
- Cambiar cómo se renderiza → modificar User
- La clase crece indefinidamente
- Difícil de testear en aislamiento

---

## ✅ Good Example - Separated Responsibilities

```typescript
// Responsabilidad: Datos y reglas de negocio del dominio
class User {
  constructor(
    public readonly id: string,
    public name: string,
    public email: string
  ) {}

  validate(): boolean {
    return this.email.includes('@') && this.name.length > 0;
  }

  updateEmail(newEmail: string): void {
    if (!newEmail.includes('@')) {
      throw new Error('Invalid email');
    }
    this.email = newEmail;
  }
}

// Responsabilidad: Persistencia
class UserRepository {
  constructor(private database: Database) {}

  async save(user: User): Promise<void> {
    await this.database.execute(
      'INSERT INTO users VALUES (?, ?, ?)',
      [user.id, user.name, user.email]
    );
  }

  async findById(id: string): Promise<User | null> {
    const data = await this.database.query(
      'SELECT * FROM users WHERE id = ?',
      [id]
    );
    return data ? new User(data.id, data.name, data.email) : null;
  }
}

// Responsabilidad: Serialización/Presentación
class UserPresenter {
  toJSON(user: User): object {
    return {
      id: user.id,
      name: user.name,
      email: user.email
    };
  }

  toHTML(user: User): string {
    return `<div class="user-card">
      <h2>${user.name}</h2>
      <p>${user.email}</p>
    </div>`;
  }
}

// Responsabilidad: Notificaciones
class UserNotificationService {
  constructor(private emailService: EmailService) {}

  sendWelcomeEmail(user: User): void {
    this.emailService.send(
      user.email,
      'Welcome!',
      `Hello ${user.name}, welcome to our platform!`
    );
  }

  sendPasswordReset(user: User, resetLink: string): void {
    this.emailService.send(
      user.email,
      'Password Reset',
      `Click here to reset: ${resetLink}`
    );
  }
}

// Responsabilidad: Logging
class UserActivityLogger {
  constructor(private logger: Logger) {}

  logCreated(user: User): void {
    this.logger.info('User created', { userId: user.id });
  }

  logUpdated(user: User, changes: object): void {
    this.logger.info('User updated', { userId: user.id, changes });
  }
}
```

---

## 🔍 How to Identify Violations

### Test 1: "Reasons to Change"
Pregúntate: ¿Por qué razones tendría que modificar esta clase?
- Si hay más de una razón → viola SRP

### Test 2: "Describe Without 'And'"
Intenta describir qué hace la clase sin usar "y":
- ❌ "Esta clase maneja usuarios **y** los guarda **y** envía emails"
- ✅ "Esta clase representa los datos de un usuario"

### Test 3: Actor Rule
¿Diferentes personas/roles solicitarían cambios a esta clase?
- DBA quiere cambiar queries → UserRepository
- Designer quiere cambiar HTML → UserPresenter
- Marketing quiere cambiar emails → UserNotificationService

---

## 🏗️ Common Refactorings

### Extract Class
```typescript
// Before: Una clase con múltiples responsabilidades
class Order {
  calculateTotal() { }
  applyDiscount() { }
  validateInventory() { }
  processPayment() { }
  sendConfirmation() { }
  generateInvoice() { }
}

// After: Extraer a clases especializadas
class Order { calculateTotal() { } applyDiscount() { } }
class InventoryService { validate(order: Order) { } }
class PaymentProcessor { process(order: Order) { } }
class NotificationService { sendConfirmation(order: Order) { } }
class InvoiceGenerator { generate(order: Order) { } }
```

### Extract Method to Class
```typescript
// Before: Método grande con múltiples responsabilidades
class ReportGenerator {
  generate(data: Data) {
    // 100 lines: fetch data
    // 100 lines: transform data
    // 100 lines: format as PDF
    // 50 lines: send email with report
  }
}

// After: Cada paso es su propia clase
class DataFetcher { fetch(): RawData { } }
class DataTransformer { transform(data: RawData): ProcessedData { } }
class PDFFormatter { format(data: ProcessedData): PDF { } }
class ReportEmailer { send(pdf: PDF, recipient: string) { } }

class ReportGenerator {
  generate(recipient: string) {
    const raw = this.fetcher.fetch();
    const processed = this.transformer.transform(raw);
    const pdf = this.formatter.format(processed);
    this.emailer.send(pdf, recipient);
  }
}
```

---

## ⚠️ Common Misconceptions

> [!warning] SRP NO significa:
> - Una clase con un solo método
> - Clases extremadamente pequeñas
> - Separar cada función en su propia clase

> [!tip] SRP significa:
> - Una clase agrupa cosas que cambian por la **misma razón**
> - Cohesión alta - las partes están relacionadas
> - Un solo "actor" o stakeholder es responsable de los cambios

---

## 📊 Benefits

| Benefit | Description |
|---------|-------------|
| **Testability** | Fácil de testear en aislamiento |
| **Maintainability** | Cambios localizados |
| **Reusability** | Clases pequeñas = más reusables |
| **Understandability** | Fácil de entender qué hace cada clase |

---

## 💡 Tips

> [!tip] Rule of Thumb
> - Si la descripción de la clase usa "y", probablemente viola SRP
> - Si cambiar un requisito afecta múltiples partes de la clase, viola SRP
> - Si es difícil nombrar la clase sin ser muy genérico, viola SRP

---

← [[Programming/Software Engineering/SOLID/SOLID Principles Complete Guide|Back to SOLID]]
