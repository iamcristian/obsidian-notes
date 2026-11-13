---
tags:
  - software-engineering
  - architectural-patterns
  - event-sourcing
  - cqrs
created: 2026-01-13
updated: 2026-11-13
status: 🟢
level: mid-to-senior
---
# Event Sourcing

> [!abstract] Hub Context
> Part of [[Programming/Software Engineering/Architectural Patterns/Architectural Patterns Complete Guide|Architectural Patterns Complete Guide]]. Deeply integrated with [[Programming/Software Engineering/Architectural Patterns/CQRS|CQRS]], [[Programming/Software Engineering/Architectural Patterns/Event-Driven Architecture|Event-Driven Architecture]], [[Programming/Software Engineering/DDD/Tactical Design & Building Blocks|Tactical Design & Building Blocks]], and [[Programming/Software Engineering/Database Design/Transactions|Transactions]].

**Event Sourcing (ES)** persists an Aggregate's authoritative state not by overwriting mutable columns in place (`UPDATE accounts SET balance_cents = 4200`), but by appending an **immutable, strictly ordered sequence of past-tense domain facts** (`AccountOpened`, `FundsDeposited`, `FundsWithdrawn`, `AccountFrozen`) to an append-only **Event Store**. Current state is derived purely in memory by left-folding the stream from event version `1` (or from a verified snapshot) up to the stream head.

---

## 1. First-Principles Mechanics & Complete Architecture

### Append-Only Log Physics vs Mutable-Row CRUD State
In traditional CRUD persistence, a single database row represents the **latest compressed state** of an entity. Every `UPDATE` physically destroys the prior state unless separate triggers or audit shadow tables try to capture deltas after the fact—and even audit tables typically fail to record **domain intent** (why `balance` changed from `$100` to `$80`: customer ATM withdrawal vs fee reversal vs fraud chargeback).

Event Sourcing inverts this model at the storage engine boundary:
1. **Events Are the System of Record**: Every state change produces one or more immutable event records appended atomically to a stream identified by `stream_id`. Historical events are never updated or deleted during normal operations.
2. **Current State Is a Pure Derived Function**: An entity's current in-memory attributes exist strictly as the mathematical reduction (fold) of its recorded events:

$$S_n = \text{fold}(\text{apply}, S_0, [e_1, e_2, \dots, e_n])$$

3. **Temporal & Structural Reconstruction**: Because every intermediate event persists in commit order, the system can reconstruct exact state at any historical timestamp or stream version, spin up brand-new [[Programming/Software Engineering/Architectural Patterns/CQRS|CQRS]] read projections months later from historical events, and inspect exact causal chains (`correlation_id`, `causation_id`) across distributed workflows.

### Separating `decide` from `apply` (The Strict Purity Invariant)
Event-sourced domain aggregates partition behavior into two distinct functions:

| Function | Signature | Purity & Side-Effect Rules | Failure Semantics |
|---|---|---|---|
| **`decide`** | `(state: S, command: C) -> Event[]` | Reads current rehydrated `state`; validates invariants; computes deterministic facts; zero direct DB/network calls inside Aggregate | **May throw** domain invariant errors (`InsufficientFundsError`, `AccountFrozenError`) |
| **`apply` (`evolve`)** | `(state: S, event: E) -> S` | **Strictly pure, deterministic, total function**. Zero I/O, zero `Date.now()`, zero `Math.random()`, zero `crypto.randomUUID()`, zero policy exceptions | **Never throws** invariant rejections; historical events are immutable past facts |

**Why `apply()` Must Never Validate or Throw**: Business policies evolve over time. Suppose in 2024 the ledger permitted promotional overdrafts down to `-$50`, leaving historical `FundsWithdrawn` events that drove `balanceCents` to `-3000`. In 2026, product disallows negative balances. If `apply()` checks `if (nextBalance < 0) throw`, rehydrating any 2024 account permanently crashes and bricks the customer's stream. Validation belongs exclusively in `decide()`, whereas `apply()` unconditionally folds historical facts.

### Canonical Event Store Schema & Indexing

A relational Event Store in PostgreSQL models two core append structures:
- **`event_streams`**: Tracks stream identity (`stream_id`), `aggregate_type`, and `current_version`. Locking the `event_streams` row (`FOR UPDATE`) or relying on `UNIQUE (stream_id, stream_version)` guarantees linearizable per-stream serialization.
- **`events`**: Stores `global_position` (`BIGSERIAL`), `stream_id` (`UUID`), `stream_version` (`INT`), `event_type`, `schema_version`, `payload` (`JSONB`), `metadata` (`JSONB` carrying `correlation_id`, `causation_id`, `actor_id`, `traceparent`), `idempotency_key`, and `occurred_at`.

### Optimistic Concurrency Control (`expectedVersion`)

When two concurrent API calls read `stream_version = 4` (`balance = $100`) and both attempt to withdraw `$80`, naive last-writer-wins appends would overdraft the account to `-$60`. The Event Store enforces atomic **Optimistic Concurrency Control (OCC)** using `expectedVersion` backed by a composite `UNIQUE (stream_id, stream_version)` constraint:

```mermaid
sequenceDiagram
    participant R1 as Request A (Withdraw 80)
    participant R2 as Request B (Withdraw 80)
    participant ES as Event Store (v=4)
    R1->>ES: Read Stream (balance 100, v=4)
    R2->>ES: Read Stream (balance 100, v=4)
    R1->>ES: Append v=5 (expectedVersion=4)
    ES-->>R1: 201 Committed (v=5)
    R2->>ES: Append v=5 (expectedVersion=4)
    ES-->>R2: 409 Conflict (PG 23505 Unique Violation)
```

Standard expected version sentinels used across PostgreSQL stores and dedicated engines (KurrentDB / EventStoreDB):
- **`NO_STREAM` (`0`)**: Asserts the stream does not yet exist; prevents duplicate stream creation (`OpenAccount` called twice concurrently).
- **`EXACT(n)` (`n >= 1`)**: Asserts the stream head is strictly at version `n`; new events insert at `n + 1 ... n + k`.
- **`ANY` (`-1`)**: Disables version checking (used solely for blind append logs such as telemetry ingestion, never for financial invariants).

When PostgreSQL raises SQLSTATE `23505` (`unique_violation`) on `(stream_id, stream_version)`, the application maps it to `OptimisticConcurrencyError`, optionally reloads the fresh stream (now at `v=5`) in a bounded retry loop (`<= 3` attempts), and re-evaluates `decide()`—which now cleanly rejects Request B with `InsufficientFundsError`!

### Snapshotting Mechanics & Invalidation

Rehydrating a stream of `50` events takes `< 0.5 ms`. Rehydrating a high-frequency merchant settlement stream with `350,000` events transfers tens of megabytes of JSONB over the network and burns hundreds of milliseconds of CPU folding every command.

**Snapshotting** caches an intermediate fold result `(stream_id, stream_version, snapshot_format_version, state_json)` whenever `stream_version` crosses a configurable threshold $N$ (typically $N = 100 \text{ to } 500$):

1. **Read Optimization**: Load the latest snapshot `S` matching the **current `SNAPSHOT_FORMAT_VERSION`**, then query `events WHERE stream_id = $1 AND stream_version > S.stream_version ORDER BY stream_version ASC` and fold only the bounded tail (`< N` events).
2. **Latency Math**: Without snapshots, $200{,}000 \text{ events} \times 15\,\mu\text{s} = 3{,}000\text{ ms}$ fold CPU + ~80 MB JSON transfer. With $N = 200$, tail fold is bounded to $\le 200 \times 15\,\mu\text{s} = 3\text{ ms}$ worst case.
3. **Sync vs Async Snapshotting**:
   - *Synchronous*: Written inside (or immediately after) the append transaction every $N$ events; simple and guarantees immediate availability, adding 1–2ms every $N$th write.
   - *Asynchronous*: Emitted by a background snapshotter worker watching stream version modulo $N$; keeps command tail latency unaffected at the cost of temporary tail growth if the worker lags.
4. **Snapshot Invalidation on Fold Logic Changes**: Snapshots are strictly **disposable caches**, never authoritative truth. If a developer fixes a bug in `apply()` or adds a new computed field (`dailyWithdrawnCents`) to `BankAccountState`, cached snapshots lack that field. Incrementing a constant `CURRENT_SNAPSHOT_FORMAT_VERSION` causes `loadLatestSnapshot` to filter `WHERE snapshot_format_version = $CURRENT`, transparently falling back to full stream replay (`fromVersion = 0`) and lazily writing fresh snapshots in the new shape!

### Event Versioning & Read-Time Upcasting (`v1 -> v2 -> v3`)

Because events represent historical facts stored inside an immutable append-only log, mutating historical rows via `UPDATE events SET payload = ...` destroys cryptographic auditability and breaks running catch-up consumers. Instead, deploy **Read-Time Upcasters**:

```mermaid
flowchart LR
    DB[("Raw Event v1")] --> U1["Upcaster v1 to v2"]
    U1 --> U2["Upcaster v2 to v3"]
    U2 --> Decrypt["PII Decrypt / Redact"]
    Decrypt --> Apply["Pure apply() v3"]
```

- Each event persists `schema_version` (`1`, `2`, `3`).
- An `UpcasterRegistry` chains pure stateless transformations `(event: RawStoredEvent) -> RawStoredEvent` sequentially (`v1 -> v2`, then `v2 -> v3`) during stream reads and projection consumption.
- Aggregate `apply()` and Projection handlers code strictly against the **latest canonical schema (`v3`)**, keeping historical schema branching isolated in pure upcaster modules.

### GDPR "Right to Be Forgotten" via Crypto-Shredding

Strict immutability collides directly with GDPR Article 17 ("Right to Erasure") whenever events store Personal Identifiable Information (`ownerEmail`, `fullName`, `iban`, `ipAddress`). Deleting or nullifying event rows corrupts audit trails and breaks balance checksums.

