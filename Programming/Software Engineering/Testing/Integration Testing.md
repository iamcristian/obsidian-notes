---
tags:
  - software-engineering
  - testing
  - integration-testing
created: 2026-01-02
status: 🔴
---
# 🔗 Integration Testing

> *"Integration tests strike a balance between unit tests and e2e tests, verifying that several units work together correctly."*

## 🎯 What is Integration Testing?

Integration testing verifica que múltiples componentes funcionan correctamente **juntos**. A diferencia de unit tests que aíslan completamente, integration tests prueban la colaboración entre unidades.

---

## 📊 Testing Spectrum

```
        Isolation                              Reality
            │                                     │
            ▼                                     ▼
┌───────────────┬─────────────────────┬──────────────────┐
│  Unit Tests   │ Integration Tests   │    E2E Tests     │
├───────────────┼─────────────────────┼──────────────────┤
│ • Fast        │ • Medium speed      │ • Slow           │
│ • Isolated    │ • Some real deps    │ • Real system    │
│ • Many        │ • Moderate amount   │ • Few            │
│ • Mocked deps │ • Real DB, APIs     │ • Real browser   │
└───────────────┴─────────────────────┴──────────────────┘
```

---

## 🔄 Types of Integration Tests

### 1. Component Integration
```typescript
// Testea service + repository juntos
describe('UserService Integration', () => {
  let db: Database;
  let userService: UserService;

  beforeAll(async () => {
    db = await createTestDatabase();
    const repository = new UserRepository(db);
    userService = new UserService(repository);
  });

  afterAll(async () => {
    await db.close();
  });

  beforeEach(async () => {
    await db.query('DELETE FROM users');
  });

  it('should persist and retrieve user', async () => {
    const created = await userService.createUser({
      name: 'John',
      email: 'john@email.com'
    });

    const retrieved = await userService.findById(created.id);

    expect(retrieved).toEqual(created);
  });

  it('should enforce unique email constraint', async () => {
    await userService.createUser({ name: 'John', email: 'test@email.com' });

    await expect(
      userService.createUser({ name: 'Jane', email: 'test@email.com' })
    ).rejects.toThrow('Email already exists');
  });
});
```

### 2. API Integration
```typescript
// Testea HTTP endpoints con DB real
describe('User API Integration', () => {
  let app: Express;
  let db: Database;

  beforeAll(async () => {
    db = await createTestDatabase();
    app = createApp(db);
  });

  afterAll(async () => {
    await db.close();
  });

  describe('POST /api/users', () => {
    it('should create user and return 201', async () => {
      const response = await request(app)
        .post('/api/users')
        .send({ name: 'John', email: 'john@email.com' })
        .expect(201);

      expect(response.body).toMatchObject({
        id: expect.any(String),
        name: 'John',
        email: 'john@email.com'
      });

      // Verify persistence
      const dbUser = await db.query('SELECT * FROM users WHERE id = $1', [response.body.id]);
      expect(dbUser.rows[0].name).toBe('John');
    });

    it('should return 400 for invalid data', async () => {
      const response = await request(app)
        .post('/api/users')
        .send({ name: '' })
        .expect(400);

      expect(response.body.error).toBeDefined();
    });
  });

  describe('GET /api/users/:id', () => {
    it('should return 404 for non-existent user', async () => {
      await request(app)
        .get('/api/users/non-existent-id')
        .expect(404);
    });
  });
});
```

### 3. External Service Integration
```typescript
// Test con servicios externos reales (sandbox/test environment)
describe('Payment Gateway Integration', () => {
  let paymentService: PaymentService;

  beforeAll(() => {
    // Usar sandbox/test API keys
    paymentService = new PaymentService({
      apiKey: process.env.STRIPE_TEST_KEY,
      environment: 'sandbox'
    });
  });

  it('should process test payment successfully', async () => {
    const result = await paymentService.charge({
      amount: 1000, // $10.00
      currency: 'usd',
      cardToken: 'tok_visa' // Stripe test token
    });

    expect(result.status).toBe('succeeded');
    expect(result.amount).toBe(1000);
  });

  it('should handle declined card', async () => {
    const result = await paymentService.charge({
      amount: 1000,
      currency: 'usd',
      cardToken: 'tok_chargeDeclined' // Stripe test token for decline
    });

    expect(result.status).toBe('failed');
    expect(result.error).toContain('declined');
  });
});
```

---

## 🗄️ Database Testing Strategies

