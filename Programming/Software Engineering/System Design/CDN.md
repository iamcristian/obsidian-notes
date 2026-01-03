---
tags:
  - software-engineering
  - system-design
  - cdn
created: 2026-01-02
status: 🔴
---
# 🌐 CDN (Content Delivery Network)

> *"Bring content closer to users for faster delivery."*

## 🎯 What is a CDN?

Un CDN es una red de servidores distribuidos geográficamente que almacenan copias de contenido estático y lo entregan desde la ubicación más cercana al usuario.

---

## 🏗️ How CDN Works

```
┌─────────────────────────────────────────────────────────────────┐
│                       CDN ARCHITECTURE                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│      User A (NYC)              User B (Tokyo)                   │
│          │                         │                            │
│          ▼                         ▼                            │
│    ┌───────────┐            ┌───────────┐                      │
│    │ Edge NYC  │            │Edge Tokyo │                      │
│    │ (PoP)     │            │ (PoP)     │                      │
│    └─────┬─────┘            └─────┬─────┘                      │
│          │                        │                             │
│          │    Cache HIT?          │    Cache HIT?               │
│          │                        │                             │
│          │     ┌─────────────────────────────┐                  │
│          └────►│                             │◄────┘            │
│                │      ORIGIN SERVER          │                  │
│                │   (Your actual server)      │                  │
│                │                             │                  │
│                └─────────────────────────────┘                  │
│                                                                 │
│  Cache HIT: Served from edge (fast, ~20ms)                      │
│  Cache MISS: Fetched from origin (slow, ~200ms+)                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 CDN Request Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                     CDN REQUEST FLOW                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. DNS Resolution                                              │
│     example.com → CDN DNS → Nearest edge IP                     │
│                                                                 │
│  2. Request to Edge                                             │
│     ┌────────┐                ┌──────────┐                     │
│     │  User  │ ──Request───► │   Edge   │                     │
│     └────────┘                │  Server  │                     │
│                               └────┬─────┘                     │
│                                    │                            │
│  3. Cache Check                    ▼                            │
│                            ┌──────────────┐                    │
│                            │ Cache Lookup │                    │
│                            └───────┬──────┘                    │
│                                    │                            │
│                    ┌───────────────┴───────────────┐           │
│                    │                               │           │
│                    ▼                               ▼           │
│              ┌─────────┐                     ┌──────────┐      │
│              │Cache HIT│                     │Cache MISS│      │
│              └────┬────┘                     └────┬─────┘      │
│                   │                               │             │
│                   ▼                               ▼             │
│              Return cached                   Fetch from        │
│              content                         origin server     │
│              (fast!)                         (cache it)        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔧 CDN Configuration

### Cache-Control Headers
```
# Cache for 1 year (immutable assets)
Cache-Control: public, max-age=31536000, immutable

# Cache for 1 hour, revalidate after
Cache-Control: public, max-age=3600, must-revalidate

# No caching (dynamic content)
Cache-Control: no-store, no-cache

# Private (user-specific)
Cache-Control: private, max-age=3600
```

### Typical Cache Policies

| Content Type | Cache Duration | Example |
|-------------|----------------|---------|
| Static assets (JS/CSS with hash) | 1 year | `app.abc123.js` |
| Images | 1 week - 1 year | `/images/logo.png` |
| HTML | No cache or short | `index.html` |
| API responses | Short or no cache | `/api/user` |
| Fonts | 1 year | `/fonts/roboto.woff2` |

### CDN Configuration (CloudFront Example)
```yaml
# AWS CloudFront Distribution
Distribution:
  Origins:
    - DomainName: my-bucket.s3.amazonaws.com
      Id: S3Origin
      S3OriginConfig:
        OriginAccessIdentity: origin-access-identity/cloudfront/XXX

  DefaultCacheBehavior:
    TargetOriginId: S3Origin
    ViewerProtocolPolicy: redirect-to-https
    CachePolicyId: 658327ea-f89d-4fab-a63d-7e88639e58f6  # CachingOptimized
    
  CacheBehaviors:
    - PathPattern: "/api/*"
      TargetOriginId: APIOrigin
      CachePolicyId: 4135ea2d-6df8-44a3-9df3-4b5a84be39ad  # CachingDisabled
    
    - PathPattern: "/static/*"
      TargetOriginId: S3Origin
      CachePolicyId: 658327ea-f89d-4fab-a63d-7e88639e58f6
      Compress: true
