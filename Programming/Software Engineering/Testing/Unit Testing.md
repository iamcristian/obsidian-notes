---
tags:
  - software-engineering
  - testing
  - unit-testing
created: 2026-01-02
status: 🔴
---
# 🔬 Unit Testing

> *"A unit test is an automated piece of code that invokes the unit of work being tested, and then checks some assumptions about a single end result."* — Roy Osherove

## 🎯 What is a Unit Test?

Un unit test verifica el comportamiento de una **unidad aislada** de código - típicamente una función, método o clase - en aislamiento de sus dependencias.

---

## 📋 The FIRST Principles

| Principle | Description |
|-----------|-------------|
| **F**ast | Ejecutar en milisegundos |
| **I**ndependent | No depender de otros tests |
| **R**epeatable | Mismo resultado siempre |
| **S**elf-validating | Pass/Fail automático |
| **T**imely | Escritos junto con el código |

---

## 🏗️ Anatomy of a Unit Test: AAA Pattern

```typescript
describe('Calculator', () => {
  it('should add two positive numbers correctly', () => {
    // ARRANGE - Setup del test
    const calculator = new Calculator();
    const a = 5;
    const b = 3;

    // ACT - Ejecutar la acción
    const result = calculator.add(a, b);

    // ASSERT - Verificar resultado
    expect(result).toBe(8);
  });
});
```

---

## 💻 Writing Good Unit Tests

### Test Behavior, Not Implementation

```typescript
// ❌ BAD - Testing implementation details
describe('UserService', () => {
  it('should call repository.save with user object', () => {
    const spy = jest.spyOn(repository, 'save');
    service.createUser({ name: 'John' });
    expect(spy).toHaveBeenCalledWith({ name: 'John', id: expect.any(String) });
  });
});

// ✅ GOOD - Testing behavior
describe('UserService', () => {
  it('should create user and make it retrievable', async () => {
    const userData = { name: 'John', email: 'john@email.com' };
    
    const createdUser = await service.createUser(userData);
    const retrievedUser = await service.getUser(createdUser.id);
    
    expect(retrievedUser.name).toBe('John');
    expect(retrievedUser.email).toBe('john@email.com');
  });
});
```

### One Concept Per Test

```typescript
// ❌ BAD - Multiple concepts
it('should validate user', () => {
  expect(User.validate({ email: '' })).toBe(false);
  expect(User.validate({ email: 'invalid' })).toBe(false);
  expect(User.validate({ email: 'valid@email.com' })).toBe(true);
  expect(User.validate({ email: 'valid@email.com', age: -1 })).toBe(false);
});

// ✅ GOOD - One concept per test
describe('User validation', () => {
  it('should reject empty email', () => {
    expect(User.validate({ email: '' })).toBe(false);
  });

  it('should reject invalid email format', () => {
    expect(User.validate({ email: 'invalid' })).toBe(false);
  });

  it('should accept valid email', () => {
    expect(User.validate({ email: 'valid@email.com' })).toBe(true);
  });

  it('should reject negative age', () => {
    expect(User.validate({ email: 'valid@email.com', age: -1 })).toBe(false);
  });
});
```

### Descriptive Test Names

```typescript
// ❌ BAD - Vague names
it('should work', () => {});
it('test1', () => {});
it('handles error', () => {});

// ✅ GOOD - Descriptive names (Given-When-Then style)
it('should return null when user is not found', () => {});
it('should throw ValidationError when email is invalid', () => {});
it('should apply 20% discount when cart total exceeds $100', () => {});

// ✅ GOOD - BDD style
describe('when user has premium membership', () => {
  it('applies free shipping', () => {});
  it('gives access to exclusive products', () => {});
});
```

---

## 🎭 Test Doubles Deep Dive

### Stubs - Control Indirect Inputs

```typescript
// Stub para controlar qué retorna una dependencia
describe('WeatherService', () => {
  it('should recommend umbrella when rain probability > 50%', async () => {
    // Stub que siempre retorna 70% probabilidad de lluvia
    const weatherApiStub = {
      getRainProbability: jest.fn().mockResolvedValue(70)
    };
    const service = new WeatherService(weatherApiStub);

    const recommendation = await service.getRecommendation();

    expect(recommendation).toContain('umbrella');
  });
});
```

### Mocks - Verify Indirect Outputs

```typescript
// Mock para verificar que se llamó correctamente
describe('OrderService', () => {
  it('should send confirmation email after order is placed', async () => {
    const emailServiceMock = {
      sendOrderConfirmation: jest.fn().mockResolvedValue(undefined)
    };
    const service = new OrderService(repo, emailServiceMock);
    const order = { customerId: '1', items: [{ productId: 'A', qty: 1 }] };

    await service.placeOrder(order);

    expect(emailServiceMock.sendOrderConfirmation).toHaveBeenCalledWith(
      expect.objectContaining({
        customerId: '1',
        status: 'confirmed'
      })
    );
  });
});
```

### Fakes - Simplified Implementations

