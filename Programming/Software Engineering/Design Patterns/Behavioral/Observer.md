---
tags:
  - software-engineering
  - design-patterns
  - behavioral
created: 2026-01-02
status: 🔴
category: Behavioral
---
# 👀 Observer Pattern

> *"Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified."*

## 🎯 Intent

- Definir dependencia **uno-a-muchos**
- Cuando el sujeto cambia, **todos los observadores** son notificados
- Promover **loose coupling** entre objetos

---

## 📊 Structure

```
┌───────────────────┐           ┌───────────────────┐
│     Subject       │           │  <<interface>>    │
├───────────────────┤           │     Observer      │
│ - observers[]     │◆─────────→├───────────────────┤
│ + attach()        │           │ + update()        │
│ + detach()        │           └───────────────────┘
│ + notify()        │                    △
└───────────────────┘                    │
                             ┌───────────┴───────────┐
                             │                       │
                    ┌────────┴────────┐    ┌────────┴────────┐
                    │ ConcreteObserverA│    │ConcreteObserverB│
                    └─────────────────┘    └─────────────────┘
```

---

## 💻 Implementation

### Basic Observer
```typescript
// Observer interface
interface Observer {
  update(data: any): void;
}

// Subject (Observable)
class Subject {
  private observers: Observer[] = [];

  attach(observer: Observer): void {
    const exists = this.observers.includes(observer);
    if (exists) return;
    this.observers.push(observer);
  }

  detach(observer: Observer): void {
    const index = this.observers.indexOf(observer);
    if (index === -1) return;
    this.observers.splice(index, 1);
  }

  notify(data: any): void {
    for (const observer of this.observers) {
      observer.update(data);
    }
  }
}

// Concrete Subject
class NewsAgency extends Subject {
  private latestNews: string = '';

  setNews(news: string): void {
    this.latestNews = news;
    this.notify(news);
  }
}

// Concrete Observers
class NewsChannel implements Observer {
  constructor(private name: string) {}

  update(news: string): void {
    console.log(`${this.name} received: ${news}`);
  }
}

// Usage
const agency = new NewsAgency();

const bbc = new NewsChannel('BBC');
const cnn = new NewsChannel('CNN');

agency.attach(bbc);
agency.attach(cnn);

agency.setNews('Breaking: Important event!');
// BBC received: Breaking: Important event!
// CNN received: Breaking: Important event!

agency.detach(cnn);
agency.setNews('Update: More details...');
// BBC received: Update: More details...
```

### Event Emitter Pattern (JavaScript Style)
```typescript
type EventHandler = (...args: any[]) => void;

class EventEmitter {
  private events: Map<string, EventHandler[]> = new Map();

  on(event: string, handler: EventHandler): void {
    if (!this.events.has(event)) {
      this.events.set(event, []);
    }
    this.events.get(event)!.push(handler);
  }

  off(event: string, handler: EventHandler): void {
    const handlers = this.events.get(event);
    if (!handlers) return;
    
    const index = handlers.indexOf(handler);
    if (index > -1) {
      handlers.splice(index, 1);
    }
  }

  emit(event: string, ...args: any[]): void {
    const handlers = this.events.get(event);
    if (!handlers) return;
    
    handlers.forEach(handler => handler(...args));
  }

  once(event: string, handler: EventHandler): void {
    const onceHandler: EventHandler = (...args) => {
      handler(...args);
      this.off(event, onceHandler);
    };
    this.on(event, onceHandler);
  }
}

// Usage
class UserService extends EventEmitter {
  createUser(userData: any) {
    const user = { id: Date.now(), ...userData };
    
    // Emit event after creating
    this.emit('user:created', user);
    
    return user;
  }
}

const userService = new UserService();

// Subscribe to events
userService.on('user:created', (user) => {
  console.log('Send welcome email to:', user.email);
});

userService.on('user:created', (user) => {
  console.log('Log analytics for:', user.id);
});

userService.createUser({ name: 'John', email: 'john@email.com' });
```

