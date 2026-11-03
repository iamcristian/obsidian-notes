---
tags:
  - software-engineering
  - design-patterns
  - structural
created: 2026-01-02
status: 🔴
category: Structural
---
# 🔌 Adapter Pattern

> *"Convert the interface of a class into another interface clients expect."*

## 🎯 Intent

- Hacer que interfaces **incompatibles** trabajen juntas
- Wrappear una clase existente con una nueva interfaz
- Permitir que clases con interfaces diferentes colaboren

---

## 📊 Structure

```
┌─────────────┐         ┌─────────────────┐
│   Client    │────────→│  <<interface>>  │
└─────────────┘         │     Target      │
                        ├─────────────────┤
                        │ + request()     │
                        └─────────────────┘
                                △
                                │
                        ┌───────┴───────┐
                        │    Adapter    │
                        ├───────────────┤
                        │ - adaptee     │────────┐
                        │ + request()   │        │
                        └───────────────┘        │
                                                 │
                                         ┌───────▼───────┐
                                         │   Adaptee     │
                                         ├───────────────┤
                                         │ + specificReq()│
                                         └───────────────┘
```

---

## 💻 Implementation

### Problem: Incompatible Interfaces
```typescript
// Sistema existente - interfaz vieja
class OldPaymentProcessor {
  processPayment(amount: number, cardNumber: string): boolean {
    console.log(`Processing ${amount} with card ${cardNumber}`);
    return true;
  }
}

// Nueva interfaz que necesitamos usar
interface PaymentGateway {
  pay(paymentDetails: PaymentDetails): PaymentResult;
}

interface PaymentDetails {
  amount: number;
  currency: string;
  paymentMethod: {
    type: string;
    cardNumber?: string;
    expiryDate?: string;
  };
}

interface PaymentResult {
  success: boolean;
  transactionId: string;
}
```

### Solution: Adapter
```typescript
// Adapter que hace compatible el viejo sistema con la nueva interfaz
class OldPaymentAdapter implements PaymentGateway {
  private oldProcessor: OldPaymentProcessor;

  constructor(oldProcessor: OldPaymentProcessor) {
    this.oldProcessor = oldProcessor;
  }

  pay(paymentDetails: PaymentDetails): PaymentResult {
    // Adaptar la llamada nueva a la vieja interfaz
    const success = this.oldProcessor.processPayment(
      paymentDetails.amount,
      paymentDetails.paymentMethod.cardNumber || ''
    );

    return {
      success,
      transactionId: success ? `TXN-${Date.now()}` : ''
    };
  }
}

// Usage
const oldProcessor = new OldPaymentProcessor();
const adapter: PaymentGateway = new OldPaymentAdapter(oldProcessor);

const result = adapter.pay({
  amount: 100,
  currency: 'USD',
  paymentMethod: {
    type: 'card',
    cardNumber: '4111111111111111',
    expiryDate: '12/25'
  }
});
```

### Real Example: Third-Party API Adapter
```typescript
// Interfaz que nuestra app espera
interface Logger {
  info(message: string, meta?: object): void;
  error(message: string, error?: Error): void;
  warn(message: string): void;
}

// Third-party library (Winston, Pino, etc.)
class WinstonLogger {
  log(level: string, message: string, metadata?: object) {
    // Winston's actual implementation
  }
}

// Adapter
class WinstonAdapter implements Logger {
  private winston: WinstonLogger;

  constructor(winstonInstance: WinstonLogger) {
    this.winston = winstonInstance;
  }

  info(message: string, meta?: object): void {
    this.winston.log('info', message, meta);
  }

  error(message: string, error?: Error): void {
    this.winston.log('error', message, { error: error?.stack });
  }

  warn(message: string): void {
    this.winston.log('warn', message);
  }
}

// Usage - el código cliente no sabe que usa Winston
class UserService {
  constructor(private logger: Logger) {}

  createUser(data: any) {
    this.logger.info('Creating user', { email: data.email });
    // ...
  }
}

const winstonLogger = new WinstonLogger();
const logger = new WinstonAdapter(winstonLogger);
const userService = new UserService(logger);
```

### Multiple Sources Adapter
```typescript
// Diferentes APIs de clima
class OpenWeatherAPI {
  getWeather(city: string): { temp_c: number; humidity: number } {
    return { temp_c: 22, humidity: 65 };
  }
}

class WeatherStackAPI {
  current(location: string): { temperature: number; humidity: number } {
    return { temperature: 21, humidity: 68 };
  }
}

// Interfaz unificada
interface WeatherService {
  getTemperature(city: string): number;
  getHumidity(city: string): number;
}

// Adapters
class OpenWeatherAdapter implements WeatherService {
  constructor(private api: OpenWeatherAPI) {}

  getTemperature(city: string): number {
    return this.api.getWeather(city).temp_c;
  }

  getHumidity(city: string): number {
    return this.api.getWeather(city).humidity;
  }
}

class WeatherStackAdapter implements WeatherService {
  constructor(private api: WeatherStackAPI) {}

  getTemperature(city: string): number {
    return this.api.current(city).temperature;
  }

  getHumidity(city: string): number {
    return this.api.current(city).humidity;
  }
}

// El cliente usa la misma interfaz
function displayWeather(service: WeatherService, city: string) {
  console.log(`Temperature: ${service.getTemperature(city)}°C`);
  console.log(`Humidity: ${service.getHumidity(city)}%`);
}
```

---

## 🔄 Object vs Class Adapter

### Object Adapter (Composition) - Preferred
```typescript
class Adapter implements Target {
  private adaptee: Adaptee;

  constructor(adaptee: Adaptee) {
    this.adaptee = adaptee;  // Composición
  }

  request(): void {
    this.adaptee.specificRequest();
  }
}
```

### Class Adapter (Inheritance) - Less Flexible
```typescript
// Solo posible en lenguajes con herencia múltiple
class Adapter extends Adaptee implements Target {
  request(): void {
    this.specificRequest();  // Hereda de Adaptee
  }
}
```

---

## ✅ When to Use

| Scenario | Example |
|----------|---------|
| Integrar código legacy | Adaptar APIs viejas |
| Usar third-party libraries | Adaptar Winston, Axios, etc. |
| Unificar interfaces diferentes | Múltiples proveedores de pago |
| Testing | Adaptar para mocks |

---

## 📊 Adapter vs Related Patterns

| Pattern | Purpose |
|---------|---------|
| **Adapter** | Hace interfaces incompatibles compatibles |
| **Decorator** | Añade comportamiento sin cambiar interfaz |
| **Facade** | Simplifica una interfaz compleja |
| **Proxy** | Mismo interfaz, controla acceso |

---

## 📋 Summary

| Aspect | Details |
|--------|---------|
| **Type** | Structural |
| **Intent** | Compatibilidad de interfaces |
| **Key Benefit** | Integrar código incompatible |
| **Trade-off** | Capa extra de indirección |
| **Common Use** | Third-party integrations |

---

← [[Programming/Software Engineering/Design Patterns/Design Patterns Complete Guide|Back to Design Patterns]]
