---
tags:
  - software-engineering
  - architectural-patterns
  - event-driven-architecture
  - sagas
  - outbox
created: 2026-01-02
updated: 2026-11-13
status: 🟢
level: mid-to-senior
---
# Event-Driven Architecture (EDA)

> [!abstract] Hub Context
> Part of [[Programming/Software Engineering/Architectural Patterns/Architectural Patterns Complete Guide|Architectural Patterns Complete Guide]]. Directly underpins [[Programming/Software Engineering/Architectural Patterns/CQRS|CQRS]], [[Programming/Software Engineering/Architectural Patterns/Event Sourcing|Event Sourcing]], [[Programming/Software Engineering/Architectural Patterns/Microservices|Microservices]], [[Programming/Software Engineering/System Design/Message Queues|Message Queues]], and [[Programming/Software Engineering/DDD/Strategic Design|Strategic Design]].

**Event-Driven Architecture (EDA)** coordinates decoupled services through immutable, asynchronously transmitted records of completed facts (**events**) and explicit asynchronous workflow actions (**commands**) transported over durable message queues or partitioned append logs. Replacing synchronous request/response chains with asynchronous event streams isolates downstream availability faults, buffers traffic bursts, and allows independent read/write scaling—at the engineering cost of eventual consistency, duplicate delivery, out-of-order arrivals, schema compatibility governance, and multi-step distributed failure compensation.

---

## 1. First-Principles Mechanics & Complete Architecture

### Synchronous RPC Chains vs Temporal Asynchronous Decoupling
When Service A calls Service B synchronously over HTTP/gRPC (`POST /checkout` -> `Inventory.reserve` -> `Payment.charge` -> `Shipping.label` -> `Email.send`), end-to-end availability compounds multiplicatively: five sequential dependencies at $99.9\%$ uptime yield $0.999^5 = 99.5\%$ effective availability, while tail latency accumulates additively and thread pools block waiting on downstream sockets.

EDA breaks **temporal coupling**: the producer commits its local state transition alongside a durable message, immediately returning to the caller. Downstream consumers read and react at their own pace without sharing runtime liveness with the producer.

### Event Taxonomy: Notification vs ECST vs Domain Events vs Event Sourcing

Misclassifying event semantics causes either synchronous callback storms under burst traffic or brittle schema coupling across bounded contexts.

| Pattern | Wire Payload Shape | Coupling & Runtime Behavior | Optimal Engineering Fit |
|---|---|---|---|
| **Event Notification** | Minimal pointer only (`{ eventId, orderId, occurredAt }`) | Producer schema stays ultra-thin; consumers **must call back synchronously** via HTTP/gRPC to fetch details—creating N+1 callback thundering herds during traffic spikes | Lightweight cache invalidations or strict PII-restricted workflows requiring fresh AuthZ checks on pull |
| **Event-Carried State Transfer (ECST)** | Full snapshot of relevant changed attributes (`{ orderId, customerId, items, totalCents, shippingAddr }`) | **Eliminates synchronous callbacks**: consumers maintain self-sufficient local read replicas and keep working even while the producer is offline; larger payloads require schema governance | Replicating [[Programming/Software Engineering/Architectural Patterns/CQRS\|CQRS]] projections, search indexing, ML feature stores, autonomous downstream services |
| **Domain Event** | Past-tense business fact inside a Bounded Context (`OrderConfirmed`, `PaymentDeclined`, `StockDepleted`) | Models explicit domain causality (`correlationId`, `causationId`); internal events stay private, whereas curated **Integration Events** cross context boundaries | Cross-context business reactions and state transitions |
| **Event Sourcing** | Intra-service write persistence mechanism (append-only event store replacing mutable DB rows) | Strictly **not** equivalent to EDA: a service can publish EDA events from mutable PostgreSQL tables via Outbox without using Event Sourcing, or run internal Event Sourcing without cross-service EDA | Financial ledgers and full-history temporal aggregates (see [[Programming/Software Engineering/Architectural Patterns/Event Sourcing\|Event Sourcing]]) |

### Point-to-Point Queues vs Partitioned Pub/Sub Append Logs

| Broker Architecture | Examples | Delivery & Consumption Model | Ordering & Replay Characteristics |
|---|---|---|---|
| **Point-to-Point Queue** | Amazon SQS, RabbitMQ Classic Queues, BullMQ | Competing workers pull messages; message becomes temporarily invisible (`VisibilityTimeout`), then is **physically deleted** upon `ACK` | No historical replay once `ACK`ed; global strict FIFO destroys throughput (unless sharded via SQS `MessageGroupId`) |
| **Partitioned Append Log** | Apache Kafka, Redpanda, NATS JetStream | Producers append sequentially to immutable partitioned topic segments; multiple independent **Consumer Groups** track separate **offset pointers** | Strict FIFO ordering **within a single partition** via partition key (`aggregateId`); retains logs for days/forever allowing independent full consumer replay |

#### Kafka Partition Keys & Consumer Group Rebalancing
- **Partition Key Invariant**: Setting the Kafka record key to the **Aggregate ID** (`orderId` or `accountId`) routes every lifecycle event (`OrderCreated` -> `OrderPaid` -> `OrderCancelled`) to `hash(orderId) % partitionCount`, guaranteeing strict causal order for that entity without serializing unrelated orders.
- **Consumer Group Rebalancing**: When pods autoscale or crash (`session.timeout.ms` / `max.poll.interval.ms` exceeded), Kafka reassigns partitions across active consumers. Classic eager protocols (`RangeAssignor`) execute a stop-the-world pause across all consumers; modern **`CooperativeStickyAssignor`** revokes only migrating partitions incrementally, avoiding global lag spikes. Because offset commits occur periodically *after* processing, every consumer **must tolerate redelivery** of uncommitted offsets after a rebalance.

### The Dual-Write Bug & Exact Atomic Cures

Executing a database write followed by a broker publish inside application code is fundamentally unsafe:

```typescript
// BROKEN DUAL-WRITE: Never deploy in production!
await db.query('UPDATE orders SET status = $1 WHERE id = $2', ['CONFIRMED', orderId]);
await kafkaProducer.send({ topic: 'orders', messages: [{ key: orderId, value: json }] });
```

1. **DB Commit + Broker Failure (Silent Split-Brain)**: If the process crashes, gets `SIGKILL`ed during rollout, or Kafka times out *after* `COMMIT`, the order is `CONFIRMED` in PostgreSQL while downstream inventory and billing never receive the event.
2. **Broker First + DB Rollback (Phantom Event)**: Reversing the calls publishes `OrderConfirmed` for an order whose transaction subsequently rolls back on a unique constraint or deadlock, charging customers for non-existent orders.
3. **Why Distributed 2PC / XA Fails**: Two-Phase Commit across PostgreSQL and Kafka/SQS is largely unsupported by modern brokers and holds coordinator locks across network hops, collapsing throughput under tail latency spikes.

#### Cure 1: Transactional Outbox + Polling Relay (`FOR UPDATE SKIP LOCKED`)
Write business rows (`orders`) and serialized event rows (`outbox_events`) inside the **exact same single ACID database transaction**:

```mermaid
flowchart LR
    Svc["Checkout Service"] -->|"Single ACID TX"| DB[("orders + outbox_events")]
    DB -->|"SKIP LOCKED or WAL CDC"| Relay["Outbox Relay"]
    Relay -->|"acks=all, key=orderId"| Broker["Kafka / Broker"]
    Broker -->|"At-Least-Once"| Inbox["Idempotent Inbox Consumer"]
```

- Multiple relay worker instances poll concurrently using `SELECT ... FROM outbox_events WHERE published_at IS NULL ORDER BY seq_id ASC LIMIT 100 FOR UPDATE SKIP LOCKED`, ensuring each worker locks and publishes disjoint batches without blocking peer workers, then sets `published_at = NOW()` using idempotent Kafka producers (`acks=all`, `enable.idempotence=true`).

#### Cure 2: Transactional Outbox + PostgreSQL WAL CDC (Debezium)
Instead of polling tables with SQL queries, configure **Debezium PostgreSQL Connector** to tail the Write-Ahead Log (`pgoutput` logical replication slot) on `outbox_events`.
- **Advantages over Polling**: Sub-50ms end-to-end propagation latency, zero `SELECT` polling pressure on primary DB indexes, and automatic transaction-commit ordering directly from the WAL stream.
- **Why Still Use an Outbox Table with CDC**: Tailing raw business tables (`orders`, `order_lines`) leaks internal normalized database schemas as external APIs and triggers multi-table joins inside CDC streams; writing formatted Integration Events into `outbox_events` and applying Debezium's `EventRouter` Single Message Transform preserves strict Bounded Context encapsulation.

### Idempotent Consumers & The Inbox Pattern

Network retries, outbox relay crashes between `kafka.send()` and `UPDATE outbox_events SET published_at`, and consumer group rebalances guarantee **At-Least-Once** delivery. End-to-end **Effectively-Once Processing** requires **At-Least-Once Delivery + Strictly Idempotent Consumers**.

Non-idempotent operations (`UPDATE inventory SET stock = stock - 2`) corrupt state on duplicate delivery. **The Inbox Deduplication Pattern** solves this atomically inside the consumer's local database transaction:
1. Begin transaction in the Consumer DB.
2. Execute:
   ```sql
   INSERT INTO processed_events (event_id, consumer_name)
   VALUES ($1, $2)
   ON CONFLICT (event_id, consumer_name) DO NOTHING;
   ```