**Crypto-Shredding** reconciles immutability with legal erasure:
1. **Per-Subject Key Management**: Provision a dedicated AES-256-GCM Data Encryption Key (DEK) $K_{\text{subject}}$ per user/customer, stored wrapped inside a mutable `subject_keys(subject_id, key_id, encrypted_dek, shredded_at)` table (envelope-encrypted by a KMS master Key Encryption Key in production).
2. **Encrypt PII Prior to Append**: Business invariants (`amountCents`, `currency`, `accountId`, `status`) remain plaintext JSONB for folding and analytical sums. Sensitive fields are encrypted with $K_{\text{subject}}$ alongside `keyId` before inserting into `events`.
3. **Execute Erasure**: Upon a verified GDPR deletion request, permanently delete (zero out) `encrypted_dek` for `subject_id` and mark `shredded_at = NOW()`. Historical events, WAL archives, and immutable backups remain structurally valid, yet the ciphertext inside them becomes mathematically unrecoverable.
4. **Deterministic Redacted Replay**: When upcasters or projection rebuilders attempt to decrypt ciphertext whose `keyId` returns `null`/shredded, they **never throw**—they substitute deterministic sentinel strings (`"[ERASED]"`), preserving exact financial totals while guaranteeing wiped PII never resurfaces during full projection rebuilds.

### Catch-Up Subscriptions, The PostgreSQL `BIGSERIAL` Visibility Gap, and Replay Safety

To query across accounts (`SELECT * FROM account_balances WHERE status = 'FROZEN'`), a **Catch-Up Projection Daemon** polls `events` sequentially by `global_position`, updates the CQRS read table, and persists `checkpoint_position` inside the **same local transaction** as the read-model upsert.

#### Critical Production Trap: The PostgreSQL `BIGSERIAL` Visibility Gap
Consider two concurrent commands running on PostgreSQL `READ COMMITTED`:
1. `TX-A` starts at $t=0\text{ ms}$, obtains `global_position = 100` from `BIGSERIAL`, and runs slowly (commits at $t=80\text{ ms}$).
2. `TX-B` starts at $t=5\text{ ms}$, obtains `global_position = 101`, and commits fast at $t=15\text{ ms}$.
3. At $t=20\text{ ms}$, the Catch-Up Projection queries `WHERE global_position > 99 ORDER BY global_position ASC`. Under MVCC snapshot isolation, `TX-A` (`100`) is **uncommitted and invisible**, while `TX-B` (`101`) is **visible**!
4. If the daemon processes `101` and sets `checkpoint_position = 101`, then when `TX-A` commits at $t=80\text{ ms}$, **`global_position = 100` is permanently skipped forever!**

**Exact Production Cures for PostgreSQL Event Stores**:
1. **Transaction Visibility Horizon (`pg_snapshot_xmin(pg_current_snapshot())`)**: Store `tx_xid xid8 NOT NULL DEFAULT pg_current_xact_id()` on every row in `events`. When the catch-up poller reads batches, filter:
   ```sql
   WHERE global_position > $checkpoint
     AND tx_xid < pg_snapshot_xmin(pg_current_snapshot())
   ORDER BY global_position ASC
   ```
   Because `pg_snapshot_xmin` returns the lowest active (uncommitted) transaction ID across the entire Postgres cluster, `TX-B` (`101`) is held back for those few milliseconds until `TX-A` (`100`) either commits or rolls back—guaranteeing **zero skipped events** without blocking writers!
2. **Strict Non-Side-Effect Invariant During Replay**: A catch-up projection daemon rebuilding read tables from `checkpoint = 0` over 5 years of history **must never trigger external side effects** (emails, SMS alerts, webhooks, Stripe transfers, or new commands). Isolate external side effects into dedicated **Live Process Managers** that either check `event.occurredAt` / live-head watermarks or emit idempotent commands once during live forward execution—never inside projection rebuild loops.

---

## 2. Real-World Use Cases & Decision Matrix

### Persistence Architecture Decision Matrix

| Dimension | Mutable CRUD | CRUD + Trigger Audit Table | CDC Outbox (Debezium) | Full Event Sourcing |
|---|---|---|---|---|
| **Storage Shape** | Latest row state only | Latest row + raw column diff JSON | Latest row + WAL change events | Append-only domain events + snapshots |
| **Captures Business Intent?** | No | No (only `old_col -> new_col`) | Partial (depends on domain outbox) | **Yes** (`AccountFrozenDueToFraud` vs `ClosedByOwner`) |
| **Time-Travel & Full Rebuild** | Impossible | Difficult (must parse column diffs) | Limited to retention window unless archived | **Native** (replay from `global_position = 1`) |
| **Schema Evolution Burden** | Low (`ALTER TABLE`) | Low | Moderate | **High** (immutable history requires Upcasters) |
| **Query Flexibility** | Native SQL indexes | Native SQL | Native SQL + Async Projections | **Requires CQRS Projections** for multi-entity queries |

### Five Concrete Engineering Scenarios

1. **Double-Entry Financial Ledger & Payment Clearing (Adopt Event Sourcing)**:
   - *Context*: Every cent must be auditable by regulators (`SOX`, `PCI-DSS`, banking licenses). Disputes require proving exact sequence of holds, captures, reversals, and fee adjustments at exact historical timestamps.
   - *Decision*: **Event Sourcing** is ideal. Balances are derived purely from immutable ledger postings; OCC prevents concurrent overdrafts; crypto-shredding isolates customer PII while leaving monetary postings intact indefinitely.
2. **Insurance Policy Underwriting & Claims Adjudication (Adopt Event Sourcing)**:
   - *Context*: Claims span months; retroactive policy endorsements alter prior coverage calculations, requiring "as-of" point-in-time state inspection (`What was the deductible on 2026-03-14 before Endorsement #3?`).
   - *Decision*: **Event Sourcing** provides native temporal replay and explicit domain transition history impossible to reconstruct cleanly from overwritten rows.
3. **Collaborative Workflows / Multi-Stage Approvals / Order Lifecycle Analytics (Adopt Event Sourcing)**:
   - *Context*: Product analytics frequently introduces retroactive metrics (*"How often do carts remove high-margin items right after seeing shipping fees?"*) that weren't modeled when the feature launched.
   - *Decision*: **Event Sourcing** allows building brand-new retrospective CQRS projections over historical streams without having instrumented custom tracking tables upfront.
4. **Standard User Preferences, Product Catalog Metadata, CMS Articles (Reject Event Sourcing)**:
   - *Context*: Simple CRUD entities where only current values matter and occasional `edited_at` history suffices.
   - *Decision*: **Do NOT use Event Sourcing**. Mutable PostgreSQL tables (optionally paired with Level 1 [[Programming/Software Engineering/Architectural Patterns/CQRS|CQRS]] or an audit log) deliver faster iteration with zero upcaster overhead.
5. **High-Throughput IoT Sensor Telemetry (`500k writes/sec` — Reject Per-Entity ES Fold)**:
   - *Context*: Devices stream temperature/GPS metrics continuously where analytics needs windowed rollups rather than sequential invariant rehydration.
   - *Decision*: **Do NOT use Event Sourcing**. Append directly to a time-series database (TimescaleDB / ClickHouse) or Kafka stream processor.

---

## 3. Step-by-Step Practical Application: Production Bank Account / Ledger Event Store

This complete, runnable TypeScript + PostgreSQL implementation builds a **Production Bank Account / Ledger Event Store** featuring:
- PostgreSQL DDL with `event_streams`, `events` (including `xid8` visibility fence & `idempotency_key`), `snapshots`, `subject_keys`, `account_balances_read_model`, and `projection_checkpoints`.
- AES-256-GCM **Crypto-Shredding** (`CryptoShredder`) and **Composable Upcasters** (`v1 -> v2 -> v3`).
- Pure `BankAccount` rehydration (`apply`) and command invariant enforcement (`decide`).
- `PostgresEventStore` with atomic OCC (`expectedVersion` + `23505` handling) and `pg_snapshot_xmin` gap-free catch-up reads.
- Idempotent `BalanceProjectionDaemon` with deterministic replay/rebuild.
- Vitest test suite covering OCC races, snapshot parity & invalidation, `v1->v2->v3` upcasting + GDPR erasure, and projection rebuild parity.

### Project Directory Layout

```text
ledger-event-sourcing/
├── package.json
├── sql/
│   └── 001_event_store.sql
└── src/
    ├── ledger/
    │   ├── crypto/
    │   │   └── CryptoShredder.ts
    │   ├── domain/
    │   │   ├── BankAccountEvents.ts
    │   │   ├── UpcasterRegistry.ts
    │   │   └── BankAccount.ts
    │   ├── store/
    │   │   └── PostgresEventStore.ts
    │   ├── service/
    │   │   └── BankAccountCommandService.ts
    │   └── projections/
    │       └── BalanceProjectionDaemon.ts
    └── tests/
        └── event-sourcing.ledger.test.ts
```

### Step 1: PostgreSQL Event Store, Snapshots & Checkpoints DDL (`sql/001_event_store.sql`)

```sql
-- sql/001_event_store.sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- Tracks aggregate stream heads for existence checks and fast version lookups
CREATE TABLE IF NOT EXISTS event_streams (
    stream_id       UUID PRIMARY KEY,
    aggregate_type  TEXT NOT NULL,
    current_version INTEGER NOT NULL DEFAULT 0,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Append-only Event Store with OCC constraint and MVCC visibility horizon column (tx_xid)
CREATE TABLE IF NOT EXISTS events (
    global_position BIGSERIAL PRIMARY KEY,
    tx_xid          XID8 NOT NULL DEFAULT pg_current_xact_id(),
    event_id        UUID NOT NULL UNIQUE DEFAULT gen_random_uuid(),
    stream_id       UUID NOT NULL REFERENCES event_streams(stream_id),
    stream_version  INTEGER NOT NULL CHECK (stream_version > 0),
    event_type      TEXT NOT NULL,
    schema_version  SMALLINT NOT NULL DEFAULT 1,
    payload         JSONB NOT NULL,
    metadata        JSONB NOT NULL DEFAULT '{}'::jsonb,
    idempotency_key TEXT NULL,
    occurred_at     TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    CONSTRAINT uq_stream_version UNIQUE (stream_id, stream_version),
    CONSTRAINT uq_stream_idempotency UNIQUE (stream_id, idempotency_key)
);

CREATE INDEX IF NOT EXISTS idx_events_global_tx
    ON events (global_position ASC, tx_xid);

-- Disposable Aggregate Snapshots keyed by format version for instant invalidation
CREATE TABLE IF NOT EXISTS snapshots (
    stream_id               UUID NOT NULL REFERENCES event_streams(stream_id),
    stream_version          INTEGER NOT NULL,
    snapshot_format_version SMALLINT NOT NULL,
    state                   JSONB NOT NULL,
    created_at              TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    PRIMARY KEY (stream_id, stream_version, snapshot_format_version)
);

-- Mutable per-subject DEK store enabling GDPR Crypto-Shredding without mutating events
CREATE TABLE IF NOT EXISTS subject_keys (
    subject_id    UUID PRIMARY KEY,
    key_id        TEXT NOT NULL UNIQUE,
    dek_hex       TEXT NULL,          -- NULL once crypto-shredded!
    shredded_at   TIMESTAMPTZ NULL,
    created_at    TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- CQRS Read Model projected asynchronously from events.global_position
CREATE TABLE IF NOT EXISTS account_balances_read_model (
    account_id      UUID PRIMARY KEY,
    owner_id        UUID NOT NULL,
    owner_email     TEXT NOT NULL,
    account_tier    TEXT NOT NULL,
    currency        CHAR(3) NOT NULL,
    balance_cents   BIGINT NOT NULL,
    status          TEXT NOT NULL,
    last_version    INTEGER NOT NULL,
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Durable projection checkpoints guaranteeing monotonic at-least-once cursor progress
CREATE TABLE IF NOT EXISTS projection_checkpoints (
    projection_name     TEXT PRIMARY KEY,
    checkpoint_position BIGINT NOT NULL DEFAULT 0,
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### Step 2: GDPR Crypto-Shredding Encryption Helper (`src/ledger/crypto/CryptoShredder.ts`)

```typescript
// src/ledger/crypto/CryptoShredder.ts
import { createCipheriv, createDecipheriv, randomBytes } from 'node:crypto';
import { Pool } from 'pg';

