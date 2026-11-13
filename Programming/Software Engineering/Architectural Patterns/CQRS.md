---
tags:
  - software-engineering
  - architectural-patterns
  - cqrs
created: 2026-01-02
updated: 2026-11-13
status: 🟢
level: mid-to-senior
---
# Command Query Responsibility Segregation (CQRS)

> [!abstract] Hub Context
> Part of [[Programming/Software Engineering/Architectural Patterns/Architectural Patterns Complete Guide|Architectural Patterns Complete Guide]]. Originating from Bertrand Meyer's CQS method invariant and generalized to architectural boundaries by Greg Young, CQRS pairs naturally with [[Programming/Software Engineering/Architectural Patterns/Hexagonal Architecture|Hexagonal Architecture]], [[Programming/Software Engineering/Architectural Patterns/Vertical Slice Architecture|Vertical Slice Architecture]], [[Programming/Software Engineering/Architectural Patterns/Event Sourcing|Event Sourcing]], and [[Programming/Software Engineering/Architectural Patterns/Event-Driven Architecture|Event-Driven Architecture]].

**Command Query Responsibility Segregation (CQRS)** partitions a system's runtime model into two independent paths: a **Write Model (Commands)** optimized for transactional invariant enforcement, concurrency safety, and state mutation, and one or more **Read Models (Queries)** optimized for low-latency retrieval, denormalized UI composition, full-text filtering, and analytical aggregation.

---

## 1. First-Principles Mechanics & Complete Architecture

### From Meyer's CQS to Greg Young's CQRS
Bertrand Meyer formulated **Command-Query Separation (CQS)** for object-oriented methods: every method must either perform an action that mutates state and return `void` (a **Command**), or return data without producing observable side effects (a **Query**), never both.

When enterprise systems force a single unified domain model (such as a JPA/TypeORM/Prisma entity graph) to serve both write invariants and complex UI reporting screens, structural conflict emerges:

1. **Write-Side Hydration Bloat**: Enforcing an order cancellation rule requires loading only `Order.status`, `Order.version`, and payment settlement flags. Rendering an order dashboard requires customer email, shipping carrier name, warehouse city, line-item SKU titles, and computed tax totals across six tables.
2. **Read-Side ORM Tax**: Reusing write-side domain repositories for multi-row read endpoints forces N+1 queries or massive Cartesian joins, constructs rich Domain Aggregates with invariant validation logic, and immediately discards those allocations during JSON DTO serialization.
3. **Indexing Contention**: Adding seven composite and GIN full-text indexes to write-heavy normalized tables degrades `INSERT` and `UPDATE` B-tree maintenance and bloats PostgreSQL Write-Ahead Log (WAL) amplification, whereas read queries still require expensive runtime joins.

Greg Young resolved this by elevating CQS from individual methods to **architectural model segregation**: the write path and the read path do not need to share the same objects, the same data-access framework, the same schema, or even the same storage engine.

### Separating CQRS from Event Sourcing
A widespread architectural misconception treats CQRS and [[Programming/Software Engineering/Architectural Patterns/Event Sourcing|Event Sourcing]] as synonymous. They solve orthogonal problems:

- **CQRS** splits **how state is mutated** from **how state is queried**. You can implement full Level 1 and Level 2 CQRS over standard mutable relational rows (`UPDATE orders SET status = 'CONFIRMED'`) without ever storing an event stream.
- **Event Sourcing** specifies **how write-side state is persisted** (an append-only sequence of domain events rather than mutable row columns). Because querying across raw serialized event streams cannot satisfy arbitrary search predicates (`WHERE customer_tier = 'VIP' AND total_cents > 50000`), Event Sourcing practically requires CQRS projections for reads—whereas CQRS does **not** require Event Sourcing.

### Greg Young's CQRS Progression: Level 0, Level 1, and Level 2

CQRS is an incremental architectural spectrum. Jumping directly to physically segregated distributed read stores introduces eventual consistency, projector operational failure modes, and schema-rebuild pipelines that trivial CRUD workloads do not justify.

| Maturity Level | Physical Storage Topology | Write Path Implementation | Read Path Implementation | Synchronization & Lag |
|---|---|---|---|---|
| **Level 0: Unified CRUD** | Single primary RDBMS | Shared ORM entity + repository | Same ORM entity + repository | `0 ms` (single ACID commit) |
| **Level 1: Logical CQRS** | Single primary RDBMS (optional read replica) | Rich Domain Aggregate or Transaction Script enforcing invariants + OCC | Direct thin parameterized SQL (`SELECT ... JOIN ...`) projecting straight to flat `ReadDTO` | `0 ms` on primary ACID commit (`5–50 ms` if read from streaming physical replica) |
| **Level 2: Physical Segregation** | Write DB (PostgreSQL) + Dedicated Read Stores (Denormalized PG, OpenSearch, Redis, ClickHouse) | Domain Aggregate + ACID commit to state tables + `outbox_events` / WAL | Query handler targeting Read Store DSL/SQL with zero domain hydration | Eventual consistency (`20 ms – 1,500 ms` async projector / CDC lag) |

#### Level 1 Architecture: Logical Split on a Single ACID Database

```mermaid
flowchart LR
    API["API Router"] -->|"ConfirmOrder"| Cmd["Command Handler"]
    API -->|"ListOrders"| Qry["Read Query Handler"]
    Cmd -->|"Loads & Validates"| Agg["Order Aggregate"]
    Agg -->|"ACID TX + OCC"| PG[("Postgres Primary")]
    Qry -->|"Direct Flat SQL"| PG
```

In **Level 1 CQRS**:
- Commands route through Application Command Handlers that enforce authentication, idempotency, domain invariants inside Aggregates, and Optimistic Concurrency Control (`WHERE version = $expected`), committing to normalized write tables (`orders`, `order_lines`).
- Queries completely bypass Domain Aggregates and Write Repositories. A `QueryHandler` issues a single parameterized SQL query directly against the PostgreSQL connection pool, leveraging covering indexes, `JOIN` expressions, and SQL projections (`SELECT o.id, o.total_cents, c.email ...`) to hydrate plain read structs.
- Because both paths target the exact same ACID primary database (or an optional synchronous Postgres view), read-your-own-writes consistency is immediate (`0 ms` lag) while eliminating ORM hydration overhead.

#### Level 2 Architecture: Physically Segregated Read Store

```mermaid
flowchart LR
    Cmd["ConfirmOrder Handler"] -->|"1. ACID TX + Outbox"| WriteDB[("Postgres Write Primary")]
    WriteDB -->|"2. WAL CDC / SKIP LOCKED"| Relay["Outbox Relay / Kafka"]
    Relay -->|"3. Partitioned by orderId"| Proj["Async Projector Worker"]
    Proj -->|"4. Monotonic Upsert"| ReadPG[("Read PG / OpenSearch")]
    Qry["SearchOrders Query"] -->|"5. Zero-Join Read"| ReadPG
```

In **Level 2 CQRS**:
- **Write Store** remains strictly normalized for ACID integrity, minimum lock duration, and minimal index write amplification (`orders`, `order_lines`, `outbox_events`).
- **Change Propagation** streams committed write mutations asynchronously via either **Transactional Outbox polling** (`FOR UPDATE SKIP LOCKED`) or **Write-Ahead Log (WAL) Change Data Capture (CDC)** such as Debezium into Kafka/Redpanda partitioned by `aggregate_id`. See [[Programming/Software Engineering/Architectural Patterns/Event-Driven Architecture|Event-Driven Architecture]] for dual-write elimination mechanics.
- **Async Projector Workers** consume domain events, merge cross-entity data into pre-joined denormalized records, compute full-text tokens or aggregations, and upsert into specialized Read Stores:
  - **Denormalized PostgreSQL read tables** (`order_search_read_model`) for sub-10ms keyset-paginated tabular dashboards.
  - **OpenSearch / Elasticsearch** indexes for fuzzy matching, multi-facet counts, and relevance scoring.
  - **Redis hashes/sorted sets** for ultra-low-latency hot lookups (`< 2 ms` p99).
  - **ClickHouse / DuckDB** columnar projections for multi-year OLAP revenue aggregations.

### Consistency & Projection Lag Engineering

Whenever read models live behind an asynchronous propagation pipeline, the system operates under **Eventual Consistency** (see [[Programming/Software Engineering/System Design/CAP Theorem|CAP Theorem]]). Every millisecond between Write DB commit and Read Store visibility creates a window where clients can observe stale state.

#### 1. Measuring End-to-End Projection Lag
Senior platforms decompose lag across three observable checkpoints rather than guessing "replication feels slow":
1. **Commit-to-Relay Lag ($L_{\text{relay}}$)**: `relay_dispatched_at - outbox.created_at` (or Debezium `MilliSecondsBehindSource` metric). Measures WAL/outbox extraction delay.
2. **Broker-to-Consumer Lag ($L_{\text{queue}}$)**: Kafka consumer group `CURRENT-OFFSET` vs `LOG-END-OFFSET` and timestamp delta between message timestamp and consumer poll time.
3. **Consumer-to-Visible Lag ($L_{\text{visible}}$)**: Time from consumer upsert start until the read store makes the document queryable (`COMMIT` in Postgres read DB, or `refresh=true` / `index.refresh_interval` elapsed in OpenSearch—which defaults to `1,000 ms`!).

Total user-observed staleness is:

$$L_{\text{total}} = L_{\text{relay}} + L_{\text{queue}} + L_{\text{visible}}$$

#### 2. Solving Read-Your-Own-Writes (RYOW) Violations
When a browser submits `POST /orders/123/confirm` (`200 OK`) and immediately redirects to `GET /orders/123` hitting a lagging Read Store, returning `DRAFT` or `404` shatters user trust and triggers duplicate submissions. Four engineering mitigations eliminate RYOW flashes without sacrificing read offloading:

1. **Synchronous In-Transaction Projection (Critical Local Views)**: For Level 1 or hybrid deployments where a read table resides inside the same PostgreSQL cluster as the write tables, update the `order_summary_read` row inside the exact same ACID transaction as the aggregate write, reserving async workers solely for OpenSearch and ClickHouse.
2. **PostgreSQL WAL LSN Sticky Tokens (Read Replicas)**: On write commit, query `SELECT pg_current_wal_lsn()::text AS lsn` inside the primary connection and return `X-Write-Lsn` (or store it in a signed cookie/Redis key with a 5-second TTL). Subsequent reads targeting a Postgres physical streaming replica check `pg_last_wal_replay_lsn() >= $token::pg_lsn`; if the replica lags behind the token, route that single caller back to the Primary Write DB (or wait briefly on PostgreSQL 17+ `WAIT FOR LSN`). See [[Programming/Software Engineering/Database Design/Replication|Replication]].
3. **Monotonic Version Barrier (`minVersion` Fallback)**: The command returns `{ orderId, version: 4 }`. The client attaches `?minVersion=4` (or `X-Min-Read-Version: 4`) on its immediate follow-up `GET`. The query handler reads the Read Store first; if `readModel.aggregateVersion < minVersion`, it executes a bounded poll (`25 ms` $\times$ 3 attempts) or performs a single fallback read directly against the Write Primary. Roughly 99.9% of general reads never supply `minVersion` (or already pass the check) and hit the fast Read Store.
4. **Command-Response Payload + Optimistic UI Patching**: Return the authoritative post-command summary DTO directly from `POST`/`PATCH` (`{ orderId, status: "CONFIRMED", version: 4, confirmedAt }`) and write it immediately into the client's query cache (e.g., TanStack Query `setQueryData`), bypassing the redundant `GET` completely.

#### 3. Monotonic Convergence Under Out-of-Order & Duplicate Delivery
Brokers provide **at-least-once** delivery; partition rebalances, retry topics, and DLQ replays frequently deliver older event sequence `seq = 2` *after* `seq = 3` already updated the read store. Blind `UPSERT` overwrites newer state with older snapshots (**stale regression corruption**).

Every read projection row **must** store a monotonic sequence fence (`last_seq` or `aggregate_version`) and enforce conditional updates at the storage engine boundary:
- **PostgreSQL**:
  ```sql
  INSERT INTO order_search_read_model (order_id, status, total_cents, aggregate_version, last_seq)
  VALUES ($1, $2, $3, $4, $5)
  ON CONFLICT (order_id) DO UPDATE
  SET status = EXCLUDED.status,
      total_cents = EXCLUDED.total_cents,
      aggregate_version = EXCLUDED.aggregate_version,
      last_seq = EXCLUDED.last_seq,
      updated_at = NOW()
  WHERE order_search_read_model.aggregate_version < EXCLUDED.aggregate_version;
  ```
- **OpenSearch / Elasticsearch**: Pass `version: event.aggregateVersion` with `version_type: "external"`. The search engine rejects any index request where `incomingVersion <= storedVersion` with HTTP `409 Conflict`, which the projector catches cleanly as a no-op ACK.

#### 4. Schema Drift & Zero-Downtime Blue/Green Read-Projection Rebuilds
Read models are **disposable derived artifacts**. Product requirements inevitably demand new columns (`warehouse_region`, weighted search boosts, or reshaped facet arrays) that older projections omitted. Never run blocking `ALTER TABLE ...` backfills or `TRUNCATE` live production read tables during business hours.

Execute **Blue/Green Projection Rebuilds**:

```mermaid
stateDiagram-v2
    [*] --> LiveV1: Read view points to v1
    LiveV1 --> ShadowBackfill: Create v2 table/index and backfill
    ShadowBackfill --> DualCatchUp: Tail live events to v2 until lag under 50ms
    DualCatchUp --> AtomicSwap: Swap DB VIEW or ES Alias to v2
    AtomicSwap --> RetireV1: Monitor v2 SLO then drop v1
```

1. **Provision Shadow Schema (`_v2`)**: Create `order_search_read_model_v2` (or OpenSearch index `orders_search_v2`) completely isolated from live reads.
2. **Backfill Historical Data**: Run a parallelized background worker scanning write-side snapshots or outbox/event streams in chunked keyset batches (`5,000` rows per batch) writing into `_v2` while `_v1` continues serving production uninterrupted.
3. **Catch Up to Live Tail**: Attach a named shadow consumer group (`orders-projector-v2`) to the live event stream from the snapshot watermark until `v2` lag matches `v1` (`< 50 ms`). Because writes enforce monotonic `aggregate_version` fences, overlap between the backfill snapshot and live tail converges safely.
4. **Atomic Pointer Swap**:
   - In **PostgreSQL**: Applications always query a stable view `order_search_active_v` (`SELECT * FROM order_search_read_model_v1`). Swap inside a metadata-lock transaction (`CREATE OR REPLACE VIEW order_search_active_v AS SELECT * FROM order_search_read_model_v2;`). Note: if columns changed compatibly (additive), the view swap takes `< 5 ms`.
   - In **OpenSearch**: Applications query the read alias `orders_search_read`. Execute an atomic cluster-state `_aliases` action removing `v1` and adding `v2` in a single request.
5. **Drain & Retire**: Keep `v1` consumer paused for 24 hours as an instant rollback target, then drop `v1`.

---

## 2. Real-World Use Cases & Decision Matrix

### Architectural Selection Matrix

| Dimension | Level 0: Unified CRUD | Level 1: Logical CQRS | Level 2: Physical CQRS | Level 2 CQRS + Event Sourcing |
|---|---|---|---|---|
| **Read / Write Traffic Skew** | $1:1$ to $5:1$ | $5:1$ to $25:1$ | $25:1$ to $500:1$ | Any skew requiring full audit replay |
| **Query Complexity** | Simple PK / indexed lookups | Multi-table joins, aggregate counts, keyset dashboards | Fuzzy full-text, faceted search, geo/vector filtering, OLAP | Same as Level 2 + temporal point-in-time rebuilds |
| **Consistency Guarantee** | Strict ACID (`0 ms` lag) | Strict ACID (`0 ms` lag on primary) | Eventual (`20–500 ms`) + RYOW barriers | Eventual projections + OCC write stream |
| **Operational Footprint** | 1 Database | 1 Database | Write DB + CDC/Outbox + Broker + Projector + Read Store | Event Store + Snapshots + Projectors + Read Stores |
| **Best Subdomain Fit** | Generic CRUD, admin config | Core transactional SaaS modules (default sweet spot) | High-scale E-commerce search, multi-tenant reporting, feeds | Financial ledgers, compliance auditing, insurance workflows |

### Five Concrete Engineering Scenarios

1. **B2B Order Fulfillment & Operations Portal (Adopt Level 1 -> Graduate to Level 2 Search)**:
   - *Context*: Fulfillment agents filter millions of orders by customer email, SKU, carrier tracking status, and date ranges while checkout workers concurrently reserve stock and confirm orders.
   - *Decision*: Start with **Level 1 CQRS** inside PostgreSQL (Write Aggregate updates `orders` + `order_lines`; Read Query executes direct flat SQL with keyset pagination). Once `ILIKE '%term%'` full-text filters saturate DB CPU, promote only the search capability to **Level 2 CQRS** projecting via Transactional Outbox into OpenSearch while keeping single-order detail reads on Level 1.
2. **E-Commerce Product Catalog & Faceted Storefront (Adopt Level 2 CQRS)**:
   - *Context*: Read/write ratio exceeds `200:1`. Backoffice merchandising edits SKU attributes and dynamic pricing in normalized Postgres; shoppers require sub-25ms faceted brand/price/availability counts across 4M SKUs.
   - *Decision*: **Level 2 CQRS** (Postgres Write Primary -> Debezium/Outbox -> OpenSearch + Redis price cache). Read models tolerate 100–300ms propagation lag; merchandising UI uses `minVersion` fallback immediately after saving a product.
3. **SaaS Multi-Tenant Executive Revenue Analytics (Adopt Level 2 CQRS)**:
   - *Context*: Running `SUM(total_cents) GROUP BY date_trunc('day', created_at), region` over a normalized 200M-row OLTP `orders` table causes buffer-pool eviction and stalls live checkouts.
   - *Decision*: Project finalized `OrderConfirmed` and `OrderRefunded` domain events asynchronously into a columnar analytical store (ClickHouse or dedicated Postgres analytical read replica with pre-aggregated rollup tables).
4. **Internal HR Directory & Feature Flag Admin (`< 50 RPS` — Reject Level 2 CQRS)**:
   - *Context*: Small internal CRUD dashboard with balanced reads and writes and trivial queries.
   - *Decision*: Stay at **Level 0** or clean **Level 1** thin queries. Introducing Kafka, Debezium, projectors, and eventual consistency adds weeks of maintenance overhead for zero measurable performance gain.
5. **High-Frequency Inventory Seat Lock / Real-Time Auction Bidding (Reject Async Read Path for Decisions)**:
   - *Context*: Showing a cached read-store seat availability status that lags 300ms causes users to click "Buy" on already-sold concert seats repeatedly.
   - *Decision*: Keep seat availability checks on the **Level 1 Write Primary** (or synchronous atomic Redis Lua reservation) rather than an eventual-consistency Level 2 projection.

---

## 3. Step-by-Step Practical Application: Production Order Search & Fulfillment Engine

The implementation below provides a complete, runnable **CQRS Level 1 & Level 2 Order Search & Fulfillment Engine** in TypeScript, PostgreSQL, OpenSearch HTTP Bulk API, and Vitest. It intentionally avoids Event Sourcing on the write side—storing normalized relational state alongside an atomic outbox—to demonstrate pure CQRS mechanics cleanly.

### Project Directory Layout

```text
order-cqrs-engine/
├── package.json
├── sql/
│   └── 001_orders_cqrs.sql
├── src/
│   ├── db/
│   │   └── postgres.ts
│   ├── orders/
│   │   ├── write/
│   │   │   ├── OrderAggregate.ts
│   │   │   └── ConfirmOrderHandler.ts
│   │   ├── read/
│   │   │   ├── OrderReadTypes.ts
│   │   │   └── ListOrderDashboardQuery.ts
│   │   └── projector/
│   │       ├── OpenSearchClient.ts
│   │       ├── OrderReadProjector.ts
│   │       └── ProjectionBlueGreenRebuilder.ts
└── tests/
    └── orders.cqrs.integration.test.ts
```

