---
tags:
  - software-engineering
  - architectural-patterns
  - cqrs
created: 2026-01-02
status: 🔴
---
# 📤📥 CQRS

> *"Command Query Responsibility Segregation"*

## 🎯 What is CQRS?

Separar las operaciones de **lectura (Query)** de las operaciones de **escritura (Command)** en modelos diferentes.

---

## 📊 Traditional vs CQRS

### Traditional (CRUD)
```
┌─────────────────────────────────────┐
│           Single Model              │
│  ┌─────────────────────────────┐   │
│  │    Create, Read, Update,    │   │
│  │         Delete              │   │
│  └─────────────────────────────┘   │
│                 │                   │
│         ┌───────┴───────┐          │
│         │   Database    │          │
│         └───────────────┘          │
└─────────────────────────────────────┘
```

### CQRS
```
┌──────────────────────────────────────────────────┐
│                                                   │
│   COMMAND SIDE            QUERY SIDE             │
│  ┌─────────────┐        ┌─────────────┐         │
│  │  Commands   │        │   Queries   │         │
│  │ (Write Ops) │        │ (Read Ops)  │         │
│  └──────┬──────┘        └──────┬──────┘         │
│         │                      │                 │
│  ┌──────┴──────┐        ┌──────┴──────┐         │
│  │ Write Model │        │ Read Model  │         │
│  └──────┬──────┘        └──────┬──────┘         │
│         │                      │                 │
│  ┌──────┴──────┐        ┌──────┴──────┐         │
│  │  Write DB   │───────→│  Read DB    │         │
│  └─────────────┘  sync  └─────────────┘         │
└──────────────────────────────────────────────────┘
```

---

## 💻 Implementation

### Basic CQRS Structure
```typescript
// ============ COMMANDS (Write) ============

// Command
interface CreateOrderCommand {
  customerId: string;
  items: { productId: string; quantity: number }[];
}

// Command Handler
class CreateOrderCommandHandler {
  constructor(
    private orderRepository: OrderWriteRepository,
    private eventBus: EventBus
  ) {}

  async handle(command: CreateOrderCommand): Promise<string> {
    // Validación y lógica de negocio
    const order = Order.create(command.customerId, command.items);
    
    // Persistir
    await this.orderRepository.save(order);
    
    // Publicar evento para sincronizar read model
    await this.eventBus.publish(new OrderCreatedEvent(order));
    
    return order.id;
  }
}

// ============ QUERIES (Read) ============

// Query
interface GetOrderQuery {
  orderId: string;
}

// Query Handler
class GetOrderQueryHandler {
  constructor(private orderReadRepository: OrderReadRepository) {}

  async handle(query: GetOrderQuery): Promise<OrderDTO | null> {
    // Lee directamente del read model optimizado
    return this.orderReadRepository.findById(query.orderId);
  }
}

// Read Model (optimizado para queries)
interface OrderDTO {
  id: string;
  customerName: string;  // Denormalizado
  items: {
    productName: string;  // Denormalizado
    quantity: number;
    price: number;
  }[];
  total: number;  // Pre-calculado
  status: string;
  createdAt: Date;
}
```

### Command and Query Bus
```typescript
// Command Bus
class CommandBus {
  private handlers = new Map<string, CommandHandler>();

  register(commandName: string, handler: CommandHandler) {
    this.handlers.set(commandName, handler);
  }

  async dispatch<T>(command: Command): Promise<T> {
    const handler = this.handlers.get(command.constructor.name);
    if (!handler) {
      throw new Error(`No handler for ${command.constructor.name}`);
    }
    return handler.handle(command);
  }
}

// Query Bus
class QueryBus {
  private handlers = new Map<string, QueryHandler>();

  register(queryName: string, handler: QueryHandler) {
    this.handlers.set(queryName, handler);
  }

  async dispatch<T>(query: Query): Promise<T> {
    const handler = this.handlers.get(query.constructor.name);
    if (!handler) {
      throw new Error(`No handler for ${query.constructor.name}`);
    }
    return handler.handle(query);
  }
}

// Usage
const commandBus = new CommandBus();
const queryBus = new QueryBus();

// Registrar handlers
commandBus.register('CreateOrderCommand', new CreateOrderCommandHandler(...));
queryBus.register('GetOrderQuery', new GetOrderQueryHandler(...));

// Controller
class OrderController {
  constructor(
    private commandBus: CommandBus,
    private queryBus: QueryBus
  ) {}

  async create(req: Request, res: Response) {
    const orderId = await this.commandBus.dispatch<string>(
      new CreateOrderCommand(req.body)
    );
    res.json({ orderId });
  }

  async get(req: Request, res: Response) {
    const order = await this.queryBus.dispatch<OrderDTO>(
      new GetOrderQuery(req.params.id)
    );
    res.json(order);
  }
}
```