export interface EncryptedField {
  keyId: string;
  ivHex: string;
  tagHex: string;
  ciphertextHex: string;
}

export class CryptoShredder {
  constructor(private readonly pool: Pool) {}

  async ensureSubjectKey(subjectId: string): Promise<{ keyId: string; key: Buffer }> {
    const existing = await this.pool.query<{ key_id: string; dek_hex: string | null }>(
      `SELECT key_id, dek_hex FROM subject_keys WHERE subject_id = $1`,
      [subjectId],
    );
    if (existing.rowCount && existing.rows[0]) {
      const row = existing.rows[0];
      if (!row.dek_hex) {
        throw new Error(`Subject ${subjectId} has already been crypto-shredded`);
      }
      return { keyId: row.key_id, key: Buffer.from(row.dek_hex, 'hex') };
    }

    const keyId = `dek_${subjectId}`;
    const dek = randomBytes(32); // 256-bit AES-GCM key
    const inserted = await this.pool.query<{ key_id: string; dek_hex: string | null }>(
      `INSERT INTO subject_keys (subject_id, key_id, dek_hex)
       VALUES ($1, $2, $3)
       ON CONFLICT (subject_id) DO UPDATE SET subject_id = EXCLUDED.subject_id
       RETURNING key_id, dek_hex`,
      [subjectId, keyId, dek.toString('hex')],
    );

    const row = inserted.rows[0];
    if (!row.dek_hex) {
      throw new Error(`Subject ${subjectId} has already been crypto-shredded`);
    }
    return { keyId: row.key_id, key: Buffer.from(row.dek_hex, 'hex') };
  }

  async encryptPii(subjectId: string, plaintext: string): Promise<EncryptedField> {
    const { keyId, key } = await this.ensureSubjectKey(subjectId);
    const iv = randomBytes(12);
    const cipher = createCipheriv('aes-256-gcm', key, iv);
    const encrypted = Buffer.concat([cipher.update(plaintext, 'utf8'), cipher.final()]);
    const tag = cipher.getAuthTag();
    return {
      keyId,
      ivHex: iv.toString('hex'),
      tagHex: tag.toString('hex'),
      ciphertextHex: encrypted.toString('hex'),
    };
  }

  // Deterministic non-throwing decryption: returns '[ERASED]' if the subject key was shredded.
  async decryptOrRedact(field: EncryptedField): Promise<string> {
    const res = await this.pool.query<{ dek_hex: string | null }>(
      `SELECT dek_hex FROM subject_keys WHERE key_id = $1`,
      [field.keyId],
    );
    const dekHex = res.rows[0]?.dek_hex;
    if (!dekHex) {
      return '[ERASED]';
    }

    const decipher = createDecipheriv(
      'aes-256-gcm',
      Buffer.from(dekHex, 'hex'),
      Buffer.from(field.ivHex, 'hex'),
    );
    decipher.setAuthTag(Buffer.from(field.tagHex, 'hex'));
    const decrypted = Buffer.concat([
      decipher.update(Buffer.from(field.ciphertextHex, 'hex')),
      decipher.final(),
    ]);
    return decrypted.toString('utf8');
  }

  async shredSubjectKey(subjectId: string): Promise<void> {
    await this.pool.query(
      `UPDATE subject_keys
       SET dek_hex = NULL, shredded_at = NOW()
       WHERE subject_id = $1`,
      [subjectId],
    );
  }
}
```

### Step 3: Domain Events & Composable `v1 -> v2 -> v3` Upcaster Registry (`src/ledger/domain/`)

```typescript
// src/ledger/domain/BankAccountEvents.ts
import { EncryptedField } from '../crypto/CryptoShredder.js';

export interface StoredEventEnvelope<P = Record<string, unknown>> {
  globalPosition: number;
  eventId: string;
  streamId: string;
  streamVersion: number;
  eventType: string;
  schemaVersion: number;
  payload: P;
  metadata: Record<string, unknown>;
  occurredAt: Date;
}

export interface NewDomainEvent<P = Record<string, unknown>> {
  eventType: BankAccountEvent['eventType'];
  schemaVersion: number;
  payload: P;
}

export type AccountTier = 'STANDARD' | 'PREMIER';

// Canonical latest event types (AccountOpened is at v3)
export type BankAccountEvent =
  | {
      eventType: 'AccountOpened';
      schemaVersion: 3;
      streamId: string;
      streamVersion: number;
      payload: {
        ownerId: string;
        ownerEmail: string; // Resolved via CryptoShredder ('[ERASED]' if shredded)
        currency: string;
        initialDepositCents: number;
        accountTier: AccountTier;
      };
    }
  | {
      eventType: 'FundsDeposited';
      schemaVersion: 1;
      streamId: string;
      streamVersion: number;
      payload: {
        amountCents: number;
        reference: string;
      };
    }
  | {
      eventType: 'FundsWithdrawn';
      schemaVersion: 1;
      streamId: string;
      streamVersion: number;
      payload: {
        amountCents: number;
        reference: string;
      };
    }
  | {
      eventType: 'AccountFrozen';
      schemaVersion: 1;
      streamId: string;
      streamVersion: number;
      payload: {
        reason: string;
      };
    };

export interface AccountOpenedPersistedV3Payload {
  ownerId: string;
  ownerEmailEncrypted: EncryptedField;
  currency: string;
  initialDepositCents: number;
  accountTier: AccountTier;
}
```

```typescript
// src/ledger/domain/UpcasterRegistry.ts
import { CryptoShredder, EncryptedField } from '../crypto/CryptoShredder.js';
import { BankAccountEvent, StoredEventEnvelope } from './BankAccountEvents.js';

type SyncUpcaster = (env: StoredEventEnvelope) => StoredEventEnvelope;

/**
 * Transforms legacy persisted schema versions cleanly into canonical domain events:
 * - AccountOpened v1 (plaintext { ownerId, ownerEmail, initialDepositCents })
 *   -> v2 (adds currency: 'USD', accountTier: 'STANDARD')
 *   -> v3 (wraps plaintext or encrypted PII into decrypted/redacted canonical shape)
 */
export class BankAccountUpcasterRegistry {
  private readonly syncChain = new Map<string, SyncUpcaster>();

  constructor(private readonly cryptoShredder: CryptoShredder) {
    // v1 -> v2: Introduce default currency & accountTier missing in 2024 schema
    this.syncChain.set('AccountOpened:1', (env) => ({
      ...env,
      schemaVersion: 2,
      payload: {
        ...env.payload,
        currency: (env.payload.currency as string | undefined) ?? 'USD',
        accountTier: 'STANDARD',
      },
    }));

    // v2 -> v3: Normalize plaintext email vs encrypted PII structure
    this.syncChain.set('AccountOpened:2', (env) => ({
      ...env,
      schemaVersion: 3,
      payload: {
        ownerId: env.payload.ownerId,
        ownerEmailPlainLegacy: env.payload.ownerEmail,
        ownerEmailEncrypted: env.payload.ownerEmailEncrypted ?? null,
        currency: env.payload.currency,
        initialDepositCents: env.payload.initialDepositCents,
        accountTier: env.payload.accountTier ?? 'STANDARD',
      },
    }));
  }

  async upcast(raw: StoredEventEnvelope): Promise<BankAccountEvent> {
    let current = raw;
    // Apply deterministic schema version transformations sequentially (v1 -> v2 -> v3)
    while (true) {
      const key = `${current.eventType}:${current.schemaVersion}`;
      const step = this.syncChain.get(key);
      if (!step) break;
      current = step(current);
    }

    if (current.eventType === 'AccountOpened') {
      let resolvedEmail = '[ERASED]';
      const encrypted = current.payload.ownerEmailEncrypted as EncryptedField | null | undefined;
      if (encrypted) {
        resolvedEmail = await this.cryptoShredder.decryptOrRedact(encrypted);
      } else if (typeof current.payload.ownerEmailPlainLegacy === 'string') {
        resolvedEmail = current.payload.ownerEmailPlainLegacy;
      }

      return {
        eventType: 'AccountOpened',
        schemaVersion: 3,
        streamId: current.streamId,
        streamVersion: current.streamVersion,
        payload: {
          ownerId: String(current.payload.ownerId),
          ownerEmail: resolvedEmail,
          currency: String(current.payload.currency),
          initialDepositCents: Number(current.payload.initialDepositCents),
          accountTier: (current.payload.accountTier as 'STANDARD' | 'PREMIER') ?? 'STANDARD',
        },
      };
    }

    return {
      eventType: current.eventType,
      schemaVersion: 1,
      streamId: current.streamId,
      streamVersion: current.streamVersion,
      payload: current.payload,
    } as BankAccountEvent;
  }

  async upcastAll(rawEvents: StoredEventEnvelope[]): Promise<BankAccountEvent[]> {
    return Promise.all(rawEvents.map((e) => this.upcast(e)));
  }
}
```

### Step 4: Pure State Rehydration & Command Invariants (`src/ledger/domain/BankAccount.ts`)

```typescript
// src/ledger/domain/BankAccount.ts
import { AccountTier, BankAccountEvent, NewDomainEvent } from './BankAccountEvents.js';