### Step 1: PostgreSQL Write Schema, Outbox, and Denormalized Read Schema (`sql/001_orders_cqrs.sql`)

```sql
-- sql/001_orders_cqrs.sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- 1. WRITE MODEL: Normalized transactional tables
CREATE TABLE IF NOT EXISTS customers (
    id          UUID PRIMARY KEY,
    tenant_id   UUID NOT NULL,
    email       TEXT NOT NULL,
    full_name   TEXT NOT NULL,
    tier        TEXT NOT NULL DEFAULT 'STANDARD',
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS orders (
    id              UUID PRIMARY KEY,
    tenant_id       UUID NOT NULL,
    customer_id     UUID NOT NULL REFERENCES customers(id),
    status          TEXT NOT NULL CHECK (status IN ('DRAFT', 'CONFIRMED', 'SHIPPED', 'CANCELLED')),
    currency        CHAR(3) NOT NULL DEFAULT 'USD',
    total_cents     BIGINT NOT NULL CHECK (total_cents >= 0),
    version         INTEGER NOT NULL DEFAULT 1,
    confirmed_at    TIMESTAMPTZ NULL,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS order_lines (
    id              UUID PRIMARY KEY,
    order_id        UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    sku             TEXT NOT NULL,
    title           TEXT NOT NULL,
    quantity        INTEGER NOT NULL CHECK (quantity > 0),
    unit_cents      BIGINT NOT NULL CHECK (unit_cents >= 0)
);

-- Command idempotency store preventing duplicate submissions on client retry
CREATE TABLE IF NOT EXISTS command_idempotency (
    tenant_id       UUID NOT NULL,
    idempotency_key TEXT NOT NULL,
    command_name    TEXT NOT NULL,
    response_json   JSONB NOT NULL,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    PRIMARY KEY (tenant_id, idempotency_key)
);

-- Transactional Outbox bridging Write Model -> Level 2 Projectors without Dual-Write bugs
CREATE TABLE IF NOT EXISTS outbox_events (
    seq_id          BIGSERIAL PRIMARY KEY,
    event_id        UUID NOT NULL UNIQUE DEFAULT gen_random_uuid(),
    aggregate_type  TEXT NOT NULL,
    aggregate_id    UUID NOT NULL,
    aggregate_ver   INTEGER NOT NULL,
    event_type      TEXT NOT NULL,
    payload         JSONB NOT NULL,
    metadata        JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    published_at    TIMESTAMPTZ NULL
);

CREATE INDEX IF NOT EXISTS idx_outbox_unpublished
    ON outbox_events (seq_id ASC)
    WHERE published_at IS NULL;

-- 2. LEVEL 2 READ MODEL: Pre-joined, search-indexed, zero-ORM table
CREATE TABLE IF NOT EXISTS order_search_read_model_v1 (
    order_id          UUID PRIMARY KEY,
    tenant_id         UUID NOT NULL,
    customer_id       UUID NOT NULL,
    customer_email    TEXT NOT NULL,
    customer_name     TEXT NOT NULL,
    status            TEXT NOT NULL,
    currency          CHAR(3) NOT NULL,
    total_cents       BIGINT NOT NULL,
    item_count        INTEGER NOT NULL,
    sku_summary       TEXT NOT NULL,
    aggregate_version INTEGER NOT NULL,
    last_seq          BIGINT NOT NULL,
    created_at        TIMESTAMPTZ NOT NULL,
    updated_at        TIMESTAMPTZ NOT NULL,
    search_tsv        TSVECTOR GENERATED ALWAYS AS (
        to_tsvector(
            'simple',
            coalesce(customer_email, '') || ' ' ||
            coalesce(customer_name, '') || ' ' ||
            coalesce(sku_summary, '') || ' ' ||
            coalesce(status, '')
        )
    ) STORED
);

-- Covering & Keyset Pagination Index: O(log N) seek with tenant isolation
CREATE INDEX IF NOT EXISTS idx_order_read_v1_tenant_keyset
    ON order_search_read_model_v1 (tenant_id, status, created_at DESC, order_id DESC)
    INCLUDE (customer_email, total_cents, item_count, aggregate_version);

CREATE INDEX IF NOT EXISTS idx_order_read_v1_tsv
    ON order_search_read_model_v1 USING GIN (search_tsv);

-- Active view pointer enabling zero-downtime Blue/Green swaps (v1 <-> v2)
CREATE OR REPLACE VIEW order_search_active_v AS
    SELECT * FROM order_search_read_model_v1;
```

### Step 2: PostgreSQL Transaction Helper (`src/db/postgres.ts`)

```typescript
// src/db/postgres.ts
import { Pool, PoolClient } from 'pg';

export async function withTransaction<T>(
  pool: Pool,
  fn: (client: PoolClient) => Promise<T>,
): Promise<T> {
  const client = await pool.connect();
  try {
    await client.query('BEGIN');
    const result = await fn(client);
    await client.query('COMMIT');
    return result;
  } catch (err) {
    await client.query('ROLLBACK');
    throw err;
  } finally {
    client.release();
  }
}
```

### Step 3: Write-Side Domain Aggregate & Command Handler (`src/orders/write/`)

```typescript
// src/orders/write/OrderAggregate.ts
export type OrderStatus = 'DRAFT' | 'CONFIRMED' | 'SHIPPED' | 'CANCELLED';

export interface OrderLineSnapshot {
  id: string;
  sku: string;
  title: string;
  quantity: number;
  unitCents: number;
}

export interface OrderSnapshot {
  id: string;
  tenantId: string;
  customerId: string;
  status: OrderStatus;
  currency: string;
  totalCents: number;
  version: number;
  lines: OrderLineSnapshot[];
  confirmedAt: Date | null;
  createdAt: Date;
}

export interface DomainEvent {
  eventType: 'OrderConfirmed' | 'OrderCancelled';
  aggregateId: string;
  aggregateVersion: number;
  payload: Record<string, unknown>;
}

export class DomainInvariantError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'DomainInvariantError';
  }
}

export class OptimisticConcurrencyError extends Error {
  constructor(orderId: string, expectedVersion: number) {
    super(`Concurrency conflict updating order ${orderId} at expected version ${expectedVersion}`);
    this.name = 'OptimisticConcurrencyError';
  }
}

export class OrderAggregate {
  private readonly pendingEvents: DomainEvent[] = [];

  private constructor(private state: OrderSnapshot) {}

  static rehydrate(snapshot: OrderSnapshot): OrderAggregate {
    return new OrderAggregate({ ...snapshot, lines: [...snapshot.lines] });
  }

  get snapshot(): Readonly<OrderSnapshot> {
    return this.state;
  }

  confirm(confirmedByActorId: string, now: Date): void {
    if (this.state.status !== 'DRAFT') {
      throw new DomainInvariantError(
        `Cannot confirm order ${this.state.id} in status ${this.state.status}`,
      );
    }
    if (this.state.lines.length === 0) {
      throw new DomainInvariantError(`Cannot confirm empty order ${this.state.id}`);
    }

    const recomputedTotal = this.state.lines.reduce(
      (sum, line) => sum + line.quantity * line.unitCents,
      0,
    );
    if (recomputedTotal <= 0) {
      throw new DomainInvariantError(`Order ${this.state.id} total must exceed zero`);
    }

    const nextVersion = this.state.version + 1;
    this.state = {
      ...this.state,
      status: 'CONFIRMED',
      totalCents: recomputedTotal,
      version: nextVersion,
      confirmedAt: now,
    };

    this.pendingEvents.push({
      eventType: 'OrderConfirmed',
      aggregateId: this.state.id,
      aggregateVersion: nextVersion,
      payload: {
        orderId: this.state.id,
        tenantId: this.state.tenantId,
        customerId: this.state.customerId,
        status: this.state.status,
        currency: this.state.currency,
        totalCents: this.state.totalCents,
        itemCount: this.state.lines.reduce((n, l) => n + l.quantity, 0),
        skuSummary: this.state.lines.map((l) => `${l.sku}:${l.title}`).join(' '),
        confirmedByActorId,
        confirmedAt: now.toISOString(),
        createdAt: this.state.createdAt.toISOString(),
      },
    });
  }

  pullPendingEvents(): DomainEvent[] {
    const events = [...this.pendingEvents];
    this.pendingEvents.length = 0;
    return events;
  }
}
```

Now implement `ConfirmOrderHandler.ts`, which guarantees command idempotency, loads the aggregate, checks optimistic concurrency `WHERE version = $expected`, writes the outbox row inside the exact same ACID transaction, and captures `pg_current_wal_lsn()` so the caller receives both `version` and `walLsn` for RYOW barriers.

