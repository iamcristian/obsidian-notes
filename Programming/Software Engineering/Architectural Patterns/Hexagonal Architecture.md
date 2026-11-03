---
tags:
  - software-engineering
  - architectural-patterns
  - hexagonal
created: 2026-01-02
status: 🔴
---
# 🔷 Hexagonal Architecture

> *"Allow an application to equally be driven by users, programs, automated tests, and to be developed and tested in isolation from its eventual run-time devices and databases."* — Alistair Cockburn

## 🎯 Also Known As

- **Ports and Adapters**
- **Clean Architecture** (variación)
- **Onion Architecture** (variación)

---

## 📊 Structure

```
                    ┌─────────────────────────────────────┐
                    │           ADAPTERS (Outside)         │
                    │  ┌─────────┐         ┌─────────┐    │
                    │  │  REST   │         │  CLI    │    │
                    │  │ Adapter │         │ Adapter │    │
                    │  └────┬────┘         └────┬────┘    │
                    │       │                   │          │
                    │       ▼                   ▼          │
                    │  ┌────────────────────────────┐     │
                    │  │    PORTS (Interfaces)      │     │
                    │  │  ┌──────────────────────┐  │     │
                    │  │  │                      │  │     │
   PRIMARY          │  │  │    APPLICATION       │  │     │    SECONDARY
   (Driving)        │  │  │      CORE            │  │     │    (Driven)
                    │  │  │                      │  │     │
                    │  │  │      DOMAIN          │  │     │
                    │  │  │                      │  │     │
                    │  │  └──────────────────────┘  │     │
                    │  └────────────────────────────┘     │
                    │       │                   │          │
                    │       ▼                   ▼          │
                    │  ┌─────────┐         ┌─────────┐    │
                    │  │Database│         │ Email   │    │
                    │  │Adapter │         │ Adapter │    │
                    │  └─────────┘         └─────────┘    │
                    └─────────────────────────────────────┘
```

---

## 🔑 Key Concepts

### Ports
> Interfaces que definen cómo la aplicación interactúa con el mundo exterior

```typescript
// PRIMARY PORT (Driving) - Cómo el exterior usa la aplicación
interface OrderService {
  createOrder(data: CreateOrderDTO): Promise<Order>;
  getOrder(id: string): Promise<Order>;
  cancelOrder(id: string): Promise<void>;
}

// SECONDARY PORT (Driven) - Cómo la aplicación usa el exterior
interface OrderRepository {
  save(order: Order): Promise<void>;
  findById(id: string): Promise<Order | null>;
  delete(id: string): Promise<void>;
}

interface EmailService {
  sendOrderConfirmation(order: Order): Promise<void>;
}
```

### Adapters
> Implementaciones concretas de los ports

```typescript
// PRIMARY ADAPTER (Driving) - REST Controller
class OrderController {
  constructor(private orderService: OrderService) {}

  async create(req: Request, res: Response) {
    const order = await this.orderService.createOrder(req.body);
    res.json(order);
  }
}

// SECONDARY ADAPTER (Driven) - Database Repository
class PostgresOrderRepository implements OrderRepository {
  constructor(private prisma: PrismaClient) {}

  async save(order: Order): Promise<void> {
    await this.prisma.order.create({ data: this.toDb(order) });
  }

  async findById(id: string): Promise<Order | null> {
    const data = await this.prisma.order.findUnique({ where: { id } });
    return data ? this.toDomain(data) : null;
  }
}
```

---

## 💻 Complete Example

### Project Structure
```
src/
├── domain/                    # Core business logic
│   ├── entities/
│   │   └── Order.ts
│   └── value-objects/
│       └── Money.ts
│
├── application/               # Use cases + Port interfaces
│   ├── ports/
│   │   ├── driving/          # Primary ports (input)
│   │   │   └── OrderService.ts
│   │   └── driven/           # Secondary ports (output)
│   │       ├── OrderRepository.ts
│   │       └── EmailService.ts
│   └── services/
│       └── OrderServiceImpl.ts
│
├── adapters/                  # Implementations
│   ├── driving/              # Primary adapters (input)
│   │   ├── rest/
│   │   │   └── OrderController.ts
│   │   └── graphql/
│   │       └── OrderResolver.ts
│   └── driven/               # Secondary adapters (output)
│       ├── persistence/
│       │   └── PostgresOrderRepository.ts
│       └── email/
│           └── SendGridEmailService.ts
│
└── infrastructure/            # Framework setup
    ├── config/
    └── di/
        └── container.ts
```

