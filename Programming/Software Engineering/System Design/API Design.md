---
tags:
  - software-engineering
  - system-design
  - api-design
created: 2026-01-02
status: 🔴
---
# 🔌 API Design

> *"A good API is not just easy to use but also hard to misuse."*

## 🎯 API Styles Comparison

| Style | Use Case | Pros | Cons |
|-------|----------|------|------|
| **REST** | Web APIs, CRUD | Simple, cacheable | Over/under fetching |
| **GraphQL** | Complex queries, mobile | Flexible, typed | Complexity, caching |
| **gRPC** | Microservices, internal | Fast, typed, streaming | Browser support |

---

## 📘 REST API Design

### Resource Naming
```
✅ Good - Sustantivos, plural
GET    /users
GET    /users/123
POST   /users
PUT    /users/123
DELETE /users/123

GET    /users/123/orders
GET    /users/123/orders/456

❌ Bad - Verbos o acciones
GET    /getUser/123
POST   /createUser
GET    /user/123/getOrders
```

### HTTP Methods
```typescript
// GET - Leer recurso(s)
GET /users         // Lista usuarios
GET /users/123     // Un usuario

// POST - Crear recurso
POST /users
Body: { "name": "John", "email": "john@email.com" }

// PUT - Reemplazar recurso completo
PUT /users/123
Body: { "name": "John", "email": "new@email.com", "role": "admin" }

// PATCH - Actualizar parcialmente
PATCH /users/123
Body: { "email": "new@email.com" }

// DELETE - Eliminar
DELETE /users/123
```

### Status Codes
```typescript
// 2xx Success
200 OK              // GET, PUT, PATCH exitoso
201 Created         // POST exitoso, nuevo recurso
204 No Content      // DELETE exitoso

// 4xx Client Error
400 Bad Request     // Validación falló
401 Unauthorized    // No autenticado
403 Forbidden       // Autenticado pero sin permiso
404 Not Found       // Recurso no existe
409 Conflict        // Conflicto (ej: email duplicado)
422 Unprocessable   // Entidad inválida
429 Too Many Req    // Rate limit

// 5xx Server Error
500 Internal Error  // Error del servidor
503 Service Unavail // Servidor no disponible
```

### Query Parameters
```typescript
// Pagination
GET /users?page=2&limit=20
GET /users?cursor=abc123&limit=20  // Cursor-based (mejor para grandes datasets)

// Filtering
GET /users?status=active&role=admin

// Sorting
GET /users?sort=created_at&order=desc
GET /users?sort=-created_at  // "-" para descending

// Field selection
GET /users?fields=id,name,email

// Search
GET /users?q=john
```

### Response Format
```typescript
// Success Response
{
  "data": {
    "id": "123",
    "name": "John Doe",
    "email": "john@email.com"
  }
}

// Collection Response with Pagination
{
  "data": [
    { "id": "1", "name": "User 1" },
    { "id": "2", "name": "User 2" }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "pages": 5
  },
  "links": {
    "self": "/users?page=1",
    "next": "/users?page=2",
    "last": "/users?page=5"
  }
}

// Error Response
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": [
      { "field": "email", "message": "Invalid email format" },
      { "field": "name", "message": "Name is required" }
    ]
  }
}
```

---

## 📗 GraphQL Basics

### Schema Definition
```graphql
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
  profile: Profile
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  createdAt: DateTime!
}

type Query {
  user(id: ID!): User
  users(limit: Int, offset: Int): [User!]!
  posts(authorId: ID): [Post!]!
}

type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
  deleteUser(id: ID!): Boolean!
}

input CreateUserInput {
  name: String!
  email: String!
}
```

### Queries
```graphql
# Fetch only what you need
query GetUser {
  user(id: "123") {
    name
    email
    posts {
      title
    }
  }
}

# Variables
query GetUsers($limit: Int!) {
  users(limit: $limit) {
    id
    name
  }
}
```

### Mutations
```graphql
mutation CreateUser {
  createUser(input: {
    name: "John"
    email: "john@email.com"
  }) {
    id
    name
  }
}
```

---

## 📕 gRPC Basics

### Protocol Buffers (.proto)
```protobuf
syntax = "proto3";

package users;

service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc ListUsers(ListUsersRequest) returns (ListUsersResponse);
  rpc CreateUser(CreateUserRequest) returns (User);
  rpc StreamUsers(StreamRequest) returns (stream User);  // Streaming
}

message User {
  string id = 1;
  string name = 2;
  string email = 3;
}

message GetUserRequest {
  string id = 1;
}

message ListUsersRequest {
  int32 page = 1;
  int32 limit = 2;
}

message ListUsersResponse {
  repeated User users = 1;
  int32 total = 2;
}
```

### Client Usage
```typescript
import { UserServiceClient } from './generated/users_grpc_pb';

const client = new UserServiceClient('localhost:50051', credentials);

// Unary call
const request = new GetUserRequest();
request.setId('123');

client.getUser(request, (error, response) => {
  console.log(response.getName());
});

// Streaming
const stream = client.streamUsers(new StreamRequest());
stream.on('data', (user) => console.log(user.getName()));
stream.on('end', () => console.log('Stream ended'));
```

---

## 🔐 API Security

### Authentication
```typescript
// JWT in Authorization header
headers: {
  'Authorization': 'Bearer eyJhbGciOiJIUzI1NiIs...'
}

// API Key
headers: {
  'X-API-Key': 'your-api-key'
}
```

### Rate Limiting
```typescript
// Response headers
{
  'X-RateLimit-Limit': '100',
  'X-RateLimit-Remaining': '95',
  'X-RateLimit-Reset': '1640000000'
}

// 429 response when exceeded
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests",
    "retryAfter": 60
  }
}
```

---

## 📊 API Versioning

### URL Path Versioning
```
GET /v1/users
GET /v2/users
```

### Header Versioning
```
GET /users
Accept: application/vnd.myapi.v2+json
```

### Query Parameter
```
GET /users?version=2
```

---

## ✅ Best Practices

| Practice | Description |
|----------|-------------|
| **Consistency** | Mismas convenciones en toda la API |
| **Documentation** | OpenAPI/Swagger para REST |
| **Versioning** | Planear desde el inicio |
| **Error Messages** | Claros y accionables |
| **Idempotency** | PUT/DELETE deben ser idempotentes |
| **HATEOAS** | Links para navegación (REST) |

---

← [[Programming/Software Engineering/System Design/_Index|Back to System Design]]
