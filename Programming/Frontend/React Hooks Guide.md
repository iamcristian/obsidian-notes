# React Hooks Guide

> **Guía completa de todos los Hooks de React 19 - Conceptos, Casos de Uso y Ejemplos**

---

## 📚 Tabla de Contenidos

1. [[#¿Qué son los Hooks y Por Qué Existen?|Conceptos Fundamentales]]
2. [[#Hooks de Estado|Estado (useState, useReducer)]]
3. [[#Hooks de Efectos|Efectos (useEffect, useLayoutEffect)]]
4. [[#Hooks de Referencia|Referencias (useRef, useImperativeHandle)]]
5. [[#Hooks de Contexto|Contexto (useContext)]]
6. [[#Hooks de Rendimiento|Rendimiento (useMemo, useCallback)]]
7. [[#Hooks de React 19|React 19 (use, useOptimistic, etc.)]]
8. [[#Custom Hooks|Custom Hooks]]

---

## ¿Qué son los Hooks y Por Qué Existen?

### El Problema Antes de Hooks

Antes de React 16.8, para tener **estado** o **efectos secundarios** necesitabas usar **clases**:

```typescript
// ❌ ANTES: Componente con clase (verboso, confuso)
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    this.handleClick = this.handleClick.bind(this); // 😫 bind manual
  }
  
  componentDidMount() {
    document.title = `Count: ${this.state.count}`;
  }
  
  componentDidUpdate() {
    document.title = `Count: ${this.state.count}`;
  }
  
  handleClick() {
    this.setState({ count: this.state.count + 1 });
  }
  
  render() {
    return <button onClick={this.handleClick}>{this.state.count}</button>;
  }
}
```

### La Solución: Hooks

Los **Hooks** permiten usar estado y otras características de React **sin escribir clases**:

```typescript
// ✅ DESPUÉS: Componente funcional con Hooks (limpio, simple)
function Counter() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);
  
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

### ¿Por Qué se Llaman "Hooks"?

Se llaman **Hooks** porque **"enganchan"** tu componente funcional a las características de React:

```
┌─────────────────────────────────────────────────────────────────┐
│                    ¿QUÉ ES UN HOOK?                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Un HOOK es una función especial que te permite "engancharte"  │
│  a características de React desde componentes funcionales.      │
│                                                                 │
│  CARACTERÍSTICAS DE REACT:                                      │
│  ┌─────────────────────┐                                        │
│  │ • Estado interno    │ ← useState() te engancha aquí         │
│  │ • Ciclo de vida     │ ← useEffect() te engancha aquí        │
│  │ • Contexto          │ ← useContext() te engancha aquí       │
│  │ • Referencias DOM   │ ← useRef() te engancha aquí           │
│  │ • Optimización      │ ← useMemo() te engancha aquí          │
│  └─────────────────────┘                                        │
│                                                                 │
│  CONVENCIÓN: Todos los hooks empiezan con "use"                 │
│  useState, useEffect, useCustomHook, etc.                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Reglas de los Hooks (OBLIGATORIAS)

```typescript
// 🚨 REGLA 1: Solo llamar Hooks en el NIVEL SUPERIOR
// No dentro de condicionales, loops, o funciones anidadas

function Component({ condition }) {
  // ✅ CORRECTO - Nivel superior
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');
  
  // ❌ INCORRECTO - Dentro de condicional
  if (condition) {
    const [value, setValue] = useState(0); // 🚫 ERROR
  }
  
  // ❌ INCORRECTO - Dentro de loop
  for (let i = 0; i < 5; i++) {
    const [item, setItem] = useState(i); // 🚫 ERROR
  }
  
  // ❌ INCORRECTO - Después de return condicional
  if (condition) return null;
  const [data, setData] = useState(null); // 🚫 ERROR
}

// ¿POR QUÉ? React identifica los hooks por su ORDEN de llamada.
// Si el orden cambia entre renders, React se confunde.
```

```typescript
// 🚨 REGLA 2: Solo llamar Hooks desde FUNCIONES DE REACT
// Componentes funcionales o Custom Hooks

// ✅ CORRECTO - En componente
function MyComponent() {
  const [count, setCount] = useState(0);
  return <div>{count}</div>;
}

// ✅ CORRECTO - En custom hook
function useCustomHook() {
  const [value, setValue] = useState(0);
  return [value, setValue];
}

// ❌ INCORRECTO - En función regular
function regularFunction() {
  const [count, setCount] = useState(0); // 🚫 ERROR
}

// ❌ INCORRECTO - En callback
function Component() {
  const handleClick = () => {
    const [count, setCount] = useState(0); // 🚫 ERROR
  };
}
```

---

## Hooks de Estado

### useState - Gestión de Estado Local

#### ¿Qué es useState?

`useState` es el hook más básico. Te permite **agregar estado** a un componente funcional.

**Estado** = datos que, cuando cambian, hacen que el componente se **re-renderice**.

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONCEPTO: useState                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  useState te da:                                                │
│  1. Una VARIABLE que persiste entre renders                     │
│  2. Una FUNCIÓN para cambiar esa variable                       │
│                                                                 │
│  const [count, setCount] = useState(0);                         │
│         │       │                   │                           │
│         │       │                   └── valor inicial           │
│         │       └── función para actualizar (setter)            │
│         └── valor actual del estado                             │
│                                                                 │
│  Cuando llamas setCount(1):                                     │
│  1. React guarda el nuevo valor (1)                             │
│  2. React re-renderiza el componente                            │
│  3. count ahora es 1                                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### ¿Cuándo usar useState?

```
✅ USA useState CUANDO:

• Necesitas que el componente "recuerde" algo entre renders
• Un cambio en ese dato debe actualizar la UI
• El dato es LOCAL a ese componente (no compartido)

EJEMPLOS:
• Contador de clicks
• Texto de un input
• Si un modal está abierto o cerrado
• Item seleccionado de una lista
• Datos cargados de una API
```

```
❌ NO USES useState CUANDO:

• El dato no afecta la UI (usa useRef)
• El dato se puede calcular de otros datos (calcula directamente)
• El dato debe compartirse entre muchos componentes (usa Context/Zustand)
• La lógica de actualización es compleja (usa useReducer)
```

#### Ejemplos de useState

```typescript
import { useState } from 'react';

// ========================================
// EJEMPLO 1: Estado simple (primitivos)
// ========================================
function Counter() {
  // Número
  const [count, setCount] = useState(0);
  
  // String
  const [name, setName] = useState('');
  
  // Boolean
  const [isOpen, setIsOpen] = useState(false);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>
        Clicks: {count}
      </button>
      
      <input 
        value={name} 
        onChange={(e) => setName(e.target.value)} 
      />
      
      <button onClick={() => setIsOpen(!isOpen)}>
        {isOpen ? 'Cerrar' : 'Abrir'}
      </button>
    </div>
  );
}

// ========================================
// EJEMPLO 2: Estado con objetos
// ========================================
interface User {
  name: string;
  email: string;
  age: number;
}

function UserForm() {
  const [user, setUser] = useState<User>({
    name: '',
    email: '',
    age: 0
  });
  
  // ⚠️ IMPORTANTE: Siempre crear objeto nuevo, no mutar
  const updateName = (name: string) => {
    // ❌ INCORRECTO - mutar directamente
    // user.name = name;
    // setUser(user);
    
    // ✅ CORRECTO - crear objeto nuevo con spread
    setUser({ ...user, name });
    
    // ✅ TAMBIÉN CORRECTO - con función
    setUser(prev => ({ ...prev, name }));
  };
  
  return (
    <form>
      <input 
        value={user.name}
        onChange={(e) => updateName(e.target.value)}
        placeholder="Nombre"
      />
      <input 
        value={user.email}
        onChange={(e) => setUser(prev => ({ ...prev, email: e.target.value }))}
        placeholder="Email"
      />
    </form>
  );
}

// ========================================
// EJEMPLO 3: Estado con arrays
// ========================================
interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

function TodoList() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [inputText, setInputText] = useState('');
  
  // Agregar item (crear array nuevo)
  const addTodo = () => {
    if (!inputText.trim()) return;
    
    const newTodo: Todo = {
      id: Date.now(),
      text: inputText,
      completed: false
    };
    
    // ✅ Spread para crear nuevo array
    setTodos([...todos, newTodo]);
    setInputText('');
  };
  
  // Eliminar item (filter crea nuevo array)
  const deleteTodo = (id: number) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };
  
  // Actualizar item (map crea nuevo array)
  const toggleTodo = (id: number) => {
    setTodos(todos.map(todo => 
      todo.id === id 
        ? { ...todo, completed: !todo.completed }
        : todo
    ));
  };
  
  return (
    <div>
      <input 
        value={inputText}
        onChange={(e) => setInputText(e.target.value)}
        onKeyPress={(e) => e.key === 'Enter' && addTodo()}
      />
      <button onClick={addTodo}>Agregar</button>
      
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <input 
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
            />
            <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
              {todo.text}
            </span>
            <button onClick={() => deleteTodo(todo.id)}>❌</button>
          </li>
        ))}
      </ul>
    </div>
  );
}

// ========================================
// EJEMPLO 4: Lazy Initialization
// ========================================
// Cuando el valor inicial es COSTOSO de calcular

function ExpensiveComponent() {
  // ❌ Esto se ejecuta en CADA render (desperdicio)
  const [data, setData] = useState(expensiveCalculation());
  
  // ✅ Esto se ejecuta SOLO en el primer render
  const [data2, setData2] = useState(() => expensiveCalculation());
  
  // ✅ Útil para leer de localStorage
  const [theme, setTheme] = useState(() => {
    const saved = localStorage.getItem('theme');
    return saved ? JSON.parse(saved) : 'light';
  });
}

// ========================================
// EJEMPLO 5: Actualización Funcional
// ========================================
// Cuando el nuevo estado depende del anterior

function Counter() {
  const [count, setCount] = useState(0);
  
  const incrementThree = () => {
    // ❌ PROBLEMA: Esto solo incrementa en 1
    // Porque React agrupa (batches) las actualizaciones
    // y count tiene el mismo valor en las 3 llamadas
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
    // Resultado: count = 1 (no 3)
    
    // ✅ SOLUCIÓN: Usar función que recibe el estado anterior
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);
    // Resultado: count = 3 ✅
  };
  
  return <button onClick={incrementThree}>+3</button>;
}
```

### useReducer - Estado Complejo con Lógica

#### ¿Qué es useReducer?

`useReducer` es una alternativa a `useState` para cuando la **lógica de actualización es compleja** o tienes **múltiples sub-valores**.

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONCEPTO: useReducer                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  useReducer sigue el patrón REDUX:                              │
│                                                                 │
│  ESTADO ──→ ACCIÓN ──→ REDUCER ──→ NUEVO ESTADO                │
│                                                                 │
│  const [state, dispatch] = useReducer(reducer, initialState);   │
│                                                                 │
│  • state: Estado actual                                         │
│  • dispatch: Función para enviar acciones                       │
│  • reducer: Función que calcula nuevo estado                    │
│  • initialState: Estado inicial                                 │
│                                                                 │
│  REDUCER: (estadoActual, acción) => nuevoEstado                │
│                                                                 │
│  function reducer(state, action) {                              │
│    switch (action.type) {                                       │
│      case 'INCREMENT':                                          │
│        return { ...state, count: state.count + 1 };            │
│      case 'DECREMENT':                                          │
│        return { ...state, count: state.count - 1 };            │
│      default:                                                   │
│        return state;                                            │
│    }                                                            │
│  }                                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### ¿Cuándo usar useReducer vs useState?

```
┌─────────────────────────────────────────────────────────────────────┐
│                useState vs useReducer                               │
├────────────────────────────────┬────────────────────────────────────┤
│          useState              │          useReducer                │
├────────────────────────────────┼────────────────────────────────────┤
│ Estado simple                  │ Estado complejo (muchas props)     │
│ 1-3 estados relacionados       │ 4+ estados relacionados            │
│ Actualizaciones simples        │ Lógica de actualización compleja   │
│ Pocas formas de actualizar     │ Muchas formas de actualizar        │
│                                │                                    │
│ Ej: isOpen, count, inputValue  │ Ej: Formulario multi-paso          │
│                                │ Ej: Carrito de compras             │
│                                │ Ej: Editor con undo/redo           │
│                                │ Ej: Juegos con múltiples estados   │
└────────────────────────────────┴────────────────────────────────────┘
```

#### Ejemplo Completo: Carrito de Compras

```typescript
import { useReducer } from 'react';

// ========================================
// TIPOS
// ========================================
interface Product {
  id: string;
  name: string;
  price: number;
}

interface CartItem extends Product {
  quantity: number;
}

interface CartState {
  items: CartItem[];
  total: number;
  itemCount: number;
  isOpen: boolean;
}

// Tipos de acciones (usa union type para type safety)
type CartAction =
  | { type: 'ADD_ITEM'; payload: Product }
  | { type: 'REMOVE_ITEM'; payload: { id: string } }
  | { type: 'UPDATE_QUANTITY'; payload: { id: string; quantity: number } }
  | { type: 'CLEAR_CART' }
  | { type: 'TOGGLE_CART' };

// ========================================
// REDUCER
// ========================================
function cartReducer(state: CartState, action: CartAction): CartState {
  switch (action.type) {
    case 'ADD_ITEM': {
      const existingItem = state.items.find(item => item.id === action.payload.id);
      
      let newItems: CartItem[];
      if (existingItem) {
        // Si existe, incrementar cantidad
        newItems = state.items.map(item =>
          item.id === action.payload.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      } else {
        // Si no existe, agregar nuevo
        newItems = [...state.items, { ...action.payload, quantity: 1 }];
      }
      
      return {
        ...state,
        items: newItems,
        total: newItems.reduce((sum, item) => sum + item.price * item.quantity, 0),
        itemCount: newItems.reduce((sum, item) => sum + item.quantity, 0),
      };
    }
    
    case 'REMOVE_ITEM': {
      const newItems = state.items.filter(item => item.id !== action.payload.id);
      
      return {
        ...state,
        items: newItems,
        total: newItems.reduce((sum, item) => sum + item.price * item.quantity, 0),
        itemCount: newItems.reduce((sum, item) => sum + item.quantity, 0),
      };
    }
    
    case 'UPDATE_QUANTITY': {
      const newItems = state.items.map(item =>
        item.id === action.payload.id
          ? { ...item, quantity: action.payload.quantity }
          : item
      ).filter(item => item.quantity > 0); // Eliminar si quantity = 0
      
      return {
        ...state,
        items: newItems,
        total: newItems.reduce((sum, item) => sum + item.price * item.quantity, 0),
        itemCount: newItems.reduce((sum, item) => sum + item.quantity, 0),
      };
    }
    
    case 'CLEAR_CART':
      return {
        ...state,
        items: [],
        total: 0,
        itemCount: 0,
      };
    
    case 'TOGGLE_CART':
      return {
        ...state,
        isOpen: !state.isOpen,
      };
    
    default:
      // TypeScript: esto asegura que manejamos todos los casos
      const _exhaustiveCheck: never = action;
      return state;
  }
}

// ========================================
// COMPONENTE
// ========================================
const initialState: CartState = {
  items: [],
  total: 0,
  itemCount: 0,
  isOpen: false,
};

function ShoppingCart() {
  const [cart, dispatch] = useReducer(cartReducer, initialState);
  
  // Las acciones son claras y descriptivas
  const addToCart = (product: Product) => {
    dispatch({ type: 'ADD_ITEM', payload: product });
  };
  
  const removeFromCart = (id: string) => {
    dispatch({ type: 'REMOVE_ITEM', payload: { id } });
  };
  
  const updateQuantity = (id: string, quantity: number) => {
    dispatch({ type: 'UPDATE_QUANTITY', payload: { id, quantity } });
  };
  
  const clearCart = () => {
    dispatch({ type: 'CLEAR_CART' });
  };
  
  const toggleCart = () => {
    dispatch({ type: 'TOGGLE_CART' });
  };
  
  return (
    <div>
      <button onClick={toggleCart}>
        🛒 Carrito ({cart.itemCount})
      </button>
      
      {cart.isOpen && (
        <div className="cart-modal">
          <h2>Tu Carrito</h2>
          
          {cart.items.length === 0 ? (
            <p>El carrito está vacío</p>
          ) : (
            <>
              {cart.items.map(item => (
                <div key={item.id} className="cart-item">
                  <span>{item.name}</span>
                  <span>${item.price}</span>
                  
                  <div className="quantity-controls">
                    <button onClick={() => updateQuantity(item.id, item.quantity - 1)}>
                      -
                    </button>
                    <span>{item.quantity}</span>
                    <button onClick={() => updateQuantity(item.id, item.quantity + 1)}>
                      +
                    </button>
                  </div>
                  
                  <button onClick={() => removeFromCart(item.id)}>
                    Eliminar
                  </button>
                </div>
              ))}
              
              <div className="cart-total">
                <strong>Total: ${cart.total.toFixed(2)}</strong>
              </div>
              
              <button onClick={clearCart}>Vaciar Carrito</button>
            </>
          )}
        </div>
      )}
    </div>
  );
}
```

---

## Hooks de Efectos

### useEffect - Efectos Secundarios

#### ¿Qué es useEffect?

`useEffect` te permite ejecutar **efectos secundarios** en componentes funcionales.

**Efecto secundario** = cualquier cosa que afecta algo FUERA del componente:
- Llamadas a APIs (fetch)
- Suscripciones (websockets, eventos)
- Manipulación del DOM (document.title)
- Timers (setTimeout, setInterval)
- Logging

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONCEPTO: useEffect                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  useEffect sincroniza tu componente con sistemas externos.      │
│                                                                 │
│  useEffect(() => {                                              │
│    // CÓDIGO DEL EFECTO                                         │
│    // Se ejecuta DESPUÉS del render                             │
│                                                                 │
│    return () => {                                               │
│      // CLEANUP (limpieza)                                      │
│      // Se ejecuta ANTES del siguiente efecto                   │
│      // o cuando el componente se desmonta                      │
│    };                                                           │
│  }, [dependencias]);                                            │
│     │                                                           │
│     └── Array de valores que el efecto "observa"                │
│         Si alguno cambia, el efecto se re-ejecuta               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Las 3 Formas de useEffect

```typescript
import { useEffect, useState } from 'react';

function Component({ userId }: { userId: string }) {
  const [data, setData] = useState(null);
  
  // ========================================
  // FORMA 1: Sin array de dependencias
  // Se ejecuta en CADA render (casi nunca lo quieres)
  // ========================================
  useEffect(() => {
    console.log('Ejecuta en CADA render');
  });
  // ⚠️ Cuidado: Puede causar loops infinitos si actualizas estado
  
  // ========================================
  // FORMA 2: Array vacío []
  // Se ejecuta SOLO al montar (una vez)
  // ========================================
  useEffect(() => {
    console.log('Ejecuta SOLO al montar');
    
    // Cleanup se ejecuta al desmontar
    return () => {
      console.log('Componente se desmontó');
    };
  }, []);
  // ✅ Útil para: setup inicial, suscripciones globales
  
  // ========================================
  // FORMA 3: Con dependencias [dep1, dep2]
  // Se ejecuta al montar Y cuando cambia alguna dependencia
  // ========================================
  useEffect(() => {
    console.log(`userId cambió a: ${userId}`);
    
    // Fetch datos del usuario
    fetchUser(userId).then(setData);
    
    return () => {
      console.log('Limpiando efecto anterior');
    };
  }, [userId]);
  // ✅ Cada vez que userId cambia:
  //    1. Ejecuta cleanup del efecto anterior
  //    2. Ejecuta el nuevo efecto
}
```

#### ¿Cuándo usar useEffect?

```
┌─────────────────────────────────────────────────────────────────┐
│                    ¿CUÁNDO USAR useEffect?                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ✅ USA useEffect PARA:                                         │
│                                                                 │
│  1. SINCRONIZAR CON SISTEMAS EXTERNOS                          │
│     • Llamadas a APIs (fetch)                                   │
│     • Suscripciones (websockets, eventos del DOM)               │
│     • Librerías externas (chart.js, mapas)                      │
│                                                                 │
│  2. EFECTOS QUE NECESITAN CLEANUP                              │
│     • Event listeners                                           │
│     • Timers (setInterval)                                      │
│     • Suscripciones                                             │
│                                                                 │
│  ❌ NO USES useEffect PARA:                                     │
│                                                                 │
│  1. CALCULAR DATOS DERIVADOS                                    │
│     // ❌ Malo                                                   │
│     useEffect(() => {                                           │
│       setFilteredItems(items.filter(i => i.active));            │
│     }, [items]);                                                │
│                                                                 │
│     // ✅ Mejor: calcula directamente                           │
│     const filteredItems = items.filter(i => i.active);          │
│                                                                 │
│  2. MANEJAR EVENTOS DEL USUARIO                                │
│     // ❌ Malo                                                   │
│     useEffect(() => {                                           │
│       if (submitted) sendForm(data);                            │
│     }, [submitted]);                                            │
│                                                                 │
│     // ✅ Mejor: en el event handler                            │
│     const handleSubmit = () => sendForm(data);                  │
│                                                                 │
│  3. INICIALIZAR LA APLICACIÓN                                  │
│     // Hazlo fuera del componente o en el entry point           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Ejemplos Prácticos de useEffect

```typescript
// ========================================
// EJEMPLO 1: Fetch de datos con cleanup
// ========================================
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    // Flag para evitar actualizar estado si el componente se desmontó
    let isMounted = true;
    
    async function fetchUser() {
      setLoading(true);
      setError(null);
      
      try {
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) throw new Error('Error al cargar usuario');
        
        const data = await response.json();
        
        // Solo actualizar si el componente sigue montado
        if (isMounted) {
          setUser(data);
          setLoading(false);
        }
      } catch (err) {
        if (isMounted) {
          setError(err.message);
          setLoading(false);
        }
      }
    }
    
    fetchUser();
    
    // Cleanup: marcar como desmontado
    return () => {
      isMounted = false;
    };
  }, [userId]); // Re-ejecutar cuando userId cambie

  if (loading) return <Spinner />;
  if (error) return <Error message={error} />;
  if (!user) return <NotFound />;
  
  return <UserCard user={user} />;
}

// ========================================
// EJEMPLO 2: Event listeners
// ========================================
function WindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });
  
  useEffect(() => {
    // Definir handler
    function handleResize() {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    }
    
    // Agregar listener
    window.addEventListener('resize', handleResize);
    
    // Cleanup: remover listener
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []); // [] = solo al montar/desmontar
  
  return <p>Window: {size.width} x {size.height}</p>;
}

// ========================================
// EJEMPLO 3: setInterval con cleanup
// ========================================
function Timer() {
  const [seconds, setSeconds] = useState(0);
  
  useEffect(() => {
    // Crear interval
    const intervalId = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);
    
    // Cleanup: limpiar interval
    return () => {
      clearInterval(intervalId);
    };
  }, []); // [] = setup una vez
  
  return <p>Segundos: {seconds}</p>;
}

// ========================================
// EJEMPLO 4: Sincronizar con localStorage
// ========================================
function useLocalStorage<T>(key: string, initialValue: T) {
  // Estado inicial desde localStorage
  const [value, setValue] = useState<T>(() => {
    const saved = localStorage.getItem(key);
    return saved ? JSON.parse(saved) : initialValue;
  });
  
  // Sincronizar cambios a localStorage
  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);
  
  return [value, setValue] as const;
}

// Uso
function Settings() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  
  return (
    <select value={theme} onChange={e => setTheme(e.target.value)}>
      <option value="light">Claro</option>
      <option value="dark">Oscuro</option>
    </select>
  );
}

// ========================================
// EJEMPLO 5: Debounce de búsqueda
// ========================================
function SearchInput() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  useEffect(() => {
    // No buscar si query está vacío
    if (!query.trim()) {
      setResults([]);
      return;
    }
    
    // Crear timeout para debounce
    const timeoutId = setTimeout(async () => {
      const response = await fetch(`/api/search?q=${query}`);
      const data = await response.json();
      setResults(data);
    }, 300); // Esperar 300ms después de que el usuario deje de escribir
    
    // Cleanup: cancelar timeout si query cambia antes de 300ms
    return () => clearTimeout(timeoutId);
  }, [query]);
  
  return (
    <div>
      <input 
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="Buscar..."
      />
      <SearchResults results={results} />
    </div>
  );
}
```

### useLayoutEffect - Efectos Síncronos

#### ¿Qué es useLayoutEffect?

Es igual que `useEffect`, pero se ejecuta **ANTES** de que el navegador pinte la pantalla.

```
┌─────────────────────────────────────────────────────────────────┐
│               useEffect vs useLayoutEffect                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  TIMELINE DE UN RENDER:                                         │
│                                                                 │
│  1. React calcula cambios      ← render                         │
│  2. React actualiza el DOM     ← commit                         │
│  3. useLayoutEffect se ejecuta ← SÍNCRONO, bloquea pintura     │
│  4. Navegador pinta en pantalla                                 │
│  5. useEffect se ejecuta       ← ASÍNCRONO, después de pintar  │
│                                                                 │
│  useLayoutEffect:                                               │
│  • Se ejecuta ANTES de pintar                                   │
│  • Bloquea la actualización visual                              │
│  • Útil para medir/modificar el DOM antes de mostrar            │
│                                                                 │
│  useEffect:                                                     │
│  • Se ejecuta DESPUÉS de pintar                                 │
│  • No bloquea la actualización visual                           │
│  • 99% de los casos usa este                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### ¿Cuándo usar useLayoutEffect?

```typescript
import { useLayoutEffect, useRef, useState } from 'react';

// ========================================
// CASO 1: Medir elementos del DOM
// ========================================
function Tooltip({ children, content }: Props) {
  const [position, setPosition] = useState({ top: 0, left: 0 });
  const triggerRef = useRef<HTMLDivElement>(null);
  
  // ✅ useLayoutEffect porque necesitamos medir ANTES de mostrar
  // Si usamos useEffect, el tooltip aparecería en posición incorrecta
  // por un frame (parpadeo)
  useLayoutEffect(() => {
    if (!triggerRef.current) return;
    
    const rect = triggerRef.current.getBoundingClientRect();
    setPosition({
      top: rect.bottom + 10,
      left: rect.left + rect.width / 2
    });
  }, []);
  
  return (
    <>
      <div ref={triggerRef}>{children}</div>
      <div style={{ position: 'fixed', top: position.top, left: position.left }}>
        {content}
      </div>
    </>
  );
}

// ========================================
// CASO 2: Scroll a posición específica
// ========================================
function Chat({ messages }: { messages: Message[] }) {
  const containerRef = useRef<HTMLDivElement>(null);
  
  // Scroll al fondo cuando llegan mensajes nuevos
  // useLayoutEffect evita que el usuario vea el scroll
  useLayoutEffect(() => {
    if (containerRef.current) {
      containerRef.current.scrollTop = containerRef.current.scrollHeight;
    }
  }, [messages]);
  
  return (
    <div ref={containerRef} style={{ height: 400, overflow: 'auto' }}>
      {messages.map(msg => <Message key={msg.id} {...msg} />)}
    </div>
  );
}
```

---

## Hooks de Referencia

### useRef - Referencias Mutables

#### ¿Qué es useRef?

`useRef` te da una "caja" que puede guardar un valor mutable que **persiste entre renders** pero **NO causa re-renders** cuando cambia.

```
┌─────────────────────────────────────────────────────────────────┐
│              useState vs useRef                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  useState:                                                      │
│  • Cuando cambia → componente re-renderiza                      │
│  • Valor disponible en el siguiente render                      │
│  • Para datos que afectan la UI                                 │
│                                                                 │
│  useRef:                                                        │
│  • Cuando cambia → NO re-renderiza                              │
│  • Valor disponible INMEDIATAMENTE                              │
│  • Para datos que NO afectan la UI                              │
│  • Para acceder a elementos del DOM                             │
│                                                                 │
│  const ref = useRef(initialValue);                              │
│  // ref.current contiene el valor                               │
│  // Puedes leer/escribir ref.current sin causar renders         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### ¿Cuándo usar useRef?

```typescript
// ========================================
// CASO 1: Acceder a elementos del DOM
// ========================================
function TextInput() {
  const inputRef = useRef<HTMLInputElement>(null);
  
  const focusInput = () => {
    // Acceder al elemento DOM directamente
    inputRef.current?.focus();
  };
  
  return (
    <>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Enfocar</button>
    </>
  );
}

// ========================================
// CASO 2: Guardar valores sin causar re-render
// ========================================
function StopWatch() {
  const [time, setTime] = useState(0);
  const [isRunning, setIsRunning] = useState(false);
  
  // ✅ useRef para el intervalId porque:
  // 1. No queremos re-render cuando cambia
  // 2. Necesitamos el valor en cleanup
  const intervalRef = useRef<NodeJS.Timeout | null>(null);
  
  const start = () => {
    if (intervalRef.current) return;
    
    setIsRunning(true);
    intervalRef.current = setInterval(() => {
      setTime(t => t + 1);
    }, 1000);
  };
  
  const stop = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
    setIsRunning(false);
  };
  
  const reset = () => {
    stop();
    setTime(0);
  };
  
  return (
    <div>
      <p>{time} segundos</p>
      <button onClick={start} disabled={isRunning}>Start</button>
      <button onClick={stop} disabled={!isRunning}>Stop</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}

// ========================================
// CASO 3: Guardar valor anterior
// ========================================
function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T>();
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref.current; // Retorna el valor anterior
}

function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);
  
  return (
    <div>
      <p>Actual: {count}, Anterior: {prevCount}</p>
      <button onClick={() => setCount(c => c + 1)}>+1</button>
    </div>
  );
}

// ========================================
// CASO 4: Evitar efectos en el primer render
// ========================================
function useUpdateEffect(effect: () => void, deps: any[]) {
  const isFirstRender = useRef(true);
  
  useEffect(() => {
    if (isFirstRender.current) {
      isFirstRender.current = false;
      return;
    }
    
    return effect();
  }, deps);
}

// Uso
function Component({ data }) {
  // Este efecto NO se ejecuta en el primer render
  useUpdateEffect(() => {
    console.log('data cambió (pero no en el primer render)');
  }, [data]);
}

// ========================================
// CASO 5: Video/Audio player control
// ========================================
function VideoPlayer({ src }: { src: string }) {
  const videoRef = useRef<HTMLVideoElement>(null);
  const [isPlaying, setIsPlaying] = useState(false);
  
  const togglePlay = () => {
    if (!videoRef.current) return;
    
    if (isPlaying) {
      videoRef.current.pause();
    } else {
      videoRef.current.play();
    }
    setIsPlaying(!isPlaying);
  };
  
  return (
    <div>
      <video ref={videoRef} src={src} />
      <button onClick={togglePlay}>
        {isPlaying ? '⏸ Pause' : '▶ Play'}
      </button>
    </div>
  );
}
```
        if (!cancelled) {
          setError(err as Error);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    }
    
    fetchUser();
    
    return () => {
      cancelled = true;
    };
  }, [userId]);

  if (loading) return <Spinner />;
  if (error) return <Error message={error.message} />;
  if (!user) return null;
  
  return <UserCard user={user} />;
}

// Ejemplo: Event listener
useEffect(() => {
  function handleResize() {
    setWindowSize({ width: window.innerWidth, height: window.innerHeight });
  }
  
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);

// Ejemplo: Intersection Observer
useEffect(() => {
  const observer = new IntersectionObserver(
    ([entry]) => setIsVisible(entry.isIntersecting),
    { threshold: 0.1 }
  );
  
  if (ref.current) observer.observe(ref.current);
  
  return () => observer.disconnect();
}, []);
```

### useContext

```typescript
import { createContext, useContext, useState, ReactNode } from 'react';

// 1. Crear el contexto con tipos
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

// 2. Custom hook para usar el contexto (recomendado)
function useTheme() {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}

// 3. Provider component
function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// 4. Uso
function App() {
  return (
    <ThemeProvider>
      <Header />
      <Main />
    </ThemeProvider>
  );
}

function Header() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <header className={theme}>
      <button onClick={toggleTheme}>
        Switch to {theme === 'light' ? 'dark' : 'light'}
      </button>
    </header>
  );
}
```

---

## Hooks de Estado Avanzados

### useReducer

```typescript
import { useReducer, Reducer } from 'react';

// Types
interface Todo {
  id: string;
  text: string;
  completed: boolean;
}

interface State {
  todos: Todo[];
  filter: 'all' | 'active' | 'completed';
}

type Action =
  | { type: 'ADD_TODO'; payload: string }
  | { type: 'TOGGLE_TODO'; payload: string }
  | { type: 'DELETE_TODO'; payload: string }
  | { type: 'SET_FILTER'; payload: State['filter'] }
  | { type: 'CLEAR_COMPLETED' };

// Reducer
const todoReducer: Reducer<State, Action> = (state, action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        ...state,
        todos: [
          ...state.todos,
          { id: crypto.randomUUID(), text: action.payload, completed: false }
        ]
      };
    
    case 'TOGGLE_TODO':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
    
    case 'DELETE_TODO':
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload)
      };
    
    case 'SET_FILTER':
      return { ...state, filter: action.payload };
    
    case 'CLEAR_COMPLETED':
      return {
        ...state,
        todos: state.todos.filter(todo => !todo.completed)
      };
    
    default:
      return state;
  }
};

// Initial state
const initialState: State = {
  todos: [],
  filter: 'all'
};

// Lazy initialization
const init = (initialTodos: Todo[]): State => ({
  todos: initialTodos,
  filter: 'all'
});

// Component
function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, initialState);
  // Con lazy init: useReducer(todoReducer, savedTodos, init);
  
  const filteredTodos = state.todos.filter(todo => {
    if (state.filter === 'active') return !todo.completed;
    if (state.filter === 'completed') return todo.completed;
    return true;
  });
  
  return (
    <div>
      <AddTodo onAdd={(text) => dispatch({ type: 'ADD_TODO', payload: text })} />
      <TodoList
        todos={filteredTodos}
        onToggle={(id) => dispatch({ type: 'TOGGLE_TODO', payload: id })}
        onDelete={(id) => dispatch({ type: 'DELETE_TODO', payload: id })}
      />
      <Filters
        current={state.filter}
        onChange={(filter) => dispatch({ type: 'SET_FILTER', payload: filter })}
      />
    </div>
  );
}
```

### useSyncExternalStore

```typescript
import { useSyncExternalStore } from 'react';

// Para suscribirse a stores externos (Redux, Zustand sin hooks, etc.)

// Ejemplo: Window width store
const windowWidthStore = {
  getSnapshot: () => window.innerWidth,
  subscribe: (callback: () => void) => {
    window.addEventListener('resize', callback);
    return () => window.removeEventListener('resize', callback);
  }
};

function WindowWidth() {
  const width = useSyncExternalStore(
    windowWidthStore.subscribe,
    windowWidthStore.getSnapshot,
    () => 0 // getServerSnapshot para SSR
  );
  
  return <span>Width: {width}px</span>;
}

// Ejemplo: Online status
function useOnlineStatus() {
  return useSyncExternalStore(
    (callback) => {
      window.addEventListener('online', callback);
      window.addEventListener('offline', callback);
      return () => {
        window.removeEventListener('online', callback);
        window.removeEventListener('offline', callback);
      };
    },
    () => navigator.onLine,
    () => true // Asumir online en servidor
  );
}

// Ejemplo: LocalStorage
function useLocalStorage<T>(key: string, initialValue: T) {
  const getSnapshot = () => {
    const item = localStorage.getItem(key);
    return item ? JSON.parse(item) : initialValue;
  };

  const subscribe = (callback: () => void) => {
    window.addEventListener('storage', callback);
    return () => window.removeEventListener('storage', callback);
  };

  const value = useSyncExternalStore(subscribe, getSnapshot, () => initialValue);
  
  const setValue = (newValue: T) => {
    localStorage.setItem(key, JSON.stringify(newValue));
    window.dispatchEvent(new Event('storage'));
  };
  
  return [value, setValue] as const;
}
```

---

## Hooks de Referencia

### useRef

```typescript
import { useRef, useEffect, forwardRef, useImperativeHandle } from 'react';

// 1. Referencia a elementos DOM
function TextInput() {
  const inputRef = useRef<HTMLInputElement>(null);
  
  const focusInput = () => {
    inputRef.current?.focus();
  };
  
  return (
    <>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus Input</button>
    </>
  );
}

// 2. Almacenar valores mutables (no causa re-render)
function Timer() {
  const [count, setCount] = useState(0);
  const intervalRef = useRef<NodeJS.Timeout | null>(null);
  const previousCountRef = useRef<number>(0);
  
  useEffect(() => {
    previousCountRef.current = count;
  }, [count]);
  
  const start = () => {
    if (intervalRef.current) return;
    intervalRef.current = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
  };
  
  const stop = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
  };
  
  return (
    <div>
      <p>Count: {count} (previous: {previousCountRef.current})</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
    </div>
  );
}

// 3. Callback ref para ejecutar código cuando ref se asigna
function MeasuredComponent() {
  const [height, setHeight] = useState(0);
  
  const measuredRef = useCallback((node: HTMLDivElement | null) => {
    if (node !== null) {
      setHeight(node.getBoundingClientRect().height);
    }
  }, []);
  
  return (
    <div ref={measuredRef}>
      Content here - Height: {height}px
    </div>
  );
}
```

### forwardRef + useImperativeHandle

```typescript
// Exponer métodos específicos al padre
interface InputHandle {
  focus: () => void;
  clear: () => void;
  getValue: () => string;
}

const CustomInput = forwardRef<InputHandle, { placeholder?: string }>(
  ({ placeholder }, ref) => {
    const inputRef = useRef<HTMLInputElement>(null);
    
    useImperativeHandle(ref, () => ({
      focus: () => inputRef.current?.focus(),
      clear: () => {
        if (inputRef.current) inputRef.current.value = '';
      },
      getValue: () => inputRef.current?.value ?? ''
    }), []);
    
    return <input ref={inputRef} placeholder={placeholder} />;
  }
);

// Uso
function Form() {
  const inputRef = useRef<InputHandle>(null);
  
  return (
    <>
      <CustomInput ref={inputRef} placeholder="Enter text..." />
      <button onClick={() => inputRef.current?.focus()}>Focus</button>
      <button onClick={() => inputRef.current?.clear()}>Clear</button>
      <button onClick={() => alert(inputRef.current?.getValue())}>Get Value</button>
    </>
  );
}
```

---

## Hooks de Rendimiento

> 🎯 **CONCEPTO CLAVE**: Los hooks de rendimiento (useMemo, useCallback, memo) existen para EVITAR trabajo innecesario. Pero usarlos incorrectamente puede hacer tu app MÁS LENTA.

### ¿Por Qué Necesitamos Optimización de Rendimiento?

```
┌─────────────────────────────────────────────────────────────────┐
│               EL PROBLEMA: RE-RENDERS EN REACT                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Cuando un componente se re-renderiza, TODO su árbol de        │
│  componentes hijos también se re-renderiza por defecto.         │
│                                                                 │
│  function Parent() {                                            │
│    const [count, setCount] = useState(0);                       │
│                                                                 │
│    return (                                                     │
│      <div>                                                      │
│        <button onClick={() => setCount(c + 1)}>+1</button>     │
│        <Child /> ← Se re-renderiza aunque no use count         │
│        <ExpensiveChild /> ← También se re-renderiza 😱         │
│      </div>                                                     │
│    );                                                           │
│  }                                                              │
│                                                                 │
│  ESTO ES UN PROBLEMA CUANDO:                                    │
│  • El hijo hace cálculos costosos                               │
│  • El hijo renderiza muchos elementos (listas grandes)          │
│  • El re-render es frecuente (input typing, drag, etc)          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### useMemo - Memoizar Valores Calculados

#### ¿Qué es useMemo?

`useMemo` "recuerda" el resultado de un cálculo y solo lo recalcula cuando sus dependencias cambian.

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONCEPTO: useMemo                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SIN useMemo:                                                   │
│  ─────────────                                                  │
│  Render 1: calcularFiltrados() → [a, b, c]                      │
│  Render 2: calcularFiltrados() → [a, b, c]  (mismo resultado)  │
│  Render 3: calcularFiltrados() → [a, b, c]  (mismo resultado)  │
│  → Se ejecuta el cálculo 3 veces aunque el resultado es igual   │
│                                                                 │
│  CON useMemo:                                                   │
│  ────────────                                                   │
│  Render 1: calcularFiltrados() → [a, b, c] (calcula)           │
│  Render 2: → [a, b, c] (usa valor guardado)                    │
│  Render 3: → [a, b, c] (usa valor guardado)                    │
│  → Solo calcula cuando las dependencias cambian                 │
│                                                                 │
│  SINTAXIS:                                                      │
│  const valorMemoizado = useMemo(                                │
│    () => calcularValor(),  // función que retorna el valor      │
│    [dep1, dep2]            // solo recalcula si estas cambian   │
│  );                                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### ¿Cuándo usar useMemo?

```
✅ USA useMemo CUANDO:

1. CÁLCULOS COSTOSOS
   • Filtrar/ordenar arrays grandes (1000+ elementos)
   • Transformaciones de datos complejas
   • Cálculos matemáticos pesados

2. PRESERVAR REFERENCIA DE OBJETOS/ARRAYS
   • Para evitar re-renders en hijos memoizados
   • Para dependencias de useEffect

❌ NO USES useMemo CUANDO:

1. CÁLCULOS SIMPLES
   • count * 2 → No lo necesitas
   • string.toUpperCase() → No lo necesitas

2. EL COMPONENTE ES RÁPIDO DE RE-RENDERIZAR
   • Si no hay problema de rendimiento, no lo añadas

3. LAS DEPENDENCIAS CAMBIAN SIEMPRE
   • Si las deps cambian en cada render, useMemo no ayuda

REGLA: Mide primero, optimiza después.
```

#### Ejemplos de useMemo

```typescript
import { useMemo, useState } from 'react';

// ========================================
// EJEMPLO 1: Cálculos costosos
// ========================================
function ProductList({ products, filter, sortBy }: Props) {
  // ✅ BUENO: Filtrar y ordenar 10,000 productos es costoso
  const filteredProducts = useMemo(() => {
    console.log('Filtrando productos...'); // Solo se ejecuta cuando es necesario
    
    return products
      .filter(p => p.category === filter)
      .sort((a, b) => a[sortBy] - b[sortBy]);
  }, [products, filter, sortBy]);
  
  // ❌ MALO: Multiplicar es barato, no necesita useMemo
  const doubledPrice = useMemo(() => price * 2, [price]); // Innecesario
  
  return <ProductGrid products={filteredProducts} />;
}

// ========================================
// EJEMPLO 2: Preservar referencia para hijos memoizados
// ========================================
const MemoizedChart = memo(({ config }: { config: ChartConfig }) => {
  console.log('Chart re-rendered');
  return <ExpensiveChart config={config} />;
});

function Dashboard({ data }: { data: number[] }) {
  const [theme, setTheme] = useState('light');
  
  // ❌ SIN useMemo: Chart re-renderiza cada vez que theme cambia
  // porque chartConfig es un objeto NUEVO en cada render
  const chartConfigBad = {
    type: 'bar',
    data: data,
    options: { animated: true }
  };
  
  // ✅ CON useMemo: Chart solo re-renderiza cuando data cambia
  const chartConfigGood = useMemo(() => ({
    type: 'bar',
    data: data,
    options: { animated: true }
  }), [data]);
  
  return (
    <div className={theme}>
      <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
      <MemoizedChart config={chartConfigGood} />
    </div>
  );
}

// ========================================
// EJEMPLO 3: Como dependencia de useEffect
// ========================================
function SearchResults({ query, filters }: Props) {
  // ✅ Memoizar el objeto de parámetros
  const searchParams = useMemo(() => ({
    query,
    ...filters,
    timestamp: Date.now() // ⚠️ Esto haría que siempre sea nuevo
  }), [query, filters]);
  
  useEffect(() => {
    // Si searchParams no está memoizado, este efecto
    // se ejecutaría en CADA render
    fetchResults(searchParams);
  }, [searchParams]);
}
```

### useCallback - Memoizar Funciones

#### ¿Qué es useCallback?

`useCallback` "recuerda" una función y solo la recrea cuando sus dependencias cambian.

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONCEPTO: useCallback                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  EN JAVASCRIPT, LAS FUNCIONES SON OBJETOS:                      │
│                                                                 │
│  const fn1 = () => console.log('hola');                         │
│  const fn2 = () => console.log('hola');                         │
│  fn1 === fn2 // false ❌ (son objetos diferentes)              │
│                                                                 │
│  PROBLEMA EN REACT:                                             │
│  ───────────────────                                            │
│  function Parent() {                                            │
│    const handleClick = () => {...}; // Nueva función cada render│
│                                                                 │
│    return <MemoizedChild onClick={handleClick} />;              │
│    // ❌ Child re-renderiza porque onClick es "diferente"       │
│  }                                                              │
│                                                                 │
│  SOLUCIÓN CON useCallback:                                      │
│  ─────────────────────────                                      │
│  function Parent() {                                            │
│    const handleClick = useCallback(() => {...}, []);            │
│    // ✅ Misma referencia entre renders                         │
│                                                                 │
│    return <MemoizedChild onClick={handleClick} />;              │
│    // ✅ Child NO re-renderiza                                  │
│  }                                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### ¿Cuándo usar useCallback?

```
✅ USA useCallback CUANDO:

1. PASAS UNA FUNCIÓN A UN COMPONENTE MEMOIZADO
   • El hijo usa memo() o React.memo
   • Sin useCallback, memo no sirve de nada

2. LA FUNCIÓN ES DEPENDENCIA DE OTRO HOOK
   • Como dependencia de useEffect
   • Como dependencia de useMemo

❌ NO USES useCallback CUANDO:

1. EL HIJO NO ESTÁ MEMOIZADO
   • Si el hijo se re-renderiza de todos modos, no sirve
   
2. LA FUNCIÓN SE USA SOLO EN EL MISMO COMPONENTE
   • No hay beneficio si no se pasa a hijos

3. LAS DEPENDENCIAS CAMBIAN SIEMPRE
   • Si las deps cambian cada render, no hay beneficio

IMPORTANTE: useCallback + memo() van juntos.
useCallback solo es útil si el receptor está memoizado.
```

#### Ejemplos de useCallback

```typescript
import { useCallback, memo, useState } from 'react';

// ========================================
// EJEMPLO 1: Con componente memoizado
// ========================================
// Hijo memoizado - solo re-renderiza si props cambian
const TodoItem = memo(({ todo, onToggle, onDelete }: Props) => {
  console.log(`TodoItem ${todo.id} rendered`);
  return (
    <li>
      <input 
        type="checkbox" 
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
      />
      {todo.text}
      <button onClick={() => onDelete(todo.id)}>Delete</button>
    </li>
  );
});

function TodoList() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [filter, setFilter] = useState('all');
  
  // ❌ SIN useCallback: TodoItem re-renderiza cuando filter cambia
  // aunque onToggle hace lo mismo
  const onToggleBad = (id: number) => {
    setTodos(todos.map(t => t.id === id ? {...t, completed: !t.completed} : t));
  };
  
  // ✅ CON useCallback: TodoItem NO re-renderiza cuando filter cambia
  const onToggle = useCallback((id: number) => {
    setTodos(prev => prev.map(t => 
      t.id === id ? {...t, completed: !t.completed} : t
    ));
  }, []); // [] porque usamos la forma funcional de setTodos
  
  const onDelete = useCallback((id: number) => {
    setTodos(prev => prev.filter(t => t.id !== id));
  }, []);
  
  return (
    <div>
      <select value={filter} onChange={e => setFilter(e.target.value)}>
        <option value="all">Todos</option>
        <option value="active">Activos</option>
        <option value="completed">Completados</option>
      </select>
      
      <ul>
        {todos.map(todo => (
          <TodoItem 
            key={todo.id}
            todo={todo}
            onToggle={onToggle}
            onDelete={onDelete}
          />
        ))}
      </ul>
    </div>
  );
}

// ========================================
// EJEMPLO 2: Como dependencia de useEffect
// ========================================
function SearchComponent({ userId }: { userId: string }) {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  // ✅ Memoizar la función de fetch
  const fetchResults = useCallback(async (searchQuery: string) => {
    const response = await fetch(`/api/search?user=${userId}&q=${searchQuery}`);
    const data = await response.json();
    setResults(data);
  }, [userId]); // Solo recrear si userId cambia
  
  useEffect(() => {
    if (query.length > 2) {
      fetchResults(query);
    }
  }, [query, fetchResults]); // fetchResults es estable
  
  return (
    <div>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      <Results data={results} />
    </div>
  );
}

// ========================================
// EJEMPLO 3: Accediendo a estado actual
// ========================================
function Counter() {
  const [count, setCount] = useState(0);
  
  // ❌ PROBLEMA: count es "viejo" (closure stale)
  const logCountBad = useCallback(() => {
    console.log('Count:', count); // Siempre logea 0
  }, []); // count no está en dependencias
  
  // ✅ SOLUCIÓN 1: Incluir count en dependencias
  const logCount = useCallback(() => {
    console.log('Count:', count);
  }, [count]);
  
  // ✅ SOLUCIÓN 2: Usar ref para valor actual
  const countRef = useRef(count);
  countRef.current = count;
  
  const logCountRef = useCallback(() => {
    console.log('Count:', countRef.current); // Siempre el valor actual
  }, []);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>+1</button>
      <MemoizedButton onClick={logCount}>Log Count</MemoizedButton>
    </div>
  );
}
```

### memo - Memoizar Componentes

#### ¿Qué es memo?

`memo` envuelve un componente y evita que se re-renderice si sus props no cambian.

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONCEPTO: memo                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SIN memo:                                                      │
│  ──────────                                                     │
│  Parent re-renderiza → Child SIEMPRE re-renderiza               │
│                                                                 │
│  CON memo:                                                      │
│  ─────────                                                      │
│  Parent re-renderiza → React compara props del Child            │
│                      → Si son iguales, NO re-renderiza          │
│                      → Si son diferentes, SÍ re-renderiza       │
│                                                                 │
│  const Child = memo(function Child({ name, onClick }) {         │
│    return <button onClick={onClick}>{name}</button>;            │
│  });                                                            │
│                                                                 │
│  ⚠️ IMPORTANTE:                                                 │
│  memo hace comparación SUPERFICIAL de props.                    │
│  Si pasas objetos/arrays/funciones nuevos, memo no ayuda.       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Ejemplo completo de optimización

```typescript
import { memo, useCallback, useMemo, useState } from 'react';

// ========================================
// COMPONENTES MEMOIZADOS
// ========================================
const Header = memo(({ title }: { title: string }) => {
  console.log('Header rendered');
  return <h1>{title}</h1>;
});

const SearchBar = memo(({ onSearch }: { onSearch: (q: string) => void }) => {
  console.log('SearchBar rendered');
  const [value, setValue] = useState('');
  
  return (
    <input 
      value={value}
      onChange={e => setValue(e.target.value)}
      onKeyDown={e => e.key === 'Enter' && onSearch(value)}
    />
  );
});

const ProductCard = memo(({ product, onAddToCart }: Props) => {
  console.log(`ProductCard ${product.id} rendered`);
  return (
    <div>
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button onClick={() => onAddToCart(product.id)}>Add</button>
    </div>
  );
});

// ========================================
// COMPONENTE PADRE OPTIMIZADO
// ========================================
function ProductPage({ products }: { products: Product[] }) {
  const [cart, setCart] = useState<string[]>([]);
  const [searchQuery, setSearchQuery] = useState('');
  const [sortBy, setSortBy] = useState<'name' | 'price'>('name');
  
  // ✅ useMemo para filtrar/ordenar
  const filteredProducts = useMemo(() => {
    return products
      .filter(p => p.name.toLowerCase().includes(searchQuery.toLowerCase()))
      .sort((a, b) => a[sortBy] > b[sortBy] ? 1 : -1);
  }, [products, searchQuery, sortBy]);
  
  // ✅ useCallback para funciones pasadas a hijos
  const handleSearch = useCallback((query: string) => {
    setSearchQuery(query);
  }, []);
  
  const handleAddToCart = useCallback((productId: string) => {
    setCart(prev => [...prev, productId]);
  }, []);
  
  // Cuando cart cambia:
  // - Header NO re-renderiza (title no cambió)
  // - SearchBar NO re-renderiza (onSearch no cambió)
  // - ProductCard NO re-renderiza (product y onAddToCart no cambiaron)
  
  return (
    <div>
      <Header title="Productos" />
      <SearchBar onSearch={handleSearch} />
      
      <div>
        <button onClick={() => setSortBy('name')}>Sort by Name</button>
        <button onClick={() => setSortBy('price')}>Sort by Price</button>
      </div>
      
      <p>Cart items: {cart.length}</p>
      
      <div className="products">
        {filteredProducts.map(product => (
          <ProductCard 
            key={product.id}
            product={product}
            onAddToCart={handleAddToCart}
          />
        ))}
      </div>
    </div>
  );
}
```

### Resumen: ¿Cuándo Usar Qué?

```
┌─────────────────────────────────────────────────────────────────────┐
│              GUÍA RÁPIDA DE OPTIMIZACIÓN                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  1. ¿TIENES UN PROBLEMA DE RENDIMIENTO?                            │
│     ├── NO → No optimices. Escribe código simple.                  │
│     └── SÍ → Continúa...                                           │
│                                                                     │
│  2. ¿QUÉ ESTÁ CAUSANDO EL PROBLEMA?                                │
│     ├── Cálculos lentos → useMemo                                  │
│     ├── Re-renders de hijos → memo + useCallback                   │
│     └── Ambos → Usa los tres                                       │
│                                                                     │
│  RECETA COMÚN:                                                      │
│  ─────────────                                                      │
│  const Child = memo(...);           // Memoizar el hijo            │
│  const handleX = useCallback(...);  // Memoizar las funciones      │
│  const dataX = useMemo(...);        // Memoizar los objetos/arrays │
│                                                                     │
│  ERRORES COMUNES:                                                   │
│  ────────────────                                                   │
│  ❌ Usar memo sin useCallback → memo es inútil                     │
│  ❌ Usar useCallback sin memo → useCallback es inútil              │
│  ❌ Optimizar todo "por si acaso" → Más lento y complejo           │
│  ❌ No incluir todas las dependencias → Bugs sutiles               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## React 19 Hooks

> 🎯 **React 19** introdujo varios hooks nuevos que simplifican patrones comunes. Estos hooks son especialmente útiles en Next.js App Router.

### use - Leer Promesas y Contexto

#### ¿Qué es use?

`use` es un hook especial que puede:
1. **Leer promesas** y suspender hasta que resuelvan
2. **Leer contexto** (alternativa a useContext)

Es el **único hook que puede llamarse condicionalmente**.

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONCEPTO: use()                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  use(promise):                                                  │
│  • "Suspende" el componente hasta que la promesa resuelve       │
│  • Trabaja con Suspense para mostrar loading                    │
│  • Lanza el error al ErrorBoundary más cercano                  │
│                                                                 │
│  use(context):                                                  │
│  • Lee un contexto (igual que useContext)                       │
│  • Puede llamarse dentro de condicionales                       │
│                                                                 │
│  // CON use()                                                   │
│  function Component({ dataPromise }) {                          │
│    const data = use(dataPromise); // Suspende si no está listo │
│    return <div>{data.name}</div>;                               │
│  }                                                              │
│                                                                 │
│  // SIN use() (el patrón viejo)                                │
│  function Component({ dataPromise }) {                          │
│    const [data, setData] = useState(null);                     │
│    const [loading, setLoading] = useState(true);               │
│    useEffect(() => {                                           │
│      dataPromise.then(d => {                                    │
│        setData(d);                                              │
│        setLoading(false);                                       │
│      });                                                        │
│    }, [dataPromise]);                                           │
│    if (loading) return <Spinner />;                             │
│    return <div>{data.name}</div>;                               │
│  }                                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Ejemplos de use

```typescript
import { use, Suspense } from 'react';

// ========================================
// EJEMPLO 1: Leer promesas con Suspense
// ========================================
// Fetch que retorna una promesa
async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}

function UserProfile({ userPromise }: { userPromise: Promise<User> }) {
  // use() "suspende" hasta que la promesa resuelve
  const user = use(userPromise);
  
  // Cuando llegamos aquí, user está garantizado que tiene valor
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

// El padre envuelve en Suspense
function UserPage({ userId }: { userId: string }) {
  // ⚠️ La promesa debe crearse FUERA del componente que usa use()
  // o memoizarse, para evitar crear una nueva en cada render
  const userPromise = useMemo(() => fetchUser(userId), [userId]);
  
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <UserProfile userPromise={userPromise} />
    </Suspense>
  );
}

// ========================================
// EJEMPLO 2: use() condicional
// ========================================
function Comments({ postId, showComments }: Props) {
  // ✅ use() puede llamarse condicionalmente (único hook que puede)
  if (showComments) {
    const commentsPromise = useMemo(() => fetchComments(postId), [postId]);
    const comments = use(commentsPromise);
    return <CommentList comments={comments} />;
  }
  
  return <button>Show Comments</button>;
}

// ========================================
// EJEMPLO 3: Leer contexto con use()
// ========================================
const ThemeContext = createContext<'light' | 'dark'>('light');

function ThemedButton({ showIcon }: { showIcon: boolean }) {
  // ✅ Puede leer contexto condicionalmente
  if (showIcon) {
    const theme = use(ThemeContext);
    return <Icon color={theme === 'dark' ? 'white' : 'black'} />;
  }
  
  return <button>Click me</button>;
}

// Con Contexto (puede ser condicional)
function Component({ showTheme }: { showTheme: boolean }) {
  if (showTheme) {
    const theme = use(ThemeContext); // ✅ Válido con use()
    return <div className={theme}>Themed content</div>;
  }
  return <div>No theme</div>;
}
```

### useTransition - Actualizaciones de Baja Prioridad

#### ¿Qué es useTransition?

`useTransition` te permite marcar actualizaciones de estado como **de baja prioridad**, permitiendo que la UI siga respondiendo mientras se procesan.

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONCEPTO: useTransition                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  EL PROBLEMA:                                                   │
│  Cuando tienes una operación costosa (filtrar 10,000 items),    │
│  la UI se congela hasta que termina.                            │
│                                                                 │
│  // El input se siente "laggy" porque el re-render es lento    │
│  const handleChange = (e) => {                                  │
│    setQuery(e.target.value);      // Actualiza input           │
│    setResults(filterItems(value)); // ¡LENTO! Bloquea la UI    │
│  };                                                             │
│                                                                 │
│  LA SOLUCIÓN:                                                   │
│  Separar en actualizaciones "urgentes" y "transiciones".        │
│                                                                 │
│  const [isPending, startTransition] = useTransition();          │
│                                                                 │
│  const handleChange = (e) => {                                  │
│    setQuery(e.target.value);  // URGENTE: input responde ya    │
│                                                                 │
│    startTransition(() => {                                      │
│      setResults(filterItems(value)); // TRANSICIÓN: puede esperar│
│    });                                                          │
│  };                                                             │
│                                                                 │
│  • isPending: true mientras la transición está en proceso       │
│  • startTransition: envuelve actualizaciones de baja prioridad  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### ¿Cuándo usar useTransition?

```
✅ USA useTransition CUANDO:

1. FILTRAR/BUSCAR EN LISTAS GRANDES
   • El usuario escribe y la lista se filtra
   • El input debe responder inmediatamente

2. NAVEGACIÓN ENTRE TABS/VISTAS
   • Cambiar de pestaña con contenido pesado
   • Mostrar indicador mientras carga

3. RENDERIZADO DE COMPONENTES COMPLEJOS
   • Actualizar un chart con muchos datos
   • Cambiar vista de lista a grid

❌ NO USES useTransition CUANDO:

1. LA OPERACIÓN YA ES RÁPIDA
   • No tiene sentido si no hay problema

2. NECESITAS EL RESULTADO INMEDIATAMENTE
   • Validaciones de formulario
   • Contadores en tiempo real
```

#### Ejemplo de useTransition

```typescript
import { useState, useTransition } from 'react';

function ProductSearch({ products }: { products: Product[] }) {
  const [query, setQuery] = useState('');
  const [filteredProducts, setFilteredProducts] = useState(products);
  const [isPending, startTransition] = useTransition();
  
  const handleSearch = (e: ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value;
    
    // ⚡ URGENTE: El input se actualiza inmediatamente
    setQuery(value);
    
    // 🐢 TRANSICIÓN: El filtrado puede tardar, no bloquea el input
    startTransition(() => {
      const filtered = products.filter(p => 
        p.name.toLowerCase().includes(value.toLowerCase()) ||
        p.description.toLowerCase().includes(value.toLowerCase())
      );
      setFilteredProducts(filtered);
    });
  };
  
  return (
    <div>
      <input 
        value={query}
        onChange={handleSearch}
        placeholder="Buscar productos..."
      />
      
      {/* Mostrar indicador mientras filtra */}
      {isPending && <span className="loading">Buscando...</span>}
      
      {/* Atenuar resultados mientras se actualizan */}
      <div style={{ opacity: isPending ? 0.7 : 1, transition: 'opacity 0.2s' }}>
        <ProductGrid products={filteredProducts} />
      </div>
    </div>
  );
}
```

### useDeferredValue - Diferir Valores

#### ¿Qué es useDeferredValue?

`useDeferredValue` es similar a `useTransition`, pero para **valores** en lugar de actualizaciones. Retorna una versión "diferida" del valor que se actualiza con menor prioridad.

```
┌─────────────────────────────────────────────────────────────────┐
│                CONCEPTO: useDeferredValue                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  useDeferredValue vs useTransition:                             │
│                                                                 │
│  useTransition:                                                 │
│  • Tú controlas CUÁNDO hacer la actualización de baja prioridad │
│  • Envuelves el setState en startTransition                     │
│                                                                 │
│  useDeferredValue:                                              │
│  • React controla cuándo actualizar                             │
│  • Solo pasas el valor, React lo difiere                        │
│  • Útil cuando NO controlas el setState (viene de props)        │
│                                                                 │
│  const deferredQuery = useDeferredValue(query);                 │
│                                                                 │
│  query:         "a" → "ab" → "abc" → "abcd" (cambia rápido)    │
│  deferredQuery: "a" → "a"  → "ab"  → "abcd" (se retrasa)       │
│                                                                 │
│  Mientras deferredQuery !== query, sabes que hay datos "stale"  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### ¿Cuándo usar useDeferredValue?

```
✅ USA useDeferredValue CUANDO:

1. EL VALOR VIENE DE PROPS (no lo controlas)
   • Un componente hijo recibe query de un padre
   
2. QUIERES DIFERIR UN VALOR PARA UN CÁLCULO COSTOSO
   • useMemo con el valor diferido
   
3. EL CÓDIGO ES MÁS SIMPLE QUE CON useTransition
   • Menos boilerplate en algunos casos

❌ PREFIERE useTransition CUANDO:

1. CONTROLAS EL setState
   • Es más explícito y fácil de entender
   
2. NECESITAS isPending
   • useDeferredValue requiere comparar valores manualmente
```

#### Ejemplo de useDeferredValue

```typescript
import { useState, useDeferredValue, useMemo } from 'react';

// CASO: El valor viene de props, no podemos usar useTransition
function SearchResults({ query }: { query: string }) {
  // Diferir el valor de query
  const deferredQuery = useDeferredValue(query);
  
  // Detectar si estamos mostrando datos "viejos"
  const isStale = query !== deferredQuery;
  
  // Calcular resultados con el valor diferido
  // Esto evita recálculos mientras el usuario escribe rápido
  const results = useMemo(() => {
    return searchDatabase(deferredQuery);
  }, [deferredQuery]); // Solo recalcula cuando deferredQuery cambia
  
  return (
    <div style={{ 
      opacity: isStale ? 0.6 : 1,
      transition: 'opacity 0.2s'
    }}>
      {isStale && <span>Actualizando...</span>}
      {results.map(item => (
        <ResultItem key={item.id} item={item} />
      ))}
    </div>
  );
}

// El padre controla el query
function SearchPage() {
  const [query, setQuery] = useState('');
  
  return (
    <div>
      <input 
        value={query} 
        onChange={e => setQuery(e.target.value)}
        placeholder="Buscar..."
      />
      <SearchResults query={query} />
    </div>
  );
}
```

### useOptimistic - Actualizaciones Optimistas

#### ¿Qué es useOptimistic?

`useOptimistic` te permite mostrar un **estado futuro esperado** mientras esperas que el servidor confirme la operación.

```
┌─────────────────────────────────────────────────────────────────┐
│                CONCEPTO: useOptimistic                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  EL PROBLEMA: La UI espera confirmación del servidor            │
│                                                                 │
│  [Usuario da like] → [Espera 200ms] → [Servidor confirma] → ❤️ │
│                      ↑                                          │
│                      El usuario ve un spinner o nada            │
│                                                                 │
│  LA SOLUCIÓN: Mostrar el resultado esperado INMEDIATAMENTE      │
│                                                                 │
│  [Usuario da like] → ❤️ (instantáneo)                          │
│                    → [Servidor confirma en background]          │
│                    → Si falla, revertir                         │
│                                                                 │
│  const [optimisticLikes, addOptimisticLike] = useOptimistic(   │
│    likes,           // Estado actual (del servidor)             │
│    (state, newVal) => state + 1  // Cómo calcular el optimista │
│  );                                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### ¿Cuándo usar useOptimistic?

```
✅ USA useOptimistic CUANDO:

1. LIKES, FAVORITOS, BOOKMARKS
   • El usuario espera retroalimentación instantánea
   • La operación casi siempre tiene éxito

2. MENSAJES DE CHAT
   • Mostrar el mensaje enviado inmediatamente
   • Indicar "enviando..." mientras confirma

3. AGREGAR/QUITAR DE LISTAS
   • Agregar al carrito
   • Eliminar items

4. TOGGLES Y SWITCHES
   • Cambiar configuraciones
   • Activar/desactivar opciones

❌ NO USES useOptimistic CUANDO:

1. LA OPERACIÓN PUEDE FALLAR FRECUENTEMENTE
   • Pagos, transferencias críticas
   • Mejor esperar confirmación

2. EL RESULTADO ES IMPREDECIBLE
   • Búsquedas, cálculos complejos
   • No sabes qué mostrar
```

#### Ejemplo de useOptimistic

```typescript
import { useOptimistic, useTransition } from 'react';

interface Message {
  id: string;
  text: string;
  sending?: boolean;  // true mientras espera confirmación
}

function Chat({ messages, onSend }: Props) {
  // Estado optimista: los mensajes que MOSTRAMOS (pueden incluir no confirmados)
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,  // Estado real del servidor
    (currentMessages, newText: string) => [
      ...currentMessages,
      { 
        id: `temp-${Date.now()}`, 
        text: newText, 
        sending: true  // Marcar como "enviando"
      }
    ]
  );
  
  const [isPending, startTransition] = useTransition();
  
  async function handleSubmit(formData: FormData) {
    const text = formData.get('message') as string;
    
    startTransition(async () => {
      // 1. Mostrar mensaje inmediatamente (optimista)
      addOptimisticMessage(text);
      
      // 2. Enviar al servidor (en background)
      await onSend(text);
      // Cuando el servidor responde, "messages" se actualiza
      // y el mensaje optimista se reemplaza con el real
    });
  }
  
  return (
    <div>
      <ul>
        {optimisticMessages.map(msg => (
          <li 
            key={msg.id}
            style={{ opacity: msg.sending ? 0.6 : 1 }}
          >
            {msg.text}
            {msg.sending && <span> ✉️ Enviando...</span>}
          </li>
        ))}
      </ul>
      
      <form action={handleSubmit}>
        <input name="message" placeholder="Escribe un mensaje..." />
        <button type="submit">Enviar</button>
      </form>
    </div>
  );
}
```

### useFormStatus - Estado del Formulario

#### ¿Qué es useFormStatus?

`useFormStatus` te da información sobre el formulario padre más cercano que está siendo enviado.

```
┌─────────────────────────────────────────────────────────────────┐
│                CONCEPTO: useFormStatus                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  useFormStatus retorna:                                         │
│                                                                 │
│  {                                                              │
│    pending: boolean,  // ¿El form está siendo enviado?          │
│    data: FormData,    // Los datos siendo enviados              │
│    method: string,    // 'get' | 'post'                         │
│    action: string     // URL del action                         │
│  }                                                              │
│                                                                 │
│  ⚠️ IMPORTANTE: Debe usarse en un componente HIJO del form     │
│                                                                 │
│  // ❌ NO funciona - mismo componente                           │
│  function Form() {                                              │
│    const { pending } = useFormStatus(); // ❌ No ve el form     │
│    return <form>...</form>;                                     │
│  }                                                              │
│                                                                 │
│  // ✅ Funciona - componente hijo                               │
│  function SubmitButton() {                                      │
│    const { pending } = useFormStatus(); // ✅ Ve el form padre  │
│    return <button disabled={pending}>...</button>;              │
│  }                                                              │
│  function Form() {                                              │
│    return <form><SubmitButton /></form>;                        │
│  }                                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Ejemplo de useFormStatus

```typescript
import { useFormStatus } from 'react-dom';

// Botón que se deshabilita durante el envío
function SubmitButton({ children }: { children: React.ReactNode }) {
  const { pending } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? (
        <>
          <Spinner size="sm" />
          Enviando...
        </>
      ) : children}
    </button>
  );
}