### Domain Layer
```typescript
// domain/entities/Order.ts
export class Order {
  private constructor(
    public readonly id: string,
    public readonly customerId: string,
    public readonly items: OrderItem[],
    private _status: OrderStatus
  ) {}

  static create(customerId: string, items: OrderItem[]): Order {
    if (items.length === 0) {
      throw new Error('Order must have at least one item');
    }
    return new Order(generateId(), customerId, items, 'pending');
  }

  get status(): OrderStatus {
    return this._status;
  }

  get total(): number {
    return this.items.reduce((sum, item) => sum + item.total, 0);
  }

  confirm(): void {
    if (this._status !== 'pending') {
      throw new Error('Can only confirm pending orders');
    }
    this._status = 'confirmed';
  }

  cancel(): void {
    if (this._status === 'shipped') {
      throw new Error('Cannot cancel shipped orders');
    }
    this._status = 'cancelled';
  }
}
```

### Application Layer (Ports + Service)
```typescript
// application/ports/driving/OrderService.ts (Primary Port)
export interface OrderService {
  createOrder(data: CreateOrderDTO): Promise<Order>;
  confirmOrder(id: string): Promise<Order>;
  cancelOrder(id: string): Promise<void>;
}

// application/ports/driven/OrderRepository.ts (Secondary Port)
export interface OrderRepository {
  save(order: Order): Promise<void>;
  findById(id: string): Promise<Order | null>;
}

// application/ports/driven/NotificationService.ts (Secondary Port)
export interface NotificationService {
  notifyOrderConfirmed(order: Order): Promise<void>;
}

// application/services/OrderServiceImpl.ts
export class OrderServiceImpl implements OrderService {
  constructor(
    private orderRepository: OrderRepository,
    private notificationService: NotificationService
  ) {}

  async createOrder(data: CreateOrderDTO): Promise<Order> {
    const order = Order.create(data.customerId, data.items);
    await this.orderRepository.save(order);
    return order;
  }

  async confirmOrder(id: string): Promise<Order> {
    const order = await this.orderRepository.findById(id);
    if (!order) throw new NotFoundError('Order', id);
    
    order.confirm();
    await this.orderRepository.save(order);
    await this.notificationService.notifyOrderConfirmed(order);
    
    return order;
  }
}
```

### Adapters Layer
```typescript
// adapters/driving/rest/OrderController.ts (Primary Adapter)
export class OrderController {
  constructor(private orderService: OrderService) {}

  async create(req: Request, res: Response): Promise<void> {
    const order = await this.orderService.createOrder({
      customerId: req.user.id,
      items: req.body.items
    });
    res.status(201).json(this.toResponse(order));
  }

  private toResponse(order: Order) {
    return {
      id: order.id,
      status: order.status,
      total: order.total,
      items: order.items.length
    };
  }
}

// adapters/driven/persistence/PostgresOrderRepository.ts (Secondary Adapter)
export class PostgresOrderRepository implements OrderRepository {
  constructor(private prisma: PrismaClient) {}

  async save(order: Order): Promise<void> {
    await this.prisma.order.upsert({
      where: { id: order.id },
      create: this.toDatabase(order),
      update: this.toDatabase(order)
    });
  }

  async findById(id: string): Promise<Order | null> {
    const data = await this.prisma.order.findUnique({
      where: { id },
      include: { items: true }
    });
    return data ? this.toDomain(data) : null;
  }

  private toDomain(data: any): Order { /* mapping */ }
  private toDatabase(order: Order): any { /* mapping */ }
}
```

---

## ✅ Benefits

| Benefit | Description |
|---------|-------------|
| **Testability** | Core se testea sin DB ni frameworks |
| **Flexibility** | Cambiar adapters sin tocar core |
| **Independence** | Framework es un detalle, no el centro |
| **Parallel Development** | Equipos trabajan en adapters diferentes |

---

## 🔄 Hexagonal vs Clean Architecture

| Aspect | Hexagonal | Clean Architecture |
|--------|-----------|-------------------|
| **Terminology** | Ports & Adapters | Entities, Use Cases |
| **Visual** | Hexagon shape | Concentric circles |
| **Focus** | External interactions | Dependency direction |
| **Essence** | Same core principles | Same core principles |

---

## 💡 Tips

> [!tip] Best Practices
> - Los **Ports** van en la capa de Application
> - Los **Adapters** son intercambiables
> - El **Domain** NO conoce los ports
> - Usa **Dependency Injection** para conectar todo

---

← [[Programming/Software Engineering/Architectural Patterns/Architectural Patterns Complete Guide|Back to Architectural Patterns]]
