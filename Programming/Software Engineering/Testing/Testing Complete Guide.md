---
tags:
  - software-engineering
  - testing
  - dashboard
created: 2026-01-02
---
# 🧪 Testing

> *"Testing shows the presence, not the absence of bugs."* — Edsger Dijkstra

## 📚 Contents

| Topic | Description | Status |
|-------|-------------|--------|
| [[Programming/Software Engineering/Testing/TDD\|TDD]] | Test-Driven Development (Red-Green-Refactor) | 🔴 |
| [[Programming/Software Engineering/Testing/Unit Testing\|Unit Testing]] | Testing isolated units with AAA pattern | 🔴 |
| [[Programming/Software Engineering/Testing/Integration Testing\|Integration Testing]] | Testing components working together | 🔴 |
| [[Programming/Software Engineering/Testing/Mocking\|Mocking]] | Test doubles, stubs, spies, fakes | 🔴 |
| [[Programming/Software Engineering/Testing/E2E Testing\|E2E Testing]] | End-to-end user flows | 🔴 |

---

## 🔺 Testing Pyramid

```
                    /\
                   /  \
                  / E2E\         Slow, Expensive
                 /──────\        Few tests
                /        \
               / Integr.  \      Medium
              /────────────\     Some tests
             /              \
            /    Unit Tests  \   Fast, Cheap
           /──────────────────\  Many tests
```

---

## 🎯 Types of Tests

### Unit Tests
```typescript
// Testear una función/clase en aislamiento
describe('Money', () => {
  it('should add two amounts with same currency', () => {
    const money1 = new Money(100, 'USD');
    const money2 = new Money(50, 'USD');
    
    const result = money1.add(money2);
    
    expect(result.amount).toBe(150);
    expect(result.currency).toBe('USD');
  });

  it('should throw when adding different currencies', () => {
    const usd = new Money(100, 'USD');
    const eur = new Money(50, 'EUR');
    
    expect(() => usd.add(eur)).toThrow('Cannot add different currencies');
  });
});
```

### Integration Tests
```typescript
// Testear múltiples componentes juntos
describe('OrderService Integration', () => {
  let orderService: OrderService;
  let db: Database;

  beforeAll(async () => {
    db = await createTestDatabase();
    const repository = new PostgresOrderRepository(db);
    orderService = new OrderService(repository);
  });

  afterAll(async () => {
    await db.close();
  });

  it('should create and retrieve an order', async () => {
    const order = await orderService.createOrder({
      customerId: 'cust-1',
      items: [{ productId: 'prod-1', quantity: 2 }]
    });

    const retrieved = await orderService.getOrder(order.id);

    expect(retrieved.id).toBe(order.id);
    expect(retrieved.items).toHaveLength(1);
  });
});
```

### E2E Tests
```typescript
// Testear el sistema completo
describe('Order API E2E', () => {
  it('should complete checkout flow', async () => {
    // Login
    const loginResponse = await request(app)
      .post('/auth/login')
      .send({ email: 'user@test.com', password: 'password' });
    
    const token = loginResponse.body.token;

    // Add to cart
    await request(app)
      .post('/cart/items')
      .set('Authorization', `Bearer ${token}`)
      .send({ productId: 'prod-1', quantity: 1 })
      .expect(200);

    // Checkout
    const orderResponse = await request(app)
      .post('/orders')
      .set('Authorization', `Bearer ${token}`)
      .send({ paymentMethod: 'card' })
      .expect(201);

    expect(orderResponse.body.status).toBe('confirmed');
  });
});
```

---

## 🎭 Test Doubles

```typescript
// Stub - Retorna valores predefinidos
const stubRepository = {
  findById: jest.fn().mockResolvedValue(mockUser)
};

// Mock - Verifica interacciones
const mockEmailService = {
  send: jest.fn()
};
expect(mockEmailService.send).toHaveBeenCalledWith('user@email.com', expect.any(String));

// Spy - Observa llamadas reales
const spy = jest.spyOn(realService, 'method');

// Fake - Implementación simplificada
class FakeUserRepository implements UserRepository {
  private users: User[] = [];
  
  async save(user: User) { this.users.push(user); }
  async findById(id: string) { return this.users.find(u => u.id === id); }
}
```

---

## 📋 Best Practices

| Practice | Description |
|----------|-------------|
| **AAA Pattern** | Arrange, Act, Assert |
| **One assertion** | Un concepto por test |
| **Descriptive names** | Nombres que explican qué se testea |
| **Independent tests** | Tests no dependen de otros |
| **Fast tests** | Unit tests < 100ms |
| **Deterministic** | Mismo resultado siempre |

---

## 📖 Resources

- 📕 **Book**: "Test-Driven Development" - Kent Beck
- 📕 **Book**: "Unit Testing Principles" - Vladimir Khorikov

---

← [[Programming/Software Engineering/Software Engineering Roadmap|Back to Software Engineering]]
