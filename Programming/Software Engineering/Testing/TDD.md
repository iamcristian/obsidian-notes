---
tags:
  - software-engineering
  - testing
  - tdd
created: 2026-01-02
status: 🔴
---
# 🔴🟢🔵 Test-Driven Development (TDD)

> *"TDD is not about testing. TDD is about design."* — Kent Beck

## 🎯 What is TDD?

Test-Driven Development es una disciplina de desarrollo donde escribes los **tests ANTES** del código de producción. Es un ciclo de feedback rápido que guía el diseño del software.

---

## 🔄 The TDD Cycle: Red-Green-Refactor

```
    ┌─────────────────────────────────────────────┐
    │                                             │
    │         🔴 RED                              │
    │         Write a failing test                │
    │                │                            │
    │                ▼                            │
    │         🟢 GREEN                            │
    │         Write minimum code to pass          │
    │                │                            │
    │                ▼                            │
    │         🔵 REFACTOR                         │
    │         Clean up the code                   │
    │                │                            │
    │                └──────────────────┐         │
    │                                   │         │
    └───────────────────────────────────┘         │
                                        │         │
                    ◄───────────────────┘         │
                                                  │
              Repeat until feature complete       │
```

---

## 📋 The Three Laws of TDD

> [!important] Uncle Bob's Three Laws
> 1. **No escribir código de producción** excepto para hacer pasar un test que falla
> 2. **No escribir más de un test** que falle a la vez
> 3. **No escribir más código de producción** del necesario para pasar el test actual

---

## 💻 TDD en Práctica: Ejemplo Completo

### Escenario: Construir una Calculadora de Precios

#### Iteration 1: 🔴 RED - Primer Test
```typescript
// price-calculator.test.ts
describe('PriceCalculator', () => {
  it('should return 0 for empty cart', () => {
    const calculator = new PriceCalculator();
    const cart: CartItem[] = [];
    
    const total = calculator.calculateTotal(cart);
    
    expect(total).toBe(0);
  });
});

// Este test FALLA porque PriceCalculator no existe
```

#### Iteration 1: 🟢 GREEN - Código Mínimo
```typescript
// price-calculator.ts
interface CartItem {
  price: number;
  quantity: number;
}

class PriceCalculator {
  calculateTotal(cart: CartItem[]): number {
    return 0; // Mínimo para pasar el test
  }
}
```

#### Iteration 2: 🔴 RED - Segundo Test
```typescript
it('should calculate total for single item', () => {
  const calculator = new PriceCalculator();
  const cart: CartItem[] = [{ price: 100, quantity: 1 }];
  
  const total = calculator.calculateTotal(cart);
  
  expect(total).toBe(100);
});
// FALLA: expected 100, received 0
```

#### Iteration 2: 🟢 GREEN
```typescript
calculateTotal(cart: CartItem[]): number {
  return cart.reduce((sum, item) => sum + item.price * item.quantity, 0);
}
```

#### Iteration 3: 🔴 RED - Añadir Descuentos
```typescript
it('should apply 10% discount for orders over 500', () => {
  const calculator = new PriceCalculator();
  const cart: CartItem[] = [{ price: 600, quantity: 1 }];
  
  const total = calculator.calculateTotal(cart);
  
  expect(total).toBe(540); // 600 - 10%
});
```

#### Iteration 3: 🟢 GREEN
```typescript
calculateTotal(cart: CartItem[]): number {
  const subtotal = cart.reduce((sum, item) => sum + item.price * item.quantity, 0);
  
  if (subtotal > 500) {
    return subtotal * 0.9;
  }
  return subtotal;
}
```

#### 🔵 REFACTOR
```typescript
class PriceCalculator {
  private readonly DISCOUNT_THRESHOLD = 500;
  private readonly DISCOUNT_RATE = 0.1;

  calculateTotal(cart: CartItem[]): number {
    const subtotal = this.calculateSubtotal(cart);
    const discount = this.calculateDiscount(subtotal);
    return subtotal - discount;
  }

  private calculateSubtotal(cart: CartItem[]): number {
    return cart.reduce((sum, item) => sum + item.price * item.quantity, 0);
  }

  private calculateDiscount(subtotal: number): number {
    if (subtotal > this.DISCOUNT_THRESHOLD) {
      return subtotal * this.DISCOUNT_RATE;
    }
    return 0;
  }
}
```

---

## 🎯 TDD Patterns

### Triangulation
Usar múltiples ejemplos para llegar a una generalización:

```typescript
// Test 1
it('should return 2 for 1 + 1', () => {
  expect(add(1, 1)).toBe(2);
});

// Implementación fake que pasa
function add(a: number, b: number): number {
  return 2; // Fake!
}

// Test 2 - Triangula hacia la solución real
it('should return 5 for 2 + 3', () => {
  expect(add(2, 3)).toBe(5);
});

// Ahora necesitas la implementación real
function add(a: number, b: number): number {
  return a + b;
}
```

### Fake It Till You Make It
```typescript
// Empezar con valor hardcodeado
function isLeapYear(year: number): boolean {
  return true; // Fake
}

// Iterar hasta la solución real
function isLeapYear(year: number): boolean {
  return year % 4 === 0 && (year % 100 !== 0 || year % 400 === 0);
}
```

### Obvious Implementation
Cuando la solución es obvia, escríbela directamente:

```typescript
it('should concatenate strings', () => {
  expect(concat('Hello', 'World')).toBe('HelloWorld');
});

// Obvio - no necesitas fake
function concat(a: string, b: string): string {
  return a + b;
}
```

---

## 🏗️ Outside-In vs Inside-Out TDD

### Outside-In (London School / Mockist)
```
                    Start here
                        │
                        ▼
┌─────────────────────────────────────────┐
│             Controller                   │
│          (Mock Service)                  │
└─────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────┐
│              Service                     │
│          (Mock Repository)               │
└─────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────┐
│             Repository                   │
└─────────────────────────────────────────┘
```

```typescript
// Outside-In: Empezamos por el controller con mocks
describe('UserController', () => {
  it('should return user when found', async () => {
    // Arrange
    const mockUserService = {
      findById: jest.fn().mockResolvedValue({ id: '1', name: 'John' })
    };
    const controller = new UserController(mockUserService);
    
    // Act
    const result = await controller.getUser('1');
    
    // Assert
    expect(result).toEqual({ id: '1', name: 'John' });
    expect(mockUserService.findById).toHaveBeenCalledWith('1');
  });
});
```

### Inside-Out (Detroit School / Classicist)
```
┌─────────────────────────────────────────┐
│             Domain/Entity                │
│          Start here                      │
└─────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────┐
│              Service                     │
│         (Uses real entities)             │
└─────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────┐
│             Controller                   │
└─────────────────────────────────────────┘
```

```typescript
// Inside-Out: Empezamos por las entidades de dominio
describe('User', () => {
  it('should create user with valid email', () => {
    const user = User.create('John', 'john@email.com');
    
    expect(user.name).toBe('John');
    expect(user.email).toBe('john@email.com');
  });

  it('should throw for invalid email', () => {
    expect(() => User.create('John', 'invalid')).toThrow('Invalid email');
  });
});
```

---

## 📊 Test Doubles en TDD

```typescript
// DUMMY - Objeto que se pasa pero nunca se usa
const dummyLogger = {} as Logger;
new Service(realRepo, dummyLogger);

// STUB - Retorna valores predefinidos
const stubRepo = {
  findById: () => ({ id: '1', name: 'Test' })
};

// SPY - Registra información sobre cómo fue llamado
const spy = jest.spyOn(service, 'save');
// Después: expect(spy).toHaveBeenCalledWith(expectedData);

// MOCK - Stub con verificación de expectativas
const mockEmailService = {
  send: jest.fn()
};
expect(mockEmailService.send).toHaveBeenCalledTimes(1);

// FAKE - Implementación funcional simplificada
class FakeUserRepository implements UserRepository {
  private users: User[] = [];
  
  async save(user: User) { 
    this.users.push(user); 
  }
  
  async findById(id: string) { 
    return this.users.find(u => u.id === id) || null; 
  }
}
```

---

## ⚠️ TDD Anti-Patterns

### ❌ Testing Implementation Details
```typescript
// BAD - Test acoplado a implementación
it('should call private method _validateEmail', () => {
  const spy = jest.spyOn(user, '_validateEmail');
  user.setEmail('test@test.com');
  expect(spy).toHaveBeenCalled();
});

// GOOD - Test del comportamiento
it('should reject invalid email', () => {
  expect(() => user.setEmail('invalid')).toThrow();
});
```

### ❌ Too Many Mocks
```typescript
// BAD - Mock de todo = test frágil
const mockA = jest.fn();
const mockB = jest.fn();
const mockC = jest.fn();
const mockD = jest.fn();
// Si refactorizas, todos los tests fallan

// GOOD - Usar fakes o tests de integración
const fakeRepo = new InMemoryRepository();
const service = new Service(fakeRepo);
```

### ❌ Test per Method
```typescript
// BAD - Un test por método
it('should call setName', () => {});
it('should call setEmail', () => {});
it('should call save', () => {});

// GOOD - Test por comportamiento/feature
it('should create a new user with validated data', () => {});
it('should prevent duplicate emails', () => {});
```

---

## 📋 TDD Checklist

> [!check] Before Writing Test
> - [ ] ¿Entiendo el requisito claramente?
> - [ ] ¿Estoy testeando comportamiento, no implementación?
> - [ ] ¿El test es simple y enfocado?

> [!check] After Test Passes
> - [ ] ¿El código es el mínimo necesario?
> - [ ] ¿Hay algo que refactorizar?
> - [ ] ¿Los nombres son claros?
> - [ ] ¿Hay duplicación que eliminar?

---

## 🎯 Benefits of TDD

| Benefit | Description |
|---------|-------------|
| **Design Feedback** | Tests difíciles = diseño problemático |
| **Documentation** | Tests son documentación ejecutable |
| **Confidence** | Refactoring sin miedo |
| **Focus** | Un problema a la vez |
| **Regression Safety** | Detectar bugs inmediatamente |

---

## 💡 Tips para Empezar

> [!tip] Getting Started
> 1. Empieza con el caso más simple posible
> 2. Escribe el test que desearías que existiera
> 3. No optimices prematuramente - primero green
> 4. Refactoriza agresivamente después de green
> 5. Si un test es difícil de escribir, el diseño puede mejorar

---

← [[Programming/Software Engineering/Testing/_Index|Back to Testing]]