3. Inspect `rowCount`:
   - If **`rowCount === 0`**: This exact `event_id` already committed previously. Commit/rollback immediately (no-op) and acknowledge the broker offset!
   - If **`rowCount === 1`**: First valid execution. Apply business mutations (`UPDATE inventory_stock ...`) inside the **exact same ACID transaction** and `COMMIT` before acknowledging the broker offset.

### Distributed Workflow Coordination: Choreography vs Orchestration Sagas

Long-running business workflows spanning multiple service databases cannot hold distributed ACID locks across services. A **Saga** decomposes a global transaction into a sequence of local ACID transactions $T_1, T_2, \dots, T_n$. If step $T_k$ fails due to a business invariant (declined card, depleted stock), the Saga executes explicit **Compensating Transactions** $C_{k-1}, \dots, C_1$—new forward business actions (`ReleaseInventoryReservation`, `IssueRefund`) that semantically undo prior committed steps.

#### Classifying Saga Transactions
Every workflow step falls into exactly one category, dictating execution order:
1. **Compensable Transactions**: Can be semantically undone by a counter-operation ($T_1$: `ReserveInventory` $\leftrightarrow$ $C_1$: `ReleaseInventory`).
2. **Pivot Transaction**: The irreversible point of no return ($T_p$: `IssueNonRefundableTicket` or `CapturePayment`). Once $T_p$ commits, backward compensation stops; the workflow must move forward.
3. **Retriable Transactions**: Steps executed **after** the Pivot that are guaranteed to succeed eventually via bounded idempotent retries ($T_r$: `MarkOrderConfirmed`, `DispatchReceiptEmail`).

#### Choreography vs Orchestration

| Dimension | Choreography (Event-Reactive) | Orchestration (Central State Machine) |
|---|---|---|
| **Control Topology** | Decentralized: `OrderService` emits `OrderCreated`; `Inventory` reacts and emits `StockReserved`; `Payment` reacts and emits `PaymentCaptured` | Central **Orchestrator** (`OrderCheckoutSagaOrchestrator`, Temporal, Step Functions) persists workflow state, dispatches explicit Commands, and consumes reply events |
| **Dependency Direction** | Producers know zero subscribers; understanding full flow requires tracing 5 repos | Orchestrator knows exact step DAG; domain services remain simple command handlers |
| **Failure & Timeouts** | Detecting a silent workflow stall after step 2 requires custom timeout monitors across services | Native per-step timeouts, explicit `COMPENSATING` state transitions, dead-man timers |
| **Circular Loop Risk** | High in complex graphs (`Service A -> Service B -> Service A`) | Zero cyclic confusion (explicit state transitions) |
| **Engineering Sweet Spot** | Simple **2–3 step** linear/fan-out notifications (`OrderPaid` -> Email + Analytics) | **4+ step** financial/booking workflows requiring strict compensation ordering & timeouts |

```mermaid
stateDiagram-v2
    [*] --> PendingCheckout: Start Saga
    PendingCheckout --> InventoryReserved: ReserveInventory OK
    InventoryReserved --> Completed: CapturePayment OK - Pivot
    InventoryReserved --> Compensating: PaymentDeclined
    Compensating --> FailedCompensated: ReleaseInventory OK
    PendingCheckout --> FailedAborted: OutOfStock
```

#### Semantic Locks for Saga Isolation Anomalies
Because Saga steps commit locally without global ACID **Isolation**, concurrent user actions can observe partial intermediate state (Dirty Read) or overwrite an in-flight reservation (Lost Update). Mitigate via **Semantic Locks**:
- Transition the order immediately to an explicit pending state (`PENDING_CHECKOUT` -> `INVENTORY_RESERVED` -> `CONFIRMED` or `COMPENSATING`).
- Reject user mutations such as `EditOrderItems` or `CancelOrder` (`409 Conflict: Checkout in progress`) until the workflow leaves the semantic lock (`COMPLETED` or `FAILED_COMPENSATED`), or queue cancellation until compensation completes.

### Schema Evolution, Causal Ordering & Poison-Pill Isolation

#### Schema Compatibility (Avro / Protobuf / JSON Schema Registry)
Producers and consumers roll out independently (`v1` and `v2` coexist concurrently, while 7-day Kafka retention stores old payloads). Configure CI/CD checks against a Schema Registry:
- **`BACKWARD`** (Upgrade consumers first): New reader processes old messages. Safe: add optional fields with defaults, delete fields. Forbidden: add required fields without defaults, rename or retype fields.
- **`FORWARD`** (Upgrade producers first): Old reader ignores unknown new fields. Safe: add fields, delete optional fields.
- **`FULL`** (Both upgrade independently in any order): Strictly additive optional fields with defaults; never delete or rename fields in place. Use **Expand-and-Contract** (`add totalMinorUnits alongside amountCents` -> migrate all consumers -> remove old field only after retention expires).

#### Causal Ordering & Poison-Pill DLQ Pipelines
- **Causal Ordering**: Never rely on distributed wall-clock timestamps (`occurredAt`) for ordering across microservices due to NTP clock skew. Propagate **`correlationId`** (root workflow trace ID) and **`causationId`** (exact parent `eventId` / `commandId`) on every envelope and partition by `aggregateId`.
- **Poison-Pill Isolation & DLQ**: If a consumer throws an unhandled `SyntaxError` or schema validation failure and crashes repeatedly, infinite immediate retries freeze that Kafka partition (**head-of-line blocking**), starving healthy orders sharing the partition!
  - Classify errors explicitly:
    1. **`TransientError`** (DB deadlock, `503` downstream timeout): Route to bounded exponential retry queue/backoff (`1s`, `5s`, `30s`), escalating to DLQ only after `maxAttempts`.
    2. **`PoisonPillError`** (corrupt JSON, invalid schema, unrecoverable domain invariant violation): Route **immediately** to `dead_letter_events` (DLQ) with original payload, headers, partition/offset, and stack trace, then **commit the partition offset** so healthy messages proceed unimpeded.

---

## 2. Real-World Use Cases & Decision Matrix

### Integration Topology Decision Matrix

| Dimension | Sync HTTP / gRPC | Point-to-Point Queue | Choreography EDA | Orchestration Saga |
|---|---|---|---|---|
| **Latency Coupling** | Caller blocks until full chain completes | Caller returns immediately (`202`) | Caller returns immediately (`202`) | Caller returns immediately (`202`) |
| **Rollback Model** | Fail fast; no cross-service compensation if single remote call | Retry until ACK or DLQ | Decentralized compensating events | Centralized deterministic backward compensation |
| **Infra Complexity** | Low (Load Balancer + API) | Low–Moderate (SQS / BullMQ) | Moderate–High (Kafka + Schema Registry + Inbox) | High (Orchestrator DB/Temporal + Outbox + Inbox + DLQ) |
| **Ideal Workload** | Auth checks, synchronous read validation, UI queries | Single-consumer background jobs (PDFs, thumbnails) | Multi-subscriber fan-out (`OrderPaid -> Analytics + Tax + Email`) | Multi-service checkout, travel booking, account provisioning |

### Five Concrete Engineering Scenarios

1. **Multi-Service E-Commerce Checkout (`Order -> Inventory -> Payment -> Shipping`)**:
   - *Decision*: **Orchestration Saga + Transactional Outbox + Idempotent Inbox**. Clear Pivot (`CapturePayment`), deterministic backward compensation (`ReleaseInventory`), and strict semantic lock states (`PENDING_CHECKOUT`).
2. **Post-Checkout Domain Fan-Out (`OrderConfirmed` -> Loyalty Points + Fraud Scoring + Email + Search Index)**:
   - *Decision*: **Choreography over Kafka + ECST**. Four unrelated bounded contexts subscribe independently to one curated Integration Event without cluttering checkout orchestration code.
3. **Interactive Coupon Code Validation (`POST /cart/apply-coupon` needing instant discount total in 40ms)**:
   - *Decision*: **Reject EDA**. Synchronous request/response RPC/HTTP is strictly superior when the caller cannot proceed without an immediate validation result.
4. **Heavy Document OCR / Video Transcoding Jobs**:
   - *Decision*: **Point-to-Point Queue (SQS / BullMQ)** rather than a full Kafka Saga. Only one worker processes each job, historical log replay is unnecessary, and visibility timeouts + DLQ handle worker crashes cleanly.
5. **Cross-Service Analytics / Search Index Synchronization**:
   - *Decision*: **Debezium Outbox WAL CDC + ECST**. Zero dual-write risk and consumer independence even during search cluster restarts.

---

## 3. Step-by-Step Practical Application: Multi-Step Order Saga + Outbox + Inbox + DLQ Replay Engine

This complete, runnable TypeScript + PostgreSQL system implements a full **Order Checkout Orchestration Saga** with:
- Atomic **Transactional Outbox** (`outbox_events`) and concurrent **`FOR UPDATE SKIP LOCKED` Relay Worker**.
- **Idempotent Inbox Consumer Dispatcher** (`processed_events`) with `TransientError` vs `PoisonPillError` classification.
- Durable **Saga Orchestrator** (`saga_instances`) executing `ReserveInventory` -> `CapturePayment` (Pivot) -> `ConfirmOrder`, and backward compensation (`ReleaseInventory` + `FailOrder`) when payment declines.
- **Dead Letter Queue (DLQ) & Replay Service** (`dead_letter_events`) isolating malformed payloads without halting healthy partition events and supporting safe idempotent replay.
- Vitest integration suite proving zero duplicate relay publications across parallel workers, strict consumer idempotency, crash-safe compensation, and DLQ replay recovery.

