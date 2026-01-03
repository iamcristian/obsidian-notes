---
tags:
  - software-engineering
  - architecture
  - event-driven
created: 2026-01-02
status: 🔴
---
# 📡 Event-Driven Architecture

> *"Don't call us, we'll call you."*

## 🎯 What is Event-Driven Architecture?

EDA es un patrón arquitectónico donde el flujo del programa está determinado por **eventos** - cambios significativos de estado que los componentes producen y consumen de forma desacoplada.

---

## 🏗️ Core Concepts

```
┌─────────────────────────────────────────────────────────────────┐
│                EVENT-DRIVEN ARCHITECTURE                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   PRODUCER                 BROKER                 CONSUMER      │
│  ┌─────────┐           ┌─────────────┐          ┌─────────┐    │
│  │ Order   │──Event───►│             │─────────►│ Email   │    │
│  │ Service │           │             │          │ Service │    │
│  └─────────┘           │   Message   │          └─────────┘    │
│                        │   Broker    │                          │
│  ┌─────────┐           │             │          ┌─────────┐    │
│  │ Payment │──Event───►│  (Kafka,    │─────────►│Inventory│    │
│  │ Service │           │   RabbitMQ, │          │ Service │    │
│  └─────────┘           │   SQS...)   │          └─────────┘    │
│                        │             │                          │
│                        └─────────────┘                          │
│                                                                 │
│   Events are:                                                   │
│   • Immutable facts about something that happened               │
│   • Broadcast to all interested consumers                       │
│   • Processed asynchronously                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Event Types

### 1. Domain Events
```typescript
// Something that happened in the domain
interface OrderCreatedEvent {
  type: 'ORDER_CREATED';
  aggregateId: string;  // Order ID
  timestamp: Date;
  payload: {
    customerId: string;
    items: OrderItem[];
    total: number;
  };
  metadata: {
    correlationId: string;
    userId: string;
  };
}

// More examples
type DomainEvent =
  | { type: 'USER_REGISTERED'; payload: { userId: string; email: string } }
  | { type: 'PAYMENT_RECEIVED'; payload: { orderId: string; amount: number } }
  | { type: 'ITEM_SHIPPED'; payload: { orderId: string; trackingNumber: string } };
```

### 2. Integration Events
```typescript
// Cross-service communication
interface IntegrationEvent {
  eventId: string;
  eventType: string;
  timestamp: Date;
  source: string;  // Which service produced it
  data: unknown;
}
```

### 3. Event Notifications
```typescript
// Minimal event - just notifies something happened
interface OrderCreatedNotification {
  type: 'ORDER_CREATED';
  orderId: string;
  timestamp: Date;
  // Consumer fetches details via API if needed
}
```

---

## 🔄 Event Patterns

### Publish-Subscribe (Pub/Sub)
```
┌─────────────────────────────────────────────────────────────────┐
│                     PUBLISH-SUBSCRIBE                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                        ┌─────────────┐                          │
│                     ┌──│ Subscriber A│                          │
│   ┌─────────┐       │  └─────────────┘                          │
│   │Publisher│──────►│  ┌─────────────┐                          │
│   └─────────┘       ├──│ Subscriber B│                          │
│                     │  └─────────────┘                          │
│        │            │  ┌─────────────┐                          │
│        ▼            └──│ Subscriber C│                          │
│      Topic             └─────────────┘                          │
│                                                                 │
│   • One event → Many consumers                                  │
│   • Consumers are independent                                   │
│   • Each consumer gets a copy                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Event Sourcing
```
┌─────────────────────────────────────────────────────────────────┐
│                      EVENT SOURCING                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Instead of storing current state, store all events            │
│                                                                 │
│   Event Store:                                                  │
│   ┌───────┬────────────────────────────────────────────────┐   │
│   │ Seq   │ Event                                           │   │
│   ├───────┼────────────────────────────────────────────────┤   │
│   │ 1     │ AccountCreated { id: "123", owner: "John" }     │   │
│   │ 2     │ MoneyDeposited { id: "123", amount: 100 }       │   │
│   │ 3     │ MoneyWithdrawn { id: "123", amount: 30 }        │   │
│   │ 4     │ MoneyDeposited { id: "123", amount: 50 }        │   │
│   └───────┴────────────────────────────────────────────────┘   │
│                                                                 │
│   Current State = Replay all events                             │
│   Balance = 0 + 100 - 30 + 50 = 120                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```typescript
// Event Sourcing implementation
interface Event {
  aggregateId: string;
  version: number;
  type: string;
  data: unknown;
  timestamp: Date;
}

class BankAccount {
  private balance = 0;
  private events: Event[] = [];

  // Apply event to update state
  private apply(event: Event) {
    switch (event.type) {
      case 'MONEY_DEPOSITED':
        this.balance += (event.data as { amount: number }).amount;
        break;
      case 'MONEY_WITHDRAWN':
        this.balance -= (event.data as { amount: number }).amount;
        break;
    }
    this.events.push(event);
  }

  // Command creates event
  deposit(amount: number) {
    if (amount <= 0) throw new Error('Amount must be positive');
    this.apply({
      aggregateId: this.id,
      version: this.events.length + 1,
      type: 'MONEY_DEPOSITED',
      data: { amount },
      timestamp: new Date()
    });
  }

