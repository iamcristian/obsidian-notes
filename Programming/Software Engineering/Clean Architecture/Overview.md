---
tags:
  - software-engineering
  - clean-architecture
  - overview
created: 2026-01-02
status: 🔴
---
# 🌐 Clean Architecture Overview

## 🎯 What is Clean Architecture?

Clean Architecture es un conjunto de principios de diseño que produce sistemas que son:

1. **Independientes del framework**
2. **Testeables**
3. **Independientes de la UI**
4. **Independientes de la base de datos**
5. **Independientes de cualquier agente externo**

---

## 🔄 The Dependency Rule

> [!important] The Most Important Rule
> Las dependencias del código fuente solo pueden apuntar **hacia adentro**.
> Nada en un círculo interno puede saber algo sobre un círculo externo.

```
Outer layers    →    Inner layers
(Frameworks)         (Entities)
    
    LOW level    →    HIGH level
    (Details)         (Policies)
```

---

## 🏗️ The Circles

### 1. Entities (Innermost)
```javascript
// Enterprise Business Rules
// Lo más estable, raramente cambia
class Order {
  constructor(id, items, customerId) {
    this.id = id;
    this.items = items;
    this.customerId = customerId;
    this.status = 'pending';
  }

  calculateTotal() {
    return this.items.reduce((sum, item) => 
      sum + item.price * item.quantity, 0
    );
  }

  canBeCancelled() {
    return this.status === 'pending';
  }
}
```

### 2. Use Cases
```javascript
// Application Business Rules
// Orquestan el flujo de datos hacia/desde las entidades
class CreateOrderUseCase {
  constructor(orderRepository, inventoryService, notificationService) {
    this.orderRepository = orderRepository;
    this.inventoryService = inventoryService;
    this.notificationService = notificationService;
  }

  async execute(orderData) {
    // 1. Validar disponibilidad
    await this.inventoryService.checkAvailability(orderData.items);
    
    // 2. Crear entidad
    const order = new Order(
      generateId(),
      orderData.items,
      orderData.customerId
    );
    
    // 3. Persistir
    await this.orderRepository.save(order);
    
    // 4. Notificar
    await this.notificationService.sendOrderConfirmation(order);
    
    return order;
  }
}
```

### 3. Interface Adapters
```javascript
// Convertidores entre formato de Use Cases y formato externo
class OrderController {
  constructor(createOrderUseCase) {
    this.createOrderUseCase = createOrderUseCase;
  }

  async create(req, res) {
    try {
      // Convertir request a formato de Use Case
      const orderData = {
        items: req.body.items,
        customerId: req.user.id
      };
      
      const order = await this.createOrderUseCase.execute(orderData);
      
      // Convertir resultado a formato de response
      res.status(201).json({
        id: order.id,
        total: order.calculateTotal(),
        status: order.status
      });
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  }
}
```

### 4. Frameworks & Drivers (Outermost)
```javascript
// Express routes, database connections, etc.
// Los detalles van aquí
const express = require('express');
const router = express.Router();

// Dependency Injection setup
const orderRepository = new PostgresOrderRepository(prisma);
const inventoryService = new InventoryServiceAdapter(externalApi);
const notificationService = new EmailNotificationService(sendgrid);

const createOrderUseCase = new CreateOrderUseCase(
  orderRepository,
  inventoryService,
  notificationService
);

const orderController = new OrderController(createOrderUseCase);

router.post('/orders', (req, res) => orderController.create(req, res));
```

---

## 🔌 Crossing Boundaries

### The Interface Pattern
```typescript
// En la capa de application (puerto)
interface IOrderRepository {
  save(order: Order): Promise<void>;
  findById(id: string): Promise<Order | null>;
}

// En la capa de infrastructure (adaptador)
class PostgresOrderRepository implements IOrderRepository {
  constructor(private prisma: PrismaClient) {}

  async save(order: Order): Promise<void> {
    await this.prisma.order.create({
      data: this.toDatabase(order)
    });
  }

  async findById(id: string): Promise<Order | null> {
    const data = await this.prisma.order.findUnique({ where: { id } });
    return data ? this.toDomain(data) : null;
  }

  private toDatabase(order: Order) { /* ... */ }
  private toDomain(data: any): Order { /* ... */ }
}
```

---

## ✅ Benefits

| Benefit | Description |
|---------|-------------|
| **Testability** | Puedes testear use cases sin frameworks |
| **Flexibility** | Fácil cambiar frameworks o databases |
| **Independence** | Equipos pueden trabajar en paralelo |
| **Maintainability** | Cambios localizados, bajo acoplamiento |
| **Understanding** | Estructura clara y predecible |

---

## ⚠️ Trade-offs

> [!warning] Consider These
> - **More initial code**: Más boilerplate y archivos
> - **Learning curve**: El equipo necesita entender la arquitectura
> - **Overkill for simple apps**: CRUD simple no lo necesita
> - **Mapping overhead**: Conversiones entre capas

---

## 💡 When to Use

> [!tip] Good Fit For
> - Aplicaciones empresariales complejas
> - Sistemas que vivirán mucho tiempo
> - Equipos grandes
> - Cuando las reglas de negocio son el core del valor
> - Cuando anticipas cambios en tecnología

> [!warning] Maybe Not For
> - MVPs o prototipos
> - CRUD simple
> - Scripts pequeños
> - Proyectos con deadline muy corto

---

← [[Programming/Software Engineering/Clean Architecture/_Index|Back to Clean Architecture]]