export const CURRENT_SNAPSHOT_FORMAT_VERSION = 2;

export type AccountStatus = 'UNINITIALIZED' | 'OPEN' | 'FROZEN';

export interface BankAccountState {
  accountId: string;
  ownerId: string;
  ownerEmail: string;
  accountTier: AccountTier;
  currency: string;
  balanceCents: number;
  transactionCount: number;
  status: AccountStatus;
  version: number;
}

export class DomainRuleViolationError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'DomainRuleViolationError';
  }
}

export function initialAccountState(accountId = ''): BankAccountState {
  return {
    accountId,
    ownerId: '',
    ownerEmail: '',
    accountTier: 'STANDARD',
    currency: 'USD',
    balanceCents: 0,
    transactionCount: 0,
    status: 'UNINITIALIZED',
    version: 0,
  };
}

/**
 * STRICTLY PURE, TOTAL STATE FOLD FUNCTION:
 * - Zero I/O, zero Date.now(), zero random values, zero invariant throws!
 * - Even if historical balances went negative under legacy rules, apply() reproduces state faithfully.
 */
export function applyBankAccountEvent(
  state: BankAccountState,
  event: BankAccountEvent,
): BankAccountState {
  switch (event.eventType) {
    case 'AccountOpened':
      return {
        ...state,
        accountId: event.streamId,
        ownerId: event.payload.ownerId,
        ownerEmail: event.payload.ownerEmail,
        accountTier: event.payload.accountTier,
        currency: event.payload.currency,
        balanceCents: event.payload.initialDepositCents,
        transactionCount: 0,
        status: 'OPEN',
        version: event.streamVersion,
      };
    case 'FundsDeposited':
      return {
        ...state,
        balanceCents: state.balanceCents + event.payload.amountCents,
        transactionCount: state.transactionCount + 1,
        version: event.streamVersion,
      };
    case 'FundsWithdrawn':
      return {
        ...state,
        balanceCents: state.balanceCents - event.payload.amountCents,
        transactionCount: state.transactionCount + 1,
        version: event.streamVersion,
      };
    case 'AccountFrozen':
      return {
        ...state,
        status: 'FROZEN',
        version: event.streamVersion,
      };
  }
}

export function rehydrateBankAccount(
  events: BankAccountEvent[],
  baseState: BankAccountState = initialAccountState(),
): BankAccountState {
  return events.reduce(applyBankAccountEvent, baseState);
}

export type BankAccountCommand =
  | {
      type: 'DepositFunds';
      amountCents: number;
      reference: string;
    }
  | {
      type: 'WithdrawFunds';
      amountCents: number;
      reference: string;
    }
  | {
      type: 'FreezeAccount';
      reason: string;
    };

export function decideBankAccountCommand(
  state: BankAccountState,
  command: BankAccountCommand,
): NewDomainEvent[] {
  if (state.status === 'UNINITIALIZED') {
    throw new DomainRuleViolationError('Account stream does not exist');
  }

  switch (command.type) {
    case 'DepositFunds': {
      if (state.status === 'FROZEN') {
        throw new DomainRuleViolationError(`Cannot deposit into frozen account ${state.accountId}`);
      }
      if (!Number.isInteger(command.amountCents) || command.amountCents <= 0) {
        throw new DomainRuleViolationError('Deposit amount must be a positive integer in cents');
      }
      return [
        {
          eventType: 'FundsDeposited',
          schemaVersion: 1,
          payload: {
            amountCents: command.amountCents,
            reference: command.reference,
          },
        },
      ];
    }

    case 'WithdrawFunds': {
      if (state.status === 'FROZEN') {
        throw new DomainRuleViolationError(`Cannot withdraw from frozen account ${state.accountId}`);
      }
      if (!Number.isInteger(command.amountCents) || command.amountCents <= 0) {
        throw new DomainRuleViolationError('Withdrawal amount must be a positive integer in cents');
      }
      if (state.balanceCents < command.amountCents) {
        throw new DomainRuleViolationError(
          `Insufficient funds: balance ${state.balanceCents} < requested ${command.amountCents}`,
        );
      }
      return [
        {
          eventType: 'FundsWithdrawn',
          schemaVersion: 1,
          payload: {
            amountCents: command.amountCents,
            reference: command.reference,
          },
        },
      ];
    }

    case 'FreezeAccount': {
      if (state.status === 'FROZEN') {
        // Idempotent no-op decision: emitting 0 events leaves stream untouched
        return [];
      }
      if (!command.reason.trim()) {
        throw new DomainRuleViolationError('Freeze reason is mandatory');
      }
      return [
        {
          eventType: 'AccountFrozen',
          schemaVersion: 1,
          payload: { reason: command.reason.trim() },
        },
      ];
    }
  }
}
```

### Step 5: `PostgresEventStore` with OCC, Idempotency, Snapshots & Gap-Free Global Reads (`src/ledger/store/PostgresEventStore.ts`)

```typescript
// src/ledger/store/PostgresEventStore.ts
import { Pool, PoolClient } from 'pg';
import { NewDomainEvent, StoredEventEnvelope } from '../domain/BankAccountEvents.js';

export const EXPECTED_NO_STREAM = 0;
export const EXPECTED_ANY_STREAM = -1;

export class OptimisticConcurrencyError extends Error {
  constructor(
    public readonly streamId: string,
    public readonly expectedVersion: number,
  ) {
    super(`Optimistic concurrency failure on stream ${streamId} (expectedVersion=${expectedVersion})`);
    this.name = 'OptimisticConcurrencyError';
  }
}

export interface SnapshotRecord<S> {
  streamId: string;
  streamVersion: number;
  snapshotFormatVersion: number;
  state: S;
}

export interface AppendResult {
  newVersion: number;
  appendedCount: number;
  deduplicated: boolean;
}

export class PostgresEventStore {
  constructor(private readonly pool: Pool) {}

  async readStream(streamId: string, afterVersion = 0): Promise<StoredEventEnvelope[]> {
    const res = await this.pool.query<{
      global_position: string;
      event_id: string;
      stream_id: string;
      stream_version: number;
      event_type: string;
      schema_version: number;
      payload: Record<string, unknown>;
      metadata: Record<string, unknown>;
      occurred_at: Date;
    }>(
      `SELECT global_position, event_id, stream_id, stream_version,
              event_type, schema_version, payload, metadata, occurred_at
       FROM events
       WHERE stream_id = $1 AND stream_version > $2
       ORDER BY stream_version ASC`,
      [streamId, afterVersion],
    );

    return res.rows.map((r) => ({
      globalPosition: Number(r.global_position),
      eventId: r.event_id,
      streamId: r.stream_id,
      streamVersion: r.stream_version,
      eventType: r.event_type,
      schemaVersion: r.schema_version,
      payload: r.payload,
      metadata: r.metadata,
      occurredAt: r.occurred_at,
    }));
  }

  /**
   * Gap-free global catch-up read:
   * Filters tx_xid < pg_snapshot_xmin(pg_current_snapshot()) so an active concurrent transaction
   * holding a lower BIGSERIAL global_position never causes higher positions to jump ahead permanently.
   */
  async readAllAfterPosition(
    afterGlobalPosition: number,
    limit: number,
  ): Promise<StoredEventEnvelope[]> {
    const res = await this.pool.query<{
      global_position: string;
      event_id: string;
      stream_id: string;
      stream_version: number;
      event_type: string;
      schema_version: number;
      payload: Record<string, unknown>;
      metadata: Record<string, unknown>;
      occurred_at: Date;
    }>(
      `SELECT global_position, event_id, stream_id, stream_version,
              event_type, schema_version, payload, metadata, occurred_at
       FROM events
       WHERE global_position > $1
         AND tx_xid < pg_snapshot_xmin(pg_current_snapshot())
       ORDER BY global_position ASC
       LIMIT $2`,
      [afterGlobalPosition, limit],
    );

    return res.rows.map((r) => ({
      globalPosition: Number(r.global_position),
      eventId: r.event_id,
      streamId: r.stream_id,
      streamVersion: r.stream_version,
      eventType: r.event_type,
      schemaVersion: r.schema_version,
      payload: r.payload,
      metadata: r.metadata,
      occurredAt: r.occurred_at,
    }));
  }

  async appendToStream(
    streamId: string,
    aggregateType: string,
    expectedVersion: number,
    events: NewDomainEvent[],
    metadata: Record<string, unknown>,
    idempotencyKey?: string,
  ): Promise<AppendResult> {
    if (events.length === 0) {
      return { newVersion: expectedVersion, appendedCount: 0, deduplicated: false };
    }

    const client = await this.pool.connect();
    try {
      await client.query('BEGIN');

      // 1. Check stream-level command idempotency before writing
      if (idempotencyKey) {
        const dup = await client.query<{ stream_version: number }>(
          `SELECT MAX(stream_version) AS stream_version
           FROM events
           WHERE stream_id = $1 AND idempotency_key = $2`,
          [streamId, idempotencyKey],
        );
        if (dup.rows[0]?.stream_version != null) {
          await client.query('COMMIT');
          return {
            newVersion: dup.rows[0].stream_version,
            appendedCount: 0,
            deduplicated: true,
          };
        }
      }

      // 2. Acquire stream row or enforce NO_STREAM creation rule
      let currentVersion: number;
      if (expectedVersion === EXPECTED_NO_STREAM) {
        try {
          await client.query(
            `INSERT INTO event_streams (stream_id, aggregate_type, current_version)
             VALUES ($1, $2, 0)`,
            [streamId, aggregateType],
          );
          currentVersion = 0;
        } catch (err: unknown) {
          if (this.isUniqueViolation(err)) {
            throw new OptimisticConcurrencyError(streamId, expectedVersion);
          }
          throw err;
        }
      } else {
        const streamRes = await client.query<{ current_version: number }>(
          `SELECT current_version FROM event_streams WHERE stream_id = $1 FOR UPDATE`,
          [streamId],
        );
        if (streamRes.rowCount === 0) {
          throw new OptimisticConcurrencyError(streamId, expectedVersion);
        }
        currentVersion = streamRes.rows[0].current_version;
        if (expectedVersion !== EXPECTED_ANY_STREAM && currentVersion !== expectedVersion) {
          throw new OptimisticConcurrencyError(streamId, expectedVersion);
        }
      }

      // 3. Insert sequentially versioned events guarded by UNIQUE (stream_id, stream_version)
      let nextVersion = currentVersion;
      for (let i = 0; i < events.length; i++) {
        nextVersion++;
        const evt = events[i];
        // Stamp idempotency_key onto the first event in the batch (or composite per index)
        const eventIdempKey = idempotencyKey ? `${idempotencyKey}:${i}` : null;
        try {
          await client.query(
            `INSERT INTO events (
               stream_id, stream_version, event_type, schema_version,
               payload, metadata, idempotency_key
             ) VALUES ($1, $2, $3, $4, $5::jsonb, $6::jsonb, $7)`,
            [
              streamId,
              nextVersion,
              evt.eventType,
              evt.schemaVersion,
              JSON.stringify(evt.payload),
              JSON.stringify(metadata),
              i === 0 && idempotencyKey ? idempotencyKey : eventIdempKey,
            ],
          );
        } catch (err: unknown) {
          if (this.isUniqueViolation(err)) {
            throw new OptimisticConcurrencyError(streamId, expectedVersion);
          }
          throw err;
        }
      }

      await client.query(
        `UPDATE event_streams SET current_version = $2 WHERE stream_id = $1`,
        [streamId, nextVersion],
      );

      await client.query('COMMIT');
      return { newVersion: nextVersion, appendedCount: events.length, deduplicated: false };
    } catch (err) {
      await client.query('ROLLBACK');
      throw err;
    } finally {
      client.release();
    }
  }