### Project Directory Layout

```text
eda-saga-engine/
├── package.json
├── sql/
│   └── 001_eda_outbox_saga.sql
└── src/
    ├── eda/
    │   ├── contracts/
    │   │   └── EventEnvelope.ts
    │   ├── outbox/
    │   │   ├── OutboxPublisher.ts
    │   │   └── OutboxRelayWorker.ts
    │   ├── consumer/
    │   │   └── IdempotentInboxDispatcher.ts
    │   ├── saga/
    │   │   └── OrderCheckoutSagaOrchestrator.ts
    │   └── dlq/
    │       └── DlqReplayService.ts
    └── tests/
        └── eda.saga.integration.test.ts
```

### Step 1: PostgreSQL Schema for Outbox, Inbox, Saga State & DLQ (`sql/001_eda_outbox_saga.sql`)

```sql
-- sql/001_eda_outbox_saga.sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- 1. Domain tables (Orders, Inventory, Payments)
CREATE TABLE IF NOT EXISTS orders (
    id          UUID PRIMARY KEY,
    customer_id UUID NOT NULL,
    sku         TEXT NOT NULL,
    quantity    INTEGER NOT NULL CHECK (quantity > 0),
    total_cents BIGINT NOT NULL CHECK (total_cents > 0),
    status      TEXT NOT NULL CHECK (
        status IN ('PENDING_CHECKOUT', 'INVENTORY_RESERVED', 'CONFIRMED', 'COMPENSATING', 'FAILED_COMPENSATED')
    ),
    failure_reason TEXT NULL,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS inventory_stock (
    sku             TEXT PRIMARY KEY,
    available_qty   INTEGER NOT NULL CHECK (available_qty >= 0),
    reserved_qty    INTEGER NOT NULL DEFAULT 0 CHECK (reserved_qty >= 0)
);

CREATE TABLE IF NOT EXISTS inventory_reservations (
    order_id    UUID PRIMARY KEY,
    sku         TEXT NOT NULL REFERENCES inventory_stock(sku),
    quantity    INTEGER NOT NULL CHECK (quantity > 0),
    status      TEXT NOT NULL CHECK (status IN ('RESERVED', 'COMMITTED', 'RELEASED')),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS payments (
    order_id    UUID PRIMARY KEY,
    amount_cents BIGINT NOT NULL CHECK (amount_cents > 0),
    status      TEXT NOT NULL CHECK (status IN ('CAPTURED', 'DECLINED')),
    reason      TEXT NULL,
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- 2. Transactional Outbox (Dual-Write Cure)
CREATE TABLE IF NOT EXISTS outbox_events (
    seq_id          BIGSERIAL PRIMARY KEY,
    event_id        UUID NOT NULL UNIQUE,
    topic           TEXT NOT NULL,
    partition_key   TEXT NOT NULL,
    event_type      TEXT NOT NULL,
    schema_version  SMALLINT NOT NULL DEFAULT 1,
    payload         JSONB NOT NULL,
    attempt_count   INTEGER NOT NULL DEFAULT 0,
    next_retry_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_error      TEXT NULL,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    published_at    TIMESTAMPTZ NULL
);

CREATE INDEX IF NOT EXISTS idx_outbox_pending
    ON outbox_events (next_retry_at ASC, seq_id ASC)
    WHERE published_at IS NULL;

-- 3. Idempotent Consumer Inbox Table
CREATE TABLE IF NOT EXISTS processed_events (
    event_id        UUID NOT NULL,
    consumer_name   TEXT NOT NULL,
    processed_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    PRIMARY KEY (event_id, consumer_name)
);

-- 4. Durable Saga Orchestrator State Machine
CREATE TABLE IF NOT EXISTS saga_instances (
    saga_id         UUID PRIMARY KEY,
    order_id        UUID NOT NULL UNIQUE,
    current_step    TEXT NOT NULL CHECK (
        current_step IN (
            'STARTED',
            'AWAITING_INVENTORY',
            'AWAITING_PAYMENT',
            'COMPENSATING_INVENTORY',
            'COMPLETED',
            'FAILED_COMPENSATED'
        )
    ),
    state_data      JSONB NOT NULL,
    version         INTEGER NOT NULL DEFAULT 1,
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- 5. Dead Letter Queue (Poison-Pill Isolation & Replay Tracking)
CREATE TABLE IF NOT EXISTS dead_letter_events (
    dlq_id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    event_id        UUID NOT NULL,
    topic           TEXT NOT NULL,
    partition_key   TEXT NOT NULL,
    consumer_name   TEXT NOT NULL,
    raw_payload     JSONB NOT NULL,
    error_class     TEXT NOT NULL,
    error_message   TEXT NOT NULL,
    status          TEXT NOT NULL DEFAULT 'UNRESOLVED' CHECK (status IN ('UNRESOLVED', 'REPLAYED')),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    replayed_at     TIMESTAMPTZ NULL
);
```

### Step 2: Expand-and-Contract Event Envelope & Runtime Validation (`src/eda/contracts/EventEnvelope.ts`)

```typescript
// src/eda/contracts/EventEnvelope.ts
import { randomUUID } from 'node:crypto';
import { z } from 'zod';

export const EventEnvelopeSchema = z.object({
  eventId: z.string().uuid(),
  eventType: z.string().min(1),
  schemaVersion: z.number().int().positive(),
  aggregateId: z.string().min(1),
  correlationId: z.string().uuid(),
  causationId: z.string().uuid(),
  occurredAt: z.string().datetime(),
  // Expand-and-Contract example: accepts legacy amountCents OR modern totalMinorUnits
  payload: z.record(z.unknown()),
});

export type EventEnvelope<P extends Record<string, unknown> = Record<string, unknown>> = {
  eventId: string;
  eventType: string;
  schemaVersion: number;
  aggregateId: string;
  correlationId: string;
  causationId: string;
  occurredAt: string;
  payload: P;
};

export function buildEnvelope<P extends Record<string, unknown>>(params: {
  eventId?: string;
  eventType: string;
  schemaVersion?: number;
  aggregateId: string;
  correlationId: string;
  causationId?: string;
  payload: P;
}): EventEnvelope<P> {
  const eventId = params.eventId ?? randomUUID();
  return {
    eventId,
    eventType: params.eventType,
    schemaVersion: params.schemaVersion ?? 1,
    aggregateId: params.aggregateId,
    correlationId: params.correlationId,
    causationId: params.causationId ?? eventId,
    occurredAt: new Date().toISOString(),
    payload: params.payload,
  };
}

// Expand-and-Contract helper normalizing legacy amountCents -> totalMinorUnits seamlessly
export function readMinorUnits(payload: Record<string, unknown>): number {
  if (typeof payload.totalMinorUnits === 'number') return payload.totalMinorUnits;
  if (typeof payload.amountCents === 'number') return payload.amountCents;
  throw new Error('Missing totalMinorUnits / amountCents monetary field');
}
```

### Step 3: Transactional Outbox Publisher & `SKIP LOCKED` Concurrent Relay (`src/eda/outbox/`)

```typescript
// src/eda/outbox/OutboxPublisher.ts
import { PoolClient } from 'pg';
import { EventEnvelope } from '../contracts/EventEnvelope.js';

export class OutboxPublisher {
  async enqueue(
    client: PoolClient,
    topic: string,
    envelope: EventEnvelope,
  ): Promise<void> {
    await client.query(
      `INSERT INTO outbox_events (
         event_id, topic, partition_key, event_type, schema_version, payload
       ) VALUES ($1, $2, $3, $4, $5, $6::jsonb)`,
      [
        envelope.eventId,
        topic,
        envelope.aggregateId, // Partition key = Aggregate ID for causal ordering
        envelope.eventType,
        envelope.schemaVersion,
        JSON.stringify(envelope),
      ],
    );
  }
}
```

