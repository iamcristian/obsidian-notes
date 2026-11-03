---
tags:
  - software-engineering
  - clean-code
  - functions
created: 2026-01-02
status: 🔴
---
# ⚡ Functions

> *"Functions should do one thing. They should do it well. They should do it only."*

## 🎯 Core Rules

### 1. Small!
```javascript
// ❌ Bad - Función demasiado larga
function processOrder(order) {
  // 50+ líneas de código validando, calculando, guardando, enviando emails...
}

// ✅ Good - Funciones pequeñas y enfocadas
function processOrder(order) {
  validateOrder(order);
  const total = calculateTotal(order);
  saveOrder(order, total);
  sendConfirmationEmail(order);
}
```

> [!info] Rule of Thumb
> - **Ideal**: 5-10 líneas
> - **Máximo**: 20 líneas
> - Si no cabe en una pantalla, es muy larga

### 2. Do One Thing
```javascript
// ❌ Bad - Hace múltiples cosas
function emailClients(clients) {
  clients.forEach(client => {
    const record = database.lookup(client);
    if (record.isActive()) {
      email(client);
    }
  });
}

// ✅ Good - Cada función hace una cosa
function emailActiveClients(clients) {
  clients
    .filter(isActiveClient)
    .forEach(email);
}

function isActiveClient(client) {
  const record = database.lookup(client);
  return record.isActive();
}
```

### 3. One Level of Abstraction
```javascript
// ❌ Bad - Mezcla niveles de abstracción
function parseBetterJSAlternative(code) {
  const REGEXES = [...];
  const statements = code.split(' ');
  const tokens = [];
  REGEXES.forEach(regex => {
    statements.forEach(statement => {
      // Parsing de bajo nivel aquí...
    });
  });
  const ast = [];
  tokens.forEach(token => {
    // AST generation aquí...
  });
}

// ✅ Good - Niveles de abstracción consistentes
function parseBetterJSAlternative(code) {
  const tokens = tokenize(code);
  const ast = buildAST(tokens);
  return ast;
}
```

---

## 📊 Function Arguments

### Ideal: Zero Arguments (Niladic)
```javascript
function getCurrentUser() { }
```

### Good: One Argument (Monadic)
```javascript
// Pregunta sobre el argumento
function fileExists(path) { }

// Transformación del argumento
function parseJson(jsonString) { }

// Evento (no retorna nada)
function passwordAttemptFailed(attempt) { }
```

### Acceptable: Two Arguments (Dyadic)
```javascript
// ✅ Okay cuando tiene sentido
function createPoint(x, y) { }
function assertEquals(expected, actual) { }

// ❌ Consider refactoring
function createUser(name, email, age, address) { }
```

### Avoid: Three+ Arguments (Triadic/Polyadic)
```javascript
// ❌ Bad
function createMenu(title, body, buttonText, cancellable) { }

// ✅ Good - Usar objeto de configuración
function createMenu({ title, body, buttonText, cancellable }) { }

// Uso
createMenu({
  title: 'Settings',
  body: 'Configure your preferences',
  buttonText: 'Save',
  cancellable: true
});
```

---

## 🚫 Side Effects

```javascript
// ❌ Bad - Side effect oculto
function checkPassword(userName, password) {
  const user = UserDatabase.find(userName);
  if (user !== null) {
    if (user.password === encrypt(password)) {
      Session.initialize(); // ⚠️ SIDE EFFECT!
      return true;
    }
  }
  return false;
}

// ✅ Good - Nombre revela lo que hace
function checkPasswordAndInitializeSession(userName, password) { }

// ✅ Better - Separar responsabilidades
function checkPassword(userName, password) { 
  // Solo verifica password
}
function initializeSession(user) {
  // Solo inicializa sesión
}
```

---

## 🔄 Command Query Separation

```javascript
// ❌ Bad - Hace dos cosas y confunde
function set(attribute, value) {
  // Setea el valor Y retorna si existía
  return previousValue !== null;
}

// Uso confuso
if (set("username", "john")) { } // ¿Qué significa true?

// ✅ Good - Separar comando y query
function attributeExists(attribute) {
  return values[attribute] !== undefined;
}

function setAttribute(attribute, value) {
  values[attribute] = value;
}

// Uso claro
if (attributeExists("username")) {
  setAttribute("username", "john");
}
```

---

## 🛡️ Error Handling

### Prefer Exceptions over Error Codes
```javascript
// ❌ Bad
function deletePage(page) {
  if (page.isValid()) {
    if (page.canDelete()) {
      if (registry.deletePage(page) === ERROR) {
        console.log("Error deleting page");
      }
    }
  }
}

// ✅ Good
function deletePage(page) {
  try {
    deletePageAndReferences(page);
  } catch (error) {
    logError(error);
  }
}

function deletePageAndReferences(page) {
  page.validate();
  page.checkDeletionPermission();
  registry.deletePage(page);
}
```

### Extract Try/Catch Blocks
```javascript
// ✅ Separar el try/catch del código de negocio
function delete(page) {
  try {
    deletePageAndAllReferences(page);
  } catch (error) {
    logError(error);
  }
}

function deletePageAndAllReferences(page) {
  // Lógica pura sin try/catch
}
```

---

## 📋 Quick Checklist

> [!check] Function Quality Check
> - [ ] ¿Hace solo UNA cosa?
> - [ ] ¿Es pequeña (< 20 líneas)?
> - [ ] ¿Tiene 0-2 argumentos?
> - [ ] ¿El nombre describe lo que hace?
> - [ ] ¿Tiene UN nivel de abstracción?
> - [ ] ¿No tiene side effects ocultos?
> - [ ] ¿No usa flags como argumentos?

---

## 💡 Tips

> [!tip] Flag Arguments
> ```javascript
> // ❌ Bad - Flag como argumento
> function createFile(name, temp) {
>   if (temp) { /* ... */ }
>   else { /* ... */ }
> }
> 
> // ✅ Good - Dos funciones separadas
> function createFile(name) { }
> function createTempFile(name) { }
> ```

---

← [[Programming/Software Engineering/Clean Code/Clean Code Complete Guide|Back to Clean Code]]