```typescript
// src/orders/write/ConfirmOrderHandler.ts
import { Pool } from 'pg';
import { withTransaction } from '../../db/postgres.js';
import {
  DomainInvariantError,
  OptimisticConcurrencyError,
  OrderAggregate,
  OrderLineSnapshot,
} from './OrderAggregate.js';

export interface ConfirmOrderCommand {
  tenantId: string;
  orderId: string;
  actorId: string;
  expectedVersion: number;
  idempotencyKey: string;
  traceparent?: string;
}

export interface ConfirmOrderResult {
  orderId: string;
  status: 'CONFIRMED';
  version: number;
  totalCents: number;
  walLsn: string;
  outboxSeqId: number;
  deduplicated: boolean;
}

export class OrderNotFoundError extends Error {
  constructor(orderId: string) {
    super(`Order ${orderId} not found`);
    this.name = 'OrderNotFoundError';
  }
}

export class ConfirmOrderHandler {
  constructor(private readonly pool: Pool) {}

  async execute(cmd: ConfirmOrderCommand): Promise<ConfirmOrderResult> {
    return withTransaction(this.pool, async (client) => {
      // 1. Check command idempotency first inside the transaction
      const existingIdemp = await client.query<{ response_json: ConfirmOrderResult }>(
        `SELECT response_json
         FROM command_idempotency
         WHERE tenant_id = $1 AND idempotency_key = $2`,
        [cmd.tenantId, cmd.idempotencyKey],
      );
      if (existingIdemp.rowCount && existingIdemp.rows[0]) {
        return { ...existingIdemp.rows[0].response_json, deduplicated: true };
      }

      // 2. Load Order write row + line items (scoped strictly by tenant_id)
      const orderRes = await client.query<{
        id: string;
        tenant_id: string;
        customer_id: string;
        status: 'DRAFT' | 'CONFIRMED' | 'SHIPPED' | 'CANCELLED';
        currency: string;
        total_cents: string;
        version: number;
        confirmed_at: Date | null;
        created_at: Date;
      }>(
        `SELECT id, tenant_id, customer_id, status, currency, total_cents, version, confirmed_at, created_at
         FROM orders
         WHERE id = $1 AND tenant_id = $2
         FOR UPDATE`,
        [cmd.orderId, cmd.tenantId],
      );

      if (orderRes.rowCount === 0) {
        throw new OrderNotFoundError(cmd.orderId);
      }

      const row = orderRes.rows[0];
      if (row.version !== cmd.expectedVersion) {
        throw new OptimisticConcurrencyError(cmd.orderId, cmd.expectedVersion);
      }

      const linesRes = await client.query<{
        id: string;
        sku: string;
        title: string;
        quantity: number;
        unit_cents: string;
      }>(
        `SELECT id, sku, title, quantity, unit_cents
         FROM order_lines
         WHERE order_id = $1
         ORDER BY id ASC`,
        [cmd.orderId],
      );

      const lines: OrderLineSnapshot[] = linesRes.rows.map((l) => ({
        id: l.id,
        sku: l.sku,
        title: l.title,
        quantity: l.quantity,
        unitCents: Number(l.unit_cents),
      }));

      // 3. Execute pure domain invariant logic inside Aggregate
      const aggregate = OrderAggregate.rehydrate({
        id: row.id,
        tenantId: row.tenant_id,
        customerId: row.customer_id,
        status: row.status,
        currency: row.currency,
        totalCents: Number(row.total_cents),
        version: row.version,
        lines,
        confirmedAt: row.confirmed_at,
        createdAt: row.created_at,
      });

      const now = new Date();
      aggregate.confirm(cmd.actorId, now);
      const snap = aggregate.snapshot;
      const events = aggregate.pullPendingEvents();

      // 4. Persist Write Model with atomic OCC guard
      const updateRes = await client.query(
        `UPDATE orders
         SET status = $1,
             total_cents = $2,
             version = $3,
             confirmed_at = $4,
             updated_at = $4
         WHERE id = $5 AND tenant_id = $6 AND version = $7`,
        [
          snap.status,
          snap.totalCents,
          snap.version,
          snap.confirmedAt,
          snap.id,
          snap.tenantId,
          cmd.expectedVersion,
        ],
      );
      if (updateRes.rowCount !== 1) {
        throw new OptimisticConcurrencyError(cmd.orderId, cmd.expectedVersion);
      }

      // 5. Look up customer email/name to enrich Event-Carried State Transfer payload
      const custRes = await client.query<{ email: string; full_name: string }>(
        `SELECT email, full_name FROM customers WHERE id = $1 AND tenant_id = $2`,
        [snap.customerId, snap.tenantId],
      );
      const customer = custRes.rows[0];
      if (!customer) {
        throw new DomainInvariantError(`Customer ${snap.customerId} missing for order ${snap.id}`);
      }

      // 6. Write Outbox Event in the EXACT same ACID transaction
      let lastSeqId = 0;
      for (const evt of events) {
        const outboxRes = await client.query<{ seq_id: string }>(
          `INSERT INTO outbox_events (
             aggregate_type, aggregate_id, aggregate_ver, event_type, payload, metadata
           ) VALUES ($1, $2, $3, $4, $5::jsonb, $6::jsonb)
           RETURNING seq_id`,
          [
            'Order',
            evt.aggregateId,
            evt.aggregateVersion,
            evt.eventType,
            JSON.stringify({
              ...evt.payload,
              customerEmail: customer.email,
              customerName: customer.full_name,
            }),
            JSON.stringify({
              actorId: cmd.actorId,
              tenantId: cmd.tenantId,
              traceparent: cmd.traceparent ?? null,
            }),
          ],
        );
        lastSeqId = Number(outboxRes.rows[0].seq_id);
      }

      // 7. Capture current WAL LSN for sticky read-replica / RYOW consistency tokens
      const lsnRes = await client.query<{ lsn: string }>(`SELECT pg_current_wal_lsn()::text AS lsn`);
      const walLsn = lsnRes.rows[0].lsn;

      const result: ConfirmOrderResult = {
        orderId: snap.id,
        status: 'CONFIRMED',
        version: snap.version,
        totalCents: snap.totalCents,
        walLsn,
        outboxSeqId: lastSeqId,
        deduplicated: false,
      };

      await client.query(
        `INSERT INTO command_idempotency (tenant_id, idempotency_key, command_name, response_json)
         VALUES ($1, $2, $3, $4::jsonb)
         ON CONFLICT (tenant_id, idempotency_key) DO NOTHING`,
        [cmd.tenantId, cmd.idempotencyKey, 'ConfirmOrder', JSON.stringify(result)],
      );

      return result;
    });
  }
}
```

### Step 4: Flat SQL Keyset-Paginated Read Query Handler (`src/orders/read/`)

Notice how `ListOrderDashboardQuery` supports both **Level 2 Read Store** queries (`order_search_active_v`) and **Level 1 / RYOW Direct Primary Fallback** when `minVersion` or `minWalLsn` detects projection or replica lag—completely bypassing `OrderAggregate` hydration.

```typescript
// src/orders/read/OrderReadTypes.ts
export interface OrderDashboardFilter {
  tenantId: string;
  status?: 'DRAFT' | 'CONFIRMED' | 'SHIPPED' | 'CANCELLED';
  searchText?: string;
  cursorCreatedAt?: string;
  cursorOrderId?: string;
  limit?: number;
  minOrderId?: string;
  minVersion?: number;
  minWalLsn?: string;
}

export interface OrderDashboardRow {
  orderId: string;
  customerEmail: string;
  customerName: string;
  status: string;
  currency: string;
  totalCents: number;
  itemCount: number;
  aggregateVersion: number;
  createdAt: string;
}

export interface OrderDashboardPage {
  items: OrderDashboardRow[];
  nextCursor: { createdAt: string; orderId: string } | null;
  servedFrom: 'READ_PROJECTION' | 'WRITE_PRIMARY_FALLBACK';
}
```

```typescript
// src/orders/read/ListOrderDashboardQuery.ts
import { Pool } from 'pg';
import { OrderDashboardFilter, OrderDashboardPage, OrderDashboardRow } from './OrderReadTypes.js';

export class ListOrderDashboardQuery {
  constructor(
    private readonly readPool: Pool,
    private readonly writePrimaryPool: Pool,
  ) {}

  async execute(filter: OrderDashboardFilter): Promise<OrderDashboardPage> {
    const limit = Math.min(Math.max(filter.limit ?? 25, 1), 100);

    // RYOW Barrier 1: If caller holds a WAL LSN token and readPool is a physical replica,
    // verify whether replica replay caught up; otherwise fall back straight to writePrimaryPool.
    if (filter.minWalLsn) {
      const lagCheck = await this.readPool.query<{ caught_up: boolean }>(
        `SELECT CASE
           WHEN pg_is_in_recovery() THEN pg_last_wal_replay_lsn() >= $1::pg_lsn
           ELSE TRUE
         END AS caught_up`,
        [filter.minWalLsn],
      );
      if (!lagCheck.rows[0]?.caught_up) {
        return this.queryDirectFromWritePrimary(filter, limit);
      }
    }

    // RYOW Barrier 2: If caller just mutated a specific order to minVersion, check read projection freshness.
    if (filter.minOrderId && filter.minVersion !== undefined) {
      const verCheck = await this.readPool.query<{ aggregate_version: number }>(
        `SELECT aggregate_version
         FROM order_search_active_v
         WHERE order_id = $1 AND tenant_id = $2`,
        [filter.minOrderId, filter.tenantId],
      );
      const currentVer = verCheck.rows[0]?.aggregate_version ?? 0;
      if (currentVer < filter.minVersion) {
        return this.queryDirectFromWritePrimary(filter, limit);
      }
    }

    return this.queryFromReadProjection(filter, limit);
  }

  private async queryFromReadProjection(
    filter: OrderDashboardFilter,
    limit: number,
  ): Promise<OrderDashboardPage> {
    const params: unknown[] = [filter.tenantId];
    const clauses: string[] = ['tenant_id = $1'];

    if (filter.status) {
      params.push(filter.status);
      clauses.push(`status = $${params.length}`);
    }

    if (filter.searchText && filter.searchText.trim().length > 0) {
      params.push(filter.searchText.trim());
      clauses.push(`search_tsv @@ plainto_tsquery('simple', $${params.length})`);
    }

    // Keyset cursor pagination: deterministic O(log N) seek, immune to offset drift
    if (filter.cursorCreatedAt && filter.cursorOrderId) {
      params.push(filter.cursorCreatedAt, filter.cursorOrderId);
      const tsIdx = params.length - 1;
      const idIdx = params.length;
      clauses.push(`(created_at, order_id) < ($${tsIdx}::timestamptz, $${idIdx}::uuid)`);
    }

    params.push(limit + 1);
    const limitIdx = params.length;

    const sql = `
      SELECT
        order_id,
        customer_email,
        customer_name,
        status,
        currency,
        total_cents,
        item_count,
        aggregate_version,
        created_at
      FROM order_search_active_v
      WHERE ${clauses.join(' AND ')}
      ORDER BY created_at DESC, order_id DESC
      LIMIT $${limitIdx}
    `;

    const res = await this.readPool.query<{
      order_id: string;
      customer_email: string;
      customer_name: string;
      status: string;
      currency: string;
      total_cents: string;
      item_count: number;
      aggregate_version: number;
      created_at: Date;
    }>(sql, params);

    return this.formatPage(res.rows, limit, 'READ_PROJECTION');
  }

  // Level 1 flat SQL fallback against Write Primary (zero Aggregate hydration)
  private async queryDirectFromWritePrimary(
    filter: OrderDashboardFilter,
    limit: number,
  ): Promise<OrderDashboardPage> {
    const params: unknown[] = [filter.tenantId];
    const clauses: string[] = ['o.tenant_id = $1'];

    if (filter.status) {
      params.push(filter.status);
      clauses.push(`o.status = $${params.length}`);
    }

    if (filter.cursorCreatedAt && filter.cursorOrderId) {
      params.push(filter.cursorCreatedAt, filter.cursorOrderId);
      const tsIdx = params.length - 1;
      const idIdx = params.length;
      clauses.push(`(o.created_at, o.id) < ($${tsIdx}::timestamptz, $${idIdx}::uuid)`);
    }

    params.push(limit + 1);
    const limitIdx = params.length;

    const sql = `
      SELECT
        o.id AS order_id,
        c.email AS customer_email,
        c.full_name AS customer_name,
        o.status,
        o.currency,
        o.total_cents,
        COALESCE(SUM(ol.quantity), 0)::int AS item_count,
        o.version AS aggregate_version,
        o.created_at
      FROM orders o
      JOIN customers c ON c.id = o.customer_id
      LEFT JOIN order_lines ol ON ol.order_id = o.id
      WHERE ${clauses.join(' AND ')}
      GROUP BY o.id, c.email, c.full_name
      ORDER BY o.created_at DESC, o.id DESC
      LIMIT $${limitIdx}
    `;

    const res = await this.writePrimaryPool.query<{
      order_id: string;
      customer_email: string;
      customer_name: string;
      status: string;
      currency: string;
      total_cents: string;
      item_count: number;
      aggregate_version: number;
      created_at: Date;
    }>(sql, params);

    return this.formatPage(res.rows, limit, 'WRITE_PRIMARY_FALLBACK');
  }

  private formatPage(
    rawRows: Array<{
      order_id: string;
      customer_email: string;
      customer_name: string;
      status: string;
      currency: string;
      total_cents: string;
      item_count: number;
      aggregate_version: number;
      created_at: Date;
    }>,
    limit: number,
    servedFrom: OrderDashboardPage['servedFrom'],
  ): OrderDashboardPage {
    const hasMore = rawRows.length > limit;
    const sliced = hasMore ? rawRows.slice(0, limit) : rawRows;
    const items: OrderDashboardRow[] = sliced.map((r) => ({
      orderId: r.order_id,
      customerEmail: r.customer_email,
      customerName: r.customer_name,
      status: r.status,
      currency: r.currency,
      totalCents: Number(r.total_cents),
      itemCount: r.item_count,
      aggregateVersion: r.aggregate_version,
      createdAt: r.created_at.toISOString(),
    }));

    const last = items[items.length - 1];
    return {
      items,
      nextCursor: hasMore && last ? { createdAt: last.createdAt, orderId: last.orderId } : null,
      servedFrom,
    };
  }
}
```

