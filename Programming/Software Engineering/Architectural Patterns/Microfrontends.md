---
tags:
  - software-engineering
  - architecture
  - microfrontends
created: 2026-01-02
status: 🔴
---
# 🧩 Microfrontends

> *"The microservice approach to frontend development."*

## 🎯 What are Microfrontends?

Microfrontends es un estilo arquitectónico donde una aplicación frontend se descompone en piezas más pequeñas e independientes, cada una desarrollada, testeada y desplegada por equipos autónomos.

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    MICROFRONTENDS ARCHITECTURE                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    Container App                         │   │
│   │   (Shell / Host Application)                             │   │
│   │                                                          │   │
│   │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐     │   │
│   │  │   Header MF  │ │   Search MF  │ │   Cart MF    │     │   │
│   │  │  (Team A)    │ │  (Team B)    │ │  (Team C)    │     │   │
│   │  └──────────────┘ └──────────────┘ └──────────────┘     │   │
│   │                                                          │   │
│   │  ┌────────────────────────────────────────────────────┐ │   │
│   │  │            Product Catalog MF                       │ │   │
│   │  │                 (Team D)                            │ │   │
│   │  │                                                     │ │   │
│   │  │    ┌─────────┐ ┌─────────┐ ┌─────────┐             │ │   │
│   │  │    │Product 1│ │Product 2│ │Product 3│             │ │   │
│   │  │    └─────────┘ └─────────┘ └─────────┘             │ │   │
│   │  └────────────────────────────────────────────────────┘ │   │
│   │                                                          │   │
│   │  ┌──────────────────────┐ ┌──────────────────────┐      │   │
│   │  │   Checkout MF        │ │   Recommendations MF │      │   │
│   │  │   (Team E)           │ │   (Team F)           │      │   │
│   │  └──────────────────────┘ └──────────────────────┘      │   │
│   │                                                          │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔧 Integration Approaches

### 1. Build-Time Integration (Package)
```json
// package.json
{
  "dependencies": {
    "@mf/header": "^1.0.0",
    "@mf/catalog": "^2.0.0",
    "@mf/cart": "^1.5.0"
  }
}
```

```tsx
// App.tsx
import Header from '@mf/header';
import Catalog from '@mf/catalog';
import Cart from '@mf/cart';

function App() {
  return (
    <>
      <Header />
      <Catalog />
      <Cart />
    </>
  );
}
```

### 2. Run-Time Integration (Module Federation)
```javascript
// webpack.config.js (Container)
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin');

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'container',
      remotes: {
        header: 'header@http://localhost:3001/remoteEntry.js',
        catalog: 'catalog@http://localhost:3002/remoteEntry.js',
        cart: 'cart@http://localhost:3003/remoteEntry.js',
      },
      shared: {
        react: { singleton: true, eager: true },
        'react-dom': { singleton: true, eager: true },
      },
    }),
  ],
};

// webpack.config.js (Header MF)
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'header',
      filename: 'remoteEntry.js',
      exposes: {
        './Header': './src/Header',
      },
      shared: {
        react: { singleton: true },
        'react-dom': { singleton: true },
      },
    }),
  ],
};
```

```tsx
// Container App.tsx
import React, { Suspense } from 'react';

const Header = React.lazy(() => import('header/Header'));
const Catalog = React.lazy(() => import('catalog/Catalog'));
const Cart = React.lazy(() => import('cart/Cart'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Header />
      <Catalog />
      <Cart />
    </Suspense>
  );
}
```

### 3. Server-Side Composition
```javascript
// Server
const express = require('express');
const { createProxyMiddleware } = require('http-proxy-middleware');

const app = express();

// Route to different MF servers
app.use('/header', createProxyMiddleware({ target: 'http://header-mf:3001' }));
app.use('/catalog', createProxyMiddleware({ target: 'http://catalog-mf:3002' }));
app.use('/cart', createProxyMiddleware({ target: 'http://cart-mf:3003' }));

// Server-side includes (SSI)
app.get('/', async (req, res) => {
  const [header, catalog, cart] = await Promise.all([
    fetch('http://header-mf:3001/fragment').then(r => r.text()),
    fetch('http://catalog-mf:3002/fragment').then(r => r.text()),
    fetch('http://cart-mf:3003/fragment').then(r => r.text()),
  ]);

  res.send(`
    <!DOCTYPE html>
    <html>
      <body>
        ${header}
        ${catalog}
        ${cart}
      </body>
    </html>
  `);
});
```

### 4. iFrame Integration
```html
<!-- Container -->
<header>
  <iframe src="http://header-mf.example.com" />
</header>
<main>
  <iframe src="http://catalog-mf.example.com" />
</main>
<aside>
  <iframe src="http://cart-mf.example.com" />
</aside>
```

### 5. Web Components
```javascript
// header-mf/src/index.js
class HeaderComponent extends HTMLElement {
  connectedCallback() {
    this.innerHTML = `
      <nav>
        <a href="/">Home</a>
        <a href="/products">Products</a>
      </nav>
    `;
  }
}

customElements.define('mf-header', HeaderComponent);
```