// Campos que se deshabilitan durante el envío
function FormFields() {
  const { pending } = useFormStatus();
  
  return (
    <fieldset disabled={pending} style={{ opacity: pending ? 0.6 : 1 }}>
      <input name="name" placeholder="Nombre" required />
      <input name="email" type="email" placeholder="Email" required />
      <textarea name="message" placeholder="Mensaje" required />
    </fieldset>
  );
}

// Formulario completo
function ContactForm() {
  async function handleSubmit(formData: FormData) {
    'use server'; // Server Action en Next.js
    await sendEmail(formData);
  }
  
  return (
    <form action={handleSubmit}>
      <FormFields />
      <SubmitButton>Enviar mensaje</SubmitButton>
    </form>
  );
}
```

### useActionState - Estado de Server Actions

#### ¿Qué es useActionState?

`useActionState` maneja el estado de server actions, incluyendo errores de validación y mensajes de éxito.

```
┌─────────────────────────────────────────────────────────────────┐
│                CONCEPTO: useActionState                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  useActionState conecta un form con una server action y         │
│  maneja el estado entre envíos.                                 │
│                                                                 │
│  const [state, formAction, isPending] = useActionState(         │
│    serverAction,   // La función que procesa el form            │
│    initialState    // Estado inicial                            │
│  );                                                             │
│                                                                 │
│  • state: Resultado de la última ejecución                      │
│  • formAction: Función para pasar al form action                │
│  • isPending: true mientras la action se ejecuta                │
│                                                                 │
│  La server action recibe:                                       │
│  async function serverAction(prevState, formData) {             │
│    // prevState: Estado anterior                                │
│    // formData: Datos del formulario                            │
│    return newState; // Esto se convierte en el nuevo state      │
│  }                                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Ejemplo completo de useActionState

