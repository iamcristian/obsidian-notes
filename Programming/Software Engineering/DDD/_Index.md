---
tags:
  - software-engineering
  - ddd
  - dashboard
created: 2026-01-02
---
# 🎯 Domain-Driven Design (DDD)

> *"Focus on the core domain and domain logic, not on the database or infrastructure."* — Eric Evans

## 📚 Contents

| Topic | Description | Status |
|-------|-------------|--------|
| [[Programming/Software Engineering/DDD/Strategic Design\|Strategic Design]] | Bounded Contexts, Ubiquitous Language | 🔴 |
| [[Programming/Software Engineering/DDD/Tactical Design\|Tactical Design]] | Entities, Value Objects, Aggregates | 🔴 |
| [[Programming/Software Engineering/DDD/Building Blocks\|Building Blocks]] | Repositories, Services, Factories | 🔴 |

---

## 🎯 Core Concepts Overview

### Strategic Design (The Big Picture)

```
┌─────────────────────────────────────────────────────────────────┐
│                        E-Commerce System                         │
├────────────────┬──────────────────┬─────────────────────────────┤
│   SALES        │    INVENTORY     │    SHIPPING                 │
│   Context      │    Context       │    Context                  │
│                │                  │                             │
│  • Order       │  • Product       │  • Shipment                │
│  • Customer    │  • Stock         │  • Carrier                 │
│  • Price       │  • Warehouse     │  • Tracking                │
│                │                  │                             │
│  Customer =    │  Product =       │  Order =                   │
│  buyer info    │  inventory item  │  shipment reference        │
└────────────────┴──────────────────┴─────────────────────────────┘
         │                │                      │
         └────────────────┼──────────────────────┘
                          │
              Context Mapping / Integration
```

### Ubiquitous Language
```markdown
Cada Bounded Context tiene su propio lenguaje:

SALES Context:
- "Customer" = persona que compra
- "Order" = pedido de compra
- "Product" = lo que el cliente quiere

INVENTORY Context:
- "Product" = SKU con stock
- "Warehouse" = ubicación física
- "Stock" = cantidad disponible

SHIPPING Context:
- "Package" = caja física a enviar
- "Order" = referencia para agrupar paquetes
```

---

## 🧱 Building Blocks

### Entities
```typescript
// Tiene identidad única - importa QUIÉN es
class Order {
  private constructor(
    public readonly id: OrderId,  // Identidad
    private items: OrderItem[],
    private status: OrderStatus
  ) {}

  static create(customerId: CustomerId): Order {
    return new Order(
      OrderId.generate(),
      [],
      OrderStatus.Draft
    );
  }

  // Comportamiento, no solo datos
  addItem(product: Product, quantity: number): void {
    if (this.status !== OrderStatus.Draft) {
      throw new Error('Cannot modify confirmed order');
    }
    this.items.push(new OrderItem(product.id, quantity, product.price));
  }
}
```

### Value Objects
```typescript
// Sin identidad - importa QUÉ es (sus valores)
class Money {
  constructor(
    public readonly amount: number,
    public readonly currency: string
  ) {
    if (amount < 0) throw new Error('Amount cannot be negative');
  }

  add(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new Error('Cannot add different currencies');
    }
    return new Money(this.amount + other.amount, this.currency);
  }

  equals(other: Money): boolean {
    return this.amount === other.amount && 
           this.currency === other.currency;
  }
}

class Address {
  constructor(
    public readonly street: string,
    public readonly city: string,
    public readonly country: string,
    public readonly zipCode: string
  ) {}

  equals(other: Address): boolean {
    return this.street === other.street &&
           this.city === other.city &&
           this.country === other.country &&
           this.zipCode === other.zipCode;
  }
}
```

### Aggregates
```typescript
// Aggregate Root: Order
// Aggregate: Order + OrderItems + ShippingAddress
class Order {
  private items: OrderItem[] = [];
  private shippingAddress?: Address;

  // Solo se accede a items A TRAVÉS del aggregate root
  addItem(productId: ProductId, quantity: number, price: Money): void {
    const existingItem = this.items.find(i => i.productId.equals(productId));
    
    if (existingItem) {
      existingItem.increaseQuantity(quantity);
    } else {
      this.items.push(new OrderItem(productId, quantity, price));
    }
  }

  // Invariantes del aggregate
  confirm(): void {
    if (this.items.length === 0) {
      throw new Error('Cannot confirm empty order');
    }
    if (!this.shippingAddress) {
      throw new Error('Shipping address required');
    }
    this.status = OrderStatus.Confirmed;
  }
}
```

### Domain Events
```typescript
abstract class DomainEvent {
  public readonly occurredAt: Date = new Date();
}

class OrderConfirmedEvent extends DomainEvent {
  constructor(
    public readonly orderId: OrderId,
    public readonly customerId: CustomerId,
    public readonly total: Money
  ) {
    super();
  }
}

class Order {
  private domainEvents: DomainEvent[] = [];

  confirm(): void {
    // ... validation
    this.status = OrderStatus.Confirmed;
    
    this.domainEvents.push(new OrderConfirmedEvent(
      this.id,
      this.customerId,
      this.calculateTotal()
    ));
  }

  pullDomainEvents(): DomainEvent[] {
    const events = [...this.domainEvents];
    this.domainEvents = [];
    return events;
  }
}
```

### Repositories
```typescript
// Interface en Domain layer
interface OrderRepository {
  save(order: Order): Promise<void>;
  findById(id: OrderId): Promise<Order | null>;
  findByCustomer(customerId: CustomerId): Promise<Order[]>;
}

// Implementación en Infrastructure layer
class PostgresOrderRepository implements OrderRepository {
  async save(order: Order): Promise<void> {
    // Mapear aggregate a tablas
    await this.prisma.$transaction([
      this.prisma.order.upsert({...}),
      ...order.items.map(item => 
        this.prisma.orderItem.upsert({...})
      )
    ]);
  }
}
```

### Domain Services
```typescript
// Lógica que no pertenece a ninguna entidad específica
class PricingService {
  calculateOrderTotal(
    items: OrderItem[],
    discounts: Discount[],
    shipping: ShippingMethod
  ): Money {
    const subtotal = items.reduce(
      (sum, item) => sum.add(item.total),
      Money.zero('USD')
    );

    const discount = this.applyDiscounts(subtotal, discounts);
    const shippingCost = shipping.calculateCost(items);

    return subtotal.subtract(discount).add(shippingCost);
  }
}
```

---

## 📖 Resources

- 📕 **Book**: "Domain-Driven Design" - Eric Evans (The Blue Book)
- 📕 **Book**: "Implementing Domain-Driven Design" - Vaughn Vernon
- 📕 **Book**: "Learning Domain-Driven Design" - Vlad Khononov

---

## 📝 Notes
```dataview
TABLE WITHOUT ID
	file.link as "Topic",
	status as "Status"
FROM "Programming/Software Engineering/DDD"
WHERE !contains(file.name, "_Index")
SORT file.name asc
```

---

← [[Programming/Software Engineering/_Index|Back to Software Engineering]]