```typescript
// Fake repository con implementación en memoria
class FakeUserRepository implements UserRepository {
  private users = new Map<string, User>();

  async save(user: User): Promise<void> {
    this.users.set(user.id, { ...user });
  }

  async findById(id: string): Promise<User | null> {
    return this.users.get(id) ?? null;
  }

  async findByEmail(email: string): Promise<User | null> {
    return Array.from(this.users.values())
      .find(u => u.email === email) ?? null;
  }

  // Helper para tests
  clear(): void {
    this.users.clear();
  }
}

describe('UserService', () => {
  let fakeRepo: FakeUserRepository;
  let service: UserService;

  beforeEach(() => {
    fakeRepo = new FakeUserRepository();
    service = new UserService(fakeRepo);
  });

  it('should not allow duplicate emails', async () => {
    await service.createUser({ name: 'John', email: 'john@email.com' });

    await expect(
      service.createUser({ name: 'Jane', email: 'john@email.com' })
    ).rejects.toThrow('Email already exists');
  });
});
```

---

## 📊 Testing Edge Cases

```typescript
describe('divide', () => {
  // Happy path
  it('should divide two numbers correctly', () => {
    expect(divide(10, 2)).toBe(5);
  });

  // Edge cases
  it('should handle division resulting in decimal', () => {
    expect(divide(10, 3)).toBeCloseTo(3.333, 2);
  });

  it('should throw when dividing by zero', () => {
    expect(() => divide(10, 0)).toThrow('Division by zero');
  });

  it('should handle negative numbers', () => {
    expect(divide(-10, 2)).toBe(-5);
    expect(divide(10, -2)).toBe(-5);
    expect(divide(-10, -2)).toBe(5);
  });

  it('should handle very large numbers', () => {
    expect(divide(Number.MAX_SAFE_INTEGER, 1)).toBe(Number.MAX_SAFE_INTEGER);
  });

  it('should return 0 when numerator is 0', () => {
    expect(divide(0, 5)).toBe(0);
  });
});
```

---

## 🧪 Testing Async Code

```typescript
// Promise-based
it('should fetch user data', async () => {
  const user = await userService.fetchUser('123');
  expect(user.name).toBe('John');
});

// Callback-based (legacy)
it('should process callback', (done) => {
  legacyService.process((error, result) => {
    expect(error).toBeNull();
    expect(result).toBe('processed');
    done();
  });
});

// Testing rejected promises
it('should reject for invalid user', async () => {
  await expect(userService.fetchUser('invalid'))
    .rejects
    .toThrow('User not found');
});

// With fake timers
it('should retry after delay', async () => {
  jest.useFakeTimers();
  
  const promise = service.retryableOperation();
  
  jest.advanceTimersByTime(1000);
  await promise;
  
  expect(mockApi.call).toHaveBeenCalledTimes(2);
  
  jest.useRealTimers();
});
```

---

## 📁 Test File Organization

```
src/
├── services/
│   ├── user.service.ts
│   └── user.service.test.ts      # Co-located
├── utils/
│   └── validators.ts
└── __tests__/                     # Or separate folder
    └── utils/
        └── validators.test.ts

# Naming conventions
user.service.test.ts    # Jest default
user.service.spec.ts    # Angular/Jasmine style
test_user_service.py    # Python style
```

---

## 📋 Test Quality Checklist

> [!check] Good Unit Test Checklist
> - [ ] Rápido (< 100ms)
> - [ ] Aislado de otras tests
> - [ ] Determinístico (sin random, sin time)
> - [ ] Testea comportamiento, no implementación
> - [ ] Un solo concepto por test
> - [ ] Nombre descriptivo
> - [ ] Sin lógica condicional en el test
> - [ ] Arrange-Act-Assert claro

---

## 🛠️ Common Testing Matchers (Jest)

```typescript
// Equality
expect(value).toBe(2);                    // ===
expect(value).toEqual({ a: 1 });          // Deep equality
expect(value).toStrictEqual({ a: 1 });    // Strict (no undefined)

// Truthiness
expect(value).toBeTruthy();
expect(value).toBeFalsy();
expect(value).toBeNull();
expect(value).toBeUndefined();
expect(value).toBeDefined();

// Numbers
expect(value).toBeGreaterThan(3);
expect(value).toBeGreaterThanOrEqual(3);
expect(value).toBeLessThan(5);
expect(value).toBeCloseTo(0.3, 5);        // Floating point

// Strings
expect(string).toMatch(/pattern/);
expect(string).toContain('substring');

// Arrays
expect(array).toContain(item);
expect(array).toHaveLength(3);
expect(array).toContainEqual({ a: 1 });

// Objects
expect(object).toHaveProperty('key');
expect(object).toHaveProperty('key', 'value');
expect(object).toMatchObject({ partial: 'match' });

// Exceptions
expect(() => fn()).toThrow();
expect(() => fn()).toThrow('message');
expect(() => fn()).toThrow(ErrorClass);

// Async
await expect(promise).resolves.toBe(value);
await expect(promise).rejects.toThrow();
```

---

← [[Programming/Software Engineering/Testing/Testing Complete Guide|Back to Testing]]