```typescript
import { useActionState } from 'react';

// Tipo del estado del formulario
interface FormState {
  success: boolean;
  message: string;
  errors?: {
    name?: string[];
    email?: string[];
    password?: string[];
  };
}

// Server action que valida y procesa el registro
async function registerAction(
  prevState: FormState,
  formData: FormData
): Promise<FormState> {
  const name = formData.get('name') as string;
  const email = formData.get('email') as string;
  const password = formData.get('password') as string;
  
  // Validación
  const errors: FormState['errors'] = {};
  
  if (!name || name.length < 2) {
    errors.name = ['El nombre debe tener al menos 2 caracteres'];
  }
  
  if (!email || !email.includes('@')) {
    errors.email = ['Email inválido'];
  }
  
  if (!password || password.length < 8) {
    errors.password = ['La contraseña debe tener al menos 8 caracteres'];
  }
  
  // Si hay errores, retornarlos
  if (Object.keys(errors).length > 0) {
    return {
      success: false,
      message: 'Por favor corrige los errores',
      errors
    };
  }
  
  // Intentar registrar
  try {
    await createUser({ name, email, password });
    return {
      success: true,
      message: '¡Registro exitoso! Revisa tu email.'
    };
  } catch (error) {
    return {
      success: false,
      message: 'Error al registrar. Intenta de nuevo.'
    };
  }
}

// Componente del formulario
function RegisterForm() {
  const [state, formAction, isPending] = useActionState(
    registerAction,
    { success: false, message: '' }  // Estado inicial
  );
  
  return (
    <form action={formAction}>
      {/* Mensaje general */}
      {state.message && (
        <div className={state.success ? 'success' : 'error'}>
          {state.message}
        </div>
      )}
      
      {/* Campo nombre */}
      <div>
        <label htmlFor="name">Nombre</label>
        <input 
          id="name"
          name="name" 
          disabled={isPending}
        />
        {state.errors?.name && (
          <span className="error">{state.errors.name[0]}</span>
        )}
      </div>
      
      {/* Campo email */}
      <div>
        <label htmlFor="email">Email</label>
        <input 
          id="email"
          name="email" 
          type="email"
          disabled={isPending}
        />
        {state.errors?.email && (
          <span className="error">{state.errors.email[0]}</span>
        )}
      </div>
      
      {/* Campo password */}
      <div>
        <label htmlFor="password">Contraseña</label>
        <input 
          id="password"
          name="password" 
          type="password"
          disabled={isPending}
        />
        {state.errors?.password && (
          <span className="error">{state.errors.password[0]}</span>
        )}
      </div>
      
      {/* Botón submit */}
      <button type="submit" disabled={isPending}>
        {isPending ? 'Registrando...' : 'Registrarse'}
      </button>
    </form>
  );
}
```