### Test Database Setup
```typescript
// jest.setup.ts
import { Pool } from 'pg';

let pool: Pool;

beforeAll(async () => {
  pool = new Pool({
    connectionString: process.env.TEST_DATABASE_URL
  });
  
  // Run migrations
  await runMigrations(pool);
});

afterAll(async () => {
  await pool.end();
});

// Clean between tests
beforeEach(async () => {
  await pool.query('BEGIN');
});

afterEach(async () => {
  await pool.query('ROLLBACK');
});
```

### Using Transactions for Isolation
```typescript
describe('OrderService', () => {
  it('should rollback on payment failure', async () => {
    const initialStock = await getStock(productId);
    
    // Payment will fail
    await expect(
      orderService.placeOrder({
        productId,
        quantity: 1,
        paymentToken: 'invalid'
      })
    ).rejects.toThrow('Payment failed');

    // Stock should be unchanged due to rollback
    const finalStock = await getStock(productId);
    expect(finalStock).toBe(initialStock);
  });
});
```

### Using Test Containers
```typescript
// Con Testcontainers para DB efímera
import { PostgreSqlContainer } from '@testcontainers/postgresql';

describe('Integration with Testcontainers', () => {
  let container: StartedPostgreSqlContainer;
  let db: Pool;

  beforeAll(async () => {
    container = await new PostgreSqlContainer()
      .withDatabase('testdb')
      .start();

    db = new Pool({
      connectionString: container.getConnectionUri()
    });

    await runMigrations(db);
  }, 60000); // Timeout más alto para container startup

  afterAll(async () => {
    await db.end();
    await container.stop();
  });

  // Tests usan db real pero aislada
});
```

---

## 🌐 API Testing Patterns

### Authentication in Tests
```typescript
describe('Protected Routes', () => {
  let authToken: string;

  beforeAll(async () => {
    // Create test user and get token
    const user = await createTestUser();
    const response = await request(app)
      .post('/api/auth/login')
      .send({ email: user.email, password: 'testpass' });
    
    authToken = response.body.token;
  });

  it('should access protected route with token', async () => {
    await request(app)
      .get('/api/profile')
      .set('Authorization', `Bearer ${authToken}`)
      .expect(200);
  });

  it('should reject without token', async () => {
    await request(app)
      .get('/api/profile')
      .expect(401);
  });
});
```

### Testing File Uploads
```typescript
it('should upload profile picture', async () => {
  const response = await request(app)
    .post('/api/users/1/avatar')
    .set('Authorization', `Bearer ${token}`)
    .attach('avatar', 'test/fixtures/test-image.jpg')
    .expect(200);

  expect(response.body.avatarUrl).toMatch(/\.jpg$/);
});
```

### Testing Webhooks
```typescript
describe('Stripe Webhook', () => {
  it('should handle payment_intent.succeeded', async () => {
    const payload = {
      type: 'payment_intent.succeeded',
      data: {
        object: {
          id: 'pi_123',
          amount: 1000,
          metadata: { orderId: 'order_123' }
        }
      }
    };

    const signature = createStripeSignature(payload);

    await request(app)
      .post('/webhooks/stripe')
      .set('stripe-signature', signature)
      .send(payload)
      .expect(200);

    // Verify order was updated
    const order = await db.query('SELECT * FROM orders WHERE id = $1', ['order_123']);
    expect(order.rows[0].status).toBe('paid');
  });
});
```

---

## ⚡ Performance Considerations

```typescript
// Parallel test execution (be careful with shared state)
describe.concurrent('Independent tests', () => {
  it.concurrent('test 1', async () => { });
  it.concurrent('test 2', async () => { });
});

// Reuse expensive setup
describe('Suite with expensive setup', () => {
  // Setup una vez, no por cada test
  beforeAll(async () => {
    await seedLargeDataset();
  });

  // Solo limpiar lo que el test modificó
  afterEach(async () => {
    await db.query('DELETE FROM orders WHERE test_marker = $1', [testId]);
  });
});
```

---

## 📋 Integration Test Checklist

> [!check] Before Writing
> - [ ] ¿Qué componentes necesitan testearse juntos?
> - [ ] ¿Qué dependencias serán reales vs mocked?
> - [ ] ¿Cómo se aislará el estado entre tests?

> [!check] Test Quality
> - [ ] Test es determinístico (sin flakiness)
> - [ ] Limpia su propio estado
> - [ ] No depende del orden de ejecución
> - [ ] Tiene timeout apropiado
> - [ ] Errores son descriptivos

---

← [[Programming/Software Engineering/Testing/_Index|Back to Testing]]