```

---

## 🎯 CDN Features

### 1. Edge Caching
```
┌─────────────────────────────────────────┐
│ Static files cached at edge locations   │
│                                         │
│ • HTML, CSS, JS                         │
│ • Images, videos                        │
│ • Fonts, documents                      │
└─────────────────────────────────────────┘
```

### 2. Compression
```nginx
# Automatic gzip/brotli compression
# Original: 500KB → Compressed: 100KB

# Nginx configuration
gzip on;
gzip_types text/plain text/css application/json application/javascript;
brotli on;
brotli_types text/plain text/css application/json application/javascript;
```

### 3. SSL/TLS Termination
```
┌────────┐  HTTPS   ┌────────┐   HTTP    ┌──────────┐
│ Client │ ───────► │  CDN   │ ────────► │  Origin  │
└────────┘          └────────┘           └──────────┘
                    
SSL handled at edge, reducing origin load
```

### 4. DDoS Protection
```
┌─────────────────────────────────────────────────────┐
│                                                     │
│   Malicious traffic absorbed by distributed        │
│   edge network before reaching origin              │
│                                                     │
│   ████████████████  ──► Edge ──► ●  Origin         │
│   (DDoS attack)       filters    (protected)       │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### 5. Edge Computing
```javascript
// Cloudflare Worker (edge function)
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request));
});

async function handleRequest(request) {
  // Run code at the edge
  const country = request.cf.country;
  
  // Route to regional origin
  if (country === 'JP') {
    return fetch('https://japan.origin.com' + request.url);
  }
  
  return fetch('https://us.origin.com' + request.url);
}
```

---

## 🔄 Cache Invalidation

### 1. Path Invalidation
```bash
# AWS CloudFront
aws cloudfront create-invalidation \
  --distribution-id XXXXX \
  --paths "/index.html" "/api/*"

# Cloudflare
curl -X POST "https://api.cloudflare.com/client/v4/zones/{zone_id}/purge_cache" \
  -H "Authorization: Bearer TOKEN" \
  -d '{"files":["https://example.com/css/styles.css"]}'
```

### 2. Cache Busting (Better Approach)
```html
<!-- Version in filename - never needs invalidation -->
<link href="/styles.abc123.css" rel="stylesheet">
<script src="/app.def456.js"></script>

<!-- Query string (less reliable) -->
<link href="/styles.css?v=123" rel="stylesheet">
```

### 3. Surrogate Keys / Tags
```
# Tag content for targeted invalidation
Cache-Tag: product-123, category-shoes

# Invalidate all products in shoes category
Purge: Cache-Tag: category-shoes
```

---

## 📊 CDN Providers Comparison

| Provider | Best For | Notable Features |
|----------|----------|------------------|
| **CloudFront** | AWS users | Lambda@Edge, integrated |
| **Cloudflare** | Security, workers | Free tier, edge compute |
| **Fastly** | Real-time purge | Instant invalidation |
| **Akamai** | Enterprise | Largest network |
| **Vercel/Netlify** | JAMstack | Git-integrated deploy |

---

## 📈 Performance Impact

```
┌─────────────────────────────────────────────────────────────────┐
│               CDN PERFORMANCE IMPROVEMENT                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   WITHOUT CDN (Origin in US-East)                               │
│   ┌────────────────────────────────────────────────────────┐   │
│   │ User Location │ Latency                                 │   │
│   ├───────────────┼────────────────────────────────────────┤   │
│   │ New York      │ 20ms   ████                            │   │
│   │ Los Angeles   │ 70ms   ██████████████                  │   │
│   │ London        │ 100ms  ████████████████████            │   │
│   │ Tokyo         │ 180ms  ████████████████████████████████│   │
│   └───────────────┴────────────────────────────────────────┘   │
│                                                                 │
│   WITH CDN (Edge locations worldwide)                           │
│   ┌────────────────────────────────────────────────────────┐   │
│   │ User Location │ Latency                                 │   │
│   ├───────────────┼────────────────────────────────────────┤   │
│   │ New York      │ 10ms   ██                              │   │
│   │ Los Angeles   │ 15ms   ███                             │   │
│   │ London        │ 12ms   ██                              │   │
│   │ Tokyo         │ 18ms   ████                            │   │
│   └───────────────┴────────────────────────────────────────┘   │
│                                                                 │
│   + Reduced origin load                                         │
│   + Better availability                                         │
│   + DDoS protection                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📚 Related

- [[Programming/Software Engineering/System Design/Caching|Caching]]
- [[Programming/Software Engineering/System Design/Scalability|Scalability]]
- [[Programming/Software Engineering/DevOps/Cloud Fundamentals|Cloud Fundamentals]]

---

← [[Programming/Software Engineering/System Design/_Index|Back to System Design]]
