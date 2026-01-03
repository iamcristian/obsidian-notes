---
tags:
  - software-engineering
  - clean-code
  - error-handling
created: 2026-01-02
status: 🔴
---
# 🛡️ Error Handling

> *"Error handling is important, but if it obscures logic, it's wrong."*

## 🎯 Core Principles

### 1. Use Exceptions Rather Than Return Codes
```javascript
// ❌ Bad - Error codes
function delete(id) {
  const record = db.find(id);
  if (record === null) {
    return -1; // Error code
  }
  const result = db.delete(record);
  if (result === 0) {
    return -2; // Another error code
  }
  return 0; // Success
}

// Calling code becomes nested mess
const result = delete(id);
if (result === 0) {
  // Success
} else if (result === -1) {
  // Handle not found
} else if (result === -2) {
  // Handle delete failed
}

// ✅ Good - Exceptions
function delete(id) {
  const record = db.find(id);
  if (record === null) {
    throw new RecordNotFoundError(id);
  }
  db.delete(record);
}

// Calling code is cleaner
try {
  delete(id);
  showSuccess();
} catch (error) {
  showError(error);
}
```

### 2. Write Your Try-Catch-Finally First
```javascript
// Start with the structure, then fill in
function readFile(path) {
  let file = null;
  try {
    file = openFile(path);
    return parseContent(file);
  } catch (error) {
    throw new FileProcessingError(path, error);
  } finally {
    if (file) {
      file.close();
    }
  }
}
```

---

## 🏗️ Custom Exceptions

### Create Meaningful Exception Classes
```javascript
// ❌ Bad - Generic error
throw new Error("Something went wrong");

// ✅ Good - Specific, informative errors
class ValidationError extends Error {
  constructor(field, message) {
    super(`Validation failed for ${field}: ${message}`);
    this.name = 'ValidationError';
    this.field = field;
  }
}

class NotFoundError extends Error {
  constructor(resource, id) {
    super(`${resource} with id ${id} not found`);
    this.name = 'NotFoundError';
    this.resource = resource;
    this.id = id;
  }
}

class NetworkError extends Error {
  constructor(url, statusCode) {
    super(`Request to ${url} failed with status ${statusCode}`);
    this.name = 'NetworkError';
    this.url = url;
    this.statusCode = statusCode;
  }
}
```

### Use Exception Hierarchy
```javascript
// Base error for your application
class AppError extends Error {
  constructor(message) {
    super(message);
    this.name = this.constructor.name;
    this.timestamp = new Date();
  }
}

// Specific errors extend base
class BusinessLogicError extends AppError { }
class DataAccessError extends AppError { }
class ExternalServiceError extends AppError { }

// Even more specific
class InsufficientFundsError extends BusinessLogicError {
  constructor(required, available) {
    super(`Insufficient funds: required ${required}, available ${available}`);
    this.required = required;
    this.available = available;
  }
}
```

---

## 🎭 The Special Case Pattern

```javascript
// ❌ Bad - Checking for null everywhere
function getExpenseTotal(expenses) {
  const meals = expenses.getMeals();
  if (meals !== null) {
    total += meals.getTotal();
  }
  const transport = expenses.getTransport();
  if (transport !== null) {
    total += transport.getTotal();
  }
  return total;
}

// ✅ Good - Use Special Case / Null Object Pattern
class NullExpense {
  getTotal() { return 0; }
}

class ExpenseReport {
  getMeals() {
    return this.meals || new NullExpense();
  }
  getTransport() {
    return this.transport || new NullExpense();
  }
}

// Clean calling code
function getExpenseTotal(expenses) {
  return expenses.getMeals().getTotal() + 
         expenses.getTransport().getTotal();
}
```

---

## 🚫 Don't Return Null

```javascript
// ❌ Bad - Returning null
function getUsers() {
  const users = db.findUsers();
  if (users.length === 0) {
    return null;
  }
  return users;
}

// Caller must check for null EVERY TIME
const users = getUsers();
if (users !== null) {
  users.forEach(/*...*/);
}

// ✅ Good - Return empty collection
function getUsers() {
  return db.findUsers(); // Returns [] if none found
}

// Caller doesn't need null checks
getUsers().forEach(/*...*/); // Works with empty array
```

### For Single Objects - Use Optional/Maybe
```javascript
// ❌ Bad
function findUser(id) {
  const user = db.find(id);
  return user || null;
}

// ✅ Good - Throw if required
function getUser(id) {
  const user = db.find(id);
  if (!user) {
    throw new NotFoundError('User', id);
  }
  return user;
}

// ✅ Good - Optional for truly optional
function findUser(id) {
  return db.find(id); // undefined if not found
}

// Use with nullish coalescing
const user = findUser(id) ?? defaultUser;
const userName = findUser(id)?.name ?? 'Guest';
```

---

## 🚫 Don't Pass Null

```javascript
// ❌ Bad - Accepting null parameters
function calculate(start, end) {
  if (start === null) {
    throw new Error("start cannot be null");
  }
  if (end === null) {
    throw new Error("end cannot be null");
  }
  return end - start;
}

// ✅ Good - TypeScript prevents null at compile time
function calculate(start: Date, end: Date): number {
  return end.getTime() - start.getTime();
}

// ✅ Good - With defaults for optional params
function calculate(start, end = new Date()) {
  return end - start;
}
```

---

## 📊 Error Handling Patterns

### Wrapper Pattern for Third-Party APIs
```javascript
// ❌ Bad - Exposing third-party exceptions throughout codebase
try {
  const response = await axios.get('/api/users');
} catch (error) {
  // AxiosError leaks everywhere
}

// ✅ Good - Wrap third-party calls
class ApiClient {
  async get(url) {
    try {
      const response = await axios.get(url);
      return response.data;
    } catch (error) {
      if (error.response?.status === 404) {
        throw new NotFoundError('Resource', url);
      }
      if (error.response?.status >= 500) {
        throw new ServiceUnavailableError(url);
      }
      throw new NetworkError(url, error.message);
    }
  }
}
```

### Result Pattern (Alternative to Exceptions)
```typescript
type Result<T, E> = 
  | { success: true; value: T }
  | { success: false; error: E };

function divide(a: number, b: number): Result<number, string> {
  if (b === 0) {
    return { success: false, error: "Cannot divide by zero" };
  }
  return { success: true, value: a / b };
}

// Usage
const result = divide(10, 0);
if (result.success) {
  console.log(result.value);
} else {
  console.log(result.error);
}
```

---

## 📋 Quick Checklist

> [!check] Error Handling Review
> - [ ] ¿Usas excepciones en lugar de códigos de error?
> - [ ] ¿Las excepciones tienen información suficiente para debug?
> - [ ] ¿No retornas null? (usa empty collections o excepciones)
> - [ ] ¿No pasas null como parámetro?
> - [ ] ¿Los third-party están wrapeados?
> - [ ] ¿El error handling no oscurece la lógica de negocio?

---

## 💡 Tips

> [!tip] Logging Errors
> ```javascript
> try {
>   processOrder(order);
> } catch (error) {
>   // Log con contexto suficiente para debugging
>   logger.error('Order processing failed', {
>     orderId: order.id,
>     userId: order.userId,
>     error: error.message,
>     stack: error.stack
>   });
>   throw error; // Re-throw si debe propagarse
> }
> ```

---

← [[Programming/Software Engineering/Clean Code/_Index|Back to Clean Code]]
