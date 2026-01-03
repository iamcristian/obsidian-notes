---
tags:
  - software-engineering
  - design-patterns
  - creational
created: 2026-01-02
status: 🔴
category: Creational
---
# 🏭 Factory Method Pattern

> *"Define an interface for creating an object, but let subclasses decide which class to instantiate."*

## 🎯 Intent

- Definir una interfaz para crear objetos
- Dejar que las **subclases** decidan qué clase instanciar
- Delegar la creación a subclases

---

## 📊 Structure

```
┌─────────────────────┐          ┌─────────────────────┐
│  <<interface>>      │          │  <<interface>>      │
│  Creator            │          │  Product            │
├─────────────────────┤          ├─────────────────────┤
│ + factoryMethod()   │─────────→│ + operation()       │
│ + someOperation()   │          └─────────────────────┘
└─────────────────────┘                    △
          △                                │
          │                    ┌───────────┴───────────┐
          │                    │                       │
┌─────────┴─────────┐  ┌───────┴───────┐  ┌───────────┴───────┐
│ ConcreteCreatorA  │  │ ConcreteProductA│  │ ConcreteProductB │
├───────────────────┤  └───────────────┘  └───────────────────┘
│ + factoryMethod() │
└───────────────────┘
```

---

## 💻 Implementation

### Example: Notification System
```typescript
// Product Interface
interface Notification {
  send(message: string): void;
}

// Concrete Products
class EmailNotification implements Notification {
  send(message: string): void {
    console.log(`📧 Email sent: ${message}`);
  }
}

class SMSNotification implements Notification {
  send(message: string): void {
    console.log(`📱 SMS sent: ${message}`);
  }
}

class PushNotification implements Notification {
  send(message: string): void {
    console.log(`🔔 Push notification: ${message}`);
  }
}

// Creator (Factory)
abstract class NotificationFactory {
  // Factory Method - subclases lo implementan
  abstract createNotification(): Notification;

  // Lógica de negocio que usa el factory method
  notify(message: string): void {
    const notification = this.createNotification();
    notification.send(message);
  }
}

// Concrete Creators
class EmailNotificationFactory extends NotificationFactory {
  createNotification(): Notification {
    return new EmailNotification();
  }
}

class SMSNotificationFactory extends NotificationFactory {
  createNotification(): Notification {
    return new SMSNotification();
  }
}

class PushNotificationFactory extends NotificationFactory {
  createNotification(): Notification {
    return new PushNotification();
  }
}

// Usage
function sendAlert(factory: NotificationFactory) {
  factory.notify("Server is down!");
}

sendAlert(new EmailNotificationFactory()); // 📧 Email sent: Server is down!
sendAlert(new SMSNotificationFactory());   // 📱 SMS sent: Server is down!
```

### Simple Factory (Variation)
```typescript
// No es el patrón completo, pero muy común
class NotificationFactory {
  static create(type: 'email' | 'sms' | 'push'): Notification {
    switch (type) {
      case 'email':
        return new EmailNotification();
      case 'sms':
        return new SMSNotification();
      case 'push':
        return new PushNotification();
      default:
        throw new Error(`Unknown notification type: ${type}`);
    }
  }
}

// Usage
const notification = NotificationFactory.create('email');
notification.send('Hello!');
```

### With Configuration
```typescript
interface NotificationConfig {
  type: 'email' | 'sms' | 'push';
  recipient: string;
  priority?: 'low' | 'high';
}

class ConfigurableNotificationFactory {
  static create(config: NotificationConfig): Notification {
    switch (config.type) {
      case 'email':
        return new EmailNotification(config.recipient);
      case 'sms':
        return new SMSNotification(config.recipient, config.priority);
      case 'push':
        return new PushNotification(config.recipient);
      default:
        throw new Error(`Unknown type`);
    }
  }
}
```

---

## 🔄 Real World Examples

### Document Creation
```typescript
interface Document {
  open(): void;
  save(): void;
  close(): void;
}

class PDFDocument implements Document {
  open() { console.log('Opening PDF...'); }
  save() { console.log('Saving PDF...'); }
  close() { console.log('Closing PDF...'); }
}

class WordDocument implements Document {
  open() { console.log('Opening Word doc...'); }
  save() { console.log('Saving Word doc...'); }
  close() { console.log('Closing Word doc...'); }
}

abstract class Application {
  abstract createDocument(): Document;
  
  newDocument(): Document {
    const doc = this.createDocument();
    doc.open();
    return doc;
  }
}

class PDFApplication extends Application {
  createDocument(): Document {
    return new PDFDocument();
  }
}
```

### Database Connections
```typescript
interface DatabaseConnection {
  connect(): void;
  query(sql: string): any;
  disconnect(): void;
}

class PostgresConnection implements DatabaseConnection {
  connect() { /* ... */ }
  query(sql: string) { /* ... */ }
  disconnect() { /* ... */ }
}

class MongoConnection implements DatabaseConnection {
  connect() { /* ... */ }
  query(sql: string) { /* ... */ }
  disconnect() { /* ... */ }
}

class DatabaseFactory {
  static create(type: 'postgres' | 'mongo', config: any): DatabaseConnection {
    switch (type) {
      case 'postgres':
        return new PostgresConnection(config);
      case 'mongo':
        return new MongoConnection(config);
      default:
        throw new Error(`Unsupported database: ${type}`);
    }
  }
}
```

---

## ✅ When to Use

| Scenario | Reason |
|----------|--------|
| No sabes de antemano qué clase necesitarás | Delegar a runtime |
| Quieres que usuarios extiendan tu librería | Subclases añaden productos |
| Quieres reusar objetos existentes | Factory puede cachear |
| Desacoplar código cliente del código de creación | Single Responsibility |

---

## 📊 Comparison: Factory Method vs Simple Factory

| Aspect | Factory Method | Simple Factory |
|--------|----------------|----------------|
| **Structure** | Usa herencia | Usa una clase estática |
| **Extensibility** | Muy extensible | Requiere modificar switch |
| **Complexity** | Más clases | Más simple |
| **Open/Closed** | ✅ Cumple | ❌ Viola |
| **When to use** | Bibliotecas, frameworks | Aplicaciones simples |

---

## ⚠️ Common Mistakes

```typescript
// ❌ Bad: Factory que hace demasiado
class NotificationFactory {
  create(type: string): Notification {
    const notification = /* ... */;
    notification.configure();  // ❌ No debería configurar
    notification.validate();   // ❌ No debería validar
    notification.connect();    // ❌ No debería conectar
    return notification;
  }
}

// ✅ Good: Factory solo crea
class NotificationFactory {
  create(type: string): Notification {
    switch (type) {
      case 'email': return new EmailNotification();
      case 'sms': return new SMSNotification();
    }
  }
}

// La configuración va aparte
const notification = factory.create('email');
notification.configure(config);
```

---

## 📋 Summary

| Aspect | Details |
|--------|---------|
| **Type** | Creational |
| **Intent** | Delegar creación a subclases |
| **Key Benefit** | Extensibilidad sin modificar código existente |
| **Trade-off** | Puede resultar en muchas clases |
| **Related** | Abstract Factory, Template Method |

---

← [[Programming/Software Engineering/Design Patterns/_Index|Back to Design Patterns]]