```typescript
// src/eda/outbox/OutboxRelayWorker.ts
import { Pool } from 'pg';
import { EventEnvelope } from '../contracts/EventEnvelope.js';

export interface MessageBrokerPort {
  publish(topic: string, partitionKey: string, envelope: EventEnvelope): Promise<void>;
}

export class OutboxRelayWorker {
  constructor(
    private readonly pool: Pool,
    private readonly broker: MessageBrokerPort,
    private readonly maxAttempts = 5,
  ) {}

  /**
   * Claims an exclusive disjoint batch via FOR UPDATE SKIP LOCKED so N parallel relay pods
   * never duplicate-publish or contend on the same rows simultaneously.
   */
  async relayBatch(batchSize = 50): Promise<{ publishedCount: number; retriedCount: number }> {
    const client = await this.pool.connect();
    let publishedCount = 0;
    let retriedCount = 0;

    try {
      await client.query('BEGIN');

      const claimed = await client.query<{
        seq_id: string;
        event_id: string;
        topic: string;
        partition_key: string;
        payload: EventEnvelope;
        attempt_count: number;
      }>(
        `SELECT seq_id, event_id, topic, partition_key, payload, attempt_count
         FROM outbox_events
         WHERE published_at IS NULL AND next_retry_at <= NOW()
         ORDER BY seq_id ASC
         LIMIT $1
         FOR UPDATE SKIP LOCKED`,
        [batchSize],
      );

      for (const row of claimed.rows) {
        try {
          await this.broker.publish(row.topic, row.partition_key, row.payload);
          await client.query(
            `UPDATE outbox_events SET published_at = NOW(), last_error = NULL WHERE seq_id = $1`,
            [row.seq_id],
          );
          publishedCount++;
        } catch (err: unknown) {
          const nextAttempt = row.attempt_count + 1;
          const message = err instanceof Error ? err.message : String(err);

          if (nextAttempt >= this.maxAttempts) {
            await client.query(
              `INSERT INTO dead_letter_events (
                 event_id, topic, partition_key, consumer_name, raw_payload, error_class, error_message
               ) VALUES ($1, $2, $3, 'outbox-relay', $4::jsonb, 'OutboxPublishExhausted', $5)`,
              [row.event_id, row.topic, row.partition_key, JSON.stringify(row.payload), message],
            );
            await client.query(
              `UPDATE outbox_events SET published_at = NOW(), attempt_count = $2, last_error = $3 WHERE seq_id = $1`,
              [row.seq_id, nextAttempt, message],
            );
          } else {
            const backoffSeconds = Math.min(60, 2 ** nextAttempt);
            await client.query(
              `UPDATE outbox_events
               SET attempt_count = $2,
                   next_retry_at = NOW() + ($3 || ' seconds')::interval,
                   last_error = $4
               WHERE seq_id = $1`,
              [row.seq_id, nextAttempt, backoffSeconds, message],
            );
            retriedCount++;
          }
        }
      }

      await client.query('COMMIT');
      return { publishedCount, retriedCount };
    } catch (err) {
      await client.query('ROLLBACK');
      throw err;
    } finally {
      client.release();
    }
  }
}
```

### Step 4: Idempotent Inbox Consumer & Poison-Pill DLQ Isolator (`src/eda/consumer/IdempotentInboxDispatcher.ts`)

```typescript
// src/eda/consumer/IdempotentInboxDispatcher.ts
import { Pool, PoolClient } from 'pg';
import { EventEnvelope, EventEnvelopeSchema } from '../contracts/EventEnvelope.js';

export class TransientConsumerError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'TransientConsumerError';
  }
}

export class PoisonPillError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'PoisonPillError';
  }
}

export type DispatchOutcome = 'PROCESSED' | 'DEDUPLICATED' | 'ROUTED_TO_DLQ';

export class IdempotentInboxDispatcher {
  constructor(private readonly pool: Pool) {}

  async dispatch(
    consumerName: string,
    topic: string,
    partitionKey: string,
    rawMessage: unknown,
    handler: (client: PoolClient, envelope: EventEnvelope) => Promise<void>,
  ): Promise<DispatchOutcome> {
    // 1. Validate envelope contract before starting business work (malformed JSON -> PoisonPill)
    const parsed = EventEnvelopeSchema.safeParse(rawMessage);
    if (!parsed.success) {
      await this.recordDeadLetter({
        eventId:
          typeof rawMessage === 'object' && rawMessage !== null && 'eventId' in rawMessage
            ? String((rawMessage as { eventId: unknown }).eventId)
            : '00000000-0000-0000-0000-000000000000',
        topic,
        partitionKey,
        consumerName,
        rawPayload: rawMessage,
        errorClass: 'PoisonPillSchemaError',
        errorMessage: parsed.error.message,
      });
      return 'ROUTED_TO_DLQ';
    }

    const envelope = parsed.data as EventEnvelope;
    const client = await this.pool.connect();

    try {
      await client.query('BEGIN');

      // 2. Atomic Inbox Deduplication inside the SAME local ACID transaction as the business mutation
      const inboxRes = await client.query(
        `INSERT INTO processed_events (event_id, consumer_name)
         VALUES ($1, $2)
         ON CONFLICT (event_id, consumer_name) DO NOTHING`,
        [envelope.eventId, consumerName],
      );

      if (inboxRes.rowCount === 0) {
        await client.query('COMMIT');
        return 'DEDUPLICATED';
      }

      // 3. Execute business handler inside the same transaction
      await handler(client, envelope);

      await client.query('COMMIT');
      return 'PROCESSED';
    } catch (err: unknown) {
      await client.query('ROLLBACK');

      // Transient failures bubble up to trigger bounded broker/worker retry without marking processed
      if (err instanceof TransientConsumerError) {
        throw err;
      }

      // Unrecoverable poison pills route cleanly to DLQ so partition offset advances!
      await this.recordDeadLetter({
        eventId: envelope.eventId,
        topic,
        partitionKey,
        consumerName,
        rawPayload: envelope,
        errorClass: err instanceof Error ? err.name : 'UnknownError',
        errorMessage: err instanceof Error ? err.message : String(err),
      });
      return 'ROUTED_TO_DLQ';
    } finally {
      client.release();
    }
  }

  private async recordDeadLetter(params: {
    eventId: string;
    topic: string;
    partitionKey: string;
    consumerName: string;
    rawPayload: unknown;
    errorClass: string;
    errorMessage: string;
  }): Promise<void> {
    await this.pool.query(
      `INSERT INTO dead_letter_events (
         event_id, topic, partition_key, consumer_name, raw_payload, error_class, error_message
       ) VALUES ($1, $2, $3, $4, $5::jsonb, $6, $7)`,
      [
        params.eventId,
        params.topic,
        params.partitionKey,
        params.consumerName,
        JSON.stringify(params.rawPayload ?? {}),
        params.errorClass,
        params.errorMessage,
      ],
    );
  }
}
```

### Step 5: Durable Order Checkout Saga Orchestrator & Compensation Handlers (`src/eda/saga/OrderCheckoutSagaOrchestrator.ts`)