### Synchronizing Read Model
```typescript
// Event Handler que actualiza Read Model
class OrderProjection {
  constructor(private readDb: ReadDatabase) {}

  @EventHandler('OrderCreated')
  async onOrderCreated(event: OrderCreatedEvent) {
    await this.readDb.orders.insert({
      id: event.orderId,
      customerName: event.customerName,
      items: event.items,
      total: event.total,
      status: 'pending',
      createdAt: event.timestamp
    });
  }

  @EventHandler('OrderShipped')
  async onOrderShipped(event: OrderShippedEvent) {
    await this.readDb.orders.update(
      { id: event.orderId },
      { status: 'shipped', shippedAt: event.timestamp }
    );
  }
}
```

---

## 🔄 Different Databases

### Simple: Same DB, Different Models
```typescript
// Write Model (normalized)
class OrderEntity {
  id: string;
  customerId: string;  // FK
  items: OrderItemEntity[];
}

// Read Model (denormalized view/table)
class OrderView {
  id: string;
  customerName: string;  // Copied
  customerEmail: string; // Copied
  itemsSummary: string;  // Computed
  total: number;         // Computed
}
```

### Advanced: Separate Databases
```typescript
// Write: PostgreSQL (strong consistency)
class PostgresOrderRepository implements OrderWriteRepository {
  async save(order: Order) {
    await this.prisma.order.create({ data: order });
  }
}

// Read: Elasticsearch (fast queries)
class ElasticsearchOrderRepository implements OrderReadRepository {
  async search(criteria: SearchCriteria) {
    return this.elastic.search({
      index: 'orders',
      body: { query: criteria }
    });
  }
}

// Read: Redis (cached frequent queries)
class RedisOrderRepository implements OrderReadRepository {
  async findById(id: string) {
    return JSON.parse(await this.redis.get(`order:${id}`));
  }
}
```

---

## ✅ Benefits

| Benefit | Description |
|---------|-------------|
| **Optimized Queries** | Read models diseñados para queries específicas |
| **Scalability** | Escalar reads y writes independientemente |
| **Simplicity** | Cada modelo es más simple |
| **Performance** | Reads no bloquean writes |
| **Flexibility** | Diferentes DBs para diferentes necesidades |

---

## ⚠️ Challenges

| Challenge | Mitigation |
|-----------|------------|
| **Eventual Consistency** | UI debe manejar delays |
| **Complexity** | Solo usar cuando se necesita |
| **Synchronization** | Eventos + projections |
| **Debugging** | Logs y tracing detallado |

---

## 🎯 When to Use

> [!tip] Good Fit
> - Muchas más reads que writes
> - Queries complejas que ralentizan writes
> - Diferentes requisitos de escalabilidad
> - Domain complejo con muchas vistas

> [!warning] Avoid When
> - CRUD simple
> - Necesitas consistencia inmediata
> - Equipo pequeño sin experiencia
> - Dominio simple

---

## 📋 Summary

| Aspect | Details |
|--------|---------|
| **Key Idea** | Separar read y write models |
| **Commands** | Cambian estado, no retornan data |
| **Queries** | Leen data, no cambian estado |
| **Trade-off** | Complejidad vs Performance/Escalabilidad |
| **Related** | Event Sourcing (combinación común) |

---

← [[Programming/Software Engineering/Architectural Patterns/Architectural Patterns Complete Guide|Back to Architectural Patterns]]