### Step 5: Level 2 Async Projector & OpenSearch External-Version Indexer (`src/orders/projector/`)

```typescript
// src/orders/projector/OpenSearchClient.ts
export interface SearchDocumentUpsert {
  index: string;
  id: string;
  externalVersion: number;
  document: Record<string, unknown>;
}

export interface SearchIndexAdapter {
  upsertWithExternalVersion(req: SearchDocumentUpsert): Promise<'INDEXED' | 'CONFLICT_STALE'>;
  swapAliasAtomically(aliasName: string, oldIndex: string, newIndex: string): Promise<void>;
}

export class HttpOpenSearchAdapter implements SearchIndexAdapter {
  constructor(private readonly baseUrl: string) {}

  async upsertWithExternalVersion(
    req: SearchDocumentUpsert,
  ): Promise<'INDEXED' | 'CONFLICT_STALE'> {
    const url = `${this.baseUrl}/${encodeURIComponent(req.index)}/_doc/${encodeURIComponent(req.id)}?version=${req.externalVersion}&version_type=external`;
    const res = await fetch(url, {
      method: 'PUT',
      headers: { 'content-type': 'application/json' },
      body: JSON.stringify(req.document),
    });

    if (res.status === 409) {
      // Expected idempotent or out-of-order stale arrival (stored version >= incoming version)
      return 'CONFLICT_STALE';
    }
    if (!res.ok) {
      const body = await res.text();
      throw new Error(`OpenSearch index error ${res.status}: ${body}`);
    }
    return 'INDEXED';
  }

  async swapAliasAtomically(aliasName: string, oldIndex: string, newIndex: string): Promise<void> {
    const res = await fetch(`${this.baseUrl}/_aliases`, {
      method: 'POST',
      headers: { 'content-type': 'application/json' },
      body: JSON.stringify({
        actions: [
          { remove: { index: oldIndex, alias: aliasName } },
          { add: { index: newIndex, alias: aliasName } },
        ],
      }),
    });
    if (!res.ok) {
      throw new Error(`OpenSearch alias swap failed ${res.status}: ${await res.text()}`);
    }
  }
}
```

```typescript
// src/orders/projector/OrderReadProjector.ts
import { Pool } from 'pg';
import { SearchIndexAdapter } from './OpenSearchClient.js';

export interface OutboxOrderEvent {
  seqId: number;
  eventId: string;
  aggregateId: string;
  aggregateVersion: number;
  eventType: string;
  payload: {
    orderId: string;
    tenantId: string;
    customerId: string;
    customerEmail: string;
    customerName: string;
    status: string;
    currency: string;
    totalCents: number;
    itemCount: number;
    skuSummary: string;
    createdAt: string;
    confirmedAt?: string;
  };
}

export class OrderReadProjector {
  constructor(
    private readonly readPool: Pool,
    private readonly searchAdapter: SearchIndexAdapter,
    private readonly targetReadTable = 'order_search_read_model_v1',
    private readonly targetSearchIndex = 'orders_search_v1',
  ) {}

  async projectEvent(event: OutboxOrderEvent): Promise<'APPLIED' | 'SKIPPED_STALE'> {
    const p = event.payload;

    // 1. Monotonic conditional UPSERT in Postgres Read Store
    // GREATEST/WHERE guard ensures replayed or out-of-order lower seqId/version never overwrites newer state.
    const pgResult = await this.readPool.query(
      `INSERT INTO ${this.targetReadTable} (
         order_id, tenant_id, customer_id, customer_email, customer_name,
         status, currency, total_cents, item_count, sku_summary,
         aggregate_version, last_seq, created_at, updated_at
       )
       VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11, $12, $13::timestamptz, NOW())
       ON CONFLICT (order_id) DO UPDATE
       SET customer_email    = EXCLUDED.customer_email,
           customer_name     = EXCLUDED.customer_name,
           status            = EXCLUDED.status,
           total_cents       = EXCLUDED.total_cents,
           item_count        = EXCLUDED.item_count,
           sku_summary       = EXCLUDED.sku_summary,
           aggregate_version = GREATEST(${this.targetReadTable}.aggregate_version, EXCLUDED.aggregate_version),
           last_seq          = GREATEST(${this.targetReadTable}.last_seq, EXCLUDED.last_seq),
           updated_at        = NOW()
       WHERE ${this.targetReadTable}.last_seq < EXCLUDED.last_seq
         AND ${this.targetReadTable}.aggregate_version <= EXCLUDED.aggregate_version`,
      [
        p.orderId,
        p.tenantId,
        p.customerId,
        p.customerEmail,
        p.customerName,
        p.status,
        p.currency,
        p.totalCents,
        p.itemCount,
        p.skuSummary,
        event.aggregateVersion,
        event.seqId,
        p.createdAt,
      ],
    );

    // 2. External optimistic version upsert into OpenSearch
    const searchStatus = await this.searchAdapter.upsertWithExternalVersion({
      index: this.targetSearchIndex,
      id: p.orderId,
      externalVersion: event.aggregateVersion,
      document: {
        orderId: p.orderId,
        tenantId: p.tenantId,
        customerEmail: p.customerEmail,
        customerName: p.customerName,
        status: p.status,
        totalCents: p.totalCents,
        itemCount: p.itemCount,
        skuSummary: p.skuSummary,
        createdAt: p.createdAt,
      },
    });

    if (pgResult.rowCount === 0 && searchStatus === 'CONFLICT_STALE') {
      return 'SKIPPED_STALE';
    }
    return 'APPLIED';
  }
}
```

### Step 6: Zero-Downtime Blue/Green Projection Rebuilder (`src/orders/projector/ProjectionBlueGreenRebuilder.ts`)

```typescript
// src/orders/projector/ProjectionBlueGreenRebuilder.ts
import { Pool } from 'pg';
import { withTransaction } from '../../db/postgres.js';
import { SearchIndexAdapter } from './OpenSearchClient.js';
import { OrderReadProjector, OutboxOrderEvent } from './OrderReadProjector.js';

export class ProjectionBlueGreenRebuilder {
  constructor(
    private readonly writePool: Pool,
    private readonly readPool: Pool,
    private readonly searchAdapter: SearchIndexAdapter,
  ) {}

  async rebuildAndSwapToV2(
    activeViewName = 'order_search_active_v',
    v2TableName = 'order_search_read_model_v2',
    oldIndex = 'orders_search_v1',
    newIndex = 'orders_search_v2',
    searchAlias = 'orders_search_read',
  ): Promise<{ backfilledRows: number }> {
    const shadowProjector = new OrderReadProjector(
      this.readPool,
      this.searchAdapter,
      v2TableName,
      newIndex,
    );

    let lastSeq = 0;
    let totalProcessed = 0;
    const batchSize = 1000;

    // Phase 1 & 2: Keyset scan over historical outbox events without blocking v1 readers
    while (true) {
      const batch = await this.writePool.query<{
        seq_id: string;
        event_id: string;
        aggregate_id: string;
        aggregate_ver: number;
        event_type: string;
        payload: OutboxOrderEvent['payload'];
      }>(
        `SELECT seq_id, event_id, aggregate_id, aggregate_ver, event_type, payload
         FROM outbox_events
         WHERE aggregate_type = 'Order' AND seq_id > $1
         ORDER BY seq_id ASC
         LIMIT $2`,
        [lastSeq, batchSize],
      );

      if (batch.rowCount === 0) break;

      for (const row of batch.rows) {
        const seqId = Number(row.seq_id);
        await shadowProjector.projectEvent({
          seqId,
          eventId: row.event_id,
          aggregateId: row.aggregate_id,
          aggregateVersion: row.aggregate_ver,
          eventType: row.event_type,
          payload: row.payload,
        });
        lastSeq = seqId;
        totalProcessed++;
      }
    }

    // Phase 3: Atomic pointer swap in PostgreSQL view and OpenSearch cluster alias
    await withTransaction(this.readPool, async (client) => {
      await client.query(`CREATE OR REPLACE VIEW ${activeViewName} AS SELECT * FROM ${v2TableName}`);
    });
    await this.searchAdapter.swapAliasAtomically(searchAlias, oldIndex, newIndex);

    return { backfilledRows: totalProcessed };
  }
}
```