  async loadLatestSnapshot<S>(
    streamId: string,
    snapshotFormatVersion: number,
  ): Promise<SnapshotRecord<S> | null> {
    const res = await this.pool.query<{
      stream_id: string;
      stream_version: number;
      snapshot_format_version: number;
      state: S;
    }>(
      `SELECT stream_id, stream_version, snapshot_format_version, state
       FROM snapshots
       WHERE stream_id = $1 AND snapshot_format_version = $2
       ORDER BY stream_version DESC
       LIMIT 1`,
      [streamId, snapshotFormatVersion],
    );

    if (res.rowCount === 0) return null;
    const r = res.rows[0];
    return {
      streamId: r.stream_id,
      streamVersion: r.stream_version,
      snapshotFormatVersion: r.snapshot_format_version,
      state: r.state,
    };
  }

  async saveSnapshot<S>(
    streamId: string,
    streamVersion: number,
    snapshotFormatVersion: number,
    state: S,
    client?: PoolClient,
  ): Promise<void> {
    const runner = client ?? this.pool;
    await runner.query(
      `INSERT INTO snapshots (stream_id, stream_version, snapshot_format_version, state)
       VALUES ($1, $2, $3, $4::jsonb)
       ON CONFLICT (stream_id, stream_version, snapshot_format_version) DO NOTHING`,
      [streamId, streamVersion, snapshotFormatVersion, JSON.stringify(state)],
    );
  }

  private isUniqueViolation(err: unknown): boolean {
    return typeof err === 'object' && err !== null && 'code' in err && (err as { code?: string }).code === '23505';
  }
}
```

### Step 6: Application Command Service & Idempotent Catch-Up Projection Daemon (`src/ledger/service/` & `projections/`)

```typescript
// src/ledger/service/BankAccountCommandService.ts
import { CryptoShredder } from '../crypto/CryptoShredder.js';
import {
  applyBankAccountEvent,
  BankAccountCommand,
  BankAccountState,
  CURRENT_SNAPSHOT_FORMAT_VERSION,
  decideBankAccountCommand,
  DomainRuleViolationError,
  initialAccountState,
  rehydrateBankAccount,
} from '../domain/BankAccount.js';
import { AccountOpenedPersistedV3Payload, AccountTier } from '../domain/BankAccountEvents.js';
import { BankAccountUpcasterRegistry } from '../domain/UpcasterRegistry.js';
import {
  EXPECTED_NO_STREAM,
  OptimisticConcurrencyError,
  PostgresEventStore,
} from '../store/PostgresEventStore.js';

export class BankAccountCommandService {
  constructor(
    private readonly eventStore: PostgresEventStore,
    private readonly upcasters: BankAccountUpcasterRegistry,
    private readonly cryptoShredder: CryptoShredder,
    private readonly snapshotInterval = 100,
  ) {}

  async openAccount(params: {
    accountId: string;
    ownerId: string;
    ownerEmail: string;
    currency: string;
    initialDepositCents: number;
    accountTier?: AccountTier;
    actorId: string;
    idempotencyKey?: string;
  }): Promise<{ accountId: string; version: number }> {
    if (params.initialDepositCents < 0) {
      throw new DomainRuleViolationError('Initial deposit cannot be negative');
    }

    const encryptedEmail = await this.cryptoShredder.encryptPii(params.ownerId, params.ownerEmail);
    const payload: AccountOpenedPersistedV3Payload = {
      ownerId: params.ownerId,
      ownerEmailEncrypted: encryptedEmail,
      currency: params.currency,
      initialDepositCents: params.initialDepositCents,
      accountTier: params.accountTier ?? 'STANDARD',
    };

    const res = await this.eventStore.appendToStream(
      params.accountId,
      'BankAccount',
      EXPECTED_NO_STREAM,
      [{ eventType: 'AccountOpened', schemaVersion: 3, payload: payload as unknown as Record<string, unknown> }],
      { actorId: params.actorId },
      params.idempotencyKey,
    );

    return { accountId: params.accountId, version: res.newVersion };
  }

  async loadAccount(accountId: string): Promise<BankAccountState> {
    const snap = await this.eventStore.loadLatestSnapshot<BankAccountState>(
      accountId,
      CURRENT_SNAPSHOT_FORMAT_VERSION,
    );
    const fromVersion = snap ? snap.streamVersion : 0;
    const baseState = snap ? snap.state : initialAccountState(accountId);

    const rawTail = await this.eventStore.readStream(accountId, fromVersion);
    const upcastedTail = await this.upcasters.upcastAll(rawTail);
    return rehydrateBankAccount(upcastedTail, baseState);
  }

  async executeCommand(
    accountId: string,
    command: BankAccountCommand,
    metadata: { actorId: string; expectedVersion?: number; idempotencyKey?: string },
    maxOccRetries = 0,
  ): Promise<{ state: BankAccountState; deduplicated: boolean }> {
    let attempt = 0;

    while (true) {
      const snap = await this.eventStore.loadLatestSnapshot<BankAccountState>(
        accountId,
        CURRENT_SNAPSHOT_FORMAT_VERSION,
      );
      const fromVersion = snap ? snap.streamVersion : 0;
      const baseState = snap ? snap.state : initialAccountState(accountId);

      const rawTail = await this.eventStore.readStream(accountId, fromVersion);
      const upcastedTail = await this.upcasters.upcastAll(rawTail);
      const currentState = rehydrateBankAccount(upcastedTail, baseState);

      const expectedVersion = metadata.expectedVersion ?? currentState.version;
      if (currentState.version !== expectedVersion) {
        throw new OptimisticConcurrencyError(accountId, expectedVersion);
      }

      const newEvents = decideBankAccountCommand(currentState, command);
      if (newEvents.length === 0) {
        return { state: currentState, deduplicated: false };
      }

      try {
        const appended = await this.eventStore.appendToStream(
          accountId,
          'BankAccount',
          expectedVersion,
          newEvents,
          { actorId: metadata.actorId },
          metadata.idempotencyKey,
        );

        // If deduplicated, reload authoritative head state
        if (appended.deduplicated) {
          return { state: await this.loadAccount(accountId), deduplicated: true };
        }

        // Fold newly appended events to compute exact post-command state
        const tailRaw = await this.eventStore.readStream(accountId, expectedVersion);
        const tailTyped = await this.upcasters.upcastAll(tailRaw);
        let nextState = currentState;
        for (const e of tailTyped) {
          nextState = applyBankAccountEvent(nextState, e);
        }

        // Persist periodic snapshot whenever stream crosses snapshotInterval boundary
        const crossedBoundary =
          Math.floor(nextState.version / this.snapshotInterval) >
          Math.floor(currentState.version / this.snapshotInterval);
        if (crossedBoundary) {
          await this.eventStore.saveSnapshot(
            accountId,
            nextState.version,
            CURRENT_SNAPSHOT_FORMAT_VERSION,
            nextState,
          );
        }

        return { state: nextState, deduplicated: false };
      } catch (err) {
        if (err instanceof OptimisticConcurrencyError && attempt < maxOccRetries && metadata.expectedVersion === undefined) {
          attempt++;
          continue;
        }
        throw err;
      }
    }
  }
}
```

```typescript
// src/ledger/projections/BalanceProjectionDaemon.ts
import { Pool, PoolClient } from 'pg';
import { BankAccountEvent } from '../domain/BankAccountEvents.js';
import { BankAccountUpcasterRegistry } from '../domain/UpcasterRegistry.js';
import { PostgresEventStore } from '../store/PostgresEventStore.js';

export class BalanceProjectionDaemon {
  public static readonly PROJECTION_NAME = 'account_balances_v1';

  constructor(
    private readonly pool: Pool,
    private readonly eventStore: PostgresEventStore,
    private readonly upcasters: BankAccountUpcasterRegistry,
  ) {}

  async runOnce(batchSize = 250): Promise<{ processedEvents: number; newCheckpoint: number }> {
    const cpRes = await this.pool.query<{ checkpoint_position: string }>(
      `SELECT checkpoint_position FROM projection_checkpoints WHERE projection_name = $1`,
      [BalanceProjectionDaemon.PROJECTION_NAME],
    );
    let checkpoint = cpRes.rowCount ? Number(cpRes.rows[0].checkpoint_position) : 0;

    const rawBatch = await this.eventStore.readAllAfterPosition(checkpoint, batchSize);
    if (rawBatch.length === 0) {
      return { processedEvents: 0, newCheckpoint: checkpoint };
    }

    const typedBatch = await this.upcasters.upcastAll(rawBatch);

    const client = await this.pool.connect();
    try {
      await client.query('BEGIN');

      for (let i = 0; i < typedBatch.length; i++) {
        await this.projectSingleEvent(client, typedBatch[i]);
        checkpoint = rawBatch[i].globalPosition;
      }

      // Advance checkpoint atomically inside the SAME read-model commit
      await client.query(
        `INSERT INTO projection_checkpoints (projection_name, checkpoint_position, updated_at)
         VALUES ($1, $2, NOW())
         ON CONFLICT (projection_name) DO UPDATE
         SET checkpoint_position = GREATEST(projection_checkpoints.checkpoint_position, EXCLUDED.checkpoint_position),
             updated_at = NOW()`,
        [BalanceProjectionDaemon.PROJECTION_NAME, checkpoint],
      );

      await client.query('COMMIT');
      return { processedEvents: typedBatch.length, newCheckpoint: checkpoint };
    } catch (err) {
      await client.query('ROLLBACK');
      throw err;
    } finally {
      client.release();
    }
  }