```typescript
// src/eda/saga/OrderCheckoutSagaOrchestrator.ts
import { randomUUID } from 'node:crypto';
import { Pool, PoolClient } from 'pg';
import { buildEnvelope, EventEnvelope, readMinorUnits } from '../contracts/EventEnvelope.js';
import { PoisonPillError } from '../consumer/IdempotentInboxDispatcher.js';
import { OutboxPublisher } from '../outbox/OutboxPublisher.js';

export interface StartCheckoutInput {
  orderId: string;
  customerId: string;
  sku: string;
  quantity: number;
  totalMinorUnits: number;
}

export class OrderCheckoutSagaOrchestrator {
  constructor(
    private readonly pool: Pool,
    private readonly outbox: OutboxPublisher,
  ) {}

  /**
   * Step 0: Creates Order under a Semantic Lock ('PENDING_CHECKOUT'), persists Saga state,
   * and emits ReserveInventoryCommand atomically via Outbox.
   */
  async startCheckout(input: StartCheckoutInput): Promise<{ sagaId: string }> {
    const client = await this.pool.connect();
    const sagaId = randomUUID();
    const correlationId = randomUUID();

    try {
      await client.query('BEGIN');

      await client.query(
        `INSERT INTO orders (id, customer_id, sku, quantity, total_cents, status)
         VALUES ($1, $2, $3, $4, $5, 'PENDING_CHECKOUT')`,
        [input.orderId, input.customerId, input.sku, input.quantity, input.totalMinorUnits],
      );

      await client.query(
        `INSERT INTO saga_instances (saga_id, order_id, current_step, state_data)
         VALUES ($1, $2, 'AWAITING_INVENTORY', $3::jsonb)`,
        [sagaId, input.orderId, JSON.stringify({ ...input, correlationId })],
      );

      await this.outbox.enqueue(
        client,
        'inventory.commands',
        buildEnvelope({
          eventType: 'ReserveInventoryCommand',
          aggregateId: input.orderId,
          correlationId,
          payload: {
            sagaId,
            orderId: input.orderId,
            sku: input.sku,
            quantity: input.quantity,
            totalMinorUnits: input.totalMinorUnits,
          },
        }),
      );

      await client.query('COMMIT');
      return { sagaId };
    } catch (err) {
      await client.query('ROLLBACK');
      throw err;
    } finally {
      client.release();
    }
  }

  /**
   * Inventory Service consumer: handles ReserveInventoryCommand and ReleaseInventoryCommand.
   */
  async handleInventoryCommand(client: PoolClient, envelope: EventEnvelope): Promise<void> {
    if (envelope.eventType === 'ReserveInventoryCommand') {
      const { orderId, sku, quantity } = envelope.payload as {
        orderId: string;
        sku: string;
        quantity: number;
      };
      if (!sku || typeof quantity !== 'number' || quantity <= 0) {
        throw new PoisonPillError('Invalid ReserveInventoryCommand payload');
      }

      const res = await client.query(
        `UPDATE inventory_stock
         SET available_qty = available_qty - $2,
             reserved_qty  = reserved_qty + $2
         WHERE sku = $1 AND available_qty >= $2`,
        [sku, quantity],
      );

      if (res.rowCount === 1) {
        await client.query(
          `INSERT INTO inventory_reservations (order_id, sku, quantity, status)
           VALUES ($1, $2, $3, 'RESERVED')
           ON CONFLICT (order_id) DO NOTHING`,
          [orderId, sku, quantity],
        );
        await this.outbox.enqueue(
          client,
          'saga.replies',
          buildEnvelope({
            eventType: 'InventoryReserved',
            aggregateId: orderId,
            correlationId: envelope.correlationId,
            causationId: envelope.eventId,
            payload: { orderId, sku, quantity },
          }),
        );
      } else {
        await this.outbox.enqueue(
          client,
          'saga.replies',
          buildEnvelope({
            eventType: 'InventoryReservationFailed',
            aggregateId: orderId,
            correlationId: envelope.correlationId,
            causationId: envelope.eventId,
            payload: { orderId, sku, reason: 'INSUFFICIENT_STOCK' },
          }),
        );
      }
      return;
    }

    if (envelope.eventType === 'ReleaseInventoryCommand') {
      const { orderId } = envelope.payload as { orderId: string };
      const resv = await client.query<{ sku: string; quantity: number; status: string }>(
        `SELECT sku, quantity, status
         FROM inventory_reservations
         WHERE order_id = $1
         FOR UPDATE`,
        [orderId],
      );
      if (resv.rowCount && resv.rows[0].status === 'RESERVED') {
        const { sku, quantity } = resv.rows[0];
        await client.query(
          `UPDATE inventory_stock
           SET available_qty = available_qty + $2,
               reserved_qty  = reserved_qty - $2
           WHERE sku = $1`,
          [sku, quantity],
        );
        await client.query(
          `UPDATE inventory_reservations SET status = 'RELEASED', updated_at = NOW() WHERE order_id = $1`,
          [orderId],
        );
      }
      await this.outbox.enqueue(
        client,
        'saga.replies',
        buildEnvelope({
          eventType: 'InventoryReleased',
          aggregateId: orderId,
          correlationId: envelope.correlationId,
          causationId: envelope.eventId,
          payload: { orderId },
        }),
      );
    }
  }

  /**
   * Payment Service consumer: Pivot transaction (CAPTURE or DECLINE).
   */
  async handlePaymentCommand(
    client: PoolClient,
    envelope: EventEnvelope,
    simulateDecline: (orderId: string, amountCents: number) => boolean,
  ): Promise<void> {
    if (envelope.eventType !== 'CapturePaymentCommand') return;

    const orderId = String(envelope.payload.orderId);
    const amountCents = readMinorUnits(envelope.payload);
    const declined = simulateDecline(orderId, amountCents);

    if (!declined) {
      await client.query(
        `INSERT INTO payments (order_id, amount_cents, status)
         VALUES ($1, $2, 'CAPTURED')
         ON CONFLICT (order_id) DO NOTHING`,
        [orderId, amountCents],
      );
      await this.outbox.enqueue(
        client,
        'saga.replies',
        buildEnvelope({
          eventType: 'PaymentCaptured',
          aggregateId: orderId,
          correlationId: envelope.correlationId,
          causationId: envelope.eventId,
          payload: { orderId, amountCents },
        }),
      );
    } else {
      await client.query(
        `INSERT INTO payments (order_id, amount_cents, status, reason)
         VALUES ($1, $2, 'DECLINED', 'CARD_DECLINED')
         ON CONFLICT (order_id) DO NOTHING`,
        [orderId, amountCents],
      );
      await this.outbox.enqueue(
        client,
        'saga.replies',
        buildEnvelope({
          eventType: 'PaymentDeclined',
          aggregateId: orderId,
          correlationId: envelope.correlationId,
          causationId: envelope.eventId,
          payload: { orderId, reason: 'CARD_DECLINED' },
        }),
      );
    }
  }

  /**
   * Saga Orchestrator state machine reacting to replies & driving backward compensation.
   */
  async handleSagaReply(client: PoolClient, envelope: EventEnvelope): Promise<void> {
    const orderId = String(envelope.payload.orderId);
    const sagaRes = await client.query<{
      saga_id: string;
      current_step: string;
      state_data: StartCheckoutInput & { correlationId: string };
      version: number;
    }>(
      `SELECT saga_id, current_step, state_data, version
       FROM saga_instances
       WHERE order_id = $1
       FOR UPDATE`,
      [orderId],
    );
    if (sagaRes.rowCount === 0) return;
    const saga = sagaRes.rows[0];

    switch (envelope.eventType) {
      case 'InventoryReserved': {
        if (saga.current_step !== 'AWAITING_INVENTORY') return;
        await client.query(
          `UPDATE orders SET status = 'INVENTORY_RESERVED', updated_at = NOW() WHERE id = $1`,
          [orderId],
        );
        await client.query(
          `UPDATE saga_instances SET current_step = 'AWAITING_PAYMENT', version = version + 1, updated_at = NOW() WHERE saga_id = $1`,
          [saga.saga_id],
        );
        await this.outbox.enqueue(
          client,
          'payment.commands',
          buildEnvelope({
            eventType: 'CapturePaymentCommand',
            aggregateId: orderId,
            correlationId: envelope.correlationId,
            causationId: envelope.eventId,
            payload: {
              orderId,
              totalMinorUnits: saga.state_data.totalMinorUnits,
            },
          }),
        );
        break;
      }

      case 'InventoryReservationFailed': {
        if (saga.current_step !== 'AWAITING_INVENTORY') return;
        await client.query(
          `UPDATE orders SET status = 'FAILED_COMPENSATED', failure_reason = 'INSUFFICIENT_STOCK', updated_at = NOW() WHERE id = $1`,
          [orderId],
        );
        await client.query(
          `UPDATE saga_instances SET current_step = 'FAILED_COMPENSATED', version = version + 1, updated_at = NOW() WHERE saga_id = $1`,
          [saga.saga_id],
        );
        break;
      }

      case 'PaymentCaptured': {
        if (saga.current_step !== 'AWAITING_PAYMENT') return;
        // Crossed Pivot Transaction: finalize order & commit inventory reservation
        await client.query(
          `UPDATE inventory_reservations SET status = 'COMMITTED', updated_at = NOW() WHERE order_id = $1`,
          [orderId],
        );
        await client.query(
          `UPDATE orders SET status = 'CONFIRMED', updated_at = NOW() WHERE id = $1`,
          [orderId],
        );
        await client.query(
          `UPDATE saga_instances SET current_step = 'COMPLETED', version = version + 1, updated_at = NOW() WHERE saga_id = $1`,
          [saga.saga_id],
        );
        break;
      }

      case 'PaymentDeclined': {
        if (saga.current_step !== 'AWAITING_PAYMENT') return;
        // Enter Semantic Lock ('COMPENSATING') and execute backward recovery!
        await client.query(
          `UPDATE orders SET status = 'COMPENSATING', failure_reason = 'CARD_DECLINED', updated_at = NOW() WHERE id = $1`,
          [orderId],
        );
        await client.query(
          `UPDATE saga_instances SET current_step = 'COMPENSATING_INVENTORY', version = version + 1, updated_at = NOW() WHERE saga_id = $1`,
          [saga.saga_id],
        );
        await this.outbox.enqueue(
          client,
          'inventory.commands',
          buildEnvelope({
            eventType: 'ReleaseInventoryCommand',
            aggregateId: orderId,
            correlationId: envelope.correlationId,
            causationId: envelope.eventId,
            payload: { orderId },
          }),
        );
        break;
      }

      case 'InventoryReleased': {
        if (saga.current_step !== 'COMPENSATING_INVENTORY') return;
        await client.query(
          `UPDATE orders SET status = 'FAILED_COMPENSATED', updated_at = NOW() WHERE id = $1`,
          [orderId],
        );
        await client.query(
          `UPDATE saga_instances SET current_step = 'FAILED_COMPENSATED', version = version + 1, updated_at = NOW() WHERE saga_id = $1`,
          [saga.saga_id],
        );
        break;
      }
    }
  }
}
```

### Step 6: Dead Letter Queue Replay Service (`src/eda/dlq/DlqReplayService.ts`)

```typescript
// src/eda/dlq/DlqReplayService.ts
import { Pool } from 'pg';
import { EventEnvelope } from '../contracts/EventEnvelope.js';
import { MessageBrokerPort } from '../outbox/OutboxRelayWorker.js';

export class DlqReplayService {
  constructor(
    private readonly pool: Pool,
    private readonly broker: MessageBrokerPort,
  ) {}

  async replayById(
    dlqId: string,
    patchEnvelope?: (broken: Record<string, unknown>) => EventEnvelope,
  ): Promise<void> {
    const res = await this.pool.query<{
      dlq_id: string;
      topic: string;
      partition_key: string;
      raw_payload: Record<string, unknown>;
      status: string;
    }>(
      `SELECT dlq_id, topic, partition_key, raw_payload, status
       FROM dead_letter_events
       WHERE dlq_id = $1`,
      [dlqId],
    );

    if (res.rowCount === 0) {
      throw new Error(`DLQ record ${dlqId} not found`);
    }
    const row = res.rows[0];
    if (row.status === 'REPLAYED') {
      return; // Idempotent replay guard
    }

    const fixedEnvelope = patchEnvelope
      ? patchEnvelope(row.raw_payload)
      : (row.raw_payload as unknown as EventEnvelope);

    await this.broker.publish(row.topic, row.partition_key, fixedEnvelope);

    await this.pool.query(
      `UPDATE dead_letter_events
       SET status = 'REPLAYED', replayed_at = NOW()
       WHERE dlq_id = $1`,
      [dlqId],
    );
  }
}
```

### Step 7: Vitest Outbox, Idempotency, Crash-Safe Compensation & DLQ Tests (`src/tests/eda.saga.integration.test.ts`)