### useId - IDs Únicos para Accesibilidad

#### ¿Qué es useId?

`useId` genera un ID único que es estable entre servidor y cliente, perfecto para conectar labels con inputs.

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONCEPTO: useId                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  EL PROBLEMA:                                                   │
│  Necesitas IDs únicos para accesibilidad (aria-*, htmlFor).     │
│  Pero Math.random() genera IDs diferentes en servidor/cliente.  │
│                                                                 │
│  // ❌ Genera hydration mismatch                                │
│  const id = `input-${Math.random()}`;                           │
│                                                                 │
│  LA SOLUCIÓN:                                                   │
│  useId genera IDs estables y únicos.                            │
│                                                                 │
│  const id = useId(); // ":r1:", ":r2:", etc.                    │
│                                                                 │
│  ⚠️ NO usar para keys en listas                                │
│     Los keys deben venir de tus datos                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Ejemplo de useId

```typescript
import { useId } from 'react';

// Componente de campo de formulario reutilizable
function FormField({ 
  label, 
  type = 'text',
  hint,
  error 
}: Props) {
  const id = useId();
  
  return (
    <div className="form-field">
      <label htmlFor={id}>{label}</label>
      
      <input 
        id={id}
        type={type}
        aria-describedby={hint ? `${id}-hint` : undefined}
        aria-invalid={!!error}
        aria-errormessage={error ? `${id}-error` : undefined}
      />
      
      {hint && (
        <p id={`${id}-hint`} className="hint">
          {hint}
        </p>
      )}
      
      {error && (
        <p id={`${id}-error`} className="error" role="alert">
          {error}
        </p>
      )}
    </div>
  );
}

// Uso
function RegistrationForm() {
  return (
    <form>
      <FormField 
        label="Email" 
        type="email"
        hint="Usaremos este email para contactarte"
      />
      <FormField 
        label="Contraseña" 
        type="password"
        hint="Mínimo 8 caracteres"
        error="La contraseña es muy corta"
      />
    </form>
  );
}
```

