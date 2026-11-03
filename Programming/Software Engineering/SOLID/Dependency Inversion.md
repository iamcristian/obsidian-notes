---
tags:
  - software-engineering
  - solid
  - dependency-inversion
created: 2026-01-02
status: 🔴
---
# 🔄 Dependency Inversion Principle (DIP)

> *"High-level modules should not depend on low-level modules. Both should depend on abstractions."* — Robert C. Martin

## 🎯 The Principle

1. **High-level modules** no deben depender de **low-level modules**
2. Ambos deben depender de **abstracciones**
3. **Abstracciones** no deben depender de **detalles**
4. **Detalles** deben depender de **abstracciones**

---

## 📊 Visual Representation

### ❌ Without DIP
```
┌─────────────────┐
│   OrderService  │  (High-level)
└────────┬────────┘
         │ depends on
         ▼
┌─────────────────┐
│ MySQLRepository │  (Low-level)
└─────────────────┘

High-level conoce y depende del low-level
```

### ✅ With DIP
```
┌─────────────────┐        ┌─────────────────────┐
│   OrderService  │───────→│  <<interface>>      │
└─────────────────┘        │  OrderRepository    │
                           └──────────△──────────┘
                                      │
                           ┌──────────┴──────────┐
                           │  MySQLOrderRepo     │
                           └─────────────────────┘

Ambos dependen de la abstracción
```

---

## ❌ Bad Example - Direct Dependency

```typescript
// Low-level module (detalle de implementación)
class MySQLDatabase {
  connect() {
    console.log('Connecting to MySQL...');
  }

  query(sql: string): any[] {
    console.log(`Executing: ${sql}`);
    return [];
  }
}

// High-level module depende directamente del low-level
class UserService {
  private database: MySQLDatabase;

  constructor() {
    this.database = new MySQLDatabase(); // ❌ Acoplamiento fuerte
  }

  getUsers(): User[] {
    this.database.connect();
    return this.database.query('SELECT * FROM users');
  }
}

// Problemas:
// 1. No puedes cambiar MySQL por PostgreSQL sin modificar UserService
// 2. No puedes testear UserService sin una DB real
// 3. UserService conoce detalles de implementación (SQL)
```

---

## ✅ Good Example - Dependency Inversion

```typescript
// Abstracción (interface) - definida por el high-level module
interface UserRepository {
  findAll(): Promise<User[]>;
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<void>;
  delete(id: string): Promise<void>;
}

// High-level module depende de la abstracción
class UserService {
  // Depende de la interfaz, no de la implementación
  constructor(private userRepository: UserRepository) {}

  async getUsers(): Promise<User[]> {
    return this.userRepository.findAll();
  }

  async getUser(id: string): Promise<User> {
    const user = await this.userRepository.findById(id);
    if (!user) throw new NotFoundError('User', id);
    return user;
  }

  async createUser(data: CreateUserDTO): Promise<User> {
    const user = new User(data);
    await this.userRepository.save(user);
    return user;
  }
}

// Low-level module implementa la abstracción
class MySQLUserRepository implements UserRepository {
  constructor(private connection: MySQLConnection) {}

  async findAll(): Promise<User[]> {
    const rows = await this.connection.query('SELECT * FROM users');
    return rows.map(this.mapToUser);
  }

  async findById(id: string): Promise<User | null> {
    const rows = await this.connection.query(
      'SELECT * FROM users WHERE id = ?',
      [id]
    );
    return rows[0] ? this.mapToUser(rows[0]) : null;
  }

  async save(user: User): Promise<void> {
    await this.connection.query(
      'INSERT INTO users (id, name, email) VALUES (?, ?, ?)',
      [user.id, user.name, user.email]
    );
  }

  async delete(id: string): Promise<void> {
    await this.connection.query('DELETE FROM users WHERE id = ?', [id]);
  }

  private mapToUser(row: any): User {
    return new User(row.id, row.name, row.email);
  }
}

// Otra implementación - fácil de intercambiar
class MongoUserRepository implements UserRepository {
  constructor(private db: MongoDatabase) {}

  async findAll(): Promise<User[]> {
    return this.db.collection('users').find().toArray();
  }

  async findById(id: string): Promise<User | null> {
    return this.db.collection('users').findOne({ id });
  }

  // ... otros métodos
}

// Mock para testing
class InMemoryUserRepository implements UserRepository {
  private users: User[] = [];

  async findAll(): Promise<User[]> {
    return [...this.users];
  }

  async findById(id: string): Promise<User | null> {
    return this.users.find(u => u.id === id) || null;
  }

  async save(user: User): Promise<void> {
    this.users.push(user);
  }

  async delete(id: string): Promise<void> {
    this.users = this.users.filter(u => u.id !== id);
  }
}
```