```typescript
// src/tests/eda.saga.integration.test.ts
import { afterAll, beforeAll, beforeEach, describe, expect, it } from 'vitest';
import { Pool } from 'pg';
import { randomUUID } from 'node:crypto';
import { buildEnvelope, EventEnvelope } from '../eda/contracts/EventEnvelope.js';
import { OutboxPublisher } from '../eda/outbox/OutboxPublisher.js';
import { MessageBrokerPort, OutboxRelayWorker } from '../eda/outbox/OutboxRelayWorker.js';
import { IdempotentInboxDispatcher } from '../eda/consumer/IdempotentInboxDispatcher.js';
import { OrderCheckoutSagaOrchestrator } from '../eda/saga/OrderCheckoutSagaOrchestrator.js';
import { DlqReplayService } from '../eda/dlq/DlqReplayService.js';

interface QueuedMsg {
  topic: string;
  partitionKey: string;
  envelope: EventEnvelope;
}

class InMemoryPartitionedBroker implements MessageBrokerPort {
  public readonly queue: QueuedMsg[] = [];
  public readonly publishedEventIds: string[] = [];

  async publish(topic: string, partitionKey: string, envelope: EventEnvelope): Promise<void> {
    this.queue.push({ topic, partitionKey, envelope });
    this.publishedEventIds.push(envelope.eventId);
  }
}

describe('EDA Outbox + Idempotent Inbox + Saga Compensation + DLQ', () => {
  let pool: Pool;
  let broker: InMemoryPartitionedBroker;
  let outbox: OutboxPublisher;
  let relay1: OutboxRelayWorker;
  let relay2: OutboxRelayWorker;
  let inbox: IdempotentInboxDispatcher;
  let orchestrator: OrderCheckoutSagaOrchestrator;

  beforeAll(async () => {
    pool = new Pool({
      connectionString: process.env.DATABASE_URL ?? 'postgres://postgres:postgres@localhost:5432/eda_test',
    });
    outbox = new OutboxPublisher();
  });

  afterAll(async () => {
    await pool.end();
  });

  beforeEach(async () => {
    broker = new InMemoryPartitionedBroker();
    relay1 = new OutboxRelayWorker(pool, broker);
    relay2 = new OutboxRelayWorker(pool, broker);
    inbox = new IdempotentInboxDispatcher(pool);
    orchestrator = new OrderCheckoutSagaOrchestrator(pool, outbox);

    await pool.query(
      'TRUNCATE dead_letter_events, saga_instances, processed_events, outbox_events, payments, inventory_reservations, inventory_stock, orders RESTART IDENTITY CASCADE',
    );
    await pool.query(`INSERT INTO inventory_stock (sku, available_qty, reserved_qty) VALUES ('SKU-100', 10, 0)`);
  });

  async function drainAll(shouldDeclinePayment: (orderId: string, amountCents: number) => boolean) {
    while (true) {
      const { publishedCount } = await relay1.relayBatch(50);
      if (broker.queue.length === 0 && publishedCount === 0) break;

      while (broker.queue.length > 0) {
        const msg = broker.queue.shift()!;
        if (msg.topic === 'inventory.commands') {
          await inbox.dispatch('inventory-svc', msg.topic, msg.partitionKey, msg.envelope, (c, e) =>
            orchestrator.handleInventoryCommand(c, e),
          );
        } else if (msg.topic === 'payment.commands') {
          await inbox.dispatch('payment-svc', msg.topic, msg.partitionKey, msg.envelope, (c, e) =>
            orchestrator.handlePaymentCommand(c, e, shouldDeclinePayment),
          );
        } else if (msg.topic === 'saga.replies') {
          await inbox.dispatch('saga-orchestrator', msg.topic, msg.partitionKey, msg.envelope, (c, e) =>
            orchestrator.handleSagaReply(c, e),
          );
        }
      }
    }
  }

  it('prevents duplicate broker publication across concurrent SKIP LOCKED relay workers', async () => {
    const orderId = randomUUID();
    await orchestrator.startCheckout({
      orderId,
      customerId: randomUUID(),
      sku: 'SKU-100',
      quantity: 3,
      totalMinorUnits: 9_000,
    });

    const [r1, r2] = await Promise.all([relay1.relayBatch(10), relay2.relayBatch(10)]);
    expect(r1.publishedCount + r2.publishedCount).toBe(1);
    expect(broker.publishedEventIds).toHaveLength(1);
  });

  it('guarantees strict consumer idempotency when identical ReserveInventoryCommand arrives 3 times', async () => {
    const orderId = randomUUID();
    const cmd = buildEnvelope({
      eventType: 'ReserveInventoryCommand',
      aggregateId: orderId,
      correlationId: randomUUID(),
      payload: { orderId, sku: 'SKU-100', quantity: 4, totalMinorUnits: 5_000 },
    });

    const d1 = await inbox.dispatch('inventory-svc', 'inventory.commands', orderId, cmd, (c, e) =>
      orchestrator.handleInventoryCommand(c, e),
    );
    const d2 = await inbox.dispatch('inventory-svc', 'inventory.commands', orderId, cmd, (c, e) =>
      orchestrator.handleInventoryCommand(c, e),
    );
    const d3 = await inbox.dispatch('inventory-svc', 'inventory.commands', orderId, cmd, (c, e) =>
      orchestrator.handleInventoryCommand(c, e),
    );

    expect(d1).toBe('PROCESSED');
    expect(d2).toBe('DEDUPLICATED');
    expect(d3).toBe('DEDUPLICATED');

    const stock = await pool.query('SELECT available_qty, reserved_qty FROM inventory_stock WHERE sku = $1', ['SKU-100']);
    expect(stock.rows[0].available_qty).toBe(6);
    expect(stock.rows[0].reserved_qty).toBe(4);
  });

  it('completes happy path AND compensates reserved inventory automatically when payment declines', async () => {
    const okOrderId = randomUUID();
    await orchestrator.startCheckout({
      orderId: okOrderId,
      customerId: randomUUID(),
      sku: 'SKU-100',
      quantity: 2,
      totalMinorUnits: 6_000,
    });
    await drainAll(() => false);

    const okOrder = await pool.query('SELECT status FROM orders WHERE id = $1', [okOrderId]);
    expect(okOrder.rows[0].status).toBe('CONFIRMED');

    // Declined payment scenario: reserves 3 units -> card declines -> compensates back to available!
    const failOrderId = randomUUID();
    await orchestrator.startCheckout({
      orderId: failOrderId,
      customerId: randomUUID(),
      sku: 'SKU-100',
      quantity: 3,
      totalMinorUnits: 15_000,
    });
    await drainAll(() => true);

    const failOrder = await pool.query('SELECT status, failure_reason FROM orders WHERE id = $1', [failOrderId]);
    expect(failOrder.rows[0].status).toBe('FAILED_COMPENSATED');
    expect(failOrder.rows[0].failure_reason).toBe('CARD_DECLINED');

    const stock = await pool.query('SELECT available_qty, reserved_qty FROM inventory_stock WHERE sku = $1', ['SKU-100']);
    expect(stock.rows[0].available_qty).toBe(8); // 10 - 2 (okOrder) - 3 + 3 (compensated)
    expect(stock.rows[0].reserved_qty).toBe(0);
  });

  it('isolates poison pills to DLQ without stalling partition and replays cleanly once patched', async () => {
    const badOrderId = randomUUID();
    const brokenPayload = {
      eventId: randomUUID(),
      eventType: 'ReserveInventoryCommand',
      schemaVersion: 1,
      aggregateId: badOrderId,
      correlationId: randomUUID(),
      causationId: randomUUID(),
      occurredAt: new Date().toISOString(),
      payload: { orderId: badOrderId, sku: 'SKU-100', quantity: -5 }, // Invalid negative qty -> PoisonPill
    };

    const outcome = await inbox.dispatch(
      'inventory-svc',
      'inventory.commands',
      badOrderId,
      brokenPayload,
      (c, e) => orchestrator.handleInventoryCommand(c, e),
    );
    expect(outcome).toBe('ROUTED_TO_DLQ');

    const dlqRow = await pool.query('SELECT dlq_id, status FROM dead_letter_events LIMIT 1');
    expect(dlqRow.rows[0].status).toBe('UNRESOLVED');

    const replayer = new DlqReplayService(pool, broker);
    await replayer.replayById(dlqRow.rows[0].dlq_id, (raw) => ({
      ...(raw as unknown as EventEnvelope),
      payload: { orderId: badOrderId, sku: 'SKU-100', quantity: 2 },
    }));

    const replayedMsg = broker.queue.shift()!;
    const replayOutcome = await inbox.dispatch(
      'inventory-svc',
      replayedMsg.topic,
      replayedMsg.partitionKey,
      replayedMsg.envelope,
      (c, e) => orchestrator.handleInventoryCommand(c, e),
    );
    expect(replayOutcome).toBe('PROCESSED');

    const stock = await pool.query('SELECT available_qty, reserved_qty FROM inventory_stock WHERE sku = $1', ['SKU-100']);
    expect(stock.rows[0].available_qty).toBe(8);
    expect(stock.rows[0].reserved_qty).toBe(2);
  });
});
```

---

## 4. Complete Construct Reference & Production Edge Cases

### Delivery & Broker Mechanics Reference

