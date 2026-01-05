# React Patterns Guide

> **Patrones avanzados de React para desarrollo profesional**

---

## 📚 Tabla de Contenidos

1. [[#Composition Patterns|Composición]]
2. [[#Render Props Pattern|Render Props]]
3. [[#Higher-Order Components|HOC]]
4. [[#Compound Components|Compound Components]]
5. [[#State Management Patterns|State Management]]
6. [[#Performance Patterns|Performance]]
7. [[#Error Handling Patterns|Error Handling]]
8. [[#Data Fetching Patterns|Data Fetching]]

---

## Composition Patterns

### Container/Presentational Pattern

```typescript
// Presentational Component - Solo UI, sin lógica
interface UserListProps {
  users: User[];
  loading: boolean;
  error: string | null;
  onUserClick: (user: User) => void;
}

const UserList: FC<UserListProps> = ({ users, loading, error, onUserClick }) => {
  if (loading) return <Spinner />;
  if (error) return <ErrorMessage message={error} />;
  
  return (
    <ul className="user-list">
      {users.map(user => (
        <li key={user.id} onClick={() => onUserClick(user)}>
          <Avatar src={user.avatar} />
          <span>{user.name}</span>
        </li>
      ))}
    </ul>
  );
};

// Container Component - Lógica y estado
const UserListContainer: FC = () => {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  useEffect(() => {
    fetchUsers()
      .then(setUsers)
      .catch(e => setError(e.message))
      .finally(() => setLoading(false));
  }, []);
  
  const handleUserClick = (user: User) => {
    navigate(`/users/${user.id}`);
  };
  
  return (
    <UserList
      users={users}
      loading={loading}
      error={error}
      onUserClick={handleUserClick}
    />
  );
};
```

### Slots Pattern

```typescript
// Componente con "slots" nombrados
interface CardProps {
  header?: ReactNode;
  children: ReactNode;
  footer?: ReactNode;
  sidebar?: ReactNode;
}

function Card({ header, children, footer, sidebar }: CardProps) {
  return (
    <article className="card">
      {header && <header className="card-header">{header}</header>}
      <div className="card-body">
        {sidebar && <aside className="card-sidebar">{sidebar}</aside>}
        <main className="card-content">{children}</main>
      </div>
      {footer && <footer className="card-footer">{footer}</footer>}
    </article>
  );
}

// Uso
function ProductCard({ product }: { product: Product }) {
  return (
    <Card
      header={<h2>{product.name}</h2>}
      footer={<Button>Add to Cart</Button>}
      sidebar={<ProductImage src={product.image} />}
    >
      <p>{product.description}</p>
      <Price value={product.price} />
    </Card>
  );
}
```

### Compound Components con Static Properties

```typescript
// Subcomponentes como propiedades estáticas
interface AccordionContextType {
  openItems: Set<string>;
  toggle: (id: string) => void;
}

const AccordionContext = createContext<AccordionContextType | null>(null);

function useAccordionContext() {
  const context = useContext(AccordionContext);
  if (!context) throw new Error('Must be used within Accordion');
  return context;
}

// Main component
function Accordion({ children, defaultOpen = [] }: {
  children: ReactNode;
  defaultOpen?: string[];
}) {
  const [openItems, setOpenItems] = useState(new Set(defaultOpen));
  
  const toggle = useCallback((id: string) => {
    setOpenItems(prev => {
      const next = new Set(prev);
      if (next.has(id)) next.delete(id);
      else next.add(id);
      return next;
    });
  }, []);
  
  return (
    <AccordionContext.Provider value={{ openItems, toggle }}>
      <div className="accordion">{children}</div>
    </AccordionContext.Provider>
  );
}

// Item subcomponent
function AccordionItem({ id, children }: { id: string; children: ReactNode }) {
  const { openItems } = useAccordionContext();
  const isOpen = openItems.has(id);
  
  return (
    <div className={`accordion-item ${isOpen ? 'open' : ''}`}>
      {children}
    </div>
  );
}

// Header subcomponent
function AccordionHeader({ id, children }: { id: string; children: ReactNode }) {
  const { toggle } = useAccordionContext();
  
  return (
    <button className="accordion-header" onClick={() => toggle(id)}>
      {children}
    </button>
  );
}

// Panel subcomponent
function AccordionPanel({ id, children }: { id: string; children: ReactNode }) {
  const { openItems } = useAccordionContext();
  
  if (!openItems.has(id)) return null;
  
  return <div className="accordion-panel">{children}</div>;
}

// Attach subcomponents
Accordion.Item = AccordionItem;
Accordion.Header = AccordionHeader;
Accordion.Panel = AccordionPanel;

// Uso
function FAQ() {
  return (
    <Accordion defaultOpen={['faq-1']}>
      <Accordion.Item id="faq-1">
        <Accordion.Header id="faq-1">What is React?</Accordion.Header>
        <Accordion.Panel id="faq-1">
          React is a JavaScript library for building user interfaces.
        </Accordion.Panel>
      </Accordion.Item>
      <Accordion.Item id="faq-2">
        <Accordion.Header id="faq-2">What are hooks?</Accordion.Header>
        <Accordion.Panel id="faq-2">
          Hooks are functions that let you use state and other React features.
        </Accordion.Panel>
      </Accordion.Item>
    </Accordion>
  );
}
```

---

## Render Props Pattern

```typescript
// Render prop: pasar una función como prop que retorna JSX

// Ejemplo: Mouse tracker
interface MousePosition {
  x: number;
  y: number;
}

interface MouseTrackerProps {
  render: (position: MousePosition) => ReactNode;
}

function MouseTracker({ render }: MouseTrackerProps) {
  const [position, setPosition] = useState<MousePosition>({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMove = (e: MouseEvent) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    
    window.addEventListener('mousemove', handleMove);
    return () => window.removeEventListener('mousemove', handleMove);
  }, []);
  
  return <>{render(position)}</>;
}

// Uso
function App() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <div>
          Mouse position: ({x}, {y})
        </div>
      )}
    />
  );
}

// Alternativa: children como función
interface MouseTrackerProps2 {
  children: (position: MousePosition) => ReactNode;
}

function MouseTracker2({ children }: MouseTrackerProps2) {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  // ... misma lógica
  return <>{children(position)}</>;
}

// Uso alternativo
<MouseTracker2>
  {({ x, y }) => <Cursor x={x} y={y} />}
</MouseTracker2>

// Ejemplo avanzado: Toggle con render props
interface ToggleRenderProps {
  on: boolean;
  toggle: () => void;
  setOn: () => void;
  setOff: () => void;
}

interface ToggleProps {
  children: (props: ToggleRenderProps) => ReactNode;
  initialOn?: boolean;
}

function Toggle({ children, initialOn = false }: ToggleProps) {
  const [on, setOn] = useState(initialOn);
  
  const toggle = useCallback(() => setOn(v => !v), []);
  const turnOn = useCallback(() => setOn(true), []);
  const turnOff = useCallback(() => setOn(false), []);
  
  return (
    <>
      {children({
        on,
        toggle,
        setOn: turnOn,
        setOff: turnOff
      })}
    </>
  );
}

// Uso flexible
<Toggle initialOn={false}>
  {({ on, toggle }) => (
    <div>
      <button onClick={toggle}>{on ? 'ON' : 'OFF'}</button>
      {on && <p>The toggle is on!</p>}
    </div>
  )}
</Toggle>
```

---

## Higher-Order Components (HOC)

```typescript
// HOC: función que toma un componente y retorna uno nuevo con funcionalidad extra

// HOC básico: withLoading
interface WithLoadingProps {
  loading: boolean;
}

function withLoading<P extends object>(
  WrappedComponent: ComponentType<P>
): FC<P & WithLoadingProps> {
  return function WithLoadingComponent({ loading, ...props }: P & WithLoadingProps) {
    if (loading) return <Spinner />;
    return <WrappedComponent {...(props as P)} />;
  };
}

// Uso
const UserListWithLoading = withLoading(UserList);
<UserListWithLoading loading={isLoading} users={users} />

// HOC: withAuth
interface WithAuthProps {
  user: User | null;
}

function withAuth<P extends WithAuthProps>(
  WrappedComponent: ComponentType<P>
): FC<Omit<P, 'user'>> {
  return function WithAuthComponent(props: Omit<P, 'user'>) {
    const { user, loading } = useAuth();
    
    if (loading) return <Spinner />;
    if (!user) return <Navigate to="/login" />;
    
    return <WrappedComponent {...(props as P)} user={user} />;
  };
}

// HOC: withErrorBoundary
function withErrorBoundary<P extends object>(
  WrappedComponent: ComponentType<P>,
  FallbackComponent: ComponentType<{ error: Error }>
): FC<P> {
  return function WithErrorBoundaryComponent(props: P) {
    return (
      <ErrorBoundary FallbackComponent={FallbackComponent}>
        <WrappedComponent {...props} />
      </ErrorBoundary>
    );
  };
}

// HOC con configuración
interface WithAnalyticsConfig {
  eventName: string;
  properties?: Record<string, unknown>;
}

function withAnalytics<P extends object>(config: WithAnalyticsConfig) {
  return function(WrappedComponent: ComponentType<P>): FC<P> {
    return function WithAnalyticsComponent(props: P) {
      useEffect(() => {
        analytics.track(config.eventName, config.properties);
      }, []);
      
      return <WrappedComponent {...props} />;
    };
  };
}

// Uso
const TrackedDashboard = withAnalytics({
  eventName: 'dashboard_viewed',
  properties: { version: '2.0' }
})(Dashboard);

// Composición de HOCs
const EnhancedComponent = compose(
  withAuth,
  withLoading,
  withErrorBoundary(ErrorFallback)
)(BaseComponent);
```

---

## Compound Components Pattern

```typescript
// Patrón para componentes que trabajan juntos compartiendo estado implícito

// Ejemplo: Select/Dropdown
interface SelectContextType {
  value: string;
  onChange: (value: string) => void;
  isOpen: boolean;
  setIsOpen: (open: boolean) => void;
}

const SelectContext = createContext<SelectContextType | null>(null);

function useSelectContext() {
  const context = useContext(SelectContext);
  if (!context) throw new Error('Must be used within Select');
  return context;
}

// Main Select component
interface SelectProps {
  value: string;
  onChange: (value: string) => void;
  children: ReactNode;
}

function Select({ value, onChange, children }: SelectProps) {
  const [isOpen, setIsOpen] = useState(false);
  const selectRef = useRef<HTMLDivElement>(null);
  
  useOnClickOutside(selectRef, () => setIsOpen(false));
  
  return (
    <SelectContext.Provider value={{ value, onChange, isOpen, setIsOpen }}>
      <div ref={selectRef} className="select">
        {children}
      </div>
    </SelectContext.Provider>
  );
}

// Trigger component
function SelectTrigger({ children, placeholder }: { children?: ReactNode; placeholder?: string }) {
  const { value, isOpen, setIsOpen } = useSelectContext();
  
  return (
    <button 
      className="select-trigger"
      onClick={() => setIsOpen(!isOpen)}
      aria-expanded={isOpen}
    >
      {children || value || placeholder}
      <ChevronIcon direction={isOpen ? 'up' : 'down'} />
    </button>
  );
}

// Options container
function SelectOptions({ children }: { children: ReactNode }) {
  const { isOpen } = useSelectContext();
  
  if (!isOpen) return null;
  
  return (
    <ul className="select-options" role="listbox">
      {children}
    </ul>
  );
}

// Individual option
interface SelectOptionProps {
  value: string;
  children: ReactNode;
  disabled?: boolean;
}

function SelectOption({ value, children, disabled }: SelectOptionProps) {
  const { value: selectedValue, onChange, setIsOpen } = useSelectContext();
  const isSelected = selectedValue === value;
  
  const handleClick = () => {
    if (disabled) return;
    onChange(value);
    setIsOpen(false);
  };
  
  return (
    <li
      role="option"
      aria-selected={isSelected}
      aria-disabled={disabled}
      className={`select-option ${isSelected ? 'selected' : ''}`}
      onClick={handleClick}
    >
      {children}
      {isSelected && <CheckIcon />}
    </li>
  );
}

// Attach subcomponents
Select.Trigger = SelectTrigger;
Select.Options = SelectOptions;
Select.Option = SelectOption;

// Uso
function CountrySelect() {
  const [country, setCountry] = useState('');
  
  return (
    <Select value={country} onChange={setCountry}>
      <Select.Trigger placeholder="Select a country" />
      <Select.Options>
        <Select.Option value="us">United States</Select.Option>
        <Select.Option value="uk">United Kingdom</Select.Option>
        <Select.Option value="ca">Canada</Select.Option>
        <Select.Option value="au" disabled>Australia (unavailable)</Select.Option>
      </Select.Options>
    </Select>
  );
}
```

---

## State Management Patterns

### Context + Reducer Pattern

```typescript
// Combinar Context con useReducer para state management escalable

// Types
interface AppState {
  user: User | null;
  theme: 'light' | 'dark';
  notifications: Notification[];
  settings: Settings;
}

type AppAction =
  | { type: 'SET_USER'; payload: User | null }
  | { type: 'TOGGLE_THEME' }
  | { type: 'ADD_NOTIFICATION'; payload: Notification }
  | { type: 'DISMISS_NOTIFICATION'; payload: string }
  | { type: 'UPDATE_SETTINGS'; payload: Partial<Settings> };

// Reducer
function appReducer(state: AppState, action: AppAction): AppState {
  switch (action.type) {
    case 'SET_USER':
      return { ...state, user: action.payload };
    case 'TOGGLE_THEME':
      return { ...state, theme: state.theme === 'light' ? 'dark' : 'light' };
    case 'ADD_NOTIFICATION':
      return { ...state, notifications: [...state.notifications, action.payload] };
    case 'DISMISS_NOTIFICATION':
      return {
        ...state,
        notifications: state.notifications.filter(n => n.id !== action.payload)
      };
    case 'UPDATE_SETTINGS':
      return { ...state, settings: { ...state.settings, ...action.payload } };
    default:
      return state;
  }
}

// Context
interface AppContextType {
  state: AppState;
  dispatch: Dispatch<AppAction>;
}

const AppContext = createContext<AppContextType | undefined>(undefined);

// Provider
function AppProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(appReducer, initialState);
  
  const value = useMemo(() => ({ state, dispatch }), [state]);
  
  return (
    <AppContext.Provider value={value}>
      {children}
    </AppContext.Provider>
  );
}

// Custom hooks para acceder al estado
function useAppState() {
  const context = useContext(AppContext);
  if (!context) throw new Error('useAppState must be used within AppProvider');
  return context.state;
}

function useAppDispatch() {
  const context = useContext(AppContext);
  if (!context) throw new Error('useAppDispatch must be used within AppProvider');
  return context.dispatch;
}

// Hooks específicos con acciones
function useUser() {
  const { user } = useAppState();
  const dispatch = useAppDispatch();
  
  const login = useCallback((user: User) => {
    dispatch({ type: 'SET_USER', payload: user });
  }, [dispatch]);
  
  const logout = useCallback(() => {
    dispatch({ type: 'SET_USER', payload: null });
  }, [dispatch]);
  
  return { user, login, logout };
}

function useNotifications() {
  const { notifications } = useAppState();
  const dispatch = useAppDispatch();
  
  const addNotification = useCallback((notification: Omit<Notification, 'id'>) => {
    dispatch({
      type: 'ADD_NOTIFICATION',
      payload: { ...notification, id: crypto.randomUUID() }
    });
  }, [dispatch]);
  
  const dismissNotification = useCallback((id: string) => {
    dispatch({ type: 'DISMISS_NOTIFICATION', payload: id });
  }, [dispatch]);
  
  return { notifications, addNotification, dismissNotification };
}
```

### Module State Pattern (Zustand-like)

```typescript
// Estado global simple sin Context
type Listener<T> = (state: T) => void;

function createStore<T>(initialState: T) {
  let state = initialState;
  const listeners = new Set<Listener<T>>();
  
  const getState = () => state;
  
  const setState = (partial: Partial<T> | ((state: T) => Partial<T>)) => {
    const nextState = typeof partial === 'function' ? partial(state) : partial;
    state = { ...state, ...nextState };
    listeners.forEach(listener => listener(state));
  };
  
  const subscribe = (listener: Listener<T>) => {
    listeners.add(listener);
    return () => listeners.delete(listener);
  };
  
  return { getState, setState, subscribe };
}

// Crear store
interface CounterState {
  count: number;
  increment: () => void;
  decrement: () => void;
}

const counterStore = createStore<CounterState>({
  count: 0,
  increment: () => {},
  decrement: () => {}
});

// Agregar acciones
counterStore.setState({
  increment: () => counterStore.setState(s => ({ count: s.count + 1 })),
  decrement: () => counterStore.setState(s => ({ count: s.count - 1 }))
});

// Hook para usar el store
function useStore<T, S>(
  store: { getState: () => T; subscribe: (l: Listener<T>) => () => void },
  selector: (state: T) => S
): S {
  return useSyncExternalStore(
    store.subscribe,
    () => selector(store.getState())
  );
}

// Uso
function Counter() {
  const count = useStore(counterStore, s => s.count);
  const { increment, decrement } = counterStore.getState();
  
  return (
    <div>
      <span>{count}</span>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </div>
  );
}
```

---

## Performance Patterns

### Virtualization Pattern

```typescript
// Para listas muy largas, solo renderizar items visibles
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualizedList({ items }: { items: Item[] }) {
  const parentRef = useRef<HTMLDivElement>(null);
  
  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50, // altura estimada de cada item
    overscan: 5 // items extra para renderizar fuera de vista
  });
  
  return (
    <div
      ref={parentRef}
      style={{ height: '400px', overflow: 'auto' }}
    >
      <div
        style={{
          height: `${virtualizer.getTotalSize()}px`,
          width: '100%',
          position: 'relative'
        }}
      >
        {virtualizer.getVirtualItems().map(virtualRow => (
          <div
            key={virtualRow.key}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: `${virtualRow.size}px`,
              transform: `translateY(${virtualRow.start}px)`
            }}
          >
            <ListItem item={items[virtualRow.index]} />
          </div>
        ))}
      </div>
    </div>
  );
}
```

### Code Splitting con lazy

```typescript
import { lazy, Suspense, startTransition } from 'react';

// Lazy loading de componentes
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));
const Profile = lazy(() => import('./pages/Profile'));

// Con named exports
const Admin = lazy(() => 
  import('./pages/Admin').then(module => ({ default: module.AdminPanel }))
);

// Prefetching
const prefetchDashboard = () => import('./pages/Dashboard');

function App() {
  return (
    <Suspense fallback={<PageSkeleton />}>
      <Routes>
        <Route path="/" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
        <Route path="/profile" element={<Profile />} />
      </Routes>
    </Suspense>
  );
}

// Link con prefetch
function NavLink({ to, children, prefetch }: NavLinkProps) {
  const handleMouseEnter = () => {
    if (prefetch) prefetch();
  };
  
  return (
    <Link to={to} onMouseEnter={handleMouseEnter}>
      {children}
    </Link>
  );
}

<NavLink to="/dashboard" prefetch={prefetchDashboard}>
  Dashboard
</NavLink>
```

### Memoization Strategy

```typescript
// Estrategia de memoización efectiva

// 1. Memoizar componentes costosos
const ExpensiveChart = memo(({ data, config }: ChartProps) => {
  // Renderizado costoso
  return <canvas ref={renderChart(data, config)} />;
});

// 2. Memoizar cálculos derivados
function DataTable({ rawData, filters, sortConfig }: Props) {
  // Solo recalcular cuando dependencias cambian
  const processedData = useMemo(() => {
    let result = [...rawData];
    
    // Filtrar
    result = result.filter(item => 
      Object.entries(filters).every(([key, value]) => 
        item[key]?.toString().includes(value)
      )
    );
    
    // Ordenar
    if (sortConfig) {
      result.sort((a, b) => {
        const aVal = a[sortConfig.key];
        const bVal = b[sortConfig.key];
        return sortConfig.direction === 'asc' 
          ? aVal.localeCompare(bVal)
          : bVal.localeCompare(aVal);
      });
    }
    
    return result;
  }, [rawData, filters, sortConfig]);
  
  return <Table data={processedData} />;
}

// 3. Memoizar callbacks para children memoizados
function Parent() {
  const [items, setItems] = useState<Item[]>([]);
  
  // Sin useCallback, MemoizedChild re-renderiza siempre
  const handleDelete = useCallback((id: string) => {
    setItems(prev => prev.filter(item => item.id !== id));
  }, []);
  
  return (
    <ul>
      {items.map(item => (
        <MemoizedChild key={item.id} item={item} onDelete={handleDelete} />
      ))}
    </ul>
  );
}

// 4. Estabilizar objetos y arrays
function SearchComponent() {
  const [query, setQuery] = useState('');
  
  // ❌ Objeto nuevo en cada render
  const config = { query, limit: 10 };
  
  // ✅ Objeto memoizado
  const config = useMemo(() => ({ query, limit: 10 }), [query]);
  
  // ❌ Array nuevo en cada render
  const deps = [query, 'users'];
  
  // ✅ Array memoizado
  const deps = useMemo(() => [query, 'users'], [query]);
  
  return <Search config={config} />;
}
```

---

## Error Handling Patterns

### Error Boundary

```typescript
// Error Boundary como clase (los hooks no pueden capturar errores de render)
interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
}

interface ErrorBoundaryProps {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
  resetKeys?: unknown[];
}

class ErrorBoundary extends Component<ErrorBoundaryProps, ErrorBoundaryState> {
  state: ErrorBoundaryState = { hasError: false, error: null };
  
  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error };
  }
  
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    this.props.onError?.(error, errorInfo);
    // Log to error reporting service
    logErrorToService(error, errorInfo);
  }
  
  componentDidUpdate(prevProps: ErrorBoundaryProps) {
    // Reset error state when resetKeys change
    if (
      this.state.hasError &&
      this.props.resetKeys &&
      prevProps.resetKeys &&
      !arraysEqual(this.props.resetKeys, prevProps.resetKeys)
    ) {
      this.setState({ hasError: false, error: null });
    }
  }
  
  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? <DefaultErrorFallback error={this.state.error} />;
    }
    return this.props.children;
  }
}

// Error fallback component
function DefaultErrorFallback({ error, resetError }: { 
  error: Error | null; 
  resetError?: () => void 
}) {
  return (
    <div role="alert" className="error-fallback">
      <h2>Something went wrong</h2>
      <pre>{error?.message}</pre>
      {resetError && (
        <button onClick={resetError}>Try again</button>
      )}
    </div>
  );
}

// Uso con react-error-boundary (librería recomendada)
import { ErrorBoundary, useErrorBoundary } from 'react-error-boundary';

function App() {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onError={(error, info) => logError(error, info)}
      onReset={() => {
        // Reset app state
      }}
      resetKeys={[userId]}
    >
      <Routes />
    </ErrorBoundary>
  );
}

// Hook para lanzar errores a Error Boundary
function UserProfile({ userId }: { userId: string }) {
  const { showBoundary } = useErrorBoundary();
  
  useEffect(() => {
    fetchUser(userId).catch(showBoundary);
  }, [userId, showBoundary]);
  
  // ...
}
```

### Async Error Handling

```typescript
// Hook para manejo de errores async
function useAsyncError() {
  const [, setError] = useState();
  
  return useCallback((error: Error) => {
    setError(() => {
      throw error; // Esto será capturado por Error Boundary
    });
  }, []);
}

// Uso
function DataLoader() {
  const throwError = useAsyncError();
  
  useEffect(() => {
    fetchData()
      .then(setData)
      .catch(throwError);
  }, [throwError]);
}

// Error handling con custom hook
interface UseQueryResult<T> {
  data: T | null;
  error: Error | null;
  loading: boolean;
  retry: () => void;
}

function useQuery<T>(queryFn: () => Promise<T>): UseQueryResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [error, setError] = useState<Error | null>(null);
  const [loading, setLoading] = useState(true);
  
  const execute = useCallback(async () => {
    setLoading(true);
    setError(null);
    
    try {
      const result = await queryFn();
      setData(result);
    } catch (err) {
      setError(err instanceof Error ? err : new Error(String(err)));
    } finally {
      setLoading(false);
    }
  }, [queryFn]);
  
  useEffect(() => {
    execute();
  }, [execute]);
  
  return { data, error, loading, retry: execute };
}
```

---

## Data Fetching Patterns

### SWR/React Query Pattern

```typescript
// Implementación simplificada del patrón stale-while-revalidate

interface CacheEntry<T> {
  data: T;
  timestamp: number;
}

const cache = new Map<string, CacheEntry<unknown>>();

interface UseDataOptions {
  revalidateOnFocus?: boolean;
  revalidateOnReconnect?: boolean;
  refreshInterval?: number;
  dedupingInterval?: number;
}

function useData<T>(
  key: string,
  fetcher: () => Promise<T>,
  options: UseDataOptions = {}
) {
  const {
    revalidateOnFocus = true,
    revalidateOnReconnect = true,
    refreshInterval,
    dedupingInterval = 2000
  } = options;
  
  const [data, setData] = useState<T | undefined>(() => {
    const cached = cache.get(key) as CacheEntry<T> | undefined;
    return cached?.data;
  });
  const [error, setError] = useState<Error | null>(null);
  const [isValidating, setIsValidating] = useState(false);
  
  const lastFetchRef = useRef<number>(0);
  
  const revalidate = useCallback(async () => {
    // Deduping: evitar múltiples requests en corto tiempo
    const now = Date.now();
    if (now - lastFetchRef.current < dedupingInterval) {
      return;
    }
    lastFetchRef.current = now;
    
    setIsValidating(true);
    
    try {
      const newData = await fetcher();
      cache.set(key, { data: newData, timestamp: now });
      setData(newData);
      setError(null);
    } catch (err) {
      setError(err as Error);
    } finally {
      setIsValidating(false);
    }
  }, [key, fetcher, dedupingInterval]);
  
  // Initial fetch
  useEffect(() => {
    revalidate();
  }, [revalidate]);
  
  // Revalidate on focus
  useEffect(() => {
    if (!revalidateOnFocus) return;
    
    const handleFocus = () => revalidate();
    window.addEventListener('focus', handleFocus);
    return () => window.removeEventListener('focus', handleFocus);
  }, [revalidateOnFocus, revalidate]);
  
  // Revalidate on reconnect
  useEffect(() => {
    if (!revalidateOnReconnect) return;
    
    const handleOnline = () => revalidate();
    window.addEventListener('online', handleOnline);
    return () => window.removeEventListener('online', handleOnline);
  }, [revalidateOnReconnect, revalidate]);
  
  // Polling
  useEffect(() => {
    if (!refreshInterval) return;
    
    const id = setInterval(revalidate, refreshInterval);
    return () => clearInterval(id);
  }, [refreshInterval, revalidate]);
  
  return {
    data,
    error,
    isLoading: !data && !error,
    isValidating,
    mutate: revalidate
  };
}

// Uso
function UserList() {
  const { data: users, error, isLoading, mutate } = useData(
    '/api/users',
    () => fetch('/api/users').then(r => r.json()),
    { refreshInterval: 30000 }
  );
  
  if (isLoading) return <Spinner />;
  if (error) return <Error message={error.message} retry={mutate} />;
  
  return <List items={users} />;
}
```

### Optimistic Updates

```typescript
// Actualizar UI inmediatamente, revertir si falla

interface Todo {
  id: string;
  text: string;
  completed: boolean;
}

function useTodos() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [optimisticTodos, setOptimisticTodos] = useOptimistic(
    todos,
    (state, { action, todo }: { action: 'add' | 'update' | 'delete'; todo: Todo }) => {
      switch (action) {
        case 'add':
          return [...state, todo];
        case 'update':
          return state.map(t => t.id === todo.id ? todo : t);
        case 'delete':
          return state.filter(t => t.id !== todo.id);
      }
    }
  );
  
  const addTodo = async (text: string) => {
    const tempTodo: Todo = {
      id: `temp-${Date.now()}`,
      text,
      completed: false
    };
    
    // Optimistic update
    setOptimisticTodos({ action: 'add', todo: tempTodo });
    
    try {
      const newTodo = await api.createTodo(text);
      setTodos(prev => [...prev.filter(t => t.id !== tempTodo.id), newTodo]);
    } catch (error) {
      // El estado optimista se revierte automáticamente
      showError('Failed to add todo');
    }
  };
  
  const toggleTodo = async (todo: Todo) => {
    const updated = { ...todo, completed: !todo.completed };
    
    setOptimisticTodos({ action: 'update', todo: updated });
    
    try {
      await api.updateTodo(updated);
      setTodos(prev => prev.map(t => t.id === todo.id ? updated : t));
    } catch (error) {
      showError('Failed to update todo');
    }
  };
  
  const deleteTodo = async (todo: Todo) => {
    setOptimisticTodos({ action: 'delete', todo });
    
    try {
      await api.deleteTodo(todo.id);
      setTodos(prev => prev.filter(t => t.id !== todo.id));
    } catch (error) {
      showError('Failed to delete todo');
    }
  };
  
  return {
    todos: optimisticTodos,
    addTodo,
    toggleTodo,
    deleteTodo
  };
}
```

---

## 🏷️ Tags

#programming #frontend #react #patterns #javascript #typescript #architecture

---

## 📚 Ver También

- [[React Complete Guide|React - Guía Completa]]
- [[React Hooks Guide|React Hooks - Guía Detallada]]
- [[Next.js Complete Guide|Next.js - Guía Completa]]