### Typed Event Emitter
```typescript
interface UserEvents {
  'user:created': (user: User) => void;
  'user:updated': (user: User, changes: Partial<User>) => void;
  'user:deleted': (userId: string) => void;
}

class TypedEventEmitter<T extends Record<string, (...args: any[]) => void>> {
  private events = new Map<keyof T, Function[]>();

  on<K extends keyof T>(event: K, handler: T[K]): void {
    if (!this.events.has(event)) {
      this.events.set(event, []);
    }
    this.events.get(event)!.push(handler);
  }

  emit<K extends keyof T>(event: K, ...args: Parameters<T[K]>): void {
    const handlers = this.events.get(event);
    if (!handlers) return;
    handlers.forEach(h => h(...args));
  }
}

// Usage with full type safety
class UserService extends TypedEventEmitter<UserEvents> {
  create(data: any): User {
    const user = { id: '1', ...data };
    this.emit('user:created', user); // ✅ Type-checked
    return user;
  }
}
```

### Stock Price Observer
```typescript
interface StockObserver {
  update(symbol: string, price: number): void;
}

class Stock {
  private observers: StockObserver[] = [];
  private price: number = 0;

  constructor(private symbol: string, initialPrice: number) {
    this.price = initialPrice;
  }

  subscribe(observer: StockObserver): () => void {
    this.observers.push(observer);
    // Return unsubscribe function
    return () => {
      this.observers = this.observers.filter(o => o !== observer);
    };
  }

  setPrice(newPrice: number): void {
    const oldPrice = this.price;
    this.price = newPrice;
    
    if (oldPrice !== newPrice) {
      this.observers.forEach(o => o.update(this.symbol, newPrice));
    }
  }

  getPrice(): number {
    return this.price;
  }
}

// Observers
class PriceDisplay implements StockObserver {
  update(symbol: string, price: number): void {
    console.log(`Display: ${symbol} = $${price}`);
  }
}

class PriceAlert implements StockObserver {
  constructor(private threshold: number) {}

  update(symbol: string, price: number): void {
    if (price > this.threshold) {
      console.log(`🚨 ALERT: ${symbol} exceeded $${this.threshold}!`);
    }
  }
}

// Usage
const appleStock = new Stock('AAPL', 150);

const display = new PriceDisplay();
const alert = new PriceAlert(160);

const unsubDisplay = appleStock.subscribe(display);
appleStock.subscribe(alert);

appleStock.setPrice(155); // Display: AAPL = $155
appleStock.setPrice(165); // Display: AAPL = $165, 🚨 ALERT!

unsubDisplay(); // Unsubscribe display
appleStock.setPrice(170); // Only alert fires
```

---

## 🔄 Push vs Pull Model

### Push Model (Subject sends data)
```typescript
interface Observer {
  update(data: CompleteData): void;
}

class Subject {
  notify() {
    const data = this.getAllData();
    this.observers.forEach(o => o.update(data)); // Push all data
  }
}
```

### Pull Model (Observer requests data)
```typescript
interface Observer {
  update(subject: Subject): void;
}

class ConcreteObserver implements Observer {
  update(subject: Subject) {
    // Pull only what you need
    const relevantData = subject.getSpecificData();
  }
}
```

---

## ✅ When to Use

| Scenario | Example |
|----------|---------|
| UI updates cuando cambian datos | React state, MobX |
| Event systems | Node.js EventEmitter |
| Notificaciones | Push notifications |
| Pub/Sub systems | Message queues |

---

## ⚠️ Considerations

> [!warning] Watch Out For
> - **Memory leaks**: Olvidar hacer unsubscribe
> - **Order of notification**: No asumas orden específico
> - **Cascade updates**: Un update triggerea otro
> - **Performance**: Muchos observers = overhead

---

## 📋 Summary

| Aspect | Details |
|--------|---------|
| **Type** | Behavioral |
| **Intent** | One-to-many notifications |
| **Key Benefit** | Loose coupling |
| **Trade-off** | Memory management, complexity |
| **JS/TS Native** | EventEmitter, addEventListener |

---

← [[Programming/Software Engineering/Design Patterns/_Index|Back to Design Patterns]]