  /**
   * PURE STATE MATERIALIZATION ONLY:
   * Never sends emails, webhooks, or external side effects during catch-up or full replay!
   * Enforces last_version < streamVersion so replaying batches is 100% idempotent.
   */
  private async projectSingleEvent(client: PoolClient, event: BankAccountEvent): Promise<void> {
    switch (event.eventType) {
      case 'AccountOpened':
        await client.query(
          `INSERT INTO account_balances_read_model (
             account_id, owner_id, owner_email, account_tier, currency,
             balance_cents, status, last_version, updated_at
           ) VALUES ($1, $2, $3, $4, $5, $6, 'OPEN', $7, NOW())
           ON CONFLICT (account_id) DO UPDATE
           SET owner_email  = EXCLUDED.owner_email,
               account_tier = EXCLUDED.account_tier,
               balance_cents = EXCLUDED.balance_cents,
               last_version = EXCLUDED.last_version,
               updated_at   = NOW()
           WHERE account_balances_read_model.last_version < EXCLUDED.last_version`,
          [
            event.streamId,
            event.payload.ownerId,
            event.payload.ownerEmail,
            event.payload.accountTier,
            event.payload.currency,
            event.payload.initialDepositCents,
            event.streamVersion,
          ],
        );
        break;

      case 'FundsDeposited':
        await client.query(
          `UPDATE account_balances_read_model
           SET balance_cents = balance_cents + $2,
               last_version  = $3,
               updated_at    = NOW()
           WHERE account_id = $1 AND last_version < $3`,
          [event.streamId, event.payload.amountCents, event.streamVersion],
        );
        break;

      case 'FundsWithdrawn':
        await client.query(
          `UPDATE account_balances_read_model
           SET balance_cents = balance_cents - $2,
               last_version  = $3,
               updated_at    = NOW()
           WHERE account_id = $1 AND last_version < $3`,
          [event.streamId, event.payload.amountCents, event.streamVersion],
        );
        break;

      case 'AccountFrozen':
        await client.query(
          `UPDATE account_balances_read_model
           SET status       = 'FROZEN',
               last_version = $2,
               updated_at   = NOW()
           WHERE account_id = $1 AND last_version < $2`,
          [event.streamId, event.streamVersion],
        );
        break;
    }
  }

  async rebuildFromScratch(): Promise<number> {
    await this.pool.query('TRUNCATE account_balances_read_model');
    await this.pool.query(
      `INSERT INTO projection_checkpoints (projection_name, checkpoint_position)
       VALUES ($1, 0)
       ON CONFLICT (projection_name) DO UPDATE SET checkpoint_position = 0, updated_at = NOW()`,
      [BalanceProjectionDaemon.PROJECTION_NAME],
    );

    let total = 0;
    while (true) {
      const { processedEvents } = await this.runOnce(500);
      if (processedEvents === 0) break;
      total += processedEvents;
    }
    return total;
  }
}
```

### Step 7: Vitest Concurrency, Snapshot, Upcaster, Crypto-Shredding & Replay Tests (`src/tests/event-sourcing.ledger.test.ts`)

```typescript
// src/tests/event-sourcing.ledger.test.ts
import { afterAll, beforeAll, beforeEach, describe, expect, it } from 'vitest';
import { Pool } from 'pg';
import { randomUUID } from 'node:crypto';
import { CryptoShredder } from '../ledger/crypto/CryptoShredder.js';
import { BankAccountUpcasterRegistry } from '../ledger/domain/UpcasterRegistry.js';
import {
  CURRENT_SNAPSHOT_FORMAT_VERSION,
  DomainRuleViolationError,
  rehydrateBankAccount,
} from '../ledger/domain/BankAccount.js';
import { OptimisticConcurrencyError, PostgresEventStore } from '../ledger/store/PostgresEventStore.js';
import { BankAccountCommandService } from '../ledger/service/BankAccountCommandService.js';
import { BalanceProjectionDaemon } from '../ledger/projections/BalanceProjectionDaemon.js';

