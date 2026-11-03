---
tags:
  - software-engineering
  - architectural-patterns
  - microservices
created: 2026-01-02
status: 🔴
---
# 🔲 Microservices Architecture

> *"Microservices are small, autonomous services that work together."* — Sam Newman

## 🎯 What Are Microservices?

Una arquitectura donde la aplicación se compone de **servicios pequeños e independientes** que:
- Se despliegan de forma independiente
- Se comunican por red (HTTP, mensajes)
- Tienen su propia base de datos
- Están organizados alrededor de capacidades de negocio

---

## 📊 Monolith vs Microservices

```
MONOLITH                         MICROSERVICES
┌────────────────────┐           ┌──────┐ ┌──────┐ ┌──────┐
│                    │           │ User │ │Order │ │ Pay  │
│    Application     │    →      │ Svc  │ │ Svc  │ │ Svc  │
│                    │           └──┬───┘ └──┬───┘ └──┬───┘
│  ┌──────────────┐  │              │        │        │
│  │   Database   │  │           ┌──┴──┐  ┌──┴──┐  ┌──┴──┐
│  └──────────────┘  │           │ DB  │  │ DB  │  │ DB  │
└────────────────────┘           └─────┘  └─────┘  └─────┘
```

---

## 🔑 Key Characteristics

### 1. Single Responsibility
```
┌─────────────────────────────────────────────────────┐
│                     E-Commerce                       │
├──────────┬──────────┬──────────┬──────────┬────────┤
│  Users   │ Products │  Orders  │ Payments │Shipping│
│ Service  │ Service  │ Service  │ Service  │Service │
└──────────┴──────────┴──────────┴──────────┴────────┘
```

### 2. Decentralized Data Management
```typescript
// User Service - tiene su propia DB
class UserService {
  private userDb: MongoDB;  // NoSQL para flexibilidad
}

// Order Service - tiene su propia DB
class OrderService {
  private orderDb: PostgreSQL;  // SQL para transacciones
}

// Product Service - tiene su propia DB
class ProductService {
  private productDb: Elasticsearch;  // Para búsquedas
}
```

### 3. Independent Deployment
```yaml
# user-service/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: user-service
        image: myapp/user-service:v2.1.0
```

---

## 📡 Communication Patterns

### Synchronous (HTTP/REST)
```typescript
// Order Service calling User Service
class OrderService {
  async createOrder(userId: string, items: Item[]) {
    // Llamada síncrona a User Service
    const user = await fetch(`http://user-service/users/${userId}`);
    
    if (!user) throw new Error('User not found');
    
    return this.orderRepository.save({
      userId,
      items,
      userEmail: user.email
    });
  }
}
```

### Asynchronous (Message Queue)
```typescript
// Order Service publica evento
class OrderService {
  constructor(private messageBroker: MessageBroker) {}

  async createOrder(data: OrderData) {
    const order = await this.orderRepository.save(data);
    
    // Publicar evento - no espera respuesta
    await this.messageBroker.publish('order.created', {
      orderId: order.id,
      userId: data.userId,
      total: order.total
    });
    
    return order;
  }
}

// Notification Service escucha eventos
class NotificationService {
  constructor(private messageBroker: MessageBroker) {
    this.messageBroker.subscribe('order.created', this.handleOrderCreated);
  }

  async handleOrderCreated(event: OrderCreatedEvent) {
    await this.sendEmail(event.userId, 'Order Confirmation', ...);
  }
}
```

---

## 🏗️ Common Patterns

### API Gateway
```
                    ┌───────────────┐
                    │  API Gateway  │
                    │   (Kong/AWS)  │
                    └───────┬───────┘
                            │
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
    ┌──────────┐      ┌──────────┐      ┌──────────┐
    │  Users   │      │  Orders  │      │ Products │
    │ Service  │      │ Service  │      │ Service  │
    └──────────┘      └──────────┘      └──────────┘
```

### Service Discovery
```typescript
// Services se registran al iniciar
class UserService {
  async onStart() {
    await serviceRegistry.register({
      name: 'user-service',
      host: 'user-svc.internal',
      port: 3000,
      healthCheck: '/health'
    });
  }
}

// Otros servicios descubren dinámicamente
class OrderService {
  async getUser(id: string) {
    const userService = await serviceRegistry.discover('user-service');
    return fetch(`${userService.url}/users/${id}`);
  }
}
```

### Circuit Breaker
```typescript
import CircuitBreaker from 'opossum';

const breaker = new CircuitBreaker(
  async (userId: string) => {
    return fetch(`http://user-service/users/${userId}`);
  },
  {
    timeout: 3000,
    errorThresholdPercentage: 50,
    resetTimeout: 30000
  }
);

breaker.fallback(() => ({ id: 'unknown', name: 'Guest' }));

// Uso
const user = await breaker.fire(userId);
```

### Saga Pattern (Distributed Transactions)
```typescript
// Orchestration-based Saga
class OrderSaga {
  async execute(orderData: OrderData) {
    const steps = [];
    
    try {
      // Step 1: Reserve inventory
      const reservation = await inventoryService.reserve(orderData.items);
      steps.push({ service: 'inventory', action: 'reserve', data: reservation });
      
      // Step 2: Process payment
      const payment = await paymentService.charge(orderData.total);
      steps.push({ service: 'payment', action: 'charge', data: payment });
      
      // Step 3: Create order
      const order = await orderService.create(orderData);
      
      return order;
    } catch (error) {
      // Compensate in reverse order
      await this.compensate(steps.reverse());
      throw error;
    }
  }

  async compensate(steps: Step[]) {
    for (const step of steps) {
      await this.rollback(step);
    }
  }
}
```

---

## ✅ Benefits

| Benefit | Description |
|---------|-------------|
| **Independent Deployment** | Actualizar un servicio sin tocar otros |
| **Technology Diversity** | Cada servicio puede usar diferente stack |
| **Scalability** | Escalar solo los servicios que lo necesitan |
| **Fault Isolation** | Un fallo no tumba todo el sistema |
| **Team Autonomy** | Equipos pequeños dueños de sus servicios |

---

## ⚠️ Challenges

| Challenge | Mitigation |
|-----------|------------|
| **Network Complexity** | Service mesh (Istio, Linkerd) |
| **Data Consistency** | Eventual consistency, Sagas |
| **Distributed Debugging** | Distributed tracing (Jaeger) |
| **Operational Overhead** | Kubernetes, automation |
| **Testing** | Contract testing (Pact) |

---

## 🎯 When to Use

> [!tip] Good Fit
> - Equipos grandes (múltiples equipos)
> - Alta necesidad de escalabilidad
> - Partes del sistema con diferentes requisitos
> - Necesidad de deployment independiente

> [!warning] Avoid When
> - Equipos pequeños (< 5-10 devs)
> - Startup con dominio no definido
> - Sistema simple sin escala
> - Sin experiencia en sistemas distribuidos

---

## 📋 Summary

| Aspect | Details |
|--------|---------|
| **Type** | Distributed Architecture |
| **Key Idea** | Servicios pequeños, independientes |
| **Communication** | HTTP, gRPC, Message Queues |
| **Data** | Database per service |
| **Challenges** | Complejidad operacional |

---

← [[Programming/Software Engineering/Architectural Patterns/Architectural Patterns Complete Guide|Back to Architectural Patterns]]
