---
tags:
  - software-engineering
  - databases
  - nosql
created: 2026-01-02
status: 🔴
---
# 📦 NoSQL Databases

> *"NoSQL databases are purpose-built for specific data models with flexible schemas."*

## 🎯 What is NoSQL?

NoSQL ("Not Only SQL") es una categoría de bases de datos que no usan el modelo relacional tradicional. Diseñadas para escalabilidad, flexibilidad y tipos de datos específicos.

---

## 📊 Types of NoSQL Databases

```
┌─────────────────────────────────────────────────────────────────┐
│                    NOSQL DATABASE TYPES                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌───────────────────┐    ┌───────────────────┐                │
│  │   KEY-VALUE       │    │    DOCUMENT       │                │
│  │                   │    │                   │                │
│  │  key ──► value    │    │  {                │                │
│  │  key ──► value    │    │    "name": "...", │                │
│  │  key ──► value    │    │    "data": {...}  │                │
│  │                   │    │  }                │                │
│  │  Redis, DynamoDB  │    │  MongoDB, Firestore│               │
│  └───────────────────┘    └───────────────────┘                │
│                                                                 │
│  ┌───────────────────┐    ┌───────────────────┐                │
│  │   WIDE-COLUMN     │    │      GRAPH        │                │
│  │                   │    │                   │                │
│  │  Row │ Col1│Col2│ │    │    (A)───►(B)     │                │
│  │  ────┼─────┼────┼ │    │     │      ▲      │                │
│  │  r1  │ v1  │ v2 │ │    │     ▼      │      │                │
│  │  r2  │ v3  │    │ │    │    (C)────►(D)    │                │
│  │                   │    │                   │                │
│  │  Cassandra,ScyllaDB│   │  Neo4j, Neptune   │                │
│  └───────────────────┘    └───────────────────┘                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔑 Key-Value Stores

### Use Cases
- Session storage
- Caching
- Real-time data
- Leaderboards

### Redis Example
```typescript
import Redis from 'ioredis';

const redis = new Redis();

// Basic operations
await redis.set('user:1:name', 'John Doe');
const name = await redis.get('user:1:name');

// With expiration (TTL)
await redis.setex('session:abc123', 3600, JSON.stringify({ userId: 1 }));

// Hash (object-like)
await redis.hset('user:1', {
  name: 'John Doe',
  email: 'john@example.com',
  age: '30'
});
const user = await redis.hgetall('user:1');

// List (queue)
await redis.rpush('queue:emails', JSON.stringify({ to: 'user@example.com' }));
const email = await redis.lpop('queue:emails');

// Set (unique values)
await redis.sadd('user:1:roles', 'admin', 'editor');
const roles = await redis.smembers('user:1:roles');
const isAdmin = await redis.sismember('user:1:roles', 'admin');

// Sorted Set (leaderboard)
await redis.zadd('leaderboard', { score: 100, member: 'user:1' });
await redis.zadd('leaderboard', { score: 150, member: 'user:2' });
const topPlayers = await redis.zrevrange('leaderboard', 0, 9, 'WITHSCORES');

// Pub/Sub
await redis.publish('notifications', JSON.stringify({ type: 'new_message' }));
```

---

## 📄 Document Databases

### Use Cases
- Content management
- User profiles
- Product catalogs
- Event logging

### MongoDB Example
```typescript
import { MongoClient, ObjectId } from 'mongodb';

const client = new MongoClient('mongodb://localhost:27017');
const db = client.db('myapp');
const users = db.collection('users');

// Insert
const result = await users.insertOne({
  name: 'John Doe',
  email: 'john@example.com',
  address: {
    street: '123 Main St',
    city: 'NYC',
    country: 'USA'
  },
  tags: ['developer', 'nodejs'],
  createdAt: new Date()
});

// Find
const user = await users.findOne({ _id: new ObjectId('...') });

const developers = await users.find({
  tags: 'developer',
  'address.country': 'USA'
}).toArray();

// Complex queries
const activeUsers = await users.find({
  createdAt: { $gte: new Date('2024-01-01') },
  $or: [
    { role: 'admin' },
    { 'stats.loginCount': { $gt: 10 } }
  ]
}).sort({ createdAt: -1 }).limit(10).toArray();

// Update
await users.updateOne(
  { _id: new ObjectId('...') },
  {
    $set: { 'address.city': 'LA' },
    $push: { tags: 'senior' },
    $inc: { 'stats.loginCount': 1 }
  }
);

// Aggregation pipeline
const usersByCountry = await users.aggregate([
  { $match: { createdAt: { $gte: new Date('2024-01-01') } } },
  { $group: {
      _id: '$address.country',
      count: { $sum: 1 },
      avgAge: { $avg: '$age' }
    }
  },
  { $sort: { count: -1 } }
]).toArray();

// Indexes
await users.createIndex({ email: 1 }, { unique: true });
await users.createIndex({ 'address.country': 1, createdAt: -1 });
await users.createIndex({ tags: 1 });  // Multikey index for arrays
```

### Document Schema Design
```typescript
// Embedded (denormalized) - good for 1:few, read together
interface User {
  _id: ObjectId;
  name: string;
  addresses: Address[];  // Embedded array
  profile: {             // Embedded document
    bio: string;
    avatar: string;
  };
}

