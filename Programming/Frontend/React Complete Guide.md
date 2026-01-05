# React Complete Guide

> **Guía profesional de React 19 para desarrollo y entrevistas técnicas**

---

## 📚 Tabla de Contenidos

1. [[#¿Qué es React y Por Qué Existe?|Conceptos Fundamentales]]
2. [[#JSX en Profundidad|JSX]]
3. [[#Componentes|Componentes]]
4. [[#Props y Children|Props]]
5. [[#Estado y Ciclo de Vida|Estado]]
6. [[#Renderizado Condicional|Renderizado]]
7. [[#Listas y Keys|Listas]]
8. [[#Eventos|Eventos]]
9. [[#Formularios|Formularios]]
10. [[#Composición vs Herencia|Composición]]

---

## ¿Qué es React y Por Qué Existe?

### El Problema que React Resuelve

Antes de React, construir interfaces de usuario dinámicas era **doloroso**:

```
┌─────────────────────────────────────────────────────────────────┐
│              EL PROBLEMA: MANIPULACIÓN MANUAL DEL DOM           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  // JavaScript vanilla - actualizar una lista de tareas        │
│                                                                 │
│  function addTask(task) {                                       │
│    // 1. Crear elemento                                         │
│    const li = document.createElement('li');                     │
│    li.textContent = task.text;                                  │
│    li.id = `task-${task.id}`;                                   │
│                                                                 │
│    // 2. Crear botón de eliminar                                │
│    const deleteBtn = document.createElement('button');          │
│    deleteBtn.textContent = 'X';                                 │
│    deleteBtn.onclick = () => removeTask(task.id);               │
│    li.appendChild(deleteBtn);                                   │
│                                                                 │
│    // 3. Agregarlo al DOM                                       │
│    document.getElementById('task-list').appendChild(li);        │
│                                                                 │
│    // 4. Actualizar contador                                    │
│    document.getElementById('task-count').textContent =          │
│      document.querySelectorAll('#task-list li').length;         │
│  }                                                              │
│                                                                 │
│  PROBLEMAS:                                                     │
│  • Código imperativo difícil de seguir                          │
│  • Fácil perder sincronía entre datos y UI                      │
│  • Cada cambio requiere saber QUÉ y DÓNDE actualizar            │
│  • Propenso a bugs cuando la app crece                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### La Solución: Programación Declarativa

React cambia el paradigma: **tú describes QUÉ quieres ver, React se encarga del CÓMO**.

```typescript
// ✅ CON REACT - Declarativo
function TaskList() {
  const [tasks, setTasks] = useState([]);
  
  // Solo describes el estado deseado
  // React actualiza el DOM automáticamente
  return (
    <div>
      <p>Total: {tasks.length}</p>
      <ul>
        {tasks.map(task => (
          <li key={task.id}>
            {task.text}
            <button onClick={() => removeTask(task.id)}>X</button>
          </li>
        ))}
      </ul>
    </div>
  );
}

// Cuando tasks cambia:
// 1. React re-renderiza el componente
// 2. Compara el nuevo JSX con el anterior
// 3. Actualiza SOLO lo que cambió en el DOM real
```

### Conceptos Clave de React

```
┌─────────────────────────────────────────────────────────────────┐
│                 PILARES DE REACT                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. DECLARATIVO                                                 │
│     Describes el estado final, no los pasos para llegar         │
│     UI = f(state) → La UI es una función del estado             │
│                                                                 │
│  2. COMPONENTES                                                 │
│     La UI se divide en piezas reutilizables e independientes    │
│     Cada componente maneja su propia lógica y presentación      │
│                                                                 │
│  3. FLUJO UNIDIRECCIONAL                                        │
│     Los datos fluyen hacia abajo (padre → hijo)                 │
│     Los eventos fluyen hacia arriba (hijo → padre)              │
│                                                                 │
│  4. VIRTUAL DOM                                                 │
│     React mantiene una copia ligera del DOM                     │
│     Compara cambios y actualiza solo lo necesario               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Virtual DOM Explicado

```
┌─────────────────────────────────────────────────────────────────┐
│                    ¿QUÉ ES EL VIRTUAL DOM?                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  El Virtual DOM es una representación en JavaScript del DOM     │
│  real. Es un OBJETO que describe cómo debería verse la UI.      │
│                                                                 │
│  // Esto:                                                       │
│  <div className="card">                                         │
│    <h1>Título</h1>                                              │
│  </div>                                                         │
│                                                                 │
│  // Se convierte en este objeto (Virtual DOM):                  │
│  {                                                              │
│    type: 'div',                                                 │
│    props: {                                                     │
│      className: 'card',                                         │
│      children: {                                                │
│        type: 'h1',                                              │
│        props: { children: 'Título' }                            │
│      }                                                          │
│    }                                                            │
│  }                                                              │
│                                                                 │
│  PROCESO:                                                       │
│  1. Estado cambia → React crea nuevo Virtual DOM                │
│  2. DIFFING: Compara nuevo vs anterior                          │
│  3. RECONCILIATION: Calcula cambios mínimos                     │
│  4. Solo actualiza las partes necesarias del DOM real           │
│                                                                 │
│  ¿POR QUÉ ES RÁPIDO?                                            │
│  • Manipular objetos JS es muy rápido                           │
│  • Manipular el DOM real es lento                               │
│  • React minimiza las operaciones en el DOM real                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```typescript
// Ejemplo de cómo React optimiza actualizaciones

function UserList({ users }) {
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

// Si users pasa de [A, B, C] a [A, B, C, D]:
// React NO recrea toda la lista
// Solo agrega el nuevo <li> para D

// Si users pasa de [A, B, C] a [A, X, C]:
// React NO recrea A y C
// Solo actualiza el texto del segundo <li>

// Por eso los KEYS son importantes:
// Le dicen a React qué elementos son los mismos entre renders
```

### React vs Otras Opciones

```
┌─────────────────────────────────────────────────────────────────────┐
│                    COMPARACIÓN: REACT vs OTROS                      │
├──────────────┬────────────────┬──────────────┬──────────────────────┤
│              │     REACT      │     VUE      │      ANGULAR         │
├──────────────┼────────────────┼──────────────┼──────────────────────┤
│ Tipo         │ Biblioteca     │ Framework    │ Framework completo   │
│              │ (solo UI)      │ progresivo   │                      │
├──────────────┼────────────────┼──────────────┼──────────────────────┤
│ Curva de     │ Media          │ Baja         │ Alta                 │
│ aprendizaje  │                │              │                      │
├──────────────┼────────────────┼──────────────┼──────────────────────┤
│ Flexibilidad │ Alta (elige    │ Media        │ Baja (todo incluido) │
│              │ tus librerías) │              │                      │
├──────────────┼────────────────┼──────────────┼──────────────────────┤
│ Ecosistema   │ Enorme         │ Grande       │ Completo             │
├──────────────┼────────────────┼──────────────┼──────────────────────┤
│ Mercado      │ #1 en demanda  │ #2           │ #3 (empresas grandes)│
│ laboral      │                │              │                      │
├──────────────┼────────────────┼──────────────┼──────────────────────┤
│ Ideal para   │ SPAs, apps     │ SPAs, apps   │ Apps empresariales   │
│              │ complejas      │ medianas     │ grandes              │
└──────────────┴────────────────┴──────────────┴──────────────────────┘
```

---

## JSX en Profundidad

### ¿Qué es JSX?

JSX (JavaScript XML) es una **extensión de sintaxis** que permite escribir markup similar a HTML dentro de JavaScript. **No es HTML**, es JavaScript con azúcar sintáctico.

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONCEPTO: JSX                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  JSX es azúcar sintáctico para React.createElement()            │
│                                                                 │
│  // Esto:                                                       │
│  const element = <h1 className="title">Hello, {name}!</h1>;     │
│                                                                 │
│  // Se transpila (convierte) a:                                 │
│  const element = React.createElement(                           │
│    'h1',                    // tipo de elemento                 │
│    { className: 'title' },  // props                            │
│    'Hello, ',               // children                         │
│    name,                                                        │
│    '!'                                                          │
│  );                                                             │
│                                                                 │
│  // El resultado es un OBJETO (Virtual DOM):                    │
│  {                                                              │
│    type: 'h1',                                                  │
│    props: {                                                     │
│      className: 'title',                                        │
│      children: ['Hello, ', 'Juan', '!']                         │
│    }                                                            │
│  }                                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Diferencias entre JSX y HTML

```typescript
// ========================================
// DIFERENCIAS CLAVE
// ========================================

// 1. className en lugar de class
<div className="container">...</div>  // ✅ JSX
<div class="container">...</div>      // ❌ HTML (no funciona en JSX)

// 2. htmlFor en lugar de for
<label htmlFor="email">Email</label>  // ✅ JSX
<label for="email">Email</label>      // ❌ HTML

// 3. camelCase para atributos
<input tabIndex={0} />                // ✅ JSX (tabIndex)
<input tabindex="0" />                // ❌ HTML (tabindex)

// 4. style es un objeto, no string
<div style={{ backgroundColor: 'red', fontSize: '16px' }}>...</div>  // ✅ JSX
<div style="background-color: red; font-size: 16px;">...</div>       // ❌ HTML

// 5. Todos los tags deben cerrarse
<img src="..." />    // ✅ Debe cerrarse
<br />               // ✅ Debe cerrarse
<input />            // ✅ Debe cerrarse

// 6. Expresiones JavaScript van entre llaves {}
<p>El resultado es: {2 + 2}</p>        // → "El resultado es: 4"
<p>Hola, {user.name.toUpperCase()}</p> // → "Hola, JUAN"
```

### Reglas de JSX

```typescript
// ✅ 1. Retornar un solo elemento raíz
function App() {
  return (
    <div>
      <Header />
      <Main />
    </div>
  );
}

// ✅ Usar Fragment para evitar div extra
function App() {
  return (
    <>
      <Header />
      <Main />
    </>
  );
}

// ✅ 2. Cerrar todas las etiquetas
<img src="photo.jpg" />
<input type="text" />
<br />

// ✅ 3. camelCase para atributos
<div className="container" tabIndex={0} onClick={handleClick}>
  <label htmlFor="input">Name</label>
</div>

// ✅ 4. Expresiones JavaScript con {}
const name = "React";
const element = <h1>Hello, {name.toUpperCase()}!</h1>;

// ✅ 5. Estilos como objeto
const styles = { backgroundColor: 'blue', fontSize: '16px' };
<div style={styles}>Styled</div>

// ✅ Inline
<div style={{ backgroundColor: 'blue', fontSize: '16px' }}>Styled</div>
```

### Expresiones en JSX

```typescript
function UserProfile({ user }) {
  return (
    <div>
      {/* Variables */}
      <h1>{user.name}</h1>
      
      {/* Expresiones */}
      <p>Age: {user.age * 2}</p>
      
      {/* Ternarios */}
      <span>{user.isAdmin ? 'Admin' : 'User'}</span>
      
      {/* Llamadas a funciones */}
      <p>{formatDate(user.createdAt)}</p>
      
      {/* Template literals */}
      <p>{`Welcome, ${user.name}!`}</p>
      
      {/* Arrays */}
      {user.hobbies.map(hobby => <span key={hobby}>{hobby}</span>)}
      
      {/* Objetos (NO directamente, necesitan stringify o acceso a propiedades) */}
      <pre>{JSON.stringify(user, null, 2)}</pre>
    </div>
  );
}
```

---

## Componentes

### ¿Qué es un Componente?

Un **componente** es una pieza independiente y reutilizable de UI. Piensa en ellos como **funciones que reciben datos y retornan UI**.

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONCEPTO: COMPONENTES                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Un componente es como una FUNCIÓN:                             │
│                                                                 │
│  ENTRADA (Props) → COMPONENTE → SALIDA (JSX/UI)                 │
│                                                                 │
│  function Saludo({ nombre }) {    // Recibe props              │
│    return <h1>Hola, {nombre}</h1>; // Retorna UI                │
│  }                                                              │
│                                                                 │
│  <Saludo nombre="Juan" />  →  <h1>Hola, Juan</h1>              │
│                                                                 │
│  CARACTERÍSTICAS:                                               │
│  • REUTILIZABLES: Úsalo múltiples veces con diferentes datos   │
│  • COMPOSABLES: Combina componentes pequeños en grandes         │
│  • AISLADOS: Cada uno tiene su propia lógica                    │
│  • DECLARATIVOS: Describes QUÉ mostrar, no CÓMO                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### ¿Por Qué Usar Componentes?

```
┌─────────────────────────────────────────────────────────────────┐
│              BENEFICIOS DE LOS COMPONENTES                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. REUTILIZACIÓN                                               │
│     Crea un botón una vez, úsalo en toda la app                 │
│                                                                 │
│     <Button>Guardar</Button>                                    │
│     <Button>Cancelar</Button>                                   │
│     <Button>Eliminar</Button>                                   │
│                                                                 │
│  2. MANTENIBILIDAD                                              │
│     Cambias el botón en un lugar, cambia en toda la app         │
│                                                                 │
│  3. TESTING                                                     │
│     Puedes probar cada componente de forma aislada              │
│                                                                 │
│  4. SEPARACIÓN DE CONCERNS                                      │
│     Cada componente tiene una sola responsabilidad              │
│                                                                 │
│  5. COLABORACIÓN                                                │
│     Diferentes desarrolladores trabajan en diferentes           │
│     componentes sin conflictos                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Tipos de Componentes

```typescript
// ========================================
// 1. COMPONENTES FUNCIONALES (RECOMENDADO - Moderno)
// ========================================
// Son funciones que retornan JSX

function Welcome({ name }: { name: string }) {
  return <h1>Hello, {name}</h1>;
}

// Arrow function (equivalente)
const Welcome = ({ name }: { name: string }) => {
  return <h1>Hello, {name}</h1>;
};

// Con return implícito (cuando es una sola expresión)
const Welcome = ({ name }: { name: string }) => <h1>Hello, {name}</h1>;

// ========================================
// 2. COMPONENTES DE CLASE (LEGACY - Solo para código antiguo)
// ========================================
// Antes de los hooks (React 16.8), era la única forma de tener estado

class Welcome extends React.Component<{ name: string }> {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}

// ⚠️ NO uses clases en código nuevo
// Los hooks reemplazaron completamente la necesidad de clases
```

### Anatomía de un Componente Profesional

```typescript
// src/components/UserCard/UserCard.tsx

// 1. IMPORTS
import { memo, useCallback, useMemo } from 'react';
import type { FC } from 'react';
import styles from './UserCard.module.css';

// 2. TIPOS (interfaces para props)
interface User {
  id: string;
  name: string;
  email: string;
  avatar?: string;
}

interface UserCardProps {
  user: User;
  onSelect?: (user: User) => void;
  isSelected?: boolean;
  className?: string;
}

// 3. COMPONENTE
export const UserCard: FC<UserCardProps> = memo(({ 
  user, 
  onSelect, 
  isSelected = false,  // Valores por defecto
  className = ''
}) => {
  // 4. LÓGICA (hooks, cálculos, callbacks)
  const initials = useMemo(() => 
    user.name.split(' ').map(n => n[0]).join('').toUpperCase(),
    [user.name]
  );
  
  const handleClick = useCallback(() => {
    onSelect?.(user);
  }, [onSelect, user]);
  
  // 5. RENDER (siempre al final)
  return (
    <article 
      className={`${styles.card} ${isSelected ? styles.selected : ''} ${className}`}
      onClick={handleClick}
      role="button"
      tabIndex={0}
      aria-selected={isSelected}
    >
      <div className={styles.avatar}>
        {user.avatar ? (
          <img src={user.avatar} alt={user.name} />
        ) : (
          <span>{initials}</span>
        )}
      </div>
      <div className={styles.info}>
        <h3>{user.name}</h3>
        <p>{user.email}</p>
      </div>
    </article>
  );
});

// 6. DISPLAY NAME (para debugging)
UserCard.displayName = 'UserCard';
```

---

## Props y Children

### ¿Qué son las Props?

**Props** (propiedades) son los datos que pasan de un componente padre a un hijo. Son **de solo lectura** - un componente nunca debe modificar sus propias props.

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONCEPTO: PROPS                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Props = Argumentos que pasas a un componente                   │
│                                                                 │
│  // Es como llamar una función:                                 │
│  saludo("Juan", 25)     ←→     <Saludo nombre="Juan" edad={25}/>│
│                                                                 │
│  REGLAS:                                                        │
│  1. INMUTABLES: No puedes modificar props dentro del componente │
│  2. UNIDIRECCIONALES: Fluyen de padre a hijo                    │
│  3. CUALQUIER VALOR: Strings, números, objetos, funciones, JSX │
│                                                                 │
│  // ❌ NUNCA hagas esto:                                        │
│  function Componente({ nombre }) {                              │
│    nombre = "Otro";  // ERROR - props son de solo lectura       │
│  }                                                              │
│                                                                 │
│  // ✅ Si necesitas modificar, usa estado local:               │
│  function Componente({ nombreInicial }) {                       │
│    const [nombre, setNombre] = useState(nombreInicial);         │
│  }                                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Props Fundamentales con TypeScript

```typescript
// ========================================
// DEFINICIÓN DE TIPOS PARA PROPS
// ========================================
interface ButtonProps {
  // Props requeridas (sin ?)
  children: React.ReactNode;    // El contenido del botón
  onClick: () => void;          // Función a ejecutar al hacer click
  
  // Props opcionales (con ?)
  variant?: 'primary' | 'secondary' | 'danger';  // Union type
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  loading?: boolean;
  
  // Atributos HTML nativos
  type?: 'button' | 'submit' | 'reset';
  className?: string;
}

// ========================================
// COMPONENTE CON PROPS TIPADAS
// ========================================
function Button({
  children,
  onClick,
  variant = 'primary',    // Valor por defecto
  size = 'md',
  disabled = false,
  loading = false,
  type = 'button',
  className = '',
}: ButtonProps) {
  return (
    <button
      type={type}
      onClick={onClick}
      disabled={disabled || loading}
      className={`btn btn-${variant} btn-${size} ${className}`}
    >
      {loading ? <Spinner /> : children}
    </button>
  );
}

// ========================================
// USOS
// ========================================
// Solo props requeridas
<Button onClick={handleSave}>Guardar</Button>

// Con props opcionales
<Button 
  onClick={handleDelete} 
  variant="danger"
  disabled={!canDelete}
>
  Eliminar
</Button>

// Con className extra
<Button onClick={handleSubmit} className="mt-4">
  Enviar
</Button>
```

### Children: El Contenido de un Componente

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONCEPTO: CHILDREN                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  children es una prop especial que representa el contenido      │
│  que pones ENTRE las etiquetas de un componente.                │
│                                                                 │
│  <Card>                                                         │
│    <p>Este es el children</p>   ← Todo esto es children         │
│    <button>Click me</button>                                    │
│  </Card>                                                        │
│                                                                 │
│  // Dentro de Card:                                             │
│  function Card({ children }) {                                  │
│    return (                                                     │
│      <div className="card">                                     │
│        {children}  ← Se renderiza aquí                          │
│      </div>                                                     │
│    );                                                           │
│  }                                                              │
│                                                                 │
│  TIPOS DE CHILDREN:                                             │
│  • React.ReactNode: Cualquier cosa renderizable                 │
│  • React.ReactElement: Solo elementos JSX                       │
│  • string: Solo texto                                           │
│  • Function: Render props pattern                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```typescript
// ========================================
// EJEMPLO: Card con children
// ========================================
interface CardProps {
  children: React.ReactNode;  // Acepta cualquier contenido
  title?: string;
}

function Card({ children, title }: CardProps) {
  return (
    <div className="card">
      {title && <h2 className="card-title">{title}</h2>}
      <div className="card-content">
        {children}
      </div>
    </div>
  );
}

// Uso
<Card title="Información del Usuario">
  <p>Nombre: Juan</p>
  <p>Email: juan@ejemplo.com</p>
  <button onClick={handleEdit}>Editar</button>
</Card>
```

// Children como función (Render Props)
interface DataFetcherProps<T> {
  url: string;
  children: (data: T | null, loading: boolean, error: Error | null) => React.ReactNode;
}

function DataFetcher<T>({ url, children }: DataFetcherProps<T>) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);
  
  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [url]);
  
  return <>{children(data, loading, error)}</>;
}

// Uso
<DataFetcher<User[]> url="/api/users">
  {(users, loading, error) => {
    if (loading) return <Spinner />;
    if (error) return <Error message={error.message} />;
    return <UserList users={users!} />;
  }}
</DataFetcher>
```

### Prop Drilling vs Composition

```typescript
// ❌ Prop Drilling (evitar)
function App() {
  const [user, setUser] = useState(null);
  return <Layout user={user} setUser={setUser} />;
}

function Layout({ user, setUser }) {
  return <Sidebar user={user} setUser={setUser} />;
}

function Sidebar({ user, setUser }) {
  return <UserMenu user={user} setUser={setUser} />;
}

// ✅ Composition Pattern
function App() {
  const [user, setUser] = useState(null);
  
  return (
    <Layout>
      <Sidebar>
        <UserMenu user={user} onLogout={() => setUser(null)} />
      </Sidebar>
    </Layout>
  );
}

function Layout({ children }) {
  return <div className="layout">{children}</div>;
}

function Sidebar({ children }) {
  return <aside className="sidebar">{children}</aside>;
}
```

---

## Estado y Ciclo de Vida

### ¿Qué es el Estado?

**Estado** es la memoria de un componente. Son datos que pueden cambiar durante la vida del componente, y cuando cambian, React re-renderiza el componente para reflejar los nuevos valores.

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONCEPTO: ESTADO                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  PROPS vs ESTADO:                                               │
│                                                                 │
│  Props       │  Estado                                          │
│  ─────────────────────────────                                  │
│  Inmutables  │  Puede cambiar                                   │
│  Del padre   │  Interno del componente                          │
│  De lectura  │  Controlado por el componente                    │
│                                                                 │
│  ¿QUÉ DEBE SER ESTADO?                                          │
│  Pregúntate:                                                    │
│  1. ¿Cambia con el tiempo? Si no → NO es estado                 │
│  2. ¿Puede calcularse de props u otro estado? → NO es estado    │
│  3. ¿Necesitas que el componente se actualice? → SÍ es estado   │
│                                                                 │
│  EJEMPLOS:                                                      │
│  • Valor de input       → Estado (cambia con usuario)           │
│  • Lista de items       → Estado (puede añadirse/eliminarse)    │
│  • Modal abierto/cerrado → Estado (toggle)                      │
│  • Nombre del usuario   → Props (viene del padre)               │
│  • Total de precios     → Calculado (no es estado)              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### ¿Cómo Funciona el Re-render?

```
┌─────────────────────────────────────────────────────────────────┐
│               CICLO DE RE-RENDERIZADO                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. EVENTO        →  Usuario hace click                         │
│                      ↓                                          │
│  2. SET STATE     →  setCount(count + 1)                        │
│                      ↓                                          │
│  3. RE-RENDER     →  React llama tu componente de nuevo         │
│                      ↓                                          │
│  4. VIRTUAL DOM   →  Crea nuevo árbol virtual                   │
│                      ↓                                          │
│  5. DIFFING       →  Compara con el anterior                    │
│                      ↓                                          │
│  6. DOM UPDATE    →  Solo actualiza lo que cambió               │
│                                                                 │
│  IMPORTANTE:                                                    │
│  • setState es ASÍNCRONO (no inmediato)                         │
│  • Los re-renders son BARATOS (gracias al Virtual DOM)          │
│  • Si no hay cambios de estado/props, no hay re-render          │
│                                                                 │
│  // ❌ Esto NO funciona como esperas:                           │
│  setCount(count + 1);                                           │
│  console.log(count);  // Sigue siendo el valor ANTERIOR         │
│                                                                 │
│  // ✅ Para acceder al nuevo valor, usa un efecto:              │
│  useEffect(() => {                                              │
│    console.log(count);  // Ahora sí es el valor actualizado     │
│  }, [count]);                                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### useState: El Hook Básico de Estado

```
┌─────────────────────────────────────────────────────────────────┐
│                   CUÁNDO USAR useState                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ✅ USAR useState CUANDO:                                       │
│  • Estado simple (booleanos, strings, números)                  │
│  • Objetos/arrays pequeños                                      │
│  • Actualizaciones independientes                               │
│  • Un componente pequeño/mediano                                │
│                                                                 │
│  ❌ CONSIDERAR useReducer CUANDO:                               │
│  • Múltiples valores de estado relacionados                     │
│  • Lógica de actualización compleja                             │
│  • El siguiente estado depende del anterior                     │
│  • Quieres testear la lógica de estado por separado             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```typescript
import { useState } from 'react';

function StateExamples() {
  // Primitivos
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');
  const [isOpen, setIsOpen] = useState(false);
  
  // Objetos
  const [user, setUser] = useState<User | null>(null);
  const [form, setForm] = useState({ email: '', password: '' });
  
  // Arrays
  const [items, setItems] = useState<string[]>([]);
  
  // Lazy initialization (para cálculos costosos)
  const [data, setData] = useState(() => {
    const saved = localStorage.getItem('data');
    return saved ? JSON.parse(saved) : initialData;
  });
  
  // Actualización basada en estado previo
  const increment = () => setCount(prev => prev + 1);
  
  // Actualizar objeto (spread operator)
  const updateEmail = (email: string) => {
    setForm(prev => ({ ...prev, email }));
  };
  
  // Actualizar array
  const addItem = (item: string) => {
    setItems(prev => [...prev, item]);
  };
  
  const removeItem = (index: number) => {
    setItems(prev => prev.filter((_, i) => i !== index));
  };
  
  const updateItem = (index: number, newValue: string) => {
    setItems(prev => prev.map((item, i) => i === index ? newValue : item));
  };
}
```

### useReducer para Estado Complejo

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONCEPTO: REDUCER                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Un REDUCER es una función pura que calcula el nuevo estado     │
│  basado en el estado actual y una acción.                       │
│                                                                 │
│  (estado actual, acción) → nuevo estado                         │
│                                                                 │
│  ¿POR QUÉ USARLO?                                               │
│  • Cuando tienes múltiples estados relacionados                 │
│  • Cuando la lógica de actualización es compleja                │
│  • Cuando quieres centralizar las actualizaciones de estado     │
│  • Cuando quieres hacer el estado más predecible y testeable    │
│                                                                 │
│  VOCABULARIO:                                                   │
│  • Reducer: La función que calcula el nuevo estado              │
│  • Action: Un objeto que describe QUÉ pasó ({type, payload})    │
│  • Dispatch: La función que envía acciones al reducer           │
│                                                                 │
│  FLUJO:                                                         │
│  click → dispatch({type:'ADD'}) → reducer → nuevo estado → UI   │
│                                                                 │
│  REGLAS DEL REDUCER:                                            │
│  1. Debe ser PURO (sin efectos secundarios)                     │
│  2. No mutar el estado, retornar uno nuevo                      │
│  3. Debe manejar todos los tipos de acción                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```typescript
import { useReducer } from 'react';

// Types
interface State {
  count: number;
  step: number;
  history: number[];
}

type Action =
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' }
  | { type: 'SET_STEP'; payload: number }
  | { type: 'RESET' }
  | { type: 'UNDO' };

// Reducer
function counterReducer(state: State, action: Action): State {
  switch (action.type) {
    case 'INCREMENT':
      return {
        ...state,
        count: state.count + state.step,
        history: [...state.history, state.count]
      };
    case 'DECREMENT':
      return {
        ...state,
        count: state.count - state.step,
        history: [...state.history, state.count]
      };
    case 'SET_STEP':
      return { ...state, step: action.payload };
    case 'RESET':
      return initialState;
    case 'UNDO':
      const previousCount = state.history[state.history.length - 1];
      return {
        ...state,
        count: previousCount ?? state.count,
        history: state.history.slice(0, -1)
      };
    default:
      return state;
  }
}

const initialState: State = { count: 0, step: 1, history: [] };

// Component
function Counter() {
  const [state, dispatch] = useReducer(counterReducer, initialState);
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <input
        type="number"
        value={state.step}
        onChange={e => dispatch({ type: 'SET_STEP', payload: Number(e.target.value) })}
      />
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
      <button onClick={() => dispatch({ type: 'UNDO' })} disabled={!state.history.length}>
        Undo
      </button>
      <button onClick={() => dispatch({ type: 'RESET' })}>Reset</button>
    </div>
  );
}
```

---

## Renderizado Condicional

```
┌─────────────────────────────────────────────────────────────────┐
│               CONCEPTO: RENDERIZADO CONDICIONAL                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  React te permite renderizar diferentes UI según condiciones.   │
│  A diferencia de HTML, todo se hace en JavaScript.              │
│                                                                 │
│  OPCIONES Y CUÁNDO USARLAS:                                     │
│                                                                 │
│  1. EARLY RETURN (if/else)                                      │
│     Usa cuando: Quieres "salir" temprano del componente         │
│     if (loading) return <Spinner />;                            │
│     if (error) return <Error />;                                │
│                                                                 │
│  2. TERNARIO (? :)                                              │
│     Usa cuando: Dos opciones mutuamente excluyentes             │
│     {isAdmin ? <AdminPanel /> : <UserPanel />}                  │
│                                                                 │
│  3. SHORT-CIRCUIT (&&)                                          │
│     Usa cuando: Mostrar algo o nada                             │
│     {hasNotifications && <Badge />}                             │
│                                                                 │
│  4. NULLISH (??)                                                │
│     Usa cuando: Valor por defecto si es null/undefined          │
│     {user.nickname ?? user.name ?? 'Anónimo'}                   │
│                                                                 │
│  ⚠️ CUIDADO CON &&:                                             │
│  {count && <Count value={count} />}  // ¡Renderiza "0"!        │
│  {count > 0 && <Count value={count} />}  // ✅ Correcto        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```typescript
function ConditionalRendering({ user, isLoading, error, items }) {
  // 1. If/else temprano (early return) - MUY RECOMENDADO
  if (isLoading) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!user) return <LoginPrompt />;
  
  // 2. Operador ternario
  return (
    <div>
      {user.isAdmin ? <AdminPanel /> : <UserDashboard />}
      
      {/* 3. Operador && (short-circuit) */}
      {user.notifications.length > 0 && <NotificationBadge count={user.notifications.length} />}
      
      {/* 4. Nullish coalescing */}
      <p>Welcome, {user.nickname ?? user.name ?? 'Guest'}</p>
      
      {/* 5. Múltiples condiciones con objeto/map */}
      {(() => {
        const statusComponents = {
          pending: <PendingStatus />,
          approved: <ApprovedStatus />,
          rejected: <RejectedStatus />,
        };
        return statusComponents[user.status] ?? <UnknownStatus />;
      })()}
      
      {/* 6. IIFE para lógica compleja */}
      {(() => {
        if (user.role === 'admin') return <AdminView />;
        if (user.role === 'moderator') return <ModeratorView />;
        if (user.isPremium) return <PremiumView />;
        return <BasicView />;
      })()}
    </div>
  );
}
```

---

## Listas y Keys

### ¿Por Qué son Importantes las Keys?

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONCEPTO: KEYS EN LISTAS                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Cuando renderizas una lista, React necesita saber qué          │
│  elementos cambiaron, se añadieron o se eliminaron.             │
│  Las KEYS son identificadores únicos que ayudan a React.        │
│                                                                 │
│  SIN KEY (React tiene que "adivinar"):                          │
│  ["A", "B", "C"]  →  ["A", "X", "B", "C"]                       │
│  React: "¿B se convirtió en X? ¿Se insertó X? 🤷"               │
│  Resultado: Puede re-renderizar TODA la lista                   │
│                                                                 │
│  CON KEY (React sabe exactamente qué pasó):                     │
│  [A:1, B:2, C:3]  →  [A:1, X:4, B:2, C:3]                       │
│  React: "¡Se insertó un elemento nuevo con key 4!"              │
│  Resultado: Solo crea el nuevo elemento                         │
│                                                                 │
│  REGLAS PARA KEYS:                                              │
│  ✅ Usar ID de los datos (item.id)                              │
│  ✅ Deben ser únicas entre hermanos                             │
│  ✅ Deben ser estables (no cambiar entre renders)               │
│                                                                 │
│  ❌ NO usar index (problemas al reordenar/filtrar)              │
│  ❌ NO usar Math.random() (cambia cada render)                  │
│  ❌ NO usar objetos ({} !== {} siempre)                         │
│                                                                 │
│  EL ÍNDICE ESTÁ BIEN SOLO SI:                                   │
│  • La lista nunca se reordena                                   │
│  • La lista nunca se filtra                                     │
│  • Los items no tienen ID                                       │
│  • Es contenido estático                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```typescript
interface Item {
  id: string;
  name: string;
  completed: boolean;
}

function TodoList({ items }: { items: Item[] }) {
  return (
    <ul>
      {items.map(item => (
        // ✅ Key única y estable (usar ID de datos)
        <li key={item.id}>
          <TodoItem item={item} />
        </li>
      ))}
    </ul>
  );
}

// ❌ Evitar: index como key (problemas con reordenamiento)
{items.map((item, index) => (
  <li key={index}>{item.name}</li>
))}

// ❌ Evitar: keys generadas aleatoriamente
{items.map(item => (
  <li key={Math.random()}>{item.name}</li>
))}

// ✅ Si no hay ID, crear uno estable
const itemsWithIds = items.map((item, index) => ({
  ...item,
  id: item.id || `${item.name}-${index}`
}));
```

### Renderizado de Listas Optimizado

```typescript
import { memo, useMemo } from 'react';

interface ListProps {
  items: Item[];
  filter: string;
  onItemClick: (item: Item) => void;
}

// Componente de item memoizado
const ListItem = memo(({ item, onClick }: { item: Item; onClick: () => void }) => {
  console.log(`Rendering: ${item.name}`);
  return (
    <li onClick={onClick}>
      {item.name}
    </li>
  );
});

function OptimizedList({ items, filter, onItemClick }: ListProps) {
  // Memoizar filtrado/ordenamiento costoso
  const filteredItems = useMemo(() => 
    items.filter(item => 
      item.name.toLowerCase().includes(filter.toLowerCase())
    ),
    [items, filter]
  );
  
  return (
    <ul>
      {filteredItems.map(item => (
        <ListItem
          key={item.id}
          item={item}
          onClick={() => onItemClick(item)}
        />
      ))}
    </ul>
  );
}
```

---

## Eventos

### ¿Cómo Funcionan los Eventos en React?

```
┌─────────────────────────────────────────────────────────────────┐
│                  CONCEPTO: SYNTHETIC EVENTS                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  React NO usa eventos nativos del DOM directamente.             │
│  En su lugar, usa EVENTOS SINTÉTICOS que envuelven              │
│  los eventos nativos para normalizar el comportamiento          │
│  entre navegadores.                                             │
│                                                                 │
│  VENTAJAS:                                                      │
│  • Misma API en todos los navegadores                           │
│  • Pool de eventos (mejor rendimiento)                          │
│  • Integración con el sistema de React                          │
│                                                                 │
│  DIFERENCIAS CON HTML:                                          │
│                                                                 │
│  HTML:     onclick="handleClick()"                              │
│  React:    onClick={handleClick}                                │
│            ↑ camelCase  ↑ función, no string                    │
│                                                                 │
│  EVENTOS COMUNES:                                               │
│  • onClick, onDoubleClick        - Mouse                        │
│  • onChange, onInput             - Formularios                  │
│  • onSubmit                      - Form submit                  │
│  • onKeyDown, onKeyUp, onKeyPress - Teclado                     │
│  • onFocus, onBlur               - Focus                        │
│  • onMouseEnter, onMouseLeave    - Hover                        │
│  • onScroll                      - Scroll                       │
│  • onDrag, onDrop                - Drag & Drop                  │
│                                                                 │
│  PREVENIR COMPORTAMIENTO:                                       │
│  • e.preventDefault()  - Evita acción por defecto              │
│  • e.stopPropagation() - Evita bubbling a padres               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Sistema de Eventos de React

```typescript
import { useState, useCallback } from 'react';
import type { 
  MouseEvent, 
  ChangeEvent, 
  FormEvent, 
  KeyboardEvent,
  FocusEvent,
  DragEvent 
} from 'react';

function EventExamples() {
  const [value, setValue] = useState('');
  
  // Mouse Events
  const handleClick = (e: MouseEvent<HTMLButtonElement>) => {
    e.preventDefault();
    e.stopPropagation();
    console.log('Clicked at:', e.clientX, e.clientY);
  };
  
  const handleMouseEnter = (e: MouseEvent<HTMLDivElement>) => {
    console.log('Mouse entered');
  };
  
  // Form Events
  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    setValue(e.target.value);
  };
  
  const handleSubmit = (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);
    console.log(Object.fromEntries(formData));
  };
  
  // Keyboard Events
  const handleKeyDown = (e: KeyboardEvent<HTMLInputElement>) => {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      // Submit
    }
    if (e.key === 'Escape') {
      // Cancel
    }
  };
  
  // Focus Events
  const handleFocus = (e: FocusEvent<HTMLInputElement>) => {
    e.target.select();
  };
  
  const handleBlur = (e: FocusEvent<HTMLInputElement>) => {
    // Validate on blur
  };
  
  // Drag Events
  const handleDragStart = (e: DragEvent<HTMLDivElement>) => {
    e.dataTransfer.setData('text/plain', 'dragged-item-id');
  };
  
  const handleDrop = (e: DragEvent<HTMLDivElement>) => {
    e.preventDefault();
    const data = e.dataTransfer.getData('text/plain');
  };
  
  return (
    <div>
      <button onClick={handleClick}>Click me</button>
      
      <form onSubmit={handleSubmit}>
        <input
          value={value}
          onChange={handleChange}
          onKeyDown={handleKeyDown}
          onFocus={handleFocus}
          onBlur={handleBlur}
        />
      </form>
      
      <div
        draggable
        onDragStart={handleDragStart}
        onDrop={handleDrop}
        onDragOver={e => e.preventDefault()}
      >
        Drag me
      </div>
    </div>
  );
}
```

### Event Handler con Parámetros

```typescript
function ItemList({ items, onDelete }) {
  // ✅ Opción 1: Arrow function inline (simple pero crea nueva función cada render)
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          {item.name}
          <button onClick={() => onDelete(item.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
  
  // ✅ Opción 2: Componente separado con useCallback
}

// Mejor para listas grandes
const Item = memo(({ item, onDelete }: { item: Item; onDelete: (id: string) => void }) => {
  const handleDelete = useCallback(() => {
    onDelete(item.id);
  }, [item.id, onDelete]);
  
  return (
    <li>
      {item.name}
      <button onClick={handleDelete}>Delete</button>
    </li>
  );
});
```

---

## Formularios

### Controlled vs Uncontrolled Components

```
┌─────────────────────────────────────────────────────────────────┐
│                CONCEPTO: CONTROLLED COMPONENTS                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  En React hay dos formas de manejar inputs:                     │
│                                                                 │
│  CONTROLLED (React es la "fuente de verdad"):                   │
│  ┌───────────────────────────────────────────┐                  │
│  │  const [value, setValue] = useState('')   │                  │
│  │  <input value={value}                     │                  │
│  │         onChange={e => setValue(e.target.value)} />          │
│  └───────────────────────────────────────────┘                  │
│  ✅ React controla el valor en todo momento                     │
│  ✅ Fácil de validar, transformar, resetear                     │
│  ✅ Recomendado para la mayoría de casos                        │
│                                                                 │
│  UNCONTROLLED (DOM es la "fuente de verdad"):                   │
│  ┌───────────────────────────────────────────┐                  │
│  │  const inputRef = useRef()                │                  │
│  │  <input ref={inputRef} defaultValue="" /> │                  │
│  │  // Leer: inputRef.current.value          │                  │
│  └───────────────────────────────────────────┘                  │
│  ✅ Menos código para formularios simples                       │
│  ✅ Necesario para inputs de archivo                            │
│  ⚠️ Menos control sobre el valor                                │
│                                                                 │
│  ¿CUÁL USAR?                                                    │
│  • Validación en tiempo real → Controlled                       │
│  • Formateo de input → Controlled                               │
│  • Formularios complejos → Controlled                           │
│  • Upload de archivos → Uncontrolled                            │
│  • Integración con código no-React → Uncontrolled               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Controlled Components

```typescript
import { useState, useCallback } from 'react';

interface FormData {
  email: string;
  password: string;
  confirmPassword: string;
  role: 'user' | 'admin';
  newsletter: boolean;
  interests: string[];
}

interface FormErrors {
  email?: string;
  password?: string;
  confirmPassword?: string;
}

function RegistrationForm() {
  const [formData, setFormData] = useState<FormData>({
    email: '',
    password: '',
    confirmPassword: '',
    role: 'user',
    newsletter: false,
    interests: []
  });
  
  const [errors, setErrors] = useState<FormErrors>({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  // Generic change handler
  const handleChange = useCallback((
    e: ChangeEvent<HTMLInputElement | HTMLSelectElement>
  ) => {
    const { name, value, type } = e.target;
    const checked = (e.target as HTMLInputElement).checked;
    
    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
    
    // Clear error on change
    if (errors[name as keyof FormErrors]) {
      setErrors(prev => ({ ...prev, [name]: undefined }));
    }
  }, [errors]);
  
  // Checkbox array handler
  const handleInterestChange = useCallback((interest: string) => {
    setFormData(prev => ({
      ...prev,
      interests: prev.interests.includes(interest)
        ? prev.interests.filter(i => i !== interest)
        : [...prev.interests, interest]
    }));
  }, []);
  
  // Validation
  const validate = useCallback((): boolean => {
    const newErrors: FormErrors = {};
    
    if (!formData.email) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Invalid email format';
    }
    
    if (!formData.password) {
      newErrors.password = 'Password is required';
    } else if (formData.password.length < 8) {
      newErrors.password = 'Password must be at least 8 characters';
    }
    
    if (formData.password !== formData.confirmPassword) {
      newErrors.confirmPassword = 'Passwords do not match';
    }
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  }, [formData]);
  
  // Submit
  const handleSubmit = async (e: FormEvent) => {
    e.preventDefault();
    
    if (!validate()) return;
    
    setIsSubmitting(true);
    try {
      await submitForm(formData);
    } catch (error) {
      console.error(error);
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return (
    <form onSubmit={handleSubmit} noValidate>
      {/* Text Input */}
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          name="email"
          type="email"
          value={formData.email}
          onChange={handleChange}
          aria-invalid={!!errors.email}
          aria-describedby={errors.email ? 'email-error' : undefined}
        />
        {errors.email && <span id="email-error" role="alert">{errors.email}</span>}
      </div>
      
      {/* Password Input */}
      <div>
        <label htmlFor="password">Password</label>
        <input
          id="password"
          name="password"
          type="password"
          value={formData.password}
          onChange={handleChange}
        />
        {errors.password && <span role="alert">{errors.password}</span>}
      </div>
      
      {/* Select */}
      <div>
        <label htmlFor="role">Role</label>
        <select id="role" name="role" value={formData.role} onChange={handleChange}>
          <option value="user">User</option>
          <option value="admin">Admin</option>
        </select>
      </div>
      
      {/* Single Checkbox */}
      <div>
        <label>
          <input
            type="checkbox"
            name="newsletter"
            checked={formData.newsletter}
            onChange={handleChange}
          />
          Subscribe to newsletter
        </label>
      </div>
      
      {/* Multiple Checkboxes */}
      <fieldset>
        <legend>Interests</legend>
        {['tech', 'sports', 'music'].map(interest => (
          <label key={interest}>
            <input
              type="checkbox"
              checked={formData.interests.includes(interest)}
              onChange={() => handleInterestChange(interest)}
            />
            {interest}
          </label>
        ))}
      </fieldset>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Register'}
      </button>
    </form>
  );
}
```

### Uncontrolled Components con useRef

```typescript
import { useRef, FormEvent } from 'react';

function UncontrolledForm() {
  const emailRef = useRef<HTMLInputElement>(null);
  const passwordRef = useRef<HTMLInputElement>(null);
  const fileRef = useRef<HTMLInputElement>(null);
  
  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();
    
    const email = emailRef.current?.value;
    const password = passwordRef.current?.value;
    const files = fileRef.current?.files;
    
    console.log({ email, password, files });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input ref={emailRef} type="email" defaultValue="" />
      <input ref={passwordRef} type="password" />
      <input ref={fileRef} type="file" multiple />
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## Composición vs Herencia

### ¿Por Qué Composición y No Herencia?

```
┌─────────────────────────────────────────────────────────────────┐
│               CONCEPTO: COMPOSICIÓN EN REACT                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  React prefiere COMPOSICIÓN sobre HERENCIA.                     │
│  En React, NUNCA necesitas herencia de clases para componentes. │
│                                                                 │
│  HERENCIA (❌ NO recomendado en React):                         │
│  class AdminButton extends Button { ... }                       │
│  Problemas: Rígido, difícil de cambiar, acoplamiento fuerte    │
│                                                                 │
│  COMPOSICIÓN (✅ Recomendado):                                  │
│  <Button variant="admin">Admin Action</Button>                  │
│  Ventajas: Flexible, reutilizable, fácil de entender           │
│                                                                 │
│  DOS PATRONES DE COMPOSICIÓN:                                   │
│                                                                 │
│  1. CONTAINMENT (Contenedores)                                  │
│     Componentes que "envuelven" otros sin saber qué contienen   │
│     <Card>                                                      │
│       <AnythingYouWant />  ← Card no sabe qué hay adentro      │
│     </Card>                                                     │
│                                                                 │
│  2. SPECIALIZATION (Especialización)                            │
│     Versiones específicas de componentes genéricos              │
│     <Dialog /> → <AlertDialog /> → <DeleteConfirmDialog />      │
│     Cada nivel añade props específicas                          │
│                                                                 │
│  PENSAMIENTO CLAVE:                                             │
│  "¿Puedo lograr esto pasando props o children?"                 │
│  Si sí → Composición                                            │
│  Si necesitas herencia → Probablemente tu diseño está mal      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Composición (Recomendado en React)

```typescript
// 1. Containment - componentes que no conocen sus children
function Card({ children, className = '' }: { children: ReactNode; className?: string }) {
  return <div className={`card ${className}`}>{children}</div>;
}

function CardHeader({ children }: { children: ReactNode }) {
  return <div className="card-header">{children}</div>;
}

function CardBody({ children }: { children: ReactNode }) {
  return <div className="card-body">{children}</div>;
}

function CardFooter({ children }: { children: ReactNode }) {
  return <div className="card-footer">{children}</div>;
}

// Uso
function UserCard({ user }) {
  return (
    <Card>
      <CardHeader>
        <h2>{user.name}</h2>
      </CardHeader>
      <CardBody>
        <p>{user.bio}</p>
      </CardBody>
      <CardFooter>
        <Button>Follow</Button>
      </CardFooter>
    </Card>
  );
}

// 2. Specialization - casos especiales de componentes
function Dialog({ title, children, actions }) {
  return (
    <div className="dialog">
      <h2>{title}</h2>
      <div className="dialog-content">{children}</div>
      <div className="dialog-actions">{actions}</div>
    </div>
  );
}

// Versión especializada
function ConfirmDialog({ title, message, onConfirm, onCancel }) {
  return (
    <Dialog
      title={title}
      actions={
        <>
          <Button variant="secondary" onClick={onCancel}>Cancel</Button>
          <Button variant="primary" onClick={onConfirm}>Confirm</Button>
        </>
      }
    >
      <p>{message}</p>
    </Dialog>
  );
}

function DeleteConfirmDialog({ itemName, onConfirm, onCancel }) {
  return (
    <ConfirmDialog
      title="Confirm Delete"
      message={`Are you sure you want to delete "${itemName}"?`}
      onConfirm={onConfirm}
      onCancel={onCancel}
    />
  );
}
```

---

## 🏷️ Tags

#programming #frontend #react #javascript #typescript #interview

---

## 📚 Ver También

- [[React Hooks Guide|React Hooks - Guía Detallada]]
- [[React Patterns Guide|React Patterns - Patrones Avanzados]]
- [[Next.js Complete Guide|Next.js - Guía Completa]]