describe('Bank Account Event Store & Projections', () => {
  let pool: Pool;
  let store: PostgresEventStore;
  let shredder: CryptoShredder;
  let upcasters: BankAccountUpcasterRegistry;
  let service: BankAccountCommandService;
  let projector: BalanceProjectionDaemon;

  beforeAll(async () => {
    pool = new Pool({
      connectionString: process.env.DATABASE_URL ?? 'postgres://postgres:postgres@localhost:5432/es_test',
    });
    store = new PostgresEventStore(pool);
    shredder = new CryptoShredder(pool);
    upcasters = new BankAccountUpcasterRegistry(shredder);
    service = new BankAccountCommandService(store, upcasters, shredder, 10);
    projector = new BalanceProjectionDaemon(pool, store, upcasters);
  });

  afterAll(async () => {
    await pool.end();
  });

  beforeEach(async () => {
    await pool.query(
      'TRUNCATE projection_checkpoints, account_balances_read_model, snapshots, events, event_streams, subject_keys RESTART IDENTITY CASCADE',
    );
  });

  it('prevents concurrent double-spend via Optimistic Concurrency Control', async () => {
    const accountId = randomUUID();
    const ownerId = randomUUID();
    await service.openAccount({
      accountId,
      ownerId,
      ownerEmail: 'alice@bank.io',
      currency: 'USD',
      initialDepositCents: 10_000, // $100.00
      actorId: ownerId,
    });

    const [a, b] = await Promise.allSettled([
      service.executeCommand(
        accountId,
        { type: 'WithdrawFunds', amountCents: 8_000, reference: 'atm-1' },
        { actorId: ownerId, expectedVersion: 1 },
      ),
      service.executeCommand(
        accountId,
        { type: 'WithdrawFunds', amountCents: 8_000, reference: 'atm-2' },
        { actorId: ownerId, expectedVersion: 1 },
      ),
    ]);

    const winners = [a, b].filter((r) => r.status === 'fulfilled');
    const losers = [a, b].filter((r) => r.status === 'rejected');
    expect(winners).toHaveLength(1);
    expect(losers).toHaveLength(1);
    expect((losers[0] as PromiseRejectedResult).reason).toBeInstanceOf(OptimisticConcurrencyError);

    // Retry second withdrawal against fresh head (balance is $20 -> throws DomainRuleViolationError)
    await expect(
      service.executeCommand(
        accountId,
        { type: 'WithdrawFunds', amountCents: 8_000, reference: 'atm-2' },
        { actorId: ownerId },
      ),
    ).rejects.toThrow(DomainRuleViolationError);

    const finalState = await service.loadAccount(accountId);
    expect(finalState.balanceCents).toBe(2_000);
    expect(finalState.version).toBe(2);
  });

  it('maintains 100% parity between snapshot+tail rehydration and full event fold', async () => {
    const accountId = randomUUID();
    const ownerId = randomUUID();
    await service.openAccount({
      accountId,
      ownerId,
      ownerEmail: 'bob@bank.io',
      currency: 'USD',
      initialDepositCents: 1_000,
      actorId: ownerId,
    });

    // Trigger 15 deposits -> crosses snapshot interval (10) at version 10, leaving 6 tail events
    for (let i = 1; i <= 15; i++) {
      await service.executeCommand(
        accountId,
        { type: 'DepositFunds', amountCents: 500, reference: `dep-${i}` },
        { actorId: ownerId },
      );
    }

    const snap = await store.loadLatestSnapshot(accountId, CURRENT_SNAPSHOT_FORMAT_VERSION);
    expect(snap).not.toBeNull();
    expect(snap?.streamVersion).toBe(10);

    const stateViaSnapshot = await service.loadAccount(accountId);
    const fullRaw = await store.readStream(accountId, 0);
    const fullTyped = await upcasters.upcastAll(fullRaw);
    const stateViaFullFold = rehydrateBankAccount(fullTyped);

    expect(stateViaSnapshot).toEqual(stateViaFullFold);
    expect(stateViaSnapshot.balanceCents).toBe(1_000 + 15 * 500);
    expect(stateViaSnapshot.version).toBe(16);
  });

  it('upcasts legacy v1 events and redacts PII deterministically after GDPR crypto-shredding', async () => {
    const accountId = randomUUID();
    const ownerId = randomUUID();

    // Insert a legacy 2024 schema_version = 1 event missing currency/tier
    await store.appendToStream(
      accountId,
      'BankAccount',
      0,
      [
        {
          eventType: 'AccountOpened',
          schemaVersion: 1,
          payload: {
            ownerId,
            ownerEmail: 'legacy@bank.io',
            initialDepositCents: 4_000,
          },
        },
      ],
      { actorId: 'legacy-migrator' },
    );

    const legacyState = await service.loadAccount(accountId);
    expect(legacyState.currency).toBe('USD');
    expect(legacyState.accountTier).toBe('STANDARD');
    expect(legacyState.ownerEmail).toBe('legacy@bank.io');

    // Now test v3 encrypted PII + GDPR shredding
    const encryptedAccId = randomUUID();
    const piiOwnerId = randomUUID();
    await service.openAccount({
      accountId: encryptedAccId,
      ownerId: piiOwnerId,
      ownerEmail: 'gdpr-user@bank.io',
      currency: 'EUR',
      initialDepositCents: 12_500,
      accountTier: 'PREMIER',
      actorId: piiOwnerId,
    });

    expect((await service.loadAccount(encryptedAccId)).ownerEmail).toBe('gdpr-user@bank.io');

    await shredder.shredSubjectKey(piiOwnerId);

    const afterShred = await service.loadAccount(encryptedAccId);
    expect(afterShred.ownerEmail).toBe('[ERASED]');
    expect(afterShred.balanceCents).toBe(12_500); // Monetary invariant preserved!
    expect(afterShred.currency).toBe('EUR');
  });

  it('guarantees projection rebuild parity and idempotency across replay', async () => {
    const accountId = randomUUID();
    const ownerId = randomUUID();
    await service.openAccount({
      accountId,
      ownerId,
      ownerEmail: 'replay@bank.io',
      currency: 'USD',
      initialDepositCents: 5_000,
      actorId: ownerId,
    });
    await service.executeCommand(
      accountId,
      { type: 'DepositFunds', amountCents: 3_000, reference: 'r-1' },
      { actorId: ownerId },
    );
    await service.executeCommand(
      accountId,
      { type: 'WithdrawFunds', amountCents: 1_500, reference: 'r-2' },
      { actorId: ownerId },
    );

    await projector.runOnce(100);
    const liveRow = await pool.query(
      'SELECT balance_cents, last_version, status FROM account_balances_read_model WHERE account_id = $1',
      [accountId],
    );
    expect(Number(liveRow.rows[0].balance_cents)).toBe(6_500);
    expect(liveRow.rows[0].last_version).toBe(3);

    // Wipe read table + reset checkpoint -> replay from 0 must match byte-for-byte
    const replayedCount = await projector.rebuildFromScratch();
    expect(replayedCount).toBe(3);

    const rebuiltRow = await pool.query(
      'SELECT balance_cents, last_version, status FROM account_balances_read_model WHERE account_id = $1',
      [accountId],
    );
    expect(rebuiltRow.rows[0]).toEqual(liveRow.rows[0]);
  });
});
```

---

## 4. Complete Construct Reference & Production Edge Cases

### Event Store Primitive Reference

| Primitive / Field | Exact Role | Production Standard |
|---|---|---|
| **`stream_id`** | Aggregate instance UUID | One stream per Aggregate boundary (`BankAccount:{uuid}`); never one global stream for all accounts |
| **`stream_version`** | `1, 2, 3...` per-stream ordinal | Strictly sequential without gaps; backed by `UNIQUE (stream_id, stream_version)` |
| **`global_position`** | Cluster-wide append sequence | Used strictly by catch-up projections and CDC connectors |
| **`tx_xid`** | Postgres `xid8` transaction ID | Prevents uncommitted lower-position transactions from causing catch-up gap skips |
| **`metadata`** | Envelope context JSONB | `correlation_id` (root flow), `causation_id` (triggering command/event), `actor_id`, `traceparent` |
| **`idempotency_key`** | Client command retry token | Scoped `UNIQUE (stream_id, idempotency_key)` prevents double-deposits on HTTP retries |

### Production Edge Cases & Structural Patterns

1. **Closing the Books (Bounding Unbounded Streams)**:
   - *Trap*: An active clearing house or utility billing account running for 10 years accumulates millions of events. Even with snapshots, storage partitioning and archival retention become unwieldy.
   - *Fix*: Partition streams along natural fiscal boundaries (`LedgerAccount:{id}:2026-11`). At month-end, append `AccountingPeriodClosed { closingBalanceCents }` to `2026-11` and immediately emit `AccountingPeriodOpened { openingBalanceCents }` at `stream_version = 1` of `2026-12`. Cold monthly partitions can be archived cleanly to S3/Parquet after legal retention checks.
2. **Event Granularity Trap ("Property-Changed" Anti-Pattern)**:
   - *Trap*: Storing generic CRUD field deltas (`CustomerUpdated { changedFields: { email: ... } }`) loses business causality, produces brittle projections, and recreates anemic CRUD with extra boilerplate.
   - *Fix*: Name events strictly as domain decisions (`BillingAddressChanged`, `SubscriptionUpgradedToAnnual`, `AccountSuspendedForChargeback`), carrying only the attributes integral to that domain fact.
3. **Hot-Stream OCC Contention Storm**:
   - *Trap*: Modeling a single global `DailyPromotionBudget` aggregate shared by 5,000 concurrent checkout workers causes massive `23505` OCC failures because only 1 transaction per stream can commit sequentially per round-trip.
   - *Fix*: Split coarse aggregates into fine-grained partitioned streams (`PromoShard:00..63`, routing users across 64 independent streams with split budget quotas) or reserve atomic Redis/Postgres row allocations before emitting finalized accounting events asynchronously.
4. **Live Process Manager vs Historical Projection Replay**:
   - *Trap*: A projection listener that updates `account_balances_read_model` also calls `emailService.sendLowBalanceAlert(...)`. When SREs run `rebuildFromScratch()`, 4 million historical alerts fire to customers!
   - *Fix*: Never combine read-model projections with external action dispatchers. Keep `BalanceProjectionDaemon` 100% side-effect-free. Run external side-effect **Reactors** against an outbox or persist an immutable `dispatched_notifications(event_id PRIMARY KEY)` ledger checked before any external API call.

---

## 5. Production Debugging, Profiling & Operations Playbook

| Symptom | Root Cause | Diagnostic SQL / Command | Production Mitigation |
|---|---|---|---|
| **Missing rows in `account_balances_read_model` after concurrent load** | Catch-up query missing `tx_xid < pg_snapshot_xmin(pg_current_snapshot())` skipped slow concurrent `BIGSERIAL` commits | Compare `SELECT stream_id, stream_version FROM event_streams EXCEPT SELECT account_id, last_version FROM account_balances_read_model;` | Add `tx_xid` visibility horizon filter to `readAllAfterPosition`; execute shadow Blue/Green rebuild to backfill skipped positions. |
| **Command p99 latency climbs linearly (`> 600 ms`) on older accounts** | Missing snapshots or `CURRENT_SNAPSHOT_FORMAT_VERSION` bumped without pre-warming hot streams | `SELECT e.stream_id, COUNT(*) AS tail_events FROM events e LEFT JOIN (SELECT stream_id, MAX(stream_version) AS sv FROM snapshots WHERE snapshot_format_version = 2 GROUP BY 1) s ON s.stream_id = e.stream_id WHERE e.stream_version > COALESCE(s.sv, 0) GROUP BY 1 ORDER BY 2 DESC LIMIT 20;` | Lower `snapshotInterval` (e.g., `100`), run background snapshot pre-warmer over top streams, or apply "Closing the Books" stream rotation. |
| **`TypeError` inside `apply()` immediately after deployment** | Newly deployed code expects `v3` payload field on legacy `v1`/`v2` event rows | Inspect failing `event_id`, `event_type`, `schema_version`, `payload` from `events WHERE stream_id = $1;` | Add missing pure transformation step in `UpcasterRegistry` (`EventType:v1 -> v2 -> v3`); verify with unit fixture containing raw legacy JSONB. |
| **Spike in HTTP `409 Conflict` / `23505` OCC errors** | Aggregate boundary encompasses unrelated concurrent actors (e.g., global counter stream) | `SELECT stream_id, COUNT(*) FROM events WHERE occurred_at > NOW() - INTERVAL '5 minutes' GROUP BY 1 ORDER BY 2 DESC LIMIT 10;` | Partition stream granularity (`PromoShard:{k}`) or add bounded OCC retry loop (`maxOccRetries = 3`) for commutative commands (`DepositFunds`). |
| **GDPR audit flags PII visible in historical stream** | Legacy pre-encryption `v1` events contain plaintext `email` alongside `v3` encrypted events | `SELECT COUNT(*) FROM events WHERE event_type = 'AccountOpened' AND schema_version = 1;` | Execute one-time encrypted stream rewrite (`events_v2` table encrypting legacy fields with per-subject DEKs), verify checksums, cut over, and purge old table + expired snapshots. |

---

## 6. Worked Exercises

### Exercise 1: Diagnosing a Silent Double-Spend, Impure `apply()` Crash, and Unbounded Fold
Inspect this Node/TypeScript Event-Sourced Wallet handler and identify **4 critical production failures**, then write the corrected implementation design:

```typescript
async function withdraw(walletId: string, amountCents: number) {
  const events = await db.query(
    'SELECT * FROM events WHERE stream_id = $1 ORDER BY stream_version ASC',
    [walletId],
  );
  const wallet = events.rows.reduce((acc, e) => {
    if (e.event_type === 'Withdrawn') {
      if (acc.balance - e.payload.amount < 0) throw new Error('Negative balance!');
      acc.balance -= e.payload.amount;
    }
    if (e.event_type === 'Deposited') acc.balance += e.payload.amount;
    return acc;
  }, { balance: 0 });

  if (wallet.balance < amountCents) throw new Error('Insufficient funds');

  await db.query(
    'INSERT INTO events (stream_id, stream_version, event_type, payload) VALUES ($1, $2, $3, $4)',
    [walletId, events.rows.length + 1, 'Withdrawn', { amount: amountCents }],
  );
  await emailClient.sendWithdrawalReceipt(walletId, amountCents);
}
```

---

### Solution 1

#### 4 Critical Production Failures
1. **Impure Validation Inside `apply` (`events.rows.reduce`)**: Throwing `'Negative balance!'` inside state reconstruction bricks any account whose historical stream ever allowed a temporary promotional overdraft or refund adjustment under older rules. Validation must execute **only** in `decide()` after rehydration finishes.
2. **Broken `stream_version` Calculation & Missing OCC Handling**: Computing `events.rows.length + 1` assumes zero event archival/truncation, and the handler never locks `event_streams`, never catches PostgreSQL `23505` (`OptimisticConcurrencyError`), and uses no `idempotency_key`—allowing duplicate withdrawals on client network retries.
3. **Unbounded Full-Stream Load**: Scanning from `stream_version = 1` without snapshots OOMs or times out once hot merchant wallets exceed tens of thousands of transactions.
4. **Unsafe Direct External Side Effect (`emailClient.sendWithdrawalReceipt`)**: Calling an external SMTP/HTTP service directly after commit risks split-brain failure (withdrawal succeeds, email crashes) and violates transactional outbox/reactor separation.

#### Corrected Production Design
- Load latest verified snapshot (`snapshot_format_version = CURRENT`) + tail `WHERE stream_version > $snapVersion`, upcast events, run pure `rehydrate()` (never throws), run `decide()`, append with `expectedVersion = state.version` + `idempotency_key` inside an ACID transaction that also inserts an outbox notification event, and catch `23505` as `OptimisticConcurrencyError` (`409 Conflict`).

---

### Exercise 2: Fixing the PostgreSQL `BIGSERIAL` Visibility Gap in a Catch-Up Projector
Your Ledger Event Store writes `1,500 events/sec` concurrently across 16 Node pods into PostgreSQL `events (global_position BIGSERIAL PRIMARY KEY, ...)`. A single catch-up projector polls every `50ms`:
```sql
SELECT * FROM events WHERE global_position > $1 ORDER BY global_position ASC LIMIT 500;
```
After running for 48 hours, finance audits reveal `17` `FundsDeposited` events exist in `events` (`global_position` values confirmed present), yet their corresponding `account_balances_read_model` rows were never credited and `projection_checkpoints.checkpoint_position` is well past those `global_position` numbers! Explain the exact PostgreSQL MVCC concurrency mechanism causing this data loss and implement the exact two-part fix.

---

### Solution 2

#### Exact MVCC Root Cause
PostgreSQL sequence generators (`BIGSERIAL` / `nextval()`) execute **non-transactionally** outside MVCC rollback/isolation:
1. Pod 1 begins `TX-1`, receives `global_position = 5000`, and takes `90ms` computing fraud/snapshot updates before `COMMIT`.
2. Pod 2 begins `TX-2` `2ms` later, receives `global_position = 5001`, and commits at `10ms`.
3. At `25ms`, the Catch-Up Projector executes `SELECT ... WHERE global_position > 4999`. PostgreSQL MVCC hides uncommitted `5000` and returns `[5001]`.
4. The projector applies `5001` and advances `checkpoint_position = 5001`. When `TX-1` commits `5000` at `90ms`, the projector's `> 5001` predicate permanently skips `5000`!

#### Two-Part Production Fix
1. **Add Transaction ID Column (`tx_xid xid8 DEFAULT pg_current_xact_id()`)**: Store the creating transaction's 64-bit XID on every `events` row with index `(global_position ASC, tx_xid)`.
2. **Filter by Cluster Visibility Horizon (`pg_snapshot_xmin`)**:
   ```sql
   SELECT *
   FROM events
   WHERE global_position > $1
     AND tx_xid < pg_snapshot_xmin(pg_current_snapshot())
   ORDER BY global_position ASC
   LIMIT 500;
   ```
   `pg_snapshot_xmin(pg_current_snapshot())` returns the lowest XID among **all currently active uncommitted transactions**. While `TX-1` remains uncommitted, `TX-2`'s `tx_xid` is `>= xmin`, so the query pauses `5001` in the result set until `TX-1` finishes committing (`5000` becomes visible first) or rolls back! Then run a shadow replay from `0` to recover the 17 missed accounts.

---

### Exercise 3: Designing a Zero-Downtime Blue/Green Projection Rebuild for 80M Events
A SaaS compliance team needs a new `monthly_merchant_settlements` table materialized from `80,000,000` historical events while production appends `1,200 events/sec`. An engineer suggests: *"Pause the current projector, `TRUNCATE` the table, change the SQL handler, and reset `checkpoint_position = 0`."* Calculate the downtime impact and design the zero-downtime Blue/Green Catch-Up architecture.

---

### Solution 3

#### Downtime Calculation
At `10,000 events/sec` batch replay throughput, replaying `80,000,000` events requires `8,000 seconds` ($\approx 2.22 \text{ hours}$). Truncating the active table blanks customer settlement dashboards completely for over 2 hours while falling further behind live `1,200/sec` ingress.

#### Zero-Downtime Blue/Green Catch-Up Architecture
1. **Provision Isolated Target (`monthly_merchant_settlements_v2`)**: Leave `v1` table and `'settlements_v1'` checkpoint running untouched in production.
2. **Register Independent Shadow Checkpoint (`'settlements_v2'`)**: Start a dedicated shadow backfill worker reading batches of `5,000` events (`WHERE global_position > $cp AND tx_xid < pg_snapshot_xmin(pg_current_snapshot())`) using high-throughput bulk SQL upserts (`UNNEST` arrays + `INSERT ... ON CONFLICT DO UPDATE ... WHERE last_version < EXCLUDED.last_version`) and defer secondary non-unique indexes until backfill finishes near tail.
3. **Tail Convergence**: Keep `'settlements_v2'` running until `max(events.global_position) - v2.checkpoint_position < 500` (`lag < 500ms`).
4. **Atomic View Swap**: Applications query `monthly_merchant_settlements_active_v`. Inside one metadata transaction execute `CREATE OR REPLACE VIEW monthly_merchant_settlements_active_v AS SELECT * FROM monthly_merchant_settlements_v2;`, keep `v1` warm for 24h rollback safety, then retire `'settlements_v1'` and drop `v1`.

---

### Exercise 4: Implementing Upcasting + GDPR Crypto-Shredding & Legacy Plaintext Remediation
An Event-Sourced customer stream contains 3 historical event schemas:
- `v1 (2024)`: `CustomerRegistered { fullName: "Ada Lovelace", email: "ada@acme.io" }` (stored plaintext).
- `v2 (2025)`: `CustomerRegistered { firstNameEnc, lastNameEnc, emailEnc, keyId: "dek_u42" }`.
- `v3 (2026)`: Same as `v2`, plus `marketingOptIn: boolean`.

User `u42` requests GDPR erasure.
1. Trace how the `v1 -> v2 -> v3` Upcaster + Crypto-Shredder executes safely on `v2`/`v3` events during projection rebuilds without crashing.
2. Explain the critical compliance flaw remaining in `2024 v1` rows and specify the remediation pipeline.

---

### Solution 4

#### 1. Upcaster Chain & Non-Throwing Redaction (`v2` & `v3`)
1. Delete `dek_hex` for `u42` in `subject_keys` (`SET dek_hex = NULL, shredded_at = NOW()`). Never alter `v2`/`v3` event rows in `events`.
2. During any stream read or projection replay:
   - `UpcasterRegistry` converts `v2` -> `v3` by injecting default `marketingOptIn: false`.
   - `CryptoShredder.decryptOrRedact(emailEnc)` finds `dek_hex === null` and returns `"[ERASED]"` instead of throwing a decryption error—allowing rehydration (`customerId`, `status`, `marketingOptIn`) and full projection rebuilds to run cleanly with zero PII leakage.

#### 2. Remediating Plaintext Legacy `v1` Events
Crypto-shredding only protects ciphertext. Plaintext `2024 v1` rows in `events` (and immutable PITR database backups) still contain `"ada@acme.io"`.
- **One-Time Offline Stream Rewrite ("Copy-and-Transform")**: Provision `events_v2`, stream all historical rows transforming `v1` plaintext PII into encrypted `v3` envelopes using newly generated per-subject DEKs (or immediate `"[ERASED]"` sentinels for already-deleted users), verify cryptographic ledger parity (`SUM` and `COUNT` assertions per stream), atomically swap tables (`events` <-> `events_v2`), drop `events_old`, and enforce backup retention expiration policies so pre-migration backups age out permanently.

---

### Exercise 5: Isolating Side Effects During Projection Replay
An engineering team adds an automated `AccountFrozen` webhook notification directly inside `BalanceProjectionDaemon.projectSingleEvent`:
```typescript
case 'AccountFrozen':
  await updateReadTable(...);
  await webhookClient.post('/merchant/account-frozen', { accountId: event.streamId });