### Step 7: Concurrency, Ordering, RYOW & Keyset Integration Tests (`tests/orders.cqrs.integration.test.ts`)

```typescript
// tests/orders.cqrs.integration.test.ts
import { afterAll, beforeAll, beforeEach, describe, expect, it } from 'vitest';
import { Pool } from 'pg';
import { randomUUID } from 'node:crypto';
import { ConfirmOrderHandler } from '../src/orders/write/ConfirmOrderHandler.js';
import { OptimisticConcurrencyError } from '../src/orders/write/OrderAggregate.js';
import { ListOrderDashboardQuery } from '../src/orders/read/ListOrderDashboardQuery.js';
import { OrderReadProjector } from '../src/orders/projector/OrderReadProjector.js';
import { SearchIndexAdapter, SearchDocumentUpsert } from '../src/orders/projector/OpenSearchClient.js';

class InMemoryVersionedSearchFake implements SearchIndexAdapter {
  public readonly docs = new Map<string, { version: number; doc: Record<string, unknown> }>();

  async upsertWithExternalVersion(req: SearchDocumentUpsert): Promise<'INDEXED' | 'CONFLICT_STALE'> {
    const existing = this.docs.get(req.id);
    if (existing && existing.version >= req.externalVersion) {
      return 'CONFLICT_STALE';
    }
    this.docs.set(req.id, { version: req.externalVersion, doc: req.document });
    return 'INDEXED';
  }

  async swapAliasAtomically(): Promise<void> {
    return;
  }
}

describe('CQRS Order Search & Fulfillment Engine Integration', () => {
  let pool: Pool;
  let searchFake: InMemoryVersionedSearchFake;
  let tenantId: string;
  let customerId: string;

  beforeAll(async () => {
    pool = new Pool({
      connectionString: process.env.DATABASE_URL ?? 'postgres://postgres:postgres@localhost:5432/cqrs_test',
    });
  });

  afterAll(async () => {
    await pool.end();
  });

  beforeEach(async () => {
    searchFake = new InMemoryVersionedSearchFake();
    tenantId = randomUUID();
    customerId = randomUUID();
    await pool.query(
      'TRUNCATE command_idempotency, outbox_events, order_lines, orders, customers, order_search_read_model_v1 RESTART IDENTITY CASCADE',
    );
    await pool.query(
      'INSERT INTO customers (id, tenant_id, email, full_name) VALUES ($1, $2, $3, $4)',
      [customerId, tenantId, 'buyer@acme.io', 'Ada Buyer'],
    );
  });

  it('commits Order confirmation + outbox atomically and blocks concurrent version conflicts', async () => {
    const orderId = randomUUID();
    await pool.query(
      `INSERT INTO orders (id, tenant_id, customer_id, status, currency, total_cents, version)
       VALUES ($1, $2, $3, 'DRAFT', 'USD', 0, 1)`,
      [orderId, tenantId, customerId],
    );
    await pool.query(
      `INSERT INTO order_lines (id, order_id, sku, title, quantity, unit_cents)
       VALUES ($1, $2, 'SKU-9', 'Mechanical Switch', 2, 2500)`,
      [randomUUID(), orderId],
    );

    const handler = new ConfirmOrderHandler(pool);
    const [r1, r2] = await Promise.allSettled([
      handler.execute({
        tenantId,
        orderId,
        actorId: 'usr-1',
        expectedVersion: 1,
        idempotencyKey: 'cmd-A',
      }),
      handler.execute({
        tenantId,
        orderId,
        actorId: 'usr-2',
        expectedVersion: 1,
        idempotencyKey: 'cmd-B',
      }),
    ]);

    const fulfilled = [r1, r2].filter((r) => r.status === 'fulfilled');
    const rejected = [r1, r2].filter((r) => r.status === 'rejected');
    expect(fulfilled).toHaveLength(1);
    expect(rejected).toHaveLength(1);
    expect((rejected[0] as PromiseRejectedResult).reason).toBeInstanceOf(OptimisticConcurrencyError);

    const outbox = await pool.query('SELECT aggregate_ver, event_type FROM outbox_events WHERE aggregate_id = $1', [
      orderId,
    ]);
    expect(outbox.rowCount).toBe(1);
    expect(outbox.rows[0].aggregate_ver).toBe(2);
  });

  it('enforces RYOW minVersion fallback before projection catches up, then serves from Read Store', async () => {
    const orderId = randomUUID();
    await pool.query(
      `INSERT INTO orders (id, tenant_id, customer_id, status, currency, total_cents, version)
       VALUES ($1, $2, $3, 'DRAFT', 'USD', 0, 1)`,
      [orderId, tenantId, customerId],
    );
    await pool.query(
      `INSERT INTO order_lines (id, order_id, sku, title, quantity, unit_cents)
       VALUES ($1, $2, 'SKU-1', 'USB-C Hub', 1, 4900)`,
      [randomUUID(), orderId],
    );

    const handler = new ConfirmOrderHandler(pool);
    const cmdResult = await handler.execute({
      tenantId,
      orderId,
      actorId: 'usr-1',
      expectedVersion: 1,
      idempotencyKey: 'ryow-1',
    });

    const query = new ListOrderDashboardQuery(pool, pool);

    // Before projector runs: read store is empty (lagging); minVersion triggers Write Primary fallback
    const immediatePage = await query.execute({
      tenantId,
      status: 'CONFIRMED',
      minOrderId: orderId,
      minVersion: cmdResult.version,
    });
    expect(immediatePage.servedFrom).toBe('WRITE_PRIMARY_FALLBACK');
    expect(immediatePage.items[0]?.orderId).toBe(orderId);
    expect(immediatePage.items[0]?.totalCents).toBe(4900);

    // Now run Level 2 async projector
    const outboxRow = await pool.query('SELECT * FROM outbox_events WHERE seq_id = $1', [cmdResult.outboxSeqId]);
    const projector = new OrderReadProjector(pool, searchFake);
    await projector.projectEvent({
      seqId: Number(outboxRow.rows[0].seq_id),
      eventId: outboxRow.rows[0].event_id,
      aggregateId: outboxRow.rows[0].aggregate_id,
      aggregateVersion: outboxRow.rows[0].aggregate_ver,
      eventType: outboxRow.rows[0].event_type,
      payload: outboxRow.rows[0].payload,
    });

    // After projector commits: query satisfies minVersion from Read Store directly
    const afterProjPage = await query.execute({
      tenantId,
      status: 'CONFIRMED',
      minOrderId: orderId,
      minVersion: cmdResult.version,
    });
    expect(afterProjPage.servedFrom).toBe('READ_PROJECTION');
    expect(afterProjPage.items[0]?.aggregateVersion).toBe(2);
  });

  it('prevents out-of-order replay from regressing Postgres and OpenSearch projections', async () => {
    const orderId = randomUUID();
    const projector = new OrderReadProjector(pool, searchFake);

    const newerEvent = {
      seqId: 20,
      eventId: randomUUID(),
      aggregateId: orderId,
      aggregateVersion: 3,
      eventType: 'OrderShipped',
      payload: {
        orderId,
        tenantId,
        customerId,
        customerEmail: 'buyer@acme.io',
        customerName: 'Ada Buyer',
        status: 'SHIPPED',
        currency: 'USD',
        totalCents: 7500,
        itemCount: 3,
        skuSummary: 'SKU-7:Dock',
        createdAt: '2026-11-10T10:00:00.000Z',
      },
    };

    const olderReplayedEvent = {
      ...newerEvent,
      seqId: 12,
      eventId: randomUUID(),
      aggregateVersion: 2,
      eventType: 'OrderConfirmed',
      payload: {
        ...newerEvent.payload,
        status: 'CONFIRMED',
        totalCents: 5000,
      },
    };

    expect(await projector.projectEvent(newerEvent)).toBe('APPLIED');
    expect(await projector.projectEvent(olderReplayedEvent)).toBe('SKIPPED_STALE');

    const row = await pool.query(
      'SELECT status, total_cents, aggregate_version, last_seq FROM order_search_read_model_v1 WHERE order_id = $1',
      [orderId],
    );
    expect(row.rows[0].status).toBe('SHIPPED');
    expect(Number(row.rows[0].total_cents)).toBe(7500);
    expect(row.rows[0].aggregate_version).toBe(3);
    expect(Number(row.rows[0].last_seq)).toBe(20);
    expect(searchFake.docs.get(orderId)?.version).toBe(3);
  });
});
```

---

## 4. Complete Framework Reference & Production Edge Cases

### Command & Query Contract Reference

| Concern | Command Contract | Query Contract |
|---|---|---|
| **Validation** | Structural (Zod/class-validator) + Domain invariants inside Aggregate | Structural filter & cursor validation + **mandatory `tenant_id` authorization predicate** |
| **HTTP Semantics** | `200 OK` / `201 Created` (sync), `202 Accepted` (async queue), `409 Conflict` (OCC version mismatch), `412 Precondition Failed` (`If-Match`) | `200 OK`, `404 Not Found` (avoid `403` leaking cross-tenant existence) |
| **Idempotency** | Mandatory `Idempotency-Key` header persisted inside write TX (`command_idempotency`) | Inherently safe & side-effect free (never mutate state or increment counters inside `GET`) |
| **Return Payload** | Identifier + `version` + `walLsn` (plus minimal optimistic UI patch) | Flat caller-tailored `ReadDTO` + `nextCursor` (never return Write Aggregate internals) |
| **Pagination** | N/A | **Keyset cursor** `(created_at, id) < ($ts, $id)` — never `OFFSET` on large datasets |