| Mechanism | Guarantee | Where It Breaks | Production Guard |
|---|---|---|---|
| **Outbox + `SKIP LOCKED`** | Atomic DB + Event enqueue | Missing partial index `WHERE published_at IS NULL` scans millions of published rows | Partition `outbox_events` daily or purge published rows + keep partial index small |
| **Debezium WAL CDC** | Sub-50ms WAL log tailing | Disconnected Debezium container leaves PostgreSQL replication slot pinning WAL segments until disk fills 100% | Monitor `pg_replication_slots.active` & `wal_status`, set `max_slot_wal_keep_size` |
| **Inbox (`processed_events`)** | Strictly idempotent local DB mutation | Unbounded table growth (`1B+` rows) slows primary index maintenance | Partition `processed_events` monthly by timestamp or retain bounded retention window (`> broker retention`) |
| **Kafka `acks=all` + `idempotence=true`** | Broker write durability + zero producer duplicates | Long consumer processing blocks `max.poll.interval.ms` triggering infinite rebalance storms | Keep consumer work async/bounded, tune `max.poll.records`, offload long tasks or pause partition |

### Subtle Distributed Production Traps

1. **The Zombie Outbox / Relay Crash Duplicate**:
   - *Trap*: Worker A publishes event `E1` to Kafka (`acks=all` succeeds), then pod OOMs 1ms before `UPDATE outbox_events SET published_at = NOW()` commits! When Postgres rolls back Worker A's claim lock, Worker B picks up `E1` and publishes it a second time.
   - *Reality*: **This is normal and expected under At-Least-Once outbox relays!** Consumer Inbox deduplication (`processed_events`) absorbs `E1`'s second arrival silently. Never assume an outbox eliminates duplicates—it eliminates **lost events**.
2. **Kafka Consumer Rebalance Storms (`max.poll.interval.ms`)**:
   - *Trap*: Consumer pulls a batch of `500` events (`max.poll.records = 500`). Each event calls a sluggish third-party tax API taking `1 second`, totaling `500s > max.poll.interval.ms (300s)`. Kafka kicks the consumer out of the group, hands the partition to Pod 2, Pod 2 pulls the exact same `500` events and times out too—creating an endless infinite loop of duplicate side effects!
   - *Fix*: Reduce `max.poll.records` (`20–50`), enforce strict HTTP timeouts (`3s`), move external HTTP calls behind a decoupled Outbox/Queue step, and monitor consumer rebalance rates.
3. **Outbox Table Autovacuum & Bloat Catastrophe**:
   - *Trap*: Marking millions of `outbox_events` rows `published_at = NOW()` (or `DELETE`) causes heavy dead-tuple bloat (`pg_stat_user_tables.n_dead_tup`), slowing `SKIP LOCKED` polls from `2ms` to `800ms`.
   - *Fix*: Partition `outbox_events` by day/week (`PARTITION BY RANGE (created_at)`) and drop expired partitions instantly via `DROP TABLE outbox_events_2026_10_01` (zero vacuum cost), or use Debezium CDC where rows can be deleted immediately or rotated cleanly via partitioned outbox tables.
4. **Circular Choreography Ping-Pong**:
   - *Trap*: `InventoryService` emits `StockAdjusted` -> `CatalogService` recalculates badge and emits `ProductUpdated` -> `InventoryService` consumes `ProductUpdated` and refreshes SKU cache emitting `StockAdjusted` -> infinite feedback storm amplifying cluster CPU to 100%.
   - *Fix*: Never emit events for no-op mutations (`if (!changed) return`), propagate `causationId` chains with hop-depth telemetry, and replace multi-hop circular reactions with an explicit Orchestrator.

---

## 5. Production Debugging, Profiling & Operations Playbook

| Symptom | Root Cause | Diagnostic Command / Query | Mitigation & Fix |
|---|---|---|---|
| **Kafka consumer lag climbs relentlessly on 1 partition while other 11 drain** | Uncaught **Poison Pill** crashing consumer before offset commit (head-of-line blocking) | `kafka-consumer-groups.sh --bootstrap-server $BROKER --describe --group inventory-svc` (inspect exact frozen `CURRENT-OFFSET` on partition `P`); inspect `dead_letter_events` | Wrap deserialization & domain validation in `PoisonPillError` -> route immediately to `dead_letter_events` and commit offset; fix payload/code and run `DlqReplayService`. |
| **PostgreSQL disk climbs 10GB/hour despite normal traffic** | Inactive Debezium logical replication slot blocking WAL recycling | `SELECT slot_name, active, pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)) AS retained_wal FROM pg_replication_slots;` | Restart Debezium connector or drop stale dev slot (`pg_drop_replication_slot(...)`); enforce `max_slot_wal_keep_size` in `postgresql.conf`. |
| **Orders stuck indefinitely in `INVENTORY_RESERVED` / `COMPENSATING`** | Downstream reply lost, worker down, or Saga stalled mid-flight | `SELECT saga_id, order_id, current_step, updated_at FROM saga_instances WHERE current_step NOT IN ('COMPLETED', 'FAILED_COMPENSATED') AND updated_at < NOW() - INTERVAL '2 minutes';` | Deploy a **Saga Timeout Sweeper** polling stale `saga_instances` (`> 2m`), re-emitting idempotent retry commands or escalating to `COMPENSATING` / manual ops alert. |
| **Outbox relay polling takes `> 500 ms` per tick** | Missing partial index or `outbox_events` bloat | `EXPLAIN (ANALYZE, BUFFERS) SELECT seq_id FROM outbox_events WHERE published_at IS NULL AND next_retry_at <= NOW() ORDER BY seq_id ASC LIMIT 50 FOR UPDATE SKIP LOCKED;` | Create partial index `WHERE published_at IS NULL`, partition table daily, and purge/drop old published partitions. |
| **Duplicate payments/emails reported during rolling deploys** | Consumer missing `processed_events` Inbox check or `max.poll.interval.ms` rebalance | `SELECT event_id, COUNT(*) FROM payments GROUP BY 1 HAVING COUNT(*) > 1;` and inspect Kafka consumer group rebalance logs | Enforce atomic `processed_events` insert in consumer TX; tune `max.poll.records` and `CooperativeStickyAssignor`. |

---

## 6. Worked Exercises

### Exercise 1: Diagnosing Dual-Write Split-Brain, Non-Idempotent Mutations & Poison Pills
Inspect this production Node/TypeScript checkout producer and Kafka inventory consumer. Identify **4 critical distributed-systems bugs** and outline the exact production replacement:

```typescript
async function placeOrder(order: Order) {
  await db.query(
    'INSERT INTO orders (id, total_cents, status) VALUES ($1, $2, $3)',
    [order.id, order.totalCents, 'PLACED'],
  );
  await kafkaProducer.send({
    topic: 'order-events',
    messages: [{ value: JSON.stringify({ eventType: 'OrderPlaced', orderId: order.id, qty: order.qty }) }],
  });
}

async function onOrderPlaced(msg: KafkaMessage) {
  const evt = JSON.parse(msg.value!.toString());
  await db.query('UPDATE inventory SET stock = stock - $1 WHERE sku = $2', [evt.qty, evt.sku]);
}
```

---

### Solution 1

#### 4 Critical Bugs
1. **Dual-Write Split-Brain**: `INSERT INTO orders` commits before `kafkaProducer.send`. If Kafka returns `LeaderNotAvailable` or the pod crashes between lines, Postgres contains `PLACED` while Inventory never receives `OrderPlaced`.
2. **Non-Idempotent Consumer & Missing `eventId`**: Neither `eventId` nor an `Inbox` table exists. Under Kafka rebalances or retries, `stock = stock - qty` executes twice, silently overselling inventory! (Also, `evt.sku` was omitted from the producer payload.)
3. **Missing Partition Key (`key: order.id`)**: Without a deterministic partition key, `OrderPlaced` and `OrderCancelled` land on arbitrary partitions and can be processed out of order.
4. **Uncaught Poison Pill (`JSON.parse`)**: A malformed message crashes the consumer repeatedly before committing the offset, halting the entire partition.

#### Production Replacement
- **Producer**: Persist `orders` + `outbox_events (event_id, topic, partition_key = order.id, payload)` inside **one ACID transaction**, published asynchronously via `SKIP LOCKED` relay or Debezium CDC.
- **Consumer**: Validate schema with Zod (routing invalid payloads to `dead_letter_events` and committing offset), open a Postgres transaction, `INSERT INTO processed_events (event_id, consumer_name) ON CONFLICT DO NOTHING`, and execute `UPDATE inventory SET stock = stock - $1 WHERE sku = $2 AND stock >= $1` **only when `rowCount === 1`**.

---

### Exercise 2: Ordering Compensable, Pivot & Retriable Transactions in a Travel Booking Saga
A travel checkout workflow integrates 3 external steps:
1. **Reserve Hotel Room** (supports `ReleaseHotelHold`, 2% failure rate).
2. **Issue Airline Ticket** (**non-refundable** carrier API—cannot be cancelled automatically once issued!).
3. **Credit Card Authorization & Capture** (supports `PlaceAuthHold`, `VoidAuth`, and `CaptureAuth`; 8% decline rate).

1. A developer implements: `1. Issue Airline Ticket -> 2. Reserve Hotel -> 3. Charge Credit Card`. Prove why this loses money on ~10% of failed checkouts.
2. Reorder the workflow explicitly into **Compensable**, **Pivot**, and **Retriable** steps, detailing failure handling at every step.

---

### Solution 2

#### 1. Why the Original Ordering Causes Unrecoverable Losses
`Issue Airline Ticket` is **non-compensable**. Placing it before `Reserve Hotel` (2% fail) and `Charge Credit Card` (8% decline) guarantees that whenever step 2 or step 3 fails, the platform has already purchased an un-refundable flight ticket it cannot void.