```
Six months later, a new BI column forces rebuilding `account_balances_read_model` from `global_position = 1`. Every merchant whose account was *ever* frozen in the past 3 years receives duplicate webhook alerts, triggering automated fraud lockouts at partner banks! Explain the architectural violation and design a replay-safe side-effect pipeline.

---

### Solution 5

#### Architectural Violation
**Conflating State Projections with Side-Effect Process Reactors**: Projections derive queryable local tables by folding history deterministically and **must be 100% pure local database state transitions** safe to execute 100 times from position `0`. External HTTP calls, emails, and partner webhooks are **non-idempotent side effects**.

#### Replay-Safe Architecture
1. **Remove All External I/O from `BalanceProjectionDaemon`**: Projections touch **only** their owned read table and `projection_checkpoints`. Replaying `80M` events produces zero network traffic.
2. **Create a Separate Live Policy Reactor (`AccountFrozenWebhookReactor`)** with two independent safety gates:
   - **High-Watermark / Bootstrap Gate**: When initializing a brand-new reactor in production, seed its checkpoint at the **current live head** `MAX(global_position)` rather than `0` (so historical events from 2024 are never treated as new live alerts).
   - **Outbox + External Idempotency Ledger**: Persist `dispatched_webhooks(event_id UUID PRIMARY KEY, dispatched_at TIMESTAMPTZ)` checked inside the reactor's checkpoint transaction (`INSERT ... ON CONFLICT DO NOTHING RETURNING 1`), and attach `Idempotency-Key: event.eventId` on the outbound HTTP POST header—guaranteeing at-most-once webhook dispatch even across reactor crashes or manual checkpoint rollbacks.

---

### Exercise 6: Solving High-Contention OCC Retry Storms on a Single Escrow Stream
An Event-Sourced marketplace credits a 2% platform commission on every order into a **single shared stream** `PlatformCommissionStream` (`stream_id = '00000000-...-0001'`). During flash sales (`800 orders/sec`), `94%` of checkout commands fail with `OptimisticConcurrencyError (23505)` and exhaust retries because every order competes for `stream_version + 1` on the exact same row in `event_streams`. Diagnose why Event Sourcing caused this serialization bottleneck and redesign the stream topology.

---

### Solution 6

#### Root Cause
In an Event Store, the **Stream (`stream_id`) IS the Consistency & Concurrency Boundary**. Funneling `800 writes/sec` from completely independent customer checkouts into a single `stream_id` forces strict global serial execution across one locked PostgreSQL row (`FOR UPDATE` + `UNIQUE (stream_id, stream_version)`). Even with a `2ms` transaction time, theoretical maximum throughput on one row is capped near `500 tx/sec`, causing massive contention queues and `23505` cascades.

#### Stream Redesign
1. **Remove Global Stream Coupling from Checkout Path**: A customer's `Order` checkout does **not** share an invariant with total platform commissions (`CommissionEarned` cannot fail because another customer paid $1!). Record `CommissionAssessed` inside the individual **Order stream** (`Order:{orderId}`, horizontal concurrency = $\infty$).
2. **Sharded Aggregation / Closing the Books (If a Ledger Stream Is Required)**: If accounting requires append-only ledger streams for commission postings, hash `orderId` across $K = 64$ independent shard streams (`CommissionBucket:2026-11-13:shard-00..63`), reducing contention per stream by $64\times$ (`~12.5 writes/sec/shard`), and roll up total daily revenue asynchronously via a **CQRS Projection** (`daily_commission_totals` keyed by `event_id` idempotency)!