---

## Custom Hooks

### ¿Qué son los Custom Hooks?

Los Custom Hooks son funciones que encapsulan lógica reutilizable usando otros hooks.

```
┌─────────────────────────────────────────────────────────────────┐
│                CONCEPTO: Custom Hooks                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  REGLAS:                                                        │
│  1. Deben empezar con "use" (useCounter, useFetch, etc.)       │
│  2. Pueden usar otros hooks (useState, useEffect, etc.)         │
│  3. Cada instancia tiene su PROPIO estado aislado               │
│                                                                 │
│  ¿POR QUÉ CREAR CUSTOM HOOKS?                                   │
│                                                                 │
│  1. REUTILIZACIÓN                                               │
│     Usar la misma lógica en múltiples componentes               │
│                                                                 │
│  2. SEPARACIÓN DE CONCERNS                                      │
│     Sacar lógica compleja fuera del componente                  │
│                                                                 │
│  3. TESTING                                                     │
│     Más fácil probar lógica aislada                             │
│                                                                 │
│  4. LEGIBILIDAD                                                 │
│     Componentes más limpios y declarativos                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Ejemplos de Custom Hooks Útiles

```typescript
// ========================================
// useToggle - Estado booleano con helpers
// ========================================
function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);
  
  const toggle = useCallback(() => setValue(v => !v), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);
  
  return { value, toggle, setTrue, setFalse };
}