#### 2. Correct Saga Ordering (Compensable -> Pivot -> Retriable)
1. **Step 1 (Compensable)**: `ReserveHotelHold` (5-minute hold; compensation: `ReleaseHotelHold`).
2. **Step 2 (Compensable)**: `PlaceCardAuthHold` (reserves funds without settling; compensation: `VoidCardAuth`).
3. **Step 3 (Pivot Transaction)**: `IssueAirlineTicket`. If the airline rejects, execute `VoidCardAuth` and `ReleaseHotelHold`—zero lost tickets and zero settled charges.
4. **Step 4 (Retriable Post-Pivot)**: Once `IssueAirlineTicket` succeeds (crossed the Pivot), execute `ConfirmHotelReservation` and `CaptureCardAuth` using bounded idempotent retries (never auto-compensating the confirmed airline ticket; escalate exhausted capture retries to Billing Operations).

---

### Exercise 3: Executing a Zero-Downtime Expand-and-Contract Event Schema Migration
Your `billing-service` publishes `InvoiceGenerated v1` to Kafka under `FULL` compatibility:
```json
{ "invoiceId": "inv_1", "amountCents": 4500, "customerId": "cust_9" }
```
Product mandates multi-currency support (`currency`) and renaming `amountCents` -> `totalMinorUnits` across 4 downstream teams (`ledger`, `tax`, `webhooks`, `analytics`). Why does deploying a PR renaming `amountCents` -> `totalMinorUnits` break production, and what is the zero-downtime Expand-and-Contract sequence?

---

### Solution 3

#### Why In-Place Renaming Breaks Production
1. Downstream consumers deploy independently and cannot cut over atomically alongside `billing-service`.
2. Kafka retains 7 days of historical `v1` events containing `"amountCents"`. Any consumer or replay expecting strictly `"totalMinorUnits"` throws `NaN` or validation errors when reading historical offsets, violating `FULL` schema compatibility checks in Schema Registry.

#### Expand-and-Contract Migration Sequence
1. **Phase 1 — Expand (Additive Schema `v1.1`)**: Update schema with optional additive fields `totalMinorUnits?: number` and `currency: string = "USD"`. Producer emits **both** `amountCents` and `totalMinorUnits` identically (`4500`) alongside `currency`. Schema Registry `FULL` validation passes cleanly!
2. **Phase 2 — Migrate Consumers Independently**: Update `ledger`, `tax`, `webhooks`, and `analytics` asynchronously to read `readMinorUnits(payload)` (`payload.totalMinorUnits ?? payload.amountCents`) and `payload.currency ?? "USD"`. Confirm via schema telemetry that zero active consumers read `amountCents`.
3. **Phase 3 — Contract (Post-Retention Cutoff)**: Once Kafka's retention window (`7 days`) purges all `v1` records lacking `totalMinorUnits` (or when cutting a clean `invoices.v2` major topic), stop emitting `amountCents`.

---

### Exercise 4: Breaking a Kafka Rebalance Storm & Partition Head-of-Line Block
A payment webhook consumer group (`12` partitions, `12` consumer pods) enters an infinite loop: consumer lag on Partition 4 freezes at offset `891,204`, the exact same 30 messages are processed and billed every 5 minutes, and Pod 4 continually triggers `CommitFailedException: rebalance in progress`. Explain the dual root cause and write the exact configuration + code fix.

---

### Solution 4

#### Dual Root Cause
1. **`max.poll.interval.ms` Breach**: `max.poll.records` (default `500`) fetched 500 webhook events in one poll. Slow downstream partner HTTP calls caused batch processing to exceed `max.poll.interval.ms` (`300s`), prompting the Kafka coordinator to evict Pod 4, rebalance Partition 4 to Pod 7, and redeliver uncommitted offsets starting at `891,204`.
2. **Partition Head-of-Line Blocking**: Offset `891,204` likely contains either a poison-pill payload or an un-bounded retry blocking commit progress for all subsequent messages in Partition 4.

#### Exact Production Fix
1. **Tune Kafka Consumer Batching & Cooperative Assignment**:
   - Reduce `max.poll.records: 20` (ensuring worst-case batch runtime $20 \times 3\text{s timeout} = 60\text{s} \ll 300\text{s}$).
   - Enable `partition.assignment.strategy: ['cooperative-sticky']` and enforce strict `3,000 ms` HTTP client timeouts.
2. **Decouple & Isolate Poison Pills**:
   - Catch non-retryable `4xx` / schema errors as `PoisonPillError`, write to `dead_letter_events`, and **commit offset `891,204`** immediately.
   - Wrap business updates in `processed_events` Inbox deduplication so any unavoidable rebalance duplicate is safely ignored (`DEDUPLICATED`).

---

### Exercise 5: Preventing Outbox Table Bloat & Debezium WAL Disk Exhaustion
A team runs two services in PostgreSQL:
- **Service A** uses a polling Outbox Relay (`SELECT ... WHERE published_at IS NULL FOR UPDATE SKIP LOCKED`), processing `3,000 events/sec` without purging old rows (`outbox_events` reaches 180M rows / 90GB).
- **Service B** uses Debezium WAL CDC (`pgoutput`). During a staging test, engineers stop the Debezium container for the weekend; by Sunday night, Service B's production-cloned PostgreSQL primary hits `100% disk full` and crashes (`PANIC: could not write to file pg_wal`).

Diagnose both operational time bombs and prescribe the exact PostgreSQL architecture fix for each.

---

### Solution 5

#### Service A: Polling Outbox Bloat Fix
- *Diagnosis*: Updating `published_at = NOW()` creates `3,000` dead tuples/sec (`260M/day`). Even with a partial index `WHERE published_at IS NULL`, table heap pages bloat, autovacuum thrashes I/O, and `DELETE FROM outbox_events` worsens fragmentation.
- *Fix*: Define `outbox_events` with PostgreSQL declarative **range partitioning by `created_at` (hourly or daily)** (`PARTITION BY RANGE (created_at)`). Once all events in yesterday's partition are published, drop the entire partition instantaneously via `DROP TABLE outbox_events_2026_11_11` (metadata DDL only, zero dead tuples, zero vacuum cost).

#### Service B: Debezium Replication Slot WAL Retention Fix
- *Diagnosis*: Debezium relies on a persistent PostgreSQL **Logical Replication Slot**. While Debezium is down, PostgreSQL refuses to delete or recycle *any* WAL segments newer than the slot's `restart_lsn`, filling `pg_wal` until the storage volume reaches 100% and crashes the cluster!
- *Fix*:
  1. Set `max_slot_wal_keep_size = '20GB'` in `postgresql.conf` (PostgreSQL 13+) so PostgreSQL invalidates an abandoned slot before crashing the primary database (falling back to Debezium snapshot re-sync if a slot drops).
  2. Monitor `pg_replication_slots` (`active = false` OR `retained_wal > 2GB`) with high-urgency SRE alerts.

---

### Exercise 6: Closing a Semantic-Lock Race Between `CancelOrder` and an In-Flight Checkout Saga
A customer clicks "Pay Now" (`StartCheckout` Saga reserves inventory at $t=0\text{ ms}$ and sends `CapturePaymentCommand` at $t=40\text{ ms}$). At $t=55\text{ ms}$, the customer clicks "Cancel Order" (`POST /orders/:id/cancel`), which blindly executes `UPDATE orders SET status = 'CANCELLED'` and emits `ReleaseInventoryCommand`. At $t=120\text{ ms}$, `PaymentCaptured` arrives from Stripe (`$200` settled!), marking the order `CONFIRMED` while inventory was already released (`CANCELLED` race -> customer charged for unreserved item!). Trace the isolation violation and implement the Semantic Lock fix.

---

### Solution 6

#### Isolation Violation (Saga Dirty Write / Lost Update)
Because Saga steps execute without global ACID Isolation, `CancelOrder` mutated `orders` while the Checkout Saga was mid-flight between `ReserveInventory` and the Pivot (`CapturePayment`), leaving the system split between `Payment = CAPTURED` and `Inventory = RELEASED`.

#### Semantic Lock Fix
1. **Enforce Explicit Pending State Transitions in `CancelOrderHandler`**:
   - Inside `BEGIN ... SELECT ... FROM orders WHERE id = $1 FOR UPDATE`, inspect `order.status`:
   - If `status IN ('PENDING_CHECKOUT', 'INVENTORY_RESERVED', 'COMPENSATING')`, **reject direct cancellation immediately with HTTP `409 Conflict`** (`"Order checkout is currently finalizing; retry cancellation once processing completes"`), **OR** set a durable flag `cancellation_requested = TRUE` on `saga_instances` so the Orchestrator evaluates it deterministically *after* the current step settles (`PaymentCaptured` -> trigger `RefundPayment` + `ReleaseInventory` cleanly; `PaymentDeclined` -> proceed with `ReleaseInventory`).
2. **State Guard in `OrderCheckoutSagaOrchestrator`**:
   - Ensure `PaymentCaptured` checks `saga.current_step === 'AWAITING_PAYMENT'` and `orders.status === 'INVENTORY_RESERVED'` under row lock (`FOR UPDATE`). If `cancellation_requested === TRUE`, transition straight to `COMPENSATING_REFUND_AND_INVENTORY` instead of `CONFIRMED`!