### Subtle Failure Modes & Concurrency Traps

1. **Multi-Aggregate Join Races in Async Projectors**:
   - *Trap*: Suppose `order_search_read_model` merges fields from `OrderConfirmed` (Order Service) and `CustomerEmailChanged` (Customer Service) emitted onto two distinct Kafka topics with different partition keys (`orderId` vs `customerId`). If `CustomerEmailChanged` consumes before `OrderConfirmed` creates the row, a naïve `UPDATE order_search_read_model SET customer_email = ... WHERE customer_id = ...` silently affects `0 rows` and drops the update forever.
   - *Fix*: Include authoritative snapshot fields inside `OrderConfirmed` (ECST), OR maintain a lightweight `customer_projection_cache(customer_id, email, customer_version)` in the read DB that `OrderProjector` joins at project time while `CustomerProjector` updates both `customer_projection_cache` (with `ON CONFLICT DO UPDATE WHERE customer_version < ...`) and existing `order_search_read_model` rows inside one transaction.
2. **Offset Pagination Drift & `O(N)` Degradation**:
   - *Trap*: Using `LIMIT 25 OFFSET 20000` forces PostgreSQL to scan and discard `20,025` rows on every request. Worse, if a background projector inserts 3 new orders while a user navigates from page 1 to page 2, rows shift downward and appear duplicated across pages.
   - *Fix*: Always use **Keyset Cursor Pagination** backed by a composite B-tree index matching the `ORDER BY (created_at DESC, order_id DESC)` direction (see [[Programming/Software Engineering/Database Design/Indexing|Indexing]]).
3. **Authorization Bypass on Thin Read Queries**:
   - *Trap*: In a classic domain layer, a central service enforces `order.assertOwnedBy(tenantId)`. When Level 1/2 read queries bypass the domain layer to query `order_search_active_v` directly, forgetting `WHERE tenant_id = $1` on a single query creates a catastrophic cross-tenant data leak!
   - *Fix*: Enforce tenant isolation at two independent layers: wrap `readPool` query construction inside a `TenantScopedDbClient` requiring `tenantId`, and enable PostgreSQL **Row-Level Security (RLS)** (`FORCE ROW LEVEL SECURITY` using `current_setting('app.tenant_id', true)::uuid`) on read tables as defense in depth.
4. **OpenSearch Refresh Interval Visibility Gap**:
   - *Trap*: Even after a Kafka projector receives HTTP `200 OK` from OpenSearch `_doc`, new documents sit inside the in-memory indexing buffer until `index.refresh_interval` fires (default `1s`), causing immediate search assertions in automated integration tests or RYOW redirects to miss the document.
   - *Fix*: Keep `refresh_interval = 1s` (or `5s`) in production for segment merge throughput, use `?refresh=wait_for` exclusively for user-interactive critical writes or test harnesses, and rely on `minVersion` Postgres fallback for immediate read-your-own-writes.

---

## 5. Production Debugging, Profiling & Operations Playbook

| Symptom | Root Cause | Diagnostic Command / Query | Production Fix |
|---|---|---|---|
| **Orders missing from search 10s+ after checkout** | Outbox relay stalled, Kafka consumer rebalance loop, or projector throwing on unmapped payload | `SELECT COUNT(*), MIN(created_at), MAX(EXTRACT(EPOCH FROM NOW() - created_at)) FROM outbox_events WHERE published_at IS NULL;` and `kafka-consumer-groups.sh --bootstrap-server $BROKER --describe --group order-projector` | Fix poison pill / scale projector pods up to partition count; add alert when oldest unpublished outbox age $> 5\text{s}$. |
| **Dashboard shows stale order status despite `200 OK` confirm** | Missing RYOW barrier or `aggregate_version` not sent by UI after mutation | Compare `SELECT version, status FROM orders WHERE id = $1;` vs `SELECT aggregate_version, last_seq, status FROM order_search_active_v WHERE order_id = $1;` | Return `version` + `walLsn` from `POST /confirm`, pass `?minVersion` on follow-up `GET`, and patch query cache optimistically. |
| **Read DB CPU 95% on dashboard `GET /orders`** | `OFFSET` pagination or missing composite `INCLUDE` index causing heap fetches | `EXPLAIN (ANALYZE, BUFFERS) SELECT ... FROM order_search_active_v WHERE tenant_id = $1 ORDER BY created_at DESC, order_id DESC LIMIT 25;` | Switch to keyset cursor `(created_at, order_id) < ($2, $3)` and add covering index `INCLUDE (customer_email, total_cents, ...)`. |
| **Search shows lower price/older status after DLQ replay** | Projector overwrote read view without monotonic version fence | Inspect `outbox_events` vs `order_search_active_v.last_seq`; inspect OpenSearch `_doc?version=true` | Add `WHERE last_seq < EXCLUDED.last_seq` in PG upsert and `version_type=external` in OpenSearch; trigger targeted stream replay for affected IDs. |
| **Outbox table bloated to 40GB slowing relay polls** | `published_at IS NOT NULL` rows never archived/pruned; autovacuum lags | `SELECT pg_size_pretty(pg_total_relation_size('outbox_events')), n_dead_tup FROM pg_stat_user_tables WHERE relname = 'outbox_events';` | Partition `outbox_events` daily/hourly (`DROP TABLE` old partitions instantly instead of `DELETE`) or move to Debezium WAL CDC reading `DELETE`-free log. |

### Distributed Trace Propagation Across the Async Boundary
To debug end-to-end latency across Level 2 CQRS in Grafana Tempo or Jaeger (see [[Programming/Software Engineering/DevOps/Observability|Observability]]), store the W3C `traceparent` header inside `outbox_events.metadata` during the Write TX. When `OrderReadProjector` processes the event, extract `metadata.traceparent` as a **Span Link** (or child span) rather than creating an orphan trace—allowing engineers to inspect Write TX duration, Outbox wait time, Kafka transit time, and Read Store commit on one flame graph.

---

## 6. Worked Exercises

### Exercise 1: Refactoring an $N+1$ ORM Read Endpoint to Level 1 CQRS
An admin dashboard endpoint `GET /v1/orders?status=CONFIRMED&page=120` currently calls `orderRepo.findByStatus('CONFIRMED')` loading 50 `Order` Domain Aggregates (averaging 25 `OrderItem` child objects each) and executes `customerRepo.findById(order.customerId)` per row before mapping DTOs in memory. Under load, p99 latency exceeds `1,900 ms`, Node heap spikes by `180 MB` per request, and page 120 occasionally repeats orders from page 119. Diagnose every structural defect and write the complete Level 1 CQRS refactor.

---

### Solution 1

#### Root Cause Analysis
1. **Write-Model Hydration Tax**: Loading $50 \times 25 = 1,250$ `OrderItem` entities executes domain validation constructors and `Money` object allocations purely to read 4 summary fields.
2. **$N+1$ Customer Lookup**: Calling `customerRepo.findById` inside a loop produces `1 + 50 = 51` database round-trips per request.
3. **Offset Pagination Instability**: `page=120` translates to `OFFSET 5950`, forcing PostgreSQL to scan and discard 6,000 rows while allowing new inserts to shift boundaries between pages.

#### Level 1 CQRS Refactor
1. Persist denormalized `total_cents` and `item_count` on `orders` inside the write-side `OrderAggregate` transaction (eliminating runtime `SUM(quantity)` across `order_lines`).
2. Replace `orderRepo.findByStatus` on the read path with a dedicated flat SQL Query Handler using keyset cursor pagination and a single `JOIN customers`:

```sql
SELECT
  o.id AS order_id,
  c.email AS customer_email,
  o.total_cents,
  o.item_count,
  o.created_at
FROM orders o
JOIN customers c ON c.id = o.customer_id
WHERE o.tenant_id = $1
  AND o.status = 'CONFIRMED'
  AND (o.created_at, o.id) < ($2::timestamptz, $3::uuid)
ORDER BY o.created_at DESC, o.id DESC
LIMIT 51;
```

3. Support this exact access path with a covering composite index:
   `CREATE INDEX idx_orders_tenant_status_keyset ON orders (tenant_id, status, created_at DESC, id DESC) INCLUDE (customer_id, total_cents, item_count);`
4. **Measured Result**: Database round-trips drop from `51` to `1`, Node heap churn drops by `> 95%`, page drift disappears completely, and p99 latency stabilizes under `8 ms` with zero eventual consistency lag.

---

### Exercise 2: Fixing an Out-of-Order Projector Regression Across PostgreSQL & OpenSearch
A team runs Level 2 CQRS: commands update PostgreSQL and emit `ProductUpdated` events via Kafka (`partitionKey = productId`). During an operational DLQ replay, an older `ProductPriceUpdated (priceCents: 9900, aggregateVersion: 3, seqId: 104)` event is consumed *after* `ProductPriceUpdated (priceCents: 14900, aggregateVersion: 4, seqId: 118)`. Both the Postgres `product_search_read` table and the OpenSearch `products_v1` index revert to `$99.00` while checkout charges `$149.00`. Design the complete monotonic storage-level defense for both read stores.

---

### Solution 2

#### 1. Stamp Monotonic Fence on Write Model
Increment `version` atomically inside the write transaction (`UPDATE products SET price_cents = $1, version = version + 1 ... RETURNING version`) and stamp both `aggregateVersion` and `seqId` onto the outbox record.

#### 2. Conditional PostgreSQL Upsert & OpenSearch External Versioning
Never execute blind `ON CONFLICT (product_id) DO UPDATE SET price_cents = EXCLUDED.price_cents` or blind `PUT /products_v1/_doc/:id`. Guard both stores atomically:

```typescript
// PostgreSQL monotonic upsert
await readDb.query(
  `INSERT INTO product_search_read (product_id, price_cents, aggregate_version, last_seq)
   VALUES ($1, $2, $3, $4)
   ON CONFLICT (product_id) DO UPDATE
   SET price_cents = EXCLUDED.price_cents,
       aggregate_version = EXCLUDED.aggregate_version,
       last_seq = EXCLUDED.last_seq
   WHERE product_search_read.aggregate_version < EXCLUDED.aggregate_version`,
  [evt.productId, evt.priceCents, evt.aggregateVersion, evt.seqId],
);

// OpenSearch external optimistic concurrency
const res = await fetch(
  `${esUrl}/products_v1/_doc/${evt.productId}?version=${evt.aggregateVersion}&version_type=external`,
  {
    method: 'PUT',
    headers: { 'content-type': 'application/json' },
    body: JSON.stringify({ priceCents: evt.priceCents, updatedAt: evt.occurredAt }),
  },
);
if (res.status === 409) {
  // Stale replay (version 3 <= current version 4): safely acknowledge and discard
  return;
}
if (!res.ok) throw new Error(`OpenSearch failed: ${res.status}`);
```

*Result*: Replaying `version: 3` after `version: 4` updates `0` rows in PostgreSQL and returns `409 Conflict` from OpenSearch, guaranteeing forward convergence regardless of Kafka retry ordering.

---

### Exercise 3: Eliminating Read-Your-Own-Writes Flash via WAL LSN & `minVersion`
A Level 2 CQRS user portal uses PostgreSQL Primary for writes, a PostgreSQL Streaming Read Replica for standard queries (~150ms replication lag), and a Redis Read Cache populated by Debezium (~300ms lag). When a user updates their billing address via `PUT /v1/profile/address` and reloads `/profile`, 40% of reloads render the stale address. Design a dual-token RYOW routing middleware that preserves Read Replica/Redis offloading for 99.9% of traffic while guaranteeing 100% consistency for the mutating user.

---

### Solution 3

1. **Write Path Emits Dual Consistency Proof**:
   - Inside the `PUT /v1/profile/address` transaction, increment `profile.version` (say to `12`) and fetch `SELECT pg_current_wal_lsn()::text AS lsn`.
   - Write a tiny Redis sticky-token `ryow:user:{userId}` (`JSON.stringify({ minVersion: 12, walLsn: '0/1A9F420' })`) with `EX 5` (5-second TTL), and return `{ ...updatedProfile, version: 12 }` in the `200 OK` HTTP body so the SPA hydrates local cache immediately.
2. **Read Path Evaluates Token Before Fallback**:
   - On `GET /v1/profile`, check `GET ryow:user:{userId}` in Redis (`< 0.5 ms`):
     - If **null** (99.9% of traffic; user did not write within 5s): read from the cached Redis read view (falling back on miss to the Read Replica).
     - If **present** (`{ minVersion: 12, walLsn }`): inspect the Redis read view's embedded `version`. If `cachedView.version >= 12`, serve directly from Redis and delete the sticky token! If `cachedView.version < 12`, route the query to the Read Replica only if `pg_last_wal_replay_lsn() >= walLsn::pg_lsn`; otherwise execute that single `SELECT` directly against the Primary Write DB.
3. **Why This Scales**: Read offloading remains intact across the entire user base; only the exact mutating user within the 300ms propagation window touches Primary.

---

### Exercise 4: Resolving Cross-Aggregate Race Conditions in a Denormalized Projector
Your `order_search_read_model` table displays `customer_email` alongside `order_status` and `total_cents`. `OrderService` publishes `OrderConfirmed` (`partitionKey = orderId`), and `CustomerService` publishes `CustomerEmailUpdated` (`partitionKey = customerId`) on separate topics. During a spike, `CustomerEmailUpdated` (changing `a@old.com` -> `b@new.com`, `customerVersion: 5`) arrives at the projector *50ms before* `OrderConfirmed` (which carried `a@old.com` captured when the order was created, `customerVersion: 4`). How does a naive projector permanently store the stale email, and how do you fix it deterministically?

---

### Solution 4

#### Why the Naive Projector Corrupts State
1. `CustomerEmailUpdated` runs `UPDATE order_search_read_model SET customer_email = 'b@new.com' WHERE customer_id = $1`, affecting `0` rows because the new order row has not been inserted yet.
2. 50ms later, `OrderConfirmed` arrives carrying the older `customerEmail: 'a@old.com'` from order checkout time and inserts `'a@old.com'`, permanently stranding stale email data!

#### Deterministic Two-Table Join-on-Project Solution
1. Maintain a small local lookup table inside the Read DB:
   `customer_read_cache (customer_id UUID PK, email TEXT, customer_version INT)` updated exclusively via monotonic `WHERE customer_read_cache.customer_version < EXCLUDED.customer_version`.
2. When `CustomerEmailUpdated` runs:
   - Inside one transaction, upsert `customer_read_cache` (persisting `'b@new.com'`, `customer_version = 5` even if zero orders exist yet!) **and** update any existing rows in `order_search_read_model WHERE customer_id = $1 AND customer_version < 5`.
3. When `OrderConfirmed` runs:
   - Include `customerVersion` (`4`) on `OrderConfirmed`, upsert `customer_read_cache` **only if `customerVersion > stored`**, then read the authoritative `email` and `customer_version` from `customer_read_cache` to insert into `order_search_read_model`!
   - Even when `OrderConfirmed` (`customerVersion: 4`) arrives second, `customer_read_cache` already holds `customer_version = 5` (`'b@new.com'`), so the inserted order row receives `'b@new.com'` immediately.

---

### Exercise 5: Zero-Downtime Blue/Green Schema Migration of a 40M-Row Read Model
Product requests adding a weighted `priority_score` integer column and changing the OpenSearch analyzer on `order_search_read_model` (40M rows in PostgreSQL, 40M documents in OpenSearch `orders_search_v1`) while writes arrive at `800 events/sec`. A junior engineer proposes pausing the projector, running `ALTER TABLE order_search_read_model ADD COLUMN ...` + `UPDATE` across 40M rows, and updating the OpenSearch mapping in place. Explain why both actions cause a production outage and design the exact Blue/Green cutover procedure.

---

### Solution 5

#### Why In-Place Mutation Fails
1. **PostgreSQL `UPDATE` over 40M rows**: Rewrites every heap tuple, generates tens of GBs of WAL, locks rows blocking the live projector, bloats dead tuples by 100%, and triggers vacuum I/O throttling—stalling read queries (`p99 > 5s`).
2. **OpenSearch Analyzer Change**: OpenSearch **cannot** change tokenizers/analyzers on existing fields in place; changing mapping compatibility forces creating a brand-new index anyway.

#### Zero-Downtime Blue/Green Migration Plan
1. **Provision `v2` Targets**: Create `order_search_read_model_v2` (including `priority_score` and indexes created *after* bulk backfill for speed, or maintained concurrently) and OpenSearch index `orders_search_v2` with the updated mapping/analyzer and `refresh_interval: -1` during initial load.
2. **Start Live Shadow Consumer Early**: Record the current max `seq_id` (or Kafka head offset `H_start`), start shadow consumer `orders-projector-v2` writing live events from `H_start` forward into `v2` immediately.
3. **Chunked Historical Backfill**: Run a throttled keyset backfill job (`5,000` rows per batch from `seq_id = 1` up to `H_start`) writing into `v2` using monotonic guards (`WHERE last_seq < EXCLUDED.last_seq`). Live tail events written at `>= H_start` cannot be overwritten by older historical rows!
4. **Verify Parity & Restore OpenSearch Refresh**: Once historical backfill finishes and `v2` consumer lag drops below `50 ms`, set `orders_search_v2` `refresh_interval = 1s`, run a count/checksum spot check (`SELECT COUNT(*), SUM(total_cents)` between `v1` and `v2`).
5. **Atomic Swap**: Execute `CREATE OR REPLACE VIEW order_search_active_v AS SELECT * FROM order_search_read_model_v2;` in PostgreSQL and `POST /_aliases` swapping `orders_search_read` from `v1` to `v2` atomically. Keep `v1` warm for 24h for instant rollback, then drop `v1`.

---

### Exercise 6: Fixing a Missing Command Idempotency & Dual-Write Illusion
A team adopts Level 2 CQRS: their `POST /orders/:id/confirm` endpoint updates `orders`, directly calls `await esClient.index(...)` synchronously inside the HTTP handler, and omits `idempotency_key`. Under mobile network retries, `POST` hits two backend pods within 40ms. One pod returns `504` after OpenSearch times out, yet Postgres committed; the retry hits `409 Conflict` (`Order already CONFIRMED`) causing the UI to display an error while the search index still holds `DRAFT`. Diagnose the architectural violations and specify the production fix.

---

### Solution 6

#### Architectural Violations
1. **Synchronous Dual-Write Inside Command Handler**: Calling `esClient.index(...)` directly from the HTTP command handler mixes the Write ACID boundary with an external read-store RPC. When OpenSearch stalls or times out, Postgres has already committed (`CONFIRMED`), leaving the read model permanently inconsistent (`DRAFT`) and causing HTTP `504` failures on valid commands.
2. **Missing Command Idempotency**: Without `command_idempotency (tenant_id, idempotency_key)`, the mobile client's automatic retry hits `Order.confirm()` when `status === 'CONFIRMED'` (or `version !== expectedVersion`), throwing `409 Conflict` even though the user's exact request already succeeded!

#### Production Fix
1. Remove `esClient.index(...)` completely from `ConfirmOrderHandler`; write an `OrderConfirmed` row to `outbox_events` inside the exact same Postgres `BEGIN...COMMIT` transaction as `UPDATE orders`. Let `OrderReadProjector` update OpenSearch asynchronously with retries and DLQ protection.
2. Require an `Idempotency-Key` UUID generated per user click by the client. Store `(tenant_id, idempotency_key, response_json)` inside the `ConfirmOrderHandler` ACID transaction (`ON CONFLICT DO NOTHING` / pre-check). When the mobile retry arrives, the second pod sees the persisted `command_idempotency` entry and cleanly replays the original `200 OK` response (`{ orderId, status: 'CONFIRMED', version: 2, walLsn }`) without throwing `409`.