// Referenced (normalized) - good for 1:many, independent access
interface Order {
  _id: ObjectId;
  userId: ObjectId;      // Reference to User
  items: OrderItem[];    // Embedded (order items always accessed with order)
}

// Hybrid approach
interface BlogPost {
  _id: ObjectId;
  title: string;
  authorId: ObjectId;    // Reference (author data changes)
  authorName: string;    // Denormalized (frequently displayed)
  comments: Comment[];   // Embedded (up to a limit)
}
```

---

## 📊 Wide-Column Stores

### Use Cases
- Time series data
- IoT sensor data
- Analytics
- High write throughput

### Cassandra Data Model
```sql
-- Keyspace (like a database)
CREATE KEYSPACE myapp
WITH replication = {
  'class': 'NetworkTopologyStrategy',
  'datacenter1': 3
};

-- Table with partition key and clustering columns
CREATE TABLE events (
    user_id UUID,
    event_date DATE,
    event_time TIMESTAMP,
    event_type TEXT,
    data TEXT,
    PRIMARY KEY ((user_id, event_date), event_time)
) WITH CLUSTERING ORDER BY (event_time DESC);

-- Partition key: (user_id, event_date) - determines data distribution
-- Clustering key: event_time - determines order within partition

-- Query patterns (must include partition key)
SELECT * FROM events 
WHERE user_id = ? AND event_date = ?
ORDER BY event_time DESC
LIMIT 100;

-- Time series pattern
CREATE TABLE sensor_readings (
    sensor_id UUID,
    bucket_date DATE,
    reading_time TIMESTAMP,
    value DOUBLE,
    PRIMARY KEY ((sensor_id, bucket_date), reading_time)
);
```

---

## 🔗 Graph Databases

### Use Cases
- Social networks
- Recommendation engines
- Fraud detection
- Knowledge graphs

### Neo4j Example (Cypher)
```cypher
// Create nodes
CREATE (john:Person {name: 'John', age: 30})
CREATE (jane:Person {name: 'Jane', age: 28})
CREATE (company:Company {name: 'TechCorp'})

// Create relationships
MATCH (john:Person {name: 'John'}), (jane:Person {name: 'Jane'})
CREATE (john)-[:KNOWS {since: 2020}]->(jane)

MATCH (john:Person {name: 'John'}), (company:Company {name: 'TechCorp'})
CREATE (john)-[:WORKS_AT {role: 'Engineer', since: 2021}]->(company)

// Query: Find friends of friends
MATCH (person:Person {name: 'John'})-[:KNOWS*2]->(fof:Person)
WHERE fof.name <> 'John'
RETURN DISTINCT fof.name

// Query: Shortest path
MATCH path = shortestPath(
  (start:Person {name: 'John'})-[:KNOWS*]-(end:Person {name: 'Alice'})
)
RETURN path

// Query: Recommendations
MATCH (user:Person {name: 'John'})-[:LIKES]->(item:Product)<-[:LIKES]-(other:Person)
MATCH (other)-[:LIKES]->(rec:Product)
WHERE NOT (user)-[:LIKES]->(rec)
RETURN rec.name, COUNT(*) AS score
ORDER BY score DESC
LIMIT 10
```

---

## ⚖️ SQL vs NoSQL Comparison

| Aspect | SQL | NoSQL |
|--------|-----|-------|
| **Schema** | Fixed, predefined | Flexible, dynamic |
| **Scaling** | Vertical (scale up) | Horizontal (scale out) |
| **Transactions** | ACID | Eventually consistent (mostly) |
| **Queries** | Complex joins | Simple lookups, denormalized |
| **Best for** | Complex relationships | High volume, simple access |

---

## 🧠 CAP Theorem

```
┌─────────────────────────────────────────────────────────────────┐
│                      CAP THEOREM                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│         Pick 2 out of 3 (you can't have all three)              │
│                                                                 │
│                   Consistency (C)                               │
│                        /\                                       │
│                       /  \                                      │
│                      /    \                                     │
│                     /      \                                    │
│                    /   CA   \                                   │
│                   /  (RDBMS) \                                  │
│                  /            \                                 │
│                 /──────────────\                                │
│                /                \                               │
│               /   CP      CP     \                              │
│              / (MongoDB) (HBase)  \                             │
│             /                      \                            │
│  Availability (A) ────────────── Partition                      │
│              \     AP            Tolerance (P)                  │
│               \ (Cassandra,                                     │
│                \ DynamoDB)                                      │
│                                                                 │
│  C = Every read receives most recent write                      │
│  A = Every request receives a response                          │
│  P = System continues despite network partitions                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📚 Related

- [[Programming/Software Engineering/Database Design/Relational Databases|Relational Databases]]
- [[Programming/Software Engineering/System Design/Caching|Caching]]

---

← [[Programming/Software Engineering/Database Design/Database Design Complete Guide|Back to Database Design]]
