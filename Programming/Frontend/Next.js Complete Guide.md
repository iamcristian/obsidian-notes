# Next.js Complete Guide

> **Guía profesional de Next.js 15 para desarrollo y entrevistas técnicas**

---

## 📚 Tabla de Contenidos

1. [[#Conceptos Fundamentales de Renderizado|Conceptos de Renderizado]]
2. [[#Fundamentos de Next.js|Fundamentos]]
3. [[#App Router|App Router]]
4. [[#Server Components vs Client Components|Server vs Client Components]]
5. [[#Data Fetching|Data Fetching]]
6. [[#Server Actions|Server Actions]]
7. [[#Routing Avanzado|Routing]]
8. [[#Caching y Revalidación|Caching]]
9. [[#Middleware|Middleware]]
10. [[#Optimizaciones|Optimizaciones]]
11. [[#Deployment|Deployment]]

---

## Conceptos Fundamentales de Renderizado

> 🎯 **IMPORTANTE PARA ENTREVISTAS**: Entender las diferencias entre CSR, SSR, SSG e ISR es CRUCIAL. Es una de las preguntas más frecuentes.

### ¿Qué es el Renderizado Web?

**Renderizado** es el proceso de convertir tu código (React, HTML, CSS, JS) en píxeles visibles en la pantalla del usuario. La pregunta clave es: **¿DÓNDE y CUÁNDO ocurre este proceso?**

```
┌─────────────────────────────────────────────────────────────────────┐
│                    LÍNEA DE TIEMPO DE UNA PÁGINA WEB                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  1. Usuario escribe URL → 2. Request al servidor → 3. Servidor     │
│     responde → 4. Navegador procesa → 5. Usuario ve contenido      │
│                                                                     │
│  La pregunta es: ¿En qué paso se genera el HTML con datos?         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

### CSR (Client-Side Rendering) - Renderizado en el Cliente

#### ¿Qué es?
El servidor envía un HTML **vacío** con JavaScript. El navegador del usuario descarga el JS, lo ejecuta, y ENTONCES genera el contenido visible.

#### ¿Cómo funciona?

```
PASO A PASO DE CSR:

1. Usuario visita tusitio.com/productos
2. Servidor envía:
   └── HTML casi vacío: <div id="root"></div>
   └── Bundle JavaScript (puede ser 500KB - 2MB)

3. Navegador del usuario:
   └── Descarga el JavaScript (1-5 segundos en conexiones lentas)
   └── Parsea y ejecuta el JavaScript
   └── JavaScript hace fetch('/api/productos')
   └── Espera respuesta de la API
   └── FINALMENTE renderiza los productos en pantalla

TIEMPO TOTAL: 3-10 segundos antes de ver contenido útil
```

#### Diagrama Visual

```
┌─────────────────────────────────────────────────────────────────┐
│                         CSR TIMELINE                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  [Request] ──→ [HTML vacío] ──→ [Descarga JS] ──→ [Ejecuta JS] │
│                                                                 │
│     0s           0.2s            1-3s              3-5s         │
│                                                                 │
│  ──→ [Fetch API] ──→ [Renderiza] ──→ [Usuario ve contenido]    │
│                                                                 │
│        4-6s           5-8s              5-10s                   │
│                                                                 │
│  📱 Durante 5-10 segundos el usuario ve: PANTALLA EN BLANCO    │
│                        o un spinner de loading                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Ejemplo de CSR (React tradicional con Vite/CRA)

```typescript
// App.tsx - CSR tradicional
// El HTML inicial está VACÍO, todo se genera en el navegador
function ProductosPage() {
  const [productos, setProductos] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Este código se ejecuta EN EL NAVEGADOR del usuario
    // El servidor NO sabe nada de esto
    fetch('/api/productos')
      .then(res => res.json())
      .then(data => {
        setProductos(data);
        setLoading(false);
      });
  }, []);

  if (loading) return <Spinner />; // El usuario ve esto primero
  
  return <ListaProductos productos={productos} />;
}
```

#### ✅ Ventajas de CSR
- **Interactividad rica**: Una vez cargado, la app es muy rápida
- **Menos carga en servidor**: El servidor solo sirve archivos estáticos
- **Ideal para dashboards**: Apps privadas que no necesitan SEO
- **Navegación instantánea**: Cambiar de página no recarga todo

#### ❌ Desventajas de CSR
- **Mal SEO**: Google ve HTML vacío (aunque ha mejorado)
- **Tiempo de carga inicial lento**: El usuario espera mucho
- **Malo en conexiones lentas**: Descargar 1MB de JS en 3G es terrible
- **Malo en dispositivos lentos**: Ejecutar JS pesado en un teléfono barato = lag

#### ¿Cuándo usar CSR?
```
✅ USA CSR CUANDO:
   • Es un dashboard privado (no necesita SEO)
   • Es una app detrás de login
   • La interactividad es más importante que la carga inicial
   • Tienes muchos estados complejos del lado del cliente
   • Ej: Panel de admin, app de edición, herramientas internas

❌ NO USES CSR CUANDO:
   • Necesitas buen SEO (blog, e-commerce, landing)
   • Tus usuarios tienen conexiones lentas
   • El contenido es mayormente estático
```

---

### SSR (Server-Side Rendering) - Renderizado en el Servidor

#### ¿Qué es?
El servidor genera el HTML **COMPLETO con datos** en cada request. El usuario recibe una página lista para ver inmediatamente.

#### ¿Cómo funciona?

```
PASO A PASO DE SSR:

1. Usuario visita tusitio.com/productos
2. Servidor:
   └── Recibe el request
   └── Consulta la base de datos
   └── Genera HTML completo con los productos
   └── Envía HTML listo para mostrar

3. Navegador del usuario:
   └── Recibe HTML completo (usuario YA VE el contenido)
   └── Descarga JavaScript en segundo plano
   └── "Hidrata" la página (hace interactiva)

TIEMPO HASTA VER CONTENIDO: 0.5-2 segundos
```

#### Diagrama Visual

```
┌─────────────────────────────────────────────────────────────────┐
│                         SSR TIMELINE                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  [Request] ──→ [Servidor consulta DB] ──→ [Genera HTML]        │
│                                                                 │
│     0s              0.1-0.5s                0.3-1s              │
│                                                                 │
│  ──→ [Envía HTML completo] ──→ [Usuario VE contenido]          │
│                                                                 │
│           0.5-1.5s                  0.5-2s ✅                   │
│                                                                 │
│  ──→ [JS se descarga] ──→ [Hidratación] ──→ [Interactivo]      │
│                                                                 │
│          1-3s               2-4s              2-5s              │
│                                                                 │
│  📱 Usuario ve contenido en 0.5-2s (aunque no es interactivo   │
│     inmediatamente, puede LEER el contenido)                    │
└─────────────────────────────────────────────────────────────────┘
```

#### Ejemplo de SSR en Next.js

```typescript
// app/productos/page.tsx - SSR en Next.js 15
// Este código se ejecuta EN EL SERVIDOR en cada request

async function ProductosPage() {
  // Esto se ejecuta en el SERVIDOR, no en el navegador
  // El usuario NUNCA ve este código
  const productos = await db.producto.findMany();
  
  // El HTML ya incluye los productos cuando llega al navegador
  return (
    <div>
      <h1>Nuestros Productos</h1>
      {productos.map(p => (
        <ProductCard key={p.id} producto={p} />
      ))}
    </div>
  );
}

export default ProductosPage;
```

#### ¿Qué es la Hidratación?

```
HIDRATACIÓN (Hydration):

El proceso donde React "conecta" el JavaScript al HTML que el servidor envió.

1. Servidor envía: <button>Comprar</button> (HTML estático, no hace nada)
2. JS se carga y React dice: "Este botón es mío, le agrego onClick"
3. Ahora: <button onClick={handleComprar}>Comprar</button> (interactivo)

PROBLEMA: Entre que el usuario VE el botón y puede USARLO hay un gap.
Esto se llama "Time to Interactive" (TTI).
```

#### ✅ Ventajas de SSR
- **Excelente SEO**: Google ve HTML completo con contenido
- **Tiempo hasta contenido rápido**: Usuario ve algo útil en <2s
- **Mejor en conexiones lentas**: HTML es más ligero que JS
- **Datos siempre frescos**: Cada request trae datos actualizados

#### ❌ Desventajas de SSR
- **Más carga en servidor**: Cada request = trabajo del servidor
- **TTFB más lento**: El servidor necesita tiempo para generar
- **Costos de infraestructura**: Más CPU/memoria en el servidor
- **No cacheable por defecto**: Cada request es único

#### ¿Cuándo usar SSR?

```
✅ USA SSR CUANDO:
   • El contenido cambia frecuentemente (cada minuto/hora)
   • Necesitas datos personalizados por usuario
   • El contenido depende de cookies/headers del request
   • Necesitas SEO con datos dinámicos
   • Ej: Feed de redes sociales, carrito de compras, búsquedas

❌ NO USES SSR CUANDO:
   • El contenido es igual para todos los usuarios
   • El contenido no cambia frecuentemente
   • No necesitas SEO
   • Tienes alto tráfico (costoso)
```

---

### SSG (Static Site Generation) - Generación Estática

#### ¿Qué es?
El HTML se genera **UNA VEZ en build time** (cuando haces `npm run build`). Después, el mismo HTML se sirve a todos los usuarios desde un CDN.

#### ¿Cómo funciona?

```
PASO A PASO DE SSG:

EN BUILD TIME (cuando despliegas):
1. Next.js ejecuta tu código
2. Consulta APIs/base de datos
3. Genera archivos HTML estáticos
4. Sube estos archivos a un CDN (Vercel, Cloudflare, etc.)

EN RUNTIME (cuando un usuario visita):
1. Usuario visita tusitio.com/productos
2. CDN sirve el HTML pre-generado (sin ejecutar código)
3. Tiempo de respuesta: 10-50ms (ULTRA RÁPIDO)

NO HAY SERVIDOR ejecutando código - solo archivos estáticos
```

#### Diagrama Visual

```
┌─────────────────────────────────────────────────────────────────┐
│                         SSG TIMELINE                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  BUILD TIME (una vez):                                          │
│  [npm run build] ──→ [Genera HTML] ──→ [Sube a CDN]            │
│                                                                 │
│  RUNTIME (cada visita):                                         │
│  [Request] ──→ [CDN sirve HTML] ──→ [Usuario ve contenido]     │
│                                                                 │
│     0s            10-50ms              50-200ms ✅               │
│                                                                 │
│  📱 ULTRA RÁPIDO - El HTML ya existe, solo se envía            │
│  📱 GRATIS - No hay servidor, solo CDN                          │
└─────────────────────────────────────────────────────────────────┘
```

#### Ejemplo de SSG en Next.js

```typescript
// app/blog/[slug]/page.tsx

// Esta función le dice a Next.js QUÉ páginas generar en build time
export async function generateStaticParams() {
  const posts = await db.post.findMany({ select: { slug: true } });
  
  // Retorna un array de parámetros
  // Next.js generará: /blog/post-1, /blog/post-2, etc.
  return posts.map(post => ({
    slug: post.slug,
  }));
}

// Este componente se ejecuta en BUILD TIME, no en runtime
async function BlogPostPage({ params }: { params: { slug: string } }) {
  const post = await db.post.findUnique({
    where: { slug: params.slug }
  });
  
  // Este HTML se genera UNA VEZ y se reutiliza
  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}

export default BlogPostPage;
```

#### ✅ Ventajas de SSG
- **Máximo rendimiento**: HTML pre-generado, 10-50ms de respuesta
- **Excelente SEO**: HTML completo disponible instantáneamente
- **Mínimo costo**: Solo necesitas un CDN (casi gratis)
- **Máxima escalabilidad**: Un CDN puede servir millones de usuarios
- **Seguridad**: Sin servidor = menos superficie de ataque

#### ❌ Desventajas de SSG
- **Datos pueden estar desactualizados**: Hasta el próximo build
- **Build time largo**: Miles de páginas = builds de minutos/horas
- **No personalizado**: Todos ven el mismo contenido
- **Necesitas rebuild**: Cada cambio de contenido requiere nuevo deploy

#### ¿Cuándo usar SSG?

```
✅ USA SSG CUANDO:
   • El contenido no cambia frecuentemente
   • El contenido es igual para todos los usuarios
   • Tienes blog, documentación, landing pages
   • Quieres máximo rendimiento y mínimo costo
   • Ej: Blog, docs, portfolio, marketing pages

❌ NO USES SSG CUANDO:
   • El contenido cambia cada minuto
   • Necesitas datos personalizados por usuario
   • Tienes miles de páginas que cambian frecuentemente
```

---

### ISR (Incremental Static Regeneration) - LO MEJOR DE AMBOS MUNDOS

#### ¿Qué es?
ISR combina SSG + SSR. Generas páginas estáticas pero se **regeneran automáticamente** en segundo plano después de un tiempo configurable.

#### ¿Cómo funciona?

```
PASO A PASO DE ISR:

1. BUILD TIME: Se generan páginas estáticas (como SSG)

2. RUNTIME - Primera visita:
   └── CDN sirve la página estática (ultra rápido)

3. DESPUÉS DE X SEGUNDOS (ej: 60):
   └── La página se considera "stale" (vieja)
   └── Siguiente usuario recibe la página vieja (aún rápido)
   └── EN SEGUNDO PLANO: Next.js regenera la página
   └── Nueva versión reemplaza la vieja

4. SIGUIENTE VISITA:
   └── Usuario recibe la página actualizada

RESULTADO: Velocidad de SSG + Frescura de SSR
```

#### Diagrama Visual

```
┌─────────────────────────────────────────────────────────────────┐
│                         ISR TIMELINE                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  t=0s: Página generada en build                                 │
│  ────────────────────────────────────────────────               │
│  │  Usuario A visita → recibe versión build (fresca)           │
│  │  Usuario B visita → recibe versión build (fresca)           │
│  ────────────────────────────────────────────────               │
│                                                                 │
│  t=60s: Página marcada como "stale"                             │
│  ────────────────────────────────────────────────               │
│  │  Usuario C visita → recibe versión vieja PERO               │
│  │                   → Next.js regenera en background          │
│  │  Usuario D visita → recibe versión NUEVA ✨                  │
│  ────────────────────────────────────────────────               │
│                                                                 │
│  📱 Usuarios SIEMPRE reciben respuesta rápida                   │
│  📱 Datos se actualizan automáticamente                         │
└─────────────────────────────────────────────────────────────────┘
```

#### Ejemplo de ISR en Next.js 15

```typescript
// app/productos/page.tsx - ISR con revalidación cada 60 segundos

// Opción 1: Revalidación basada en tiempo
export const revalidate = 60; // Regenerar cada 60 segundos

async function ProductosPage() {
  // Esto se ejecuta:
  // - Una vez en build time
  // - Luego cada 60 segundos cuando hay tráfico
  const productos = await db.producto.findMany();
  
  return (
    <div>
      <h1>Productos</h1>
      <p>Última actualización: {new Date().toISOString()}</p>
      {productos.map(p => <ProductCard key={p.id} producto={p} />)}
    </div>
  );
}

export default ProductosPage;
```

```typescript
// Opción 2: Revalidación bajo demanda (On-Demand ISR)
// Regenerar cuando TÚ lo decides (ej: cuando se actualiza un producto)

// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from 'next/cache';
import { NextRequest } from 'next/server';

export async function POST(request: NextRequest) {
  const { path, tag, secret } = await request.json();
  
  // Verificar secret para seguridad
  if (secret !== process.env.REVALIDATION_SECRET) {
    return Response.json({ error: 'Invalid secret' }, { status: 401 });
  }
  
  // Revalidar por path
  if (path) {
    revalidatePath(path);
    return Response.json({ revalidated: true, path });
  }
  
  // Revalidar por tag (más flexible)
  if (tag) {
    revalidateTag(tag);
    return Response.json({ revalidated: true, tag });
  }
}

// Uso con tags:
// app/productos/page.tsx
async function ProductosPage() {
  const productos = await fetch('https://api.example.com/productos', {
    next: { tags: ['productos'] } // Etiquetar este fetch
  });
  
  return <ListaProductos productos={productos} />;
}

// Cuando actualizas un producto en tu CMS:
// POST /api/revalidate { tag: 'productos', secret: 'tu-secret' }
// → Todas las páginas que usan tag 'productos' se regeneran
```

#### ✅ Ventajas de ISR
- **Velocidad de SSG**: Páginas pre-generadas, respuesta ultra rápida
- **Frescura de SSR**: Datos se actualizan automáticamente
- **Escalabilidad**: CDN sirve la mayoría del tráfico
- **Costo eficiente**: Regeneración solo cuando necesario
- **On-demand**: Puedes forzar regeneración al instante

#### ❌ Desventajas de ISR
- **Complejidad**: Más difícil de entender y debuggear
- **Puede mostrar datos viejos**: Usuario C ve datos de hace 60s
- **Solo en plataformas compatibles**: Vercel, Netlify (con adaptadores)

#### ¿Cuándo usar ISR?

```
✅ USA ISR CUANDO:
   • Tienes contenido que cambia cada hora/día
   • Quieres velocidad de SSG pero datos actualizados
   • Tienes e-commerce con productos que cambian
   • Tienes un blog con comentarios
   • Quieres controlar CUÁNDO se actualizan las páginas
   • Ej: E-commerce, blog con CMS, páginas con estadísticas

❌ NO USES ISR CUANDO:
   • Necesitas datos en tiempo real (usa SSR)
   • El contenido NUNCA cambia (usa SSG puro)
   • Necesitas personalización por usuario (usa SSR)
```

---

### Comparación Final: ¿Cuál Elegir?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TABLA COMPARATIVA DE RENDERIZADO                 │
├──────────────┬──────────────┬──────────────┬───────────┬────────────┤
│              │     CSR      │     SSR      │    SSG    │    ISR     │
├──────────────┼──────────────┼──────────────┼───────────┼────────────┤
│ HTML genera  │ Navegador    │ Servidor     │ Build     │ Build +    │
│ en           │              │ (cada req)   │ time      │ Background │
├──────────────┼──────────────┼──────────────┼───────────┼────────────┤
│ Velocidad    │ ❌ Lenta     │ ⚡ Media     │ ⚡⚡ Rápida│ ⚡⚡ Rápida │
│ inicial      │ (3-10s)      │ (1-3s)       │ (50-200ms)│ (50-200ms) │
├──────────────┼──────────────┼──────────────┼───────────┼────────────┤
│ SEO          │ ❌ Malo      │ ✅ Excelente │ ✅ Excelente│ ✅ Excelente│
├──────────────┼──────────────┼──────────────┼───────────┼────────────┤
│ Datos        │ ✅ Siempre   │ ✅ Siempre   │ ❌ Solo    │ ⚡ Casi    │
│ frescos      │ frescos      │ frescos      │ en build  │ siempre    │
├──────────────┼──────────────┼──────────────┼───────────┼────────────┤
│ Costo        │ ⚡ Bajo      │ ❌ Alto      │ ⚡ Muy bajo│ ⚡ Bajo    │
│ servidor     │ (solo CDN)   │ (CPU/RAM)    │ (solo CDN)│ (CDN+poco) │
├──────────────┼──────────────┼──────────────┼───────────┼────────────┤
│ Personaliz.  │ ✅ Sí        │ ✅ Sí        │ ❌ No     │ ❌ No      │
│ por usuario  │              │              │           │            │
├──────────────┼──────────────┼──────────────┼───────────┼────────────┤
│ Ideal para   │ Dashboards   │ Feeds,       │ Blogs,    │ E-commerce │
│              │ Apps internas│ búsquedas    │ docs      │ con CMS    │
└──────────────┴──────────────┴──────────────┴───────────┴────────────┘
```

#### Decisión Rápida

```
¿Necesitas SEO?
├── NO → ¿Es un dashboard/app privada? 
│        └── SÍ → CSR ✅
│
└── SÍ → ¿El contenido cambia cada minuto?
         ├── SÍ → ¿Es personalizado por usuario?
         │        ├── SÍ → SSR ✅
         │        └── NO → SSR o ISR (revalidate: 60) ✅
         │
         └── NO → ¿El contenido cambia cada hora/día?
                  ├── SÍ → ISR ✅
                  └── NO → SSG ✅
```

---

## Fundamentos de Next.js

### ¿Qué es Next.js?

Next.js es un **framework de React** que resuelve los problemas de React puro:

```
┌─────────────────────────────────────────────────────────────────┐
│              REACT PURO vs NEXT.JS                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  REACT PURO (Vite/CRA):                                         │
│  • Solo CSR (client-side rendering)                             │
│  • Tú configuras el routing (React Router)                      │
│  • Tú configuras SSR si lo necesitas (complicado)               │
│  • Tú optimizas imágenes, fonts, etc.                           │
│  • Tú configuras webpack/bundler                                │
│                                                                 │
│  NEXT.JS:                                                       │
│  • SSR, SSG, ISR, CSR - tú eliges                               │
│  • Routing basado en archivos (automático)                      │
│  • Optimización automática de imágenes y fonts                  │
│  • Configuración zero-config (funciona de caja)                 │
│  • Server Components de React                                   │
│  • API Routes integradas                                        │
│  • Middleware, caching, y más                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### ¿Por qué usar Next.js?

1. **Rendimiento optimizado**: Imágenes, fonts, scripts optimizados automáticamente
2. **SEO listo**: SSR/SSG para que Google indexe tu contenido
3. **DX (Developer Experience)**: Hot reload, TypeScript, ESLint configurados
4. **Full-stack**: API Routes te permiten tener backend en el mismo proyecto
5. **Despliegue fácil**: Vercel (creadores de Next.js) hace deploy en 1 click

### Instalación y Setup

```bash
# Crear nuevo proyecto
npx create-next-app@latest my-app

# Opciones interactivas:
# ✔ Would you like to use TypeScript? Yes
# ✔ Would you like to use ESLint? Yes
# ✔ Would you like to use Tailwind CSS? Yes
# ✔ Would you like to use `src/` directory? Yes
# ✔ Would you like to use App Router? Yes
# ✔ Would you like to customize the default import alias? No
```

### Estructura del Proyecto

```
my-app/
├── src/
│   ├── app/                    # App Router
│   │   ├── layout.tsx          # Root layout
│   │   ├── page.tsx            # Home page
│   │   ├── loading.tsx         # Loading UI
│   │   ├── error.tsx           # Error UI
│   │   ├── not-found.tsx       # 404 page
│   │   ├── globals.css         # Global styles
│   │   ├── api/                # API routes
│   │   │   └── route.ts
│   │   ├── dashboard/          # Route segment
│   │   │   ├── page.tsx
│   │   │   └── layout.tsx
│   │   └── [...slug]/          # Catch-all route
│   │       └── page.tsx
│   ├── components/             # React components
│   ├── lib/                    # Utility functions
│   └── hooks/                  # Custom hooks
├── public/                     # Static files
├── next.config.js              # Next.js config
├── tailwind.config.ts          # Tailwind config
└── tsconfig.json               # TypeScript config
```

---

## App Router

### Convenciones de Archivos

```typescript
// app/page.tsx - Página (UI única para una ruta)
export default function Page() {
  return <h1>Home</h1>;
}

// app/layout.tsx - Layout (UI compartida entre rutas)
export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}

// app/loading.tsx - Loading UI (Suspense fallback)
export default function Loading() {
  return <div>Loading...</div>;
}

// app/error.tsx - Error UI (Error Boundary)
'use client';
export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}

// app/not-found.tsx - 404 UI
export default function NotFound() {
  return (
    <div>
      <h2>Not Found</h2>
      <p>Could not find requested resource</p>
    </div>
  );
}

// app/template.tsx - Similar a layout pero se re-monta en navegación
export default function Template({ children }: { children: React.ReactNode }) {
  return <div>{children}</div>;
}
```

### Rutas Dinámicas

```typescript
// app/blog/[slug]/page.tsx
interface PageProps {
  params: Promise<{ slug: string }>;
}

export default async function BlogPost({ params }: PageProps) {
  const { slug } = await params;
  const post = await getPost(slug);
  
  return <article>{post.content}</article>;
}

// Generar rutas estáticas en build time
export async function generateStaticParams() {
  const posts = await getPosts();
  
  return posts.map((post) => ({
    slug: post.slug,
  }));
}

// app/shop/[...slug]/page.tsx - Catch-all segments
// Matches: /shop/a, /shop/a/b, /shop/a/b/c
interface PageProps {
  params: Promise<{ slug: string[] }>;
}

export default async function ShopPage({ params }: PageProps) {
  const { slug } = await params;
  // slug es un array: ['a', 'b', 'c']
  return <div>Path: {slug.join('/')}</div>;
}

// app/shop/[[...slug]]/page.tsx - Optional catch-all
// También matches: /shop (slug será undefined)
```

### Route Groups

```typescript
// Organizar rutas sin afectar URL
// app/(marketing)/about/page.tsx → /about
// app/(marketing)/contact/page.tsx → /contact
// app/(shop)/products/page.tsx → /products

// app/(marketing)/layout.tsx
export default function MarketingLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="marketing-layout">
      <MarketingHeader />
      {children}
      <MarketingFooter />
    </div>
  );
}

// app/(shop)/layout.tsx
export default function ShopLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="shop-layout">
      <ShopHeader />
      <ShopSidebar />
      {children}
    </div>
  );
}
```

### Parallel Routes

```typescript
// Renderizar múltiples páginas en la misma vista
// app/@team/page.tsx
// app/@analytics/page.tsx
// app/layout.tsx

// app/layout.tsx
export default function Layout({
  children,
  team,
  analytics,
}: {
  children: React.ReactNode;
  team: React.ReactNode;
  analytics: React.ReactNode;
}) {
  return (
    <div>
      <main>{children}</main>
      <aside>
        {team}
        {analytics}
      </aside>
    </div>
  );
}

// Conditional rendering basado en autenticación
// app/@auth/login/page.tsx
// app/@auth/default.tsx (fallback cuando no hay match)
export default function Layout({
  children,
  auth,
}: {
  children: React.ReactNode;
  auth: React.ReactNode;
}) {
  const isLoggedIn = checkAuth();
  return isLoggedIn ? children : auth;
}
```

### Intercepting Routes

```typescript
// Interceptar una ruta para mostrar en modal
// app/feed/page.tsx - Lista de posts
// app/feed/@modal/(.)post/[id]/page.tsx - Modal interceptando /post/[id]
// app/post/[id]/page.tsx - Página completa del post

// (.) intercepta el mismo nivel
// (..) intercepta un nivel arriba
// (...) intercepta desde root

// app/feed/@modal/(.)post/[id]/page.tsx
export default async function PostModal({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  const post = await getPost(id);
  
  return (
    <Modal>
      <PostContent post={post} />
    </Modal>
  );
}

// app/feed/layout.tsx
export default function FeedLayout({
  children,
  modal,
}: {
  children: React.ReactNode;
  modal: React.ReactNode;
}) {
  return (
    <>
      {children}
      {modal}
    </>
  );
}
```

---

## Server Components vs Client Components

> 🎯 **CONCEPTO CRUCIAL**: Esta es una de las innovaciones más importantes de React 18/19 y Next.js 13+. Entenderlo bien es FUNDAMENTAL.

### ¿Qué es un Componente en React?

Antes de entender Server vs Client Components, recordemos qué es un componente:

```
COMPONENTE = Función que recibe datos (props) y retorna UI (JSX)

function Saludo({ nombre }) {
  return <h1>Hola, {nombre}</h1>;
}

La pregunta es: ¿DÓNDE se ejecuta esta función?
```

### El Problema del Modelo Tradicional (Solo Cliente)

En React tradicional (CSR), **TODO** se ejecuta en el navegador del usuario:

```
┌─────────────────────────────────────────────────────────────────┐
│          REACT TRADICIONAL - TODO EN EL CLIENTE                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SERVIDOR:                          NAVEGADOR DEL USUARIO:      │
│  ┌──────────────────────┐          ┌──────────────────────┐    │
│  │ Solo sirve archivos  │   ──→    │ Descarga JS (500KB+) │    │
│  │ estáticos (HTML, JS) │          │ Ejecuta TODOS los    │    │
│  └──────────────────────┘          │ componentes          │    │
│                                    │ Hace fetch a APIs    │    │
│                                    │ Renderiza UI         │    │
│                                    └──────────────────────┘    │
│                                                                 │
│  PROBLEMAS:                                                     │
│  • El navegador descarga TODO el código de la app               │
│  • Incluye librerías que solo se usan para procesar datos       │
│  • El teléfono del usuario ejecuta código que no necesita       │
│  • Datos sensibles pueden filtrarse al bundle                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### La Solución: Server Components

React Server Components (RSC) divide los componentes en dos tipos:

```
┌─────────────────────────────────────────────────────────────────┐
│            NUEVO MODELO: SERVER + CLIENT COMPONENTS             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SERVIDOR:                          NAVEGADOR:                  │
│  ┌──────────────────────┐          ┌──────────────────────┐    │
│  │ Server Components    │   ──→    │ Client Components    │    │
│  │ • Consultan DB       │   HTML   │ • Interactividad     │    │
│  │ • Procesan datos     │   +      │ • Hooks (useState)   │    │
│  │ • Acceden a archivos │   JSON   │ • Event handlers     │    │
│  │ • Usan secretos      │          │ • Browser APIs       │    │
│  │                      │          │                      │    │
│  │ NO se envían al      │          │ Solo estos se envían │    │
│  │ navegador ✅         │          │ al navegador         │    │
│  └──────────────────────┘          └──────────────────────┘    │
│                                                                 │
│  RESULTADO: Bundle más pequeño, más rápido, más seguro          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Server Components - En Detalle

#### ¿Qué es un Server Component?

Un Server Component es un componente que se ejecuta **ÚNICAMENTE en el servidor**. Su código JavaScript NUNCA llega al navegador del usuario.

```typescript
// app/productos/page.tsx - Este es un SERVER COMPONENT (por defecto)

// ✅ Este import NUNCA llega al navegador
import { db } from '@/lib/database';  // Conexión a PostgreSQL
import { formatPrice } from 'accounting';  // Librería de 50KB

async function ProductosPage() {
  // ✅ Este código se ejecuta EN EL SERVIDOR
  // El usuario NUNCA ve este código en DevTools
  
  const productos = await db.producto.findMany({
    include: { categoria: true }
  });
  
  // ✅ Puedes usar async/await directamente (sin useEffect)
  // ✅ La librería 'accounting' NO se envía al navegador
  
  return (
    <div>
      <h1>Productos</h1>
      {productos.map(p => (
        <div key={p.id}>
          <h2>{p.nombre}</h2>
          {/* formatPrice se ejecutó en servidor, solo el resultado llega al cliente */}
          <p>{formatPrice(p.precio)}</p>
        </div>
      ))}
    </div>
  );
}

export default ProductosPage;
```

#### ¿Qué pueden hacer los Server Components?

```
✅ SERVER COMPONENTS PUEDEN:

1. ACCEDER A BASE DE DATOS DIRECTAMENTE
   const users = await prisma.user.findMany();
   
2. USAR VARIABLES DE ENTORNO SECRETAS
   const apiKey = process.env.SECRET_API_KEY; // Seguro
   
3. LEER ARCHIVOS DEL SISTEMA
   const content = await fs.readFile('./data.json');
   
4. HACER FETCH A APIs PRIVADAS
   const data = await fetch(internalApi, { headers: { secret } });
   
5. IMPORTAR LIBRERÍAS PESADAS (sin afectar bundle)
   import { marked } from 'marked';  // 30KB que NO van al cliente
   import hljs from 'highlight.js';  // 1MB que NO va al cliente
   
6. SER ASYNC (sin useEffect)
   async function Page() {
     const data = await getData();
     return <div>{data}</div>;
   }
```

```
❌ SERVER COMPONENTS NO PUEDEN:

1. USAR HOOKS DE REACT
   useState()     // ❌ Error
   useEffect()    // ❌ Error
   useContext()   // ❌ Error (pero sí puedes leer de Context)
   
2. AGREGAR EVENT HANDLERS
   <button onClick={...}>  // ❌ Error
   <input onChange={...}>  // ❌ Error
   
3. USAR BROWSER APIs
   window.localStorage     // ❌ No existe en servidor
   document.getElementById // ❌ No existe en servidor
   navigator.geolocation   // ❌ No existe en servidor
   
4. USAR EFECTOS O ESTADO
   No hay ciclo de vida del componente
   No hay re-renders
   Se ejecuta UNA vez y ya
```

### Client Components - En Detalle

#### ¿Qué es un Client Component?

Un Client Component es un componente que se ejecuta en el navegador del usuario. Necesita la directiva `'use client'` al inicio del archivo.

```typescript
// components/AddToCartButton.tsx
'use client';  // <-- Esta línea hace que sea Client Component

import { useState } from 'react';

// Este componente se ejecuta en el NAVEGADOR del usuario
// Su código SÍ se envía al navegador

export function AddToCartButton({ productId }: { productId: string }) {
  // ✅ Puedes usar hooks
  const [isAdding, setIsAdding] = useState(false);
  const [quantity, setQuantity] = useState(1);
  
  // ✅ Puedes usar event handlers
  async function handleAddToCart() {
    setIsAdding(true);
    
    await fetch('/api/cart', {
      method: 'POST',
      body: JSON.stringify({ productId, quantity })
    });
    
    setIsAdding(false);
  }
  
  return (
    <div>
      <input 
        type="number" 
        value={quantity} 
        onChange={e => setQuantity(Number(e.target.value))} // ✅ Event handler
      />
      <button onClick={handleAddToCart} disabled={isAdding}>
        {isAdding ? 'Agregando...' : 'Agregar al carrito'}
      </button>
    </div>
  );
}
```

#### ¿Qué pueden hacer los Client Components?

```
✅ CLIENT COMPONENTS PUEDEN:

1. USAR HOOKS DE REACT
   const [state, setState] = useState();
   useEffect(() => {...}, []);
   const context = useContext(MyContext);
   
2. AGREGAR INTERACTIVIDAD
   <button onClick={handleClick}>
   <input onChange={handleChange}>
   <form onSubmit={handleSubmit}>
   
3. USAR BROWSER APIs
   localStorage.getItem('token')
   window.innerWidth
   navigator.geolocation.getCurrentPosition()
   
4. MANEJAR ESTADO LOCAL
   Re-renders cuando el estado cambia
   Ciclo de vida del componente
   
5. USAR LIBRERÍAS QUE REQUIEREN BROWSER
   import { motion } from 'framer-motion';
   import { Chart } from 'chart.js';
```

```
❌ CLIENT COMPONENTS NO PUEDEN:

1. IMPORTAR CÓDIGO SOLO-SERVIDOR
   import { db } from '@/lib/database';  // ❌ Error en cliente
   
2. SER ASYNC DIRECTAMENTE
   async function Component() {...}  // ❌ Error
   // (debe usar useEffect + useState)
   
3. ACCEDER A SECRETOS
   process.env.SECRET_KEY  // ❌ No disponible (solo NEXT_PUBLIC_*)
```

### ¿Cuándo Usar Cada Uno?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DECISIÓN: SERVER vs CLIENT                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  USA SERVER COMPONENT CUANDO:            USA CLIENT COMPONENT CUANDO:│
│  ─────────────────────────────          ─────────────────────────── │
│                                                                     │
│  • Mostrar datos de DB/API              • Necesitas onClick/onChange │
│  • Procesar markdown/datos              • Necesitas useState        │
│  • Acceder a sistema de archivos        • Necesitas useEffect       │
│  • Usar librerías pesadas               • Necesitas useContext      │
│  • Mantener secretos seguros            • Animaciones (framer)      │
│  • El componente no necesita            • Formularios interactivos  │
│    interactividad                       • Acceder a localStorage    │
│                                         • Geolocalización           │
│  Ejemplos:                              • Tiempo real (websockets)  │
│  • Página de producto                                               │
│  • Lista de artículos                   Ejemplos:                   │
│  • Perfil de usuario (lectura)          • Carrito de compras        │
│  • Sidebar con datos                    • Formulario de login       │
│  • Footer con links                     • Modal/Dropdown            │
│                                         • Slider/Carousel           │
│                                         • Chat en tiempo real       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Composición: Server + Client Components Juntos

El poder real está en **combinarlos**:

```typescript
// app/productos/[id]/page.tsx - SERVER COMPONENT (página)
import { db } from '@/lib/database';
import { AddToCartButton } from '@/components/AddToCartButton';
import { ProductGallery } from '@/components/ProductGallery';
import { marked } from 'marked';  // Librería pesada, no va al cliente

async function ProductoPage({ params }: { params: { id: string } }) {
  // ✅ Fetch en servidor - rápido, seguro, sin loading states
  const producto = await db.producto.findUnique({
    where: { id: params.id },
    include: { imagenes: true, reviews: true }
  });
  
  // ✅ Procesar markdown en servidor (marked no va al bundle del cliente)
  const descripcionHtml = marked(producto.descripcion);
  
  return (
    <div>
      {/* Server Component renderiza datos estáticos */}
      <h1>{producto.nombre}</h1>
      <p className="precio">${producto.precio}</p>
      
      {/* Client Component para interactividad */}
      <ProductGallery imagenes={producto.imagenes} />
      
      {/* Otro Client Component */}
      <AddToCartButton productId={producto.id} />
      
      {/* HTML procesado en servidor */}
      <div dangerouslySetInnerHTML={{ __html: descripcionHtml }} />
      
      {/* Server Component para reviews */}
      <ReviewsList reviews={producto.reviews} />
    </div>
  );
}
```

```typescript
// components/ProductGallery.tsx - CLIENT COMPONENT
'use client';

import { useState } from 'react';
import Image from 'next/image';

interface Props {
  imagenes: { url: string; alt: string }[];
}

export function ProductGallery({ imagenes }: Props) {
  // ✅ Estado para imagen activa
  const [activeIndex, setActiveIndex] = useState(0);
  
  return (
    <div>
      {/* Imagen principal */}
      <Image 
        src={imagenes[activeIndex].url}
        alt={imagenes[activeIndex].alt}
        width={500}
        height={500}
      />
      
      {/* Thumbnails con interactividad */}
      <div className="thumbnails">
        {imagenes.map((img, i) => (
          <button 
            key={i}
            onClick={() => setActiveIndex(i)}  // ✅ Event handler
            className={i === activeIndex ? 'active' : ''}
          >
            <Image src={img.url} alt={img.alt} width={80} height={80} />
          </button>
        ))}
      </div>
    </div>
  );
}
```

### Patrón Importante: Server Component como Children

```typescript
// ❌ PROBLEMA: No puedes importar Server Component dentro de Client Component

// components/Modal.tsx
'use client';
import { ServerContent } from './ServerContent';  // ❌ Esto NO funciona

export function Modal() {
  return <div className="modal"><ServerContent /></div>;
}
```

```typescript
// ✅ SOLUCIÓN: Pasar Server Component como children

// components/Modal.tsx - Client Component
'use client';

import { useState } from 'react';

interface Props {
  children: React.ReactNode;  // Aceptar children
  trigger: React.ReactNode;
}

export function Modal({ children, trigger }: Props) {
  const [isOpen, setIsOpen] = useState(false);
  
  return (
    <>
      <button onClick={() => setIsOpen(true)}>{trigger}</button>
      
      {isOpen && (
        <div className="modal-overlay">
          <div className="modal-content">
            {children}  {/* ✅ Server Component puede ir aquí */}
            <button onClick={() => setIsOpen(false)}>Cerrar</button>
          </div>
        </div>
      )}
    </>
  );
}

// app/page.tsx - Server Component
import { Modal } from '@/components/Modal';
import { ProductDetails } from '@/components/ProductDetails';

export default async function Page() {
  return (
    <Modal trigger="Ver detalles">
      {/* ProductDetails es Server Component, se pasa como children */}
      <ProductDetails productId="123" />
    </Modal>
  );
}
```

### Resumen Visual

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SERVER vs CLIENT - RESUMEN                       │
├────────────────────────────┬────────────────────────────────────────┤
│     SERVER COMPONENT       │         CLIENT COMPONENT               │
├────────────────────────────┼────────────────────────────────────────┤
│ Ejecuta en: SERVIDOR       │ Ejecuta en: NAVEGADOR                  │
│ Directiva: (ninguna)       │ Directiva: 'use client'                │
│ Puede ser async: SÍ        │ Puede ser async: NO                    │
│ Hooks: NO                  │ Hooks: SÍ                              │
│ Events: NO                 │ Events: SÍ                             │
│ DB Access: SÍ              │ DB Access: NO (usa API)                │
│ Secretos: SÍ               │ Secretos: NO                           │
│ Bundle size: 0 bytes       │ Bundle size: Código enviado            │
├────────────────────────────┴────────────────────────────────────────┤
│                                                                     │
│  REGLA DE ORO: Por defecto, usa Server Component.                   │
│  Solo usa 'use client' cuando NECESITES interactividad.             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## App Router - Convenciones de Archivos

### Archivos Especiales

```typescript
// app/page.tsx - Página (UI única para una ruta)
export default function Page() {
  return <h1>Home</h1>;
}

// app/layout.tsx - Layout (UI compartida entre rutas)
export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}

// app/loading.tsx - Loading UI (Suspense fallback)
export default function Loading() {
  return <div>Loading...</div>;
}

// app/error.tsx - Error UI (Error Boundary)
'use client';
export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}

// app/not-found.tsx - 404 UI
export default function NotFound() {
  return (
    <div>
      <h2>Not Found</h2>
      <p>Could not find requested resource</p>
    </div>
  );
}

// app/template.tsx - Similar a layout pero se re-monta en navegación
export default function Template({ children }: { children: React.ReactNode }) {
  return <div>{children}</div>;
}
```
      ))}
    </ul>
  );
}

// Request deduplication - Next.js automáticamente deduplica
// requests idénticos en el mismo render pass
async function Layout({ children }) {
  const user = await getUser(); // Request 1
  return <div>{children}</div>;
}

async function Page() {
  const user = await getUser(); // No hace otro request, usa el mismo
  return <div>{user.name}</div>;
}
```

### Data Fetching Patterns

```typescript
// 1. Sequential Data Fetching
async function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  
  // Sequential - uno después de otro
  const artist = await getArtist(id);
  const albums = await getAlbums(artist.id); // Espera a artist
  
  return (
    <div>
      <ArtistInfo artist={artist} />
      <Albums albums={albums} />
    </div>
  );
}

// 2. Parallel Data Fetching
async function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  
  // Parallel - simultáneo
  const [artist, albums, relatedArtists] = await Promise.all([
    getArtist(id),
    getAlbums(id),
    getRelatedArtists(id)
  ]);
  
  return (
    <div>
      <ArtistInfo artist={artist} />
      <Albums albums={albums} />
      <RelatedArtists artists={relatedArtists} />
    </div>
  );
}

// 3. Preloading Data
// lib/data.ts
import { cache } from 'react';

// cache() de React evita duplicados en el mismo render
export const getUser = cache(async (id: string) => {
  const res = await fetch(`/api/users/${id}`);
  return res.json();
});

// Preload function para empezar fetch antes
export const preloadUser = (id: string) => {
  void getUser(id);
};

// Uso
import { preloadUser, getUser } from '@/lib/data';

export default async function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  preloadUser(id); // Empieza el fetch inmediatamente
  
  // ... otro código que puede tomar tiempo
  
  const user = await getUser(id); // Ya puede estar listo
  return <Profile user={user} />;
}
```

### ORM y Database

```typescript
// Usar Prisma con Server Components
// lib/prisma.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma = globalForPrisma.prisma ?? new PrismaClient();

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;

// app/users/page.tsx
import { prisma } from '@/lib/prisma';

async function getUsers() {
  return prisma.user.findMany({
    include: { posts: true },
    orderBy: { createdAt: 'desc' }
  });
}

export default async function UsersPage() {
  const users = await getUsers();
  
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          {user.name} - {user.posts.length} posts
        </li>
      ))}
    </ul>
  );
}

// Con Drizzle ORM
// lib/db.ts
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
});

export const db = drizzle(pool);

// app/posts/page.tsx
import { db } from '@/lib/db';
import { posts, users } from '@/lib/schema';
import { eq } from 'drizzle-orm';

export default async function PostsPage() {
  const allPosts = await db
    .select()
    .from(posts)
    .leftJoin(users, eq(posts.authorId, users.id));
  
  return <PostList posts={allPosts} />;
}
```

---

## Server Actions

### Definir Server Actions

```typescript
// 1. Inline en Server Component
// app/posts/page.tsx
export default function PostsPage() {
  async function createPost(formData: FormData) {
    'use server';
    
    const title = formData.get('title') as string;
    const content = formData.get('content') as string;
    
    await db.post.create({
      data: { title, content }
    });
    
    revalidatePath('/posts');
  }
  
  return (
    <form action={createPost}>
      <input name="title" placeholder="Title" />
      <textarea name="content" placeholder="Content" />
      <button type="submit">Create Post</button>
    </form>
  );
}

// 2. En archivo separado
// app/actions.ts
'use server';

import { revalidatePath, revalidateTag } from 'next/cache';
import { redirect } from 'next/navigation';
import { z } from 'zod';

const PostSchema = z.object({
  title: z.string().min(1).max(100),
  content: z.string().min(10),
});

export async function createPost(prevState: any, formData: FormData) {
  // Validación
  const validatedFields = PostSchema.safeParse({
    title: formData.get('title'),
    content: formData.get('content'),
  });
  
  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
      message: 'Invalid fields',
    };
  }
  
  const { title, content } = validatedFields.data;
  
  try {
    await db.post.create({ data: { title, content } });
  } catch (error) {
    return { message: 'Database error' };
  }
  
  revalidatePath('/posts');
  redirect('/posts');
}

export async function deletePost(id: string) {
  await db.post.delete({ where: { id } });
  revalidateTag('posts');
}
```

### Usar Server Actions en Client Components

```typescript
// components/CreatePostForm.tsx
'use client';

import { useActionState } from 'react';
import { useFormStatus } from 'react-dom';
import { createPost } from '@/app/actions';

function SubmitButton() {
  const { pending } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Creating...' : 'Create Post'}
    </button>
  );
}

export function CreatePostForm() {
  const initialState = { message: '', errors: {} };
  const [state, formAction] = useActionState(createPost, initialState);
  
  return (
    <form action={formAction}>
      <div>
        <label htmlFor="title">Title</label>
        <input id="title" name="title" />
        {state.errors?.title && (
          <p className="error">{state.errors.title}</p>
        )}
      </div>
      
      <div>
        <label htmlFor="content">Content</label>
        <textarea id="content" name="content" />
        {state.errors?.content && (
          <p className="error">{state.errors.content}</p>
        )}
      </div>
      
      {state.message && (
        <p className="error">{state.message}</p>
      )}
      
      <SubmitButton />
    </form>
  );
}

// Usar Server Action sin form
'use client';

import { deletePost } from '@/app/actions';
import { useTransition } from 'react';

export function DeleteButton({ postId }: { postId: string }) {
  const [isPending, startTransition] = useTransition();
  
  const handleDelete = () => {
    startTransition(async () => {
      await deletePost(postId);
    });
  };
  
  return (
    <button onClick={handleDelete} disabled={isPending}>
      {isPending ? 'Deleting...' : 'Delete'}
    </button>
  );
}
```

### Optimistic Updates con Server Actions

```typescript
'use client';

import { useOptimistic } from 'react';
import { likePost } from '@/app/actions';

interface Post {
  id: string;
  likes: number;
  isLiked: boolean;
}

export function LikeButton({ post }: { post: Post }) {
  const [optimisticPost, addOptimisticLike] = useOptimistic(
    post,
    (state, optimisticValue: boolean) => ({
      ...state,
      likes: optimisticValue ? state.likes + 1 : state.likes - 1,
      isLiked: optimisticValue,
    })
  );
  
  async function handleLike() {
    addOptimisticLike(!optimisticPost.isLiked);
    await likePost(post.id);
  }
  
  return (
    <form action={handleLike}>
      <button type="submit">
        {optimisticPost.isLiked ? '❤️' : '🤍'} {optimisticPost.likes}
      </button>
    </form>
  );
}
```

---

## Caching y Revalidación

### Tipos de Cache en Next.js

```
┌─────────────────────────────────────────────────────────┐
│                   NEXT.JS CACHING                       │
├─────────────────────────────────────────────────────────┤
│  1. Request Memoization (React)                         │
│     - Deduplica fetch durante un render                 │
│                                                         │
│  2. Data Cache (Next.js)                                │
│     - Persiste resultados de fetch entre requests       │
│                                                         │
│  3. Full Route Cache (Next.js)                          │
│     - Cachea HTML y RSC payload en build                │
│                                                         │
│  4. Router Cache (Client)                               │
│     - Cachea RSC payload en navegación client-side      │
└─────────────────────────────────────────────────────────┘
```

### Configurar Caching

```typescript
// 1. Por fetch individual
// Cache indefinido (default)
fetch('https://api.example.com/data');

// Sin cache
fetch('https://api.example.com/data', { cache: 'no-store' });

// Revalidación por tiempo
fetch('https://api.example.com/data', { 
  next: { revalidate: 3600 } // Revalidar cada hora
});

// Con tags para revalidación on-demand
fetch('https://api.example.com/posts', { 
  next: { tags: ['posts'] } 
});

// 2. Por segmento de ruta
// app/posts/page.tsx
export const revalidate = 3600; // Revalidar cada hora
export const dynamic = 'force-dynamic'; // Sin cache
export const fetchCache = 'force-no-store'; // Sin cache en fetches

// 3. En next.config.js
module.exports = {
  experimental: {
    staleTimes: {
      dynamic: 30, // Router Cache para páginas dinámicas
      static: 180, // Router Cache para páginas estáticas
    },
  },
};
```

### Revalidación

```typescript
// app/actions.ts
'use server';

import { revalidatePath, revalidateTag } from 'next/cache';

// Revalidar una ruta específica
export async function updatePost(id: string, data: PostData) {
  await db.post.update({ where: { id }, data });
  
  revalidatePath('/posts');           // Revalida /posts
  revalidatePath('/posts/[id]', 'page'); // Revalida páginas dinámicas
  revalidatePath('/', 'layout');      // Revalida todo bajo root layout
}

// Revalidar por tag
export async function deletePost(id: string) {
  await db.post.delete({ where: { id } });
  
  revalidateTag('posts'); // Revalida todos los fetch con tag 'posts'
}

// Revalidación on-demand vía API
// app/api/revalidate/route.ts
import { revalidateTag } from 'next/cache';
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  const { tag, secret } = await request.json();
  
  if (secret !== process.env.REVALIDATION_SECRET) {
    return NextResponse.json({ error: 'Invalid secret' }, { status: 401 });
  }
  
  revalidateTag(tag);
  
  return NextResponse.json({ revalidated: true, now: Date.now() });
}
```

---

## Middleware

```typescript
// middleware.ts (en root del proyecto)
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // 1. Redirect
  if (request.nextUrl.pathname === '/old-page') {
    return NextResponse.redirect(new URL('/new-page', request.url));
  }
  
  // 2. Rewrite (cambiar destino sin cambiar URL)
  if (request.nextUrl.pathname.startsWith('/api/v1')) {
    return NextResponse.rewrite(
      new URL(request.nextUrl.pathname.replace('/api/v1', '/api/v2'), request.url)
    );
  }
  
  // 3. Agregar headers
  const response = NextResponse.next();
  response.headers.set('x-custom-header', 'value');
  
  // 4. Leer cookies
  const token = request.cookies.get('token');
  
  // 5. Autenticación
  if (request.nextUrl.pathname.startsWith('/dashboard')) {
    if (!token) {
      return NextResponse.redirect(new URL('/login', request.url));
    }
  }
  
  return response;
}

// Configurar en qué rutas ejecutar
export const config = {
  matcher: [
    // Ejecutar en todas excepto static files y api routes
    '/((?!_next/static|_next/image|favicon.ico|api).*)',
    // O especificar rutas específicas
    '/dashboard/:path*',
    '/admin/:path*',
  ],
};
```

### Middleware con Auth

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import { verifyToken } from '@/lib/auth';

const publicRoutes = ['/', '/login', '/register', '/about'];
const authRoutes = ['/login', '/register'];

export async function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;
  const token = request.cookies.get('token')?.value;
  
  // Verificar token si existe
  const user = token ? await verifyToken(token) : null;
  const isAuthenticated = !!user;
  
  // Rutas de auth - redirigir a dashboard si ya autenticado
  if (authRoutes.includes(pathname) && isAuthenticated) {
    return NextResponse.redirect(new URL('/dashboard', request.url));
  }
  
  // Rutas protegidas - redirigir a login si no autenticado
  if (!publicRoutes.includes(pathname) && !isAuthenticated) {
    const loginUrl = new URL('/login', request.url);
    loginUrl.searchParams.set('callbackUrl', pathname);
    return NextResponse.redirect(loginUrl);
  }
  
  // Agregar user info al request para uso posterior
  const response = NextResponse.next();
  if (user) {
    response.headers.set('x-user-id', user.id);
    response.headers.set('x-user-role', user.role);
  }
  
  return response;
}

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico|api).*)'],
};
```

---

## Optimizaciones

### Image Optimization

```typescript
import Image from 'next/image';

// Imagen local
import profilePic from '@/public/profile.jpg';

function Avatar() {
  return (
    <Image
      src={profilePic}
      alt="Profile picture"
      // width y height se infieren del import
      placeholder="blur" // Blur placeholder automático para imports locales
    />
  );
}

// Imagen remota
function RemoteImage() {
  return (
    <Image
      src="https://example.com/image.jpg"
      alt="Remote image"
      width={500}
      height={300}
      // Opciones adicionales
      quality={75} // Default 75
      priority // Cargar con prioridad (para LCP)
      loading="lazy" // Default, o "eager"
      placeholder="blur"
      blurDataURL="data:image/..." // Requerido para placeholder blur con remoto
      sizes="(max-width: 768px) 100vw, 50vw" // Responsive sizes
    />
  );
}

// Configurar dominios permitidos
// next.config.js
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'example.com',
        pathname: '/images/**',
      },
      {
        protocol: 'https',
        hostname: '**.cloudinary.com',
      },
    ],
    // O usar domains (deprecated pero simple)
    domains: ['example.com', 'cdn.example.com'],
  },
};

// Fill container
function BackgroundImage() {
  return (
    <div className="relative h-64 w-full">
      <Image
        src="/hero.jpg"
        alt="Hero"
        fill
        className="object-cover"
        sizes="100vw"
      />
    </div>
  );
}
```

### Font Optimization

```typescript
// app/layout.tsx
import { Inter, Roboto_Mono } from 'next/font/google';

// Google Fonts - automaticamente self-hosted
const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter',
});

const robotoMono = Roboto_Mono({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-roboto-mono',
});

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={`${inter.variable} ${robotoMono.variable}`}>
      <body className={inter.className}>{children}</body>
    </html>
  );
}

// Local fonts
import localFont from 'next/font/local';

const myFont = localFont({
  src: [
    { path: './fonts/MyFont-Regular.woff2', weight: '400', style: 'normal' },
    { path: './fonts/MyFont-Bold.woff2', weight: '700', style: 'normal' },
  ],
  variable: '--font-my-font',
});

// Usar en CSS/Tailwind
// globals.css
:root {
  --font-sans: var(--font-inter);
  --font-mono: var(--font-roboto-mono);
}

// tailwind.config.ts
module.exports = {
  theme: {
    extend: {
      fontFamily: {
        sans: ['var(--font-inter)'],
        mono: ['var(--font-roboto-mono)'],
      },
    },
  },
};
```

### Metadata y SEO

```typescript
// app/layout.tsx - Metadata estática
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: {
    default: 'My App',
    template: '%s | My App', // Para páginas hijas
  },
  description: 'My awesome Next.js application',
  keywords: ['Next.js', 'React', 'TypeScript'],
  authors: [{ name: 'John Doe', url: 'https://johndoe.com' }],
  creator: 'John Doe',
  openGraph: {
    type: 'website',
    locale: 'en_US',
    url: 'https://myapp.com',
    siteName: 'My App',
    title: 'My App',
    description: 'My awesome Next.js application',
    images: [
      {
        url: 'https://myapp.com/og-image.jpg',
        width: 1200,
        height: 630,
        alt: 'My App OG Image',
      },
    ],
  },
  twitter: {
    card: 'summary_large_image',
    title: 'My App',
    description: 'My awesome Next.js application',
    creator: '@johndoe',
    images: ['https://myapp.com/twitter-image.jpg'],
  },
  robots: {
    index: true,
    follow: true,
  },
  icons: {
    icon: '/favicon.ico',
    apple: '/apple-icon.png',
  },
  manifest: '/manifest.json',
};

// app/blog/[slug]/page.tsx - Metadata dinámica
import type { Metadata } from 'next';

interface Props {
  params: Promise<{ slug: string }>;
}

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { slug } = await params;
  const post = await getPost(slug);
  
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [post.coverImage],
      type: 'article',
      publishedTime: post.publishedAt,
      authors: [post.author.name],
    },
  };
}

export default async function BlogPost({ params }: Props) {
  const { slug } = await params;
  const post = await getPost(slug);
  return <Article post={post} />;
}
```

### Script Optimization

```typescript
import Script from 'next/script';

export default function Page() {
  return (
    <>
      {/* Estrategia: beforeInteractive - carga antes de hidratación */}
      <Script
        src="https://example.com/critical-script.js"
        strategy="beforeInteractive"
      />
      
      {/* afterInteractive (default) - carga después de hidratación */}
      <Script
        src="https://www.googletagmanager.com/gtag/js?id=GA_ID"
        strategy="afterInteractive"
      />
      
      {/* lazyOnload - carga durante idle time */}
      <Script
        src="https://example.com/widget.js"
        strategy="lazyOnload"
      />
      
      {/* worker - ejecuta en web worker (experimental) */}
      <Script
        src="https://example.com/heavy-script.js"
        strategy="worker"
      />
      
      {/* Inline script */}
      <Script id="gtag-init" strategy="afterInteractive">
        {`
          window.dataLayer = window.dataLayer || [];
          function gtag(){dataLayer.push(arguments);}
          gtag('js', new Date());
          gtag('config', 'GA_ID');
        `}
      </Script>
      
      {/* Con callbacks */}
      <Script
        src="https://example.com/script.js"
        onLoad={() => console.log('Script loaded')}
        onReady={() => console.log('Script ready')}
        onError={(e) => console.error('Script error', e)}
      />
    </>
  );
}
```

---

## API Routes

```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';

// GET /api/users
export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const page = searchParams.get('page') ?? '1';
  const limit = searchParams.get('limit') ?? '10';
  
  const users = await db.user.findMany({
    skip: (parseInt(page) - 1) * parseInt(limit),
    take: parseInt(limit),
  });
  
  return NextResponse.json(users);
}

// POST /api/users
export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const user = await db.user.create({ data: body });
    
    return NextResponse.json(user, { status: 201 });
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to create user' },
      { status: 500 }
    );
  }
}

// app/api/users/[id]/route.ts
interface RouteParams {
  params: Promise<{ id: string }>;
}

// GET /api/users/:id
export async function GET(request: NextRequest, { params }: RouteParams) {
  const { id } = await params;
  
  const user = await db.user.findUnique({ where: { id } });
  
  if (!user) {
    return NextResponse.json({ error: 'User not found' }, { status: 404 });
  }
  
  return NextResponse.json(user);
}

// PUT /api/users/:id
export async function PUT(request: NextRequest, { params }: RouteParams) {
  const { id } = await params;
  const body = await request.json();
  
  const user = await db.user.update({
    where: { id },
    data: body,
  });
  
  return NextResponse.json(user);
}

// DELETE /api/users/:id
export async function DELETE(request: NextRequest, { params }: RouteParams) {
  const { id } = await params;
  
  await db.user.delete({ where: { id } });
  
  return new NextResponse(null, { status: 204 });
}

// Configuración de ruta
export const runtime = 'nodejs'; // 'edge' para Edge Runtime
export const dynamic = 'force-dynamic'; // Sin cache
export const revalidate = 0;
```

---

## Deployment

### Build y Análisis

```bash
# Build de producción
npm run build

# Analizar bundle
npm install @next/bundle-analyzer

# next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  // config
});

# Ejecutar análisis
ANALYZE=true npm run build
```

### Configuración de Producción

```typescript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Output mode
  output: 'standalone', // Para Docker
  // output: 'export', // Para static export
  
  // Compilador
  compiler: {
    removeConsole: process.env.NODE_ENV === 'production',
  },
  
  // Imágenes
  images: {
    remotePatterns: [
      { hostname: 'cdn.example.com' },
    ],
    formats: ['image/avif', 'image/webp'],
  },
  
  // Headers de seguridad
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          { key: 'X-Frame-Options', value: 'DENY' },
          { key: 'X-Content-Type-Options', value: 'nosniff' },
          { key: 'Referrer-Policy', value: 'origin-when-cross-origin' },
        ],
      },
    ];
  },
  
  // Redirects
  async redirects() {
    return [
      {
        source: '/old-blog/:slug',
        destination: '/blog/:slug',
        permanent: true,
      },
    ];
  },
  
  // Rewrites
  async rewrites() {
    return [
      {
        source: '/api/:path*',
        destination: 'https://api.example.com/:path*',
      },
    ];
  },
};

module.exports = nextConfig;
```

### Docker

```dockerfile
# Dockerfile
FROM node:20-alpine AS base

# Dependencies
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app
COPY package*.json ./
RUN npm ci

# Builder
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
ENV NEXT_TELEMETRY_DISABLED 1
RUN npm run build

# Runner
FROM base AS runner
WORKDIR /app
ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000
ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]
```

---

## 🏷️ Tags

#programming #frontend #nextjs #react #fullstack #ssr #typescript

---

## 📚 Ver También

- [[React Complete Guide|React - Guía Completa]]
- [[React Hooks Guide|React Hooks]]
- [[NestJS Complete Guide|NestJS - Guía Completa]]