---

## 🔌 Dependency Injection

DIP se implementa típicamente con **Dependency Injection**:

### Constructor Injection (Recomendado)
```typescript
class OrderService {
  constructor(
    private orderRepository: OrderRepository,
    private paymentGateway: PaymentGateway,
    private notificationService: NotificationService
  ) {}
}
```

### Method Injection
```typescript
class ReportGenerator {
  generate(data: Data, formatter: ReportFormatter): Report {
    return formatter.format(data);
  }
}
```

### Property Injection (Menos recomendado)
```typescript
class Service {
  logger?: Logger;

  doSomething() {
    this.logger?.log('Doing something');
  }
}
```

---

## 🏭 DI Container Example

```typescript
// Configuración del container
class DIContainer {
  private services = new Map<string, any>();

  register<T>(token: string, factory: () => T): void {
    this.services.set(token, factory);
  }

  resolve<T>(token: string): T {
    const factory = this.services.get(token);
    if (!factory) throw new Error(`Service ${token} not found`);
    return factory();
  }
}

// Setup
const container = new DIContainer();

// Registrar implementaciones
container.register('UserRepository', () => 
  new MySQLUserRepository(container.resolve('Database'))
);

container.register('UserService', () =>
  new UserService(container.resolve('UserRepository'))
);

// Uso
const userService = container.resolve<UserService>('UserService');
```

### Con InversifyJS (Biblioteca popular)
```typescript
import { Container, injectable, inject } from 'inversify';

@injectable()
class UserService {
  constructor(
    @inject('UserRepository') private userRepository: UserRepository
  ) {}
}

const container = new Container();
container.bind<UserRepository>('UserRepository').to(MySQLUserRepository);
container.bind<UserService>(UserService).toSelf();

const userService = container.get(UserService);
```

---

## 🧪 Testing Benefits

```typescript
// Sin DIP - difícil de testear
class UserService {
  private db = new MySQLDatabase(); // No puedes mockear
  
  getUsers() {
    return this.db.query('SELECT * FROM users'); // Necesita DB real
  }
}

// Con DIP - fácil de testear
describe('UserService', () => {
  it('should return all users', async () => {
    // Arrange
    const mockRepository = new InMemoryUserRepository();
    mockRepository.save(new User('1', 'John', 'john@email.com'));
    
    const service = new UserService(mockRepository);
    
    // Act
    const users = await service.getUsers();
    
    // Assert
    expect(users).toHaveLength(1);
    expect(users[0].name).toBe('John');
  });
});
```

---

## 📋 Summary

| Aspect | Without DIP | With DIP |
|--------|-------------|----------|
| **Coupling** | Alto - conoce detalles | Bajo - solo interfaces |
| **Testing** | Necesita infra real | Mocks fáciles |
| **Flexibility** | Difícil cambiar impl | Fácil intercambiar |
| **Reusability** | Bajo | Alto |

---

## 💡 Tips

> [!tip] Best Practices
> - Las interfaces las define el **consumidor** (high-level), no el proveedor
> - Usa Dependency Injection, preferentemente constructor injection
> - Evita `new` dentro de clases de negocio
> - Las abstracciones deben ser **estables** (cambiar poco)

---

← [[Programming/Software Engineering/SOLID/SOLID Principles Complete Guide|Back to SOLID]]