// Uso
function Modal() {
  const { value: isOpen, toggle, setFalse: close } = useToggle();
  
  return (
    <>
      <button onClick={toggle}>Toggle Modal</button>
      {isOpen && <Dialog onClose={close} />}
    </>
  );
}

// ========================================
// useLocalStorage - Persistir estado
// ========================================
function useLocalStorage<T>(key: string, initialValue: T) {
  // Leer valor inicial de localStorage
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });
  
  // Guardar en localStorage cuando cambia
  const setValue = useCallback((value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error('Error saving to localStorage:', error);
    }
  }, [key, storedValue]);
  
  return [storedValue, setValue] as const;
}

// Uso
function Settings() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  const [fontSize, setFontSize] = useLocalStorage('fontSize', 16);
  
  return (
    <div>
      <select value={theme} onChange={e => setTheme(e.target.value)}>
        <option value="light">Claro</option>
        <option value="dark">Oscuro</option>
      </select>
      
      <input 
        type="range" 
        value={fontSize}
        onChange={e => setFontSize(Number(e.target.value))}
        min={12}
        max={24}
      />
    </div>
  );
}

// ========================================
// useFetch - Fetching con estados
// ========================================
interface UseFetchResult<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
  refetch: () => void;
}

function useFetch<T>(url: string): UseFetchResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);
  
  const fetchData = useCallback(async () => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await fetch(url);
      if (!response.ok) throw new Error('Network response was not ok');
      const json = await response.json();
      setData(json);
    } catch (err) {
      setError(err instanceof Error ? err : new Error('Unknown error'));
    } finally {
      setLoading(false);
    }
  }, [url]);
  
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  
  return { data, loading, error, refetch: fetchData };
}