```html
<!-- Container -->
<script src="http://header-mf.example.com/bundle.js"></script>
<mf-header></mf-header>
```

---

## 📡 Communication Between MFs

### 1. Custom Events
```typescript
// Publish (from Cart MF)
const event = new CustomEvent('cart:item-added', {
  detail: { productId: '123', quantity: 1 }
});
window.dispatchEvent(event);

// Subscribe (in Header MF)
window.addEventListener('cart:item-added', (event: CustomEvent) => {
  const { productId, quantity } = event.detail;
  updateCartCount(quantity);
});
```

### 2. Shared State (Event Bus)
```typescript
// shared/event-bus.ts
type EventCallback = (data: unknown) => void;

class EventBus {
  private events: Map<string, EventCallback[]> = new Map();

  subscribe(event: string, callback: EventCallback) {
    if (!this.events.has(event)) {
      this.events.set(event, []);
    }
    this.events.get(event)!.push(callback);
    
    return () => {
      const callbacks = this.events.get(event)!;
      const index = callbacks.indexOf(callback);
      callbacks.splice(index, 1);
    };
  }

  publish(event: string, data: unknown) {
    this.events.get(event)?.forEach(callback => callback(data));
  }
}

// Singleton instance
export const eventBus = new EventBus();

// Usage
eventBus.publish('user:logged-in', { userId: '123' });
eventBus.subscribe('user:logged-in', (data) => console.log(data));
```

### 3. Props/Callbacks (Module Federation)
```tsx
// Container
import { useState } from 'react';

const Header = React.lazy(() => import('header/Header'));
const Cart = React.lazy(() => import('cart/Cart'));

function App() {
  const [cartItems, setCartItems] = useState([]);

  const handleAddToCart = (item) => {
    setCartItems(prev => [...prev, item]);
  };

  return (
    <>
      <Header cartCount={cartItems.length} />
      <Catalog onAddToCart={handleAddToCart} />
      <Cart items={cartItems} />
    </>
  );
}
```

### 4. URL/Query Parameters
```typescript
// Navigation-based communication
// Product MF links to checkout with product info in URL
<Link to={`/checkout?product=${productId}&qty=${quantity}`}>
  Checkout
</Link>

// Checkout MF reads from URL
const params = new URLSearchParams(window.location.search);
const productId = params.get('product');
```

---

## 🎨 Styling Strategies

### CSS Modules / Scoped CSS
```css
/* header.module.css */
.nav {
  display: flex;
}
```

```tsx
import styles from './header.module.css';

function Header() {
  return <nav className={styles.nav}>...</nav>;
}
```

### Shadow DOM (Web Components)
```typescript
class HeaderComponent extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
  }

  connectedCallback() {
    this.shadowRoot!.innerHTML = `
      <style>
        /* Styles are scoped to this component */
        nav { display: flex; }
      </style>
      <nav>...</nav>
    `;
  }
}
```

### CSS-in-JS with Namespace
```typescript
// Each MF prefixes its styles
const useStyles = makeStyles({
  'header-nav': {
    display: 'flex'
  }
});
```

---

## 🔄 Deployment Strategies

```
┌─────────────────────────────────────────────────────────────────┐
│               INDEPENDENT DEPLOYMENT                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Team A                                                        │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ Header MF                                                │   │
│   │ Git Repo ──► CI/CD ──► CDN/Server                       │   │
│   │              │                                           │   │
│   │              └──► http://cdn.example.com/header/v1.2.3  │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   Team B                                                        │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ Catalog MF                                               │   │
│   │ Git Repo ──► CI/CD ──► CDN/Server                       │   │
│   │              │                                           │   │
│   │              └──► http://cdn.example.com/catalog/v2.0.0 │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   Container loads latest versions dynamically                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## ⚖️ Pros vs Cons

| Pros | Cons |
|------|------|
| ✅ Independent deployments | ❌ Increased complexity |
| ✅ Team autonomy | ❌ Duplicate dependencies |
| ✅ Technology flexibility | ❌ Performance overhead |
| ✅ Smaller codebases | ❌ Inconsistent UX risk |
| ✅ Isolated failures | ❌ Testing complexity |

---

## 📊 When to Use

```
✅ USE WHEN:
• Large teams (10+ frontend developers)
• Multiple teams need autonomy
• Different parts evolve at different speeds
• Legacy migration (strangler pattern)
• Enterprise applications

❌ AVOID WHEN:
• Small team (< 5 developers)
• Simple applications
• Consistent look & feel is critical
• Performance is paramount
• Team is inexperienced
```

---

## 📚 Related

- [[Programming/Software Engineering/Architectural Patterns/Microservices|Microservices]]
- [[Programming/Software Engineering/System Design/Scalability|Scalability]]

---

← [[Programming/Software Engineering/Architectural Patterns/_Index|Back to Architectural Patterns]]