  // Rebuild from events
  static fromEvents(events: Event[]): BankAccount {
    const account = new BankAccount();
    events.forEach(e => account.apply(e));
    return account;
  }
}
```

### CQRS + Event Sourcing
```
┌─────────────────────────────────────────────────────────────────┐
│                   CQRS + EVENT SOURCING                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Write Side                        Read Side                   │
│   ┌─────────────┐                  ┌─────────────┐             │
│   │  Commands   │                  │   Queries   │             │
│   └──────┬──────┘                  └──────┬──────┘             │
│          │                                │                     │
│          ▼                                ▼                     │
│   ┌─────────────┐                  ┌─────────────┐             │
│   │  Aggregate  │                  │  Read Model │             │
│   └──────┬──────┘                  │  (Projected)│             │
│          │                         └──────▲──────┘             │
│          ▼                                │                     │
│   ┌─────────────┐     Events      ┌─────────────┐             │
│   │ Event Store │────────────────►│  Projector  │             │
│   └─────────────┘                 └─────────────┘             │
│                                                                 │
│   Write: Store events                                           │
│   Read: Query optimized projections                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 💻 Implementation Examples

### Kafka Producer/Consumer (Node.js)
```typescript
import { Kafka, Producer, Consumer, EachMessagePayload } from 'kafkajs';

// Setup
const kafka = new Kafka({
  clientId: 'my-app',
  brokers: ['localhost:9092']
});

// Producer
const producer: Producer = kafka.producer();

async function publishEvent(topic: string, event: object) {
  await producer.connect();
  await producer.send({
    topic,
    messages: [{
      key: event.aggregateId,
      value: JSON.stringify(event),
      headers: {
        'event-type': event.type,
        'correlation-id': event.correlationId
      }
    }]
  });
}

// Usage
await publishEvent('orders', {
  type: 'ORDER_CREATED',
  aggregateId: 'order-123',
  correlationId: 'req-456',
  data: { customerId: 'cust-789', total: 99.99 }
});

// Consumer
const consumer: Consumer = kafka.consumer({ groupId: 'email-service' });

async function startConsumer() {
  await consumer.connect();
  await consumer.subscribe({ topic: 'orders', fromBeginning: false });

  await consumer.run({
    eachMessage: async ({ topic, partition, message }: EachMessagePayload) => {
      const event = JSON.parse(message.value!.toString());
      
      switch (event.type) {
        case 'ORDER_CREATED':
          await sendOrderConfirmationEmail(event.data);
          break;
        case 'ORDER_SHIPPED':
          await sendShippingNotification(event.data);
          break;
      }
    }
  });
}
```

### AWS EventBridge
```typescript
import { EventBridgeClient, PutEventsCommand } from '@aws-sdk/client-eventbridge';

const client = new EventBridgeClient({ region: 'us-east-1' });

async function publishEvent(event: object) {
  await client.send(new PutEventsCommand({
    Entries: [{
      Source: 'com.myapp.orders',
      DetailType: 'OrderCreated',
      Detail: JSON.stringify(event),
      EventBusName: 'my-event-bus'
    }]
  }));
}

// Lambda handler for consuming
export const handler = async (event: any) => {
  const orderEvent = event.detail;
  // Process event
  console.log('Order created:', orderEvent);
};
```

---

## 🔧 Event Design Best Practices

### Event Structure
```typescript
interface WellDesignedEvent {
  // Identity
  eventId: string;           // Unique ID for idempotency
  eventType: string;         // What happened
  aggregateId: string;       // What entity it happened to
  aggregateType: string;     // Type of entity
  
  // Versioning
  version: number;           // Schema version
  timestamp: Date;           // When it happened
  
  // Payload
  data: EventPayload;        // The actual data
  
  // Context
  metadata: {
    correlationId: string;   // Track across services
    causationId: string;     // What caused this event
    userId?: string;         // Who triggered it
    traceId?: string;        // Distributed tracing
  };
}
```

### Idempotent Consumers
```typescript
class IdempotentConsumer {
  private processedEvents: Set<string> = new Set();
  
  async handleEvent(event: Event) {
    // Check if already processed
    if (await this.isProcessed(event.eventId)) {
      console.log(`Event ${event.eventId} already processed, skipping`);
      return;
    }
    
    // Process event
    await this.processEvent(event);
    
    // Mark as processed
    await this.markProcessed(event.eventId);
  }
  
  private async isProcessed(eventId: string): Promise<boolean> {
    // Check database/cache
    return this.processedEvents.has(eventId);
  }
  
  private async markProcessed(eventId: string): Promise<void> {
    // Store in database/cache with TTL
    this.processedEvents.add(eventId);
  }
}
```

---

## ⚠️ Challenges & Solutions

| Challenge | Solution |
|-----------|----------|
| **Event ordering** | Partition by aggregate ID, sequence numbers |
| **Duplicate events** | Idempotent consumers, deduplication |
| **Schema evolution** | Event versioning, backward compatibility |
| **Event replay** | Snapshots, selective replay |
| **Debugging** | Correlation IDs, distributed tracing |
| **Eventual consistency** | Saga pattern, compensating actions |

---

## 📊 When to Use EDA

```
✅ USE WHEN:
• Microservices needing loose coupling
• Real-time data processing
• Audit/compliance requirements (event sourcing)
• Complex workflows with multiple services
• High scalability requirements

❌ AVOID WHEN:
• Simple CRUD applications
• Strong consistency required everywhere
• Team unfamiliar with async patterns
• Low complexity, few services
```

---

## 📚 Related

- [[Programming/Software Engineering/Architectural Patterns/CQRS|CQRS]]
- [[Programming/Software Engineering/Architectural Patterns/Microservices|Microservices]]
- [[Programming/Software Engineering/System Design/Scalability|Scalability]]

---

← [[Programming/Software Engineering/Architectural Patterns/_Index|Back to Architectural Patterns]]