// Uso
function UserProfile({ userId }: { userId: string }) {
  const { data: user, loading, error, refetch } = useFetch<User>(
    `/api/users/${userId}`
  );
  
  if (loading) return <Spinner />;
  if (error) return <Error message={error.message} onRetry={refetch} />;
  if (!user) return <NotFound />;
  
  return <UserCard user={user} />;
}

// ========================================
// useDebounce - Diferir valores
// ========================================
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => clearTimeout(timer);
  }, [value, delay]);
  
  return debouncedValue;
}

// Uso
function SearchInput() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 300);
  
  // Este efecto solo se ejecuta 300ms después de que el usuario
  // deja de escribir
  useEffect(() => {
    if (debouncedQuery) {
      searchAPI(debouncedQuery);
    }
  }, [debouncedQuery]);
  
  return (
    <input 
      value={query}
      onChange={e => setQuery(e.target.value)}
      placeholder="Buscar..."
    />
  );
}

// ========================================
// useMediaQuery - Responsive hooks
// ========================================
function useMediaQuery(query: string): boolean {
  const [matches, setMatches] = useState(false);
  
  useEffect(() => {
    const media = window.matchMedia(query);
    
    // Set initial value
    setMatches(media.matches);
    
    // Listen for changes
    const listener = (e: MediaQueryListEvent) => setMatches(e.matches);
    media.addEventListener('change', listener);
    
    return () => media.removeEventListener('change', listener);
  }, [query]);
  
  return matches;
}

// Uso
function ResponsiveLayout() {
  const isMobile = useMediaQuery('(max-width: 768px)');
  const isDarkMode = useMediaQuery('(prefers-color-scheme: dark)');
  
  return (
    <div className={isDarkMode ? 'dark' : 'light'}>
      {isMobile ? <MobileNav /> : <DesktopNav />}
    </div>
  );
}

// ========================================
// usePrevious - Valor anterior
// ========================================
function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T>();
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref.current;
}

// Uso
function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);
  
  return (
    <div>
      <p>Actual: {count}</p>
      <p>Anterior: {prevCount}</p>
      <button onClick={() => setCount(c => c + 1)}>+1</button>
    </div>
  );
}
```

---

## 🏷️ Tags

#programming #frontend #react #hooks #javascript #typescript #react19

---

## 📚 Ver También

- [[React Complete Guide|React - Guía Completa]]
- [[React Patterns Guide|React Patterns - Patrones Avanzados]]
- [[Next.js Complete Guide|Next.js - Guía Completa]]
