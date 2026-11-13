---
tags:
  - software-engineering
  - architectural-patterns
  - hexagonal-architecture
  - ports-and-adapters
  - nestjs
  - typescript
created: 2026-01-02
updated: 2026-11-13
status: 🟢
level: mid-to-senior
---
# Hexagonal Architecture (Ports & Adapters)

> [!abstract] Vault Navigation
> Part of [[Programming/Software Engineering/Architectural Patterns/Architectural Patterns Complete Guide|Architectural Patterns Complete Guide]]. Shares the inward dependency invariant with [[Programming/Software Engineering/Clean Architecture/Clean Architecture Complete Guide|Clean Architecture Complete Guide]], [[Programming/Software Engineering/Clean Architecture/Dependency Rule|Dependency Rule]], and [[Programming/Software Engineering/SOLID/Dependency Inversion|Dependency Inversion]]. Pairs operationally with [[Programming/Software Engineering/Design Patterns/Structural/Adapter|Adapter]], [[Programming/Software Engineering/Architectural Patterns/CQRS|CQRS]], and [[Programming/Backend/NestJS Complete Guide|NestJS Complete Guide]].

Proposed by Alistair Cockburn in 2005, **Hexagonal Architecture (Ports & Adapters)** partitions a software system into two strictly asymmetric dependency zones—an **Inside (Application + Domain Core)** and an **Outside (External Actors & Infrastructure Adapters)**—governed by a completely **symmetric operational boundary**. Unlike traditional horizontal layered systems that place a UI at the "top" and a database at the "bottom," Hexagonal Architecture models every external stimulus and every external side effect identically as actors living outside the polygon, communicating with the core strictly through explicit **Ports** implemented or invoked by replaceable **Adapters**.

---

## 1. First-Principles Mechanics & Complete Architecture

### 1.1 The Symmetric Inside/Outside Boundary Model

In vertical N-tier stacks, the database sits beneath business logic, creating a mental bias where storage engines are treated as structural foundations rather than volatile external I/O boundaries. Hexagonal Architecture eliminates top-versus-bottom hierarchy. Every piece of code whose lifecycle belongs to an external protocol, transport, clock, filesystem, database engine, vendor SDK, or test harness lives **outside** the hexagon.

The polygon metaphor carries two concrete engineering invariants (not six mandatory sides—Cockburn chose a hexagon simply to provide enough visual faces for diverse external actors):

1. **Every Boundary Crossing Requires an Explicit Port**: The inside core defines named protocols (**Ports**) specifying what external capabilities it exposes or requires using **strictly domain vocabulary**.
2. **All Source-Code Dependencies Point Inward**: Adapters import Ports and Domain types; Ports and Domain Entities never import HTTP frameworks, SQL drivers, ORMs, or third-party vendor SDKs.

```mermaid
flowchart LR
    HTTP["HTTP / gRPC Adapter"] -->|"Invokes"| IN["Driving Port (UseCase)"]
    CRON["CLI / Cron / Test"] -->|"Invokes"| IN
    IN --> CORE["Application + Domain Core"]
    CORE -->|"Calls"| OUT["Driven Port (SPI)"]
    PG["Postgres / Redis"] -.->|"Implements"| OUT
    STRIPE["Stripe / Fake"] -.->|"Implements"| OUT
```

### 1.2 Driving (Primary) vs Driven (Secondary) Mechanics

Confusion in production Hexagonal implementations almost always stems from inverting **who owns the interface** versus **who initiates the function call** at runtime.

| Dimension | Driving Side (Primary) | Driven Side (Secondary) |
|---|---|---|
| **Alternative Terminology** | Inbound Port + Primary Adapter | Outbound Port + Secondary Adapter / SPI |
| **Runtime Control Flow** | Outside Actor $\rightarrow$ Primary Adapter $\rightarrow$ Driving Port $\rightarrow$ Use Case | Use Case $\rightarrow$ Driven Port $\rightarrow$ Secondary Adapter $\rightarrow$ External I/O |
| **Compile-Time Source Ownership** | Port interface and Use-Case Command/Result DTOs live **inside Core** | Port interface and Entity/Value Object arguments live **inside Core** |
| **Common Production Adapters** | Fastify/Express/NestJS HTTP controllers, gRPC server handlers, GraphQL resolvers, CLI batch runners, Kafka/BullMQ consumers, Vitest test suites | PostgreSQL/Prisma/Drizzle repositories, Redis read-through decorators, Stripe/Adyen HTTP clients, SMTP/SES mailers, system clocks, in-memory fakes |
| **Adapter Responsibility** | Parse transport framing, verify signatures/JWTs, map external payload into a Domain Command, invoke Port, translate domain outcome or error to HTTP/gRPC status | Translate Domain Entities/Commands into SQL statements, Redis keys, or HTTP payloads; handle retries/timeouts; map vendor exceptions into domain outcomes |

Trace both runtime call stacks inside a single subscription renewal operation:

1. **Driving Invocation**: A NestJS controller (`SubscriptionRenewalHttpController`) or a scheduled CLI job (`RenewalBatchCliRunner`) parses its input, constructs a pure `ProcessRenewalCommand`, and calls `processRenewalUseCase.execute(cmd)`. Neither the HTTP request object nor CLI `process.argv` crosses the Driving Port boundary.
2. **Core Orchestration**: `ProcessRenewalService` (inside Core) invokes `subscriptionRepo.findById(id)` and `clock.now()` through Driven Port interfaces. At runtime, polymorphism dispatches directly into `PgSubscriptionRepository` or `InMemorySubscriptionRepository`.
3. **Driven Response Translation**: `StripePaymentGatewayAdapter` catches `Stripe.errors.StripeCardError` inside the outer adapter, maps it to `{ status: 'DECLINED', declineCode: 'INSUFFICIENT_FUNDS', retryable: false }`, and returns pure domain data back into Core.

### 1.3 Port/DTO Ownership Rules & the Anti-Corruption Boundary

A port that resides in `core/ports/` yet accepts a framework type provides **zero architectural isolation**. Examine the three critical Port/DTO ownership rules:

1. **Zero ORM/Prisma/TypeORM Types in Ports**: If `SubscriptionRepositoryPort.save(sub: Prisma.SubscriptionCreateInput)` references Prisma's generated namespace, upgrading Prisma or normalizing a SQL join table breaks Core compilation and forces every test harness to construct Prisma payload graphs.
2. **Zero HTTP/Transport Types in Driving Ports**: If `ProcessRenewalUseCase.execute(req: FastifyRequest)` accepts Fastify or Express objects, invoking that same use case from a cron batch runner or Kafka consumer becomes impossible without fabricating mock HTTP sockets.
3. **Zero Vendor SDK Types in Driven Ports (Anti-Corruption Layer)**: A `PaymentGatewayPort` must never accept `Stripe.PaymentIntentCreateParams` or return `Stripe.Response<Stripe.PaymentIntent>`. Translating between vendor wire formats and clean domain contracts at the adapter boundary forms an **Anti-Corruption Layer (ACL)**.

#### The Mapping Tax vs CQRS Read Bypass

Pure Hexagonal Ports impose a measurable engineering cost: translating `SQL Row -> Domain Entity -> Response DTO` on every endpoint allocates extra objects, maintains mapper boilerplate, and degrades latency on read-heavy endpoints joining six tables.

Senior architectures resolve this trade-off deliberately using [[Programming/Software Engineering/Architectural Patterns/CQRS|CQRS]] read segregation:
- **Write Commands (Mutations)**: Route strictly through **Rich Domain Entities** and **Secondary Write Ports** (`findById`, `save`). Here invariant protection (`PAST_DUE -> SUSPENDED`, idempotent billing cycles, optimistic concurrency) justifies domain hydration and ACL mapping.
- **Read Queries (Dashboards, Search, CSV Exports)**: Define a narrow **Read Query Port** (`SubscriptionSummaryQueryPort`) whose adapter runs an optimized flat SQL `SELECT ... JOIN` returning a flat `SubscriptionSummaryView` DTO directly—bypassing domain entity hydration and eliminating the read-path mapper tax while keeping the Core boundary intact.

```mermaid
flowchart LR
    CMD["Command Adapter"] -->|"Write Port"| CORE["Domain Entity + Write Port"]
    CORE -->|"Hydrate & Persist"| WDB[("Write Tables")]
    QRY["Query Adapter"] -->|"Read Port (CQRS Bypass)"| SQL["Flat SQL Read Adapter"]
    SQL -->|"Direct Projection"| RDB[("Read Replica / Views")]
```

### 1.4 Composition Root Wiring Mechanics

Because Core code must never import concrete Adapters, **wiring concrete implementations to abstract Ports happens exclusively at the Composition Root**—the outermost module executed at process boot (`main.ts` / `AppModule`).

Two idiomatic wiring topologies dominate TypeScript systems:

| Approach | Mechanism | Strengths | Failure Risks |
|---|---|---|---|
| **Pure DI (Manual Factory)** | Plain TypeScript function instantiates adapters (`new PgSubscriptionRepository(pool)`) and injects them via `new ProcessRenewalService(repo, gateway, clock)` | Zero framework lock-in, compile-time verification of every constructor argument, millisecond cold boot, trivial stack traces | Verbose manual wiring once graph exceeds 40+ services without sub-factories |
| **NestJS IoC Container** | `Symbol` injection tokens (`SUBSCRIPTION_REPO_PORT`) bound via `useClass` or `useFactory` inside feature `@Module` metadata | Integrates cleanly with lifecycle hooks (`OnModuleDestroy`), request scopes, and config injection; overrides trivially in `Test.createTestingModule` | Runtime `UnknownDependenciesException` if token forgotten; developers accidentally injecting `@InjectRepository` concrete classes directly into services |

### 1.5 Hexagonal vs Onion vs Clean Architecture vs Vertical Slice

| Dimension | Hexagonal (Ports & Adapters) | Onion Architecture | Clean Architecture | Vertical Slice Architecture |
|---|---|---|---|---|
| **Primary Organizing Principle** | Symmetry of outside actors around a Core connected via Ports | Concentric inward rings centered on DDD Domain Model | Four strict rings with explicit Interactors & Output Presenters | Feature/endpoint cohesion along axes of change |
| **Internal Core Prescription** | Unopinionated (use DDD Aggregates, Transaction Scripts, or Slices inside) | Prescribes Domain Model $\rightarrow$ Domain Services $\rightarrow$ App Services | Prescribes Enterprise Entities + Use Case Interactors | Per-slice implementation freedom per endpoint |
| **Adapter Symmetry** | Explicit Driving vs Driven Port parity across REST, CLI, Queue, DB, API | Focuses heavily on infrastructure inversion rather than multi-driver parity | Separates Controllers from Presenters symmetrically | Couples HTTP route, validation, and SQL in one slice |
| **Boilerplate Overhead** | Moderate (Ports + ACL mappers on boundaries) | Moderate–High | Highest (Input/Output Ports + Presenters + ViewModels) | Lowest per endpoint |
| **Best Architectural Fit** | Services swapping I/O providers or driven by REST + Cron + Workers | Complex DDD enterprise monoliths | Multi-team regulated enterprise cores | High-churn SaaS APIs with light-to-moderate cross-endpoint overlap |

---

## 2. Real-World Use Cases & Decision Matrix

### 2.1 Concrete Adoption Scenarios

1. **Multi-Driver Subscription Billing & Settlement Engines**: When the exact same renewal state machine must execute from an interactive REST `POST /renew` retry endpoint, a nightly Kubernetes CronJob, and a Stripe `invoice.payment_failed` webhook consumer, a single Driving Port (`ProcessRenewalUseCase`) prevents split business rules across controllers and workers.
2. **Volatile Vendor SDK Abstraction (Payments, LLMs, Logistics)**: Production systems integrating Stripe plus Adyen fallback, or swapping OpenAI/Anthropic/Vertex model providers, insulate core workflow pricing and retry policies behind clean Driven Ports (`PaymentGatewayPort`, `InferencePort`). Swapping vendors alters only one adapter file.
3. **Time-Sensitive & Non-Deterministic Business Rules**: Grace-period expirations, SLA timers, and token TTL checks fail unpredictably under tests calling `Date.now()` directly. Modeling `ClockPort` as a Driven Port makes temporal boundary tests (`23:59:59Z` vs `00:00:01Z` 7 days later) instantaneous and deterministic.
4. **Zero-Docker Sub-Second Core Test Suites**: When integration pipelines take 12 minutes to spin up Postgres, Redis, and Stripe mocks for every business logic edge case, swapping Driven Ports for stateful In-Memory Fakes runs 1,000 renewal scenarios in under 150ms.

### 2.2 Decision Matrix: When to Adopt vs When to Avoid

| System Profile | Verdict | Engineering Rationale & Trade-Off |
|---|---|---|
| **Core Billing, Ledger, Trading, or Workflow State Machine** | **Adopt Full Hexagonal** | Complex state transitions, strict idempotency, external payment/ledger APIs, and multi-trigger invocation justify Port + ACL separation |
| **SaaS Backend Integrating 3+ Swappable External Providers** | **Adopt Hexagonal** | Anti-Corruption Adapters isolate SDK breaking changes and provider fallbacks cleanly |
| **Internal CRUD Admin Panel / Backoffice Form Service** | **Do NOT Use** | Defining Driving/Driven interfaces + ACL mappers around simple `INSERT`/`SELECT` endpoints triples file count with zero protective gain; use [[Programming/Software Engineering/Architectural Patterns/Vertical Slice Architecture\|Vertical Slice Architecture]] or N-tier |
| **Read-Heavy Analytics & Search Endpoints (`>10:1` Read/Write)** | **Hybrid: CQRS Read Bypass** | Keep write paths Hexagonal; bypass domain entity mapping on reads via narrow Query Ports returning flat SQL projections |
| **Latency-Critical HFT / Streaming Packet Pipeline (`<500us`)** | **Avoid Abstract Port Tax** | Virtual dispatch and boundary DTO allocation cycles add unnecessary GC pressure; keep I/O buffers zero-copy |

---

## 3. Step-by-Step Practical Application: Subscription Billing Hexagonal Core

This section constructs a complete, production-ready **Subscription Billing Hexagonal Core** in modern TypeScript (`Node 22` / `NestJS` / `pg` / `stripe` / `zod` / `vitest`). Every file is self-contained, strictly typed, and free of placeholder stubs.

### 3.1 Project Directory Structure

Notice how the directory tree enforces the inside/outside split structurally: `core/` has zero dependencies on `adapters/` or `composition/`.

```text
src/billing/
  core/
    domain/
      money.vo.ts
      subscription.entity.ts
      billing.errors.ts
    ports/
      driving/
        process-renewal.use-case.ts
      driven/
        subscription-repository.port.ts
        payment-gateway.port.ts
        clock.port.ts
        unit-of-work.port.ts
    application/
      process-renewal.service.ts
  adapters/
    primary/
      http/
        process-renewal.controller.ts
      cli/
        renewal-batch.runner.ts
    secondary/
      postgres/
        pg-subscription.repository.ts
        pg-unit-of-work.ts
      stripe/
        stripe-payment-gateway.adapter.ts
      clock/
        system-clock.adapter.ts
      fakes/
        in-memory-subscription.repository.ts
        fake-payment-gateway.adapter.ts
        fixed-clock.adapter.ts
  composition/
    pure-di.factory.ts
    billing.module.ts
  tests/
    process-renewal.unit.spec.ts
    pg-subscription.integration.spec.ts
```

### 3.2 Pure Domain Value Objects, Aggregate Entity & Domain Errors

First, build the innermost Domain Model in `src/billing/core/domain/`. Notice integer-cent arithmetic (`Money`) to prevent IEEE-754 floating-point drift and strict state transition methods inside `Subscription`.

```typescript
// src/billing/core/domain/billing.errors.ts
export abstract class BillingDomainError extends Error {
  abstract readonly code: string;

  protected constructor(message: string) {
    super(message);
    this.name = new.target.name;
  }
}

export class InvalidMoneyError extends BillingDomainError {
  readonly code = 'INVALID_MONEY';
  constructor(message: string) {
    super(message);
  }
}

export class SubscriptionNotFoundError extends BillingDomainError {
  readonly code = 'SUBSCRIPTION_NOT_FOUND';
  constructor(readonly subscriptionId: string) {
    super(`Subscription ${subscriptionId} does not exist`);
  }
}

export class InvalidSubscriptionStateError extends BillingDomainError {
  readonly code = 'INVALID_SUBSCRIPTION_STATE';
  constructor(message: string) {
    super(message);
  }
}

export class RenewalNotDueYetError extends BillingDomainError {
  readonly code = 'RENEWAL_NOT_DUE_YET';
  constructor(
    readonly subscriptionId: string,
    readonly nextBillingAt: Date,
  ) {
    super(`Subscription ${subscriptionId} is not due until ${nextBillingAt.toISOString()}`);
  }
}

export class OptimisticConcurrencyError extends BillingDomainError {
  readonly code = 'OPTIMISTIC_CONCURRENCY_CONFLICT';
  constructor(readonly subscriptionId: string) {
    super(`Concurrent modification detected for subscription ${subscriptionId}`);
  }
}
```

```typescript
// src/billing/core/domain/money.vo.ts
import { InvalidMoneyError } from './billing.errors';

export type SupportedCurrency = 'USD' | 'EUR' | 'GBP';

export class Money {
  private constructor(
    readonly amountCents: number,
    readonly currency: SupportedCurrency,
  ) {
    Object.freeze(this);
  }

  static fromCents(amountCents: number, currency: SupportedCurrency): Money {
    if (!Number.isInteger(amountCents)) {
      throw new InvalidMoneyError(`Amount must be an integer in minor units, received ${amountCents}`);
    }
    if (amountCents < 0) {
      throw new InvalidMoneyError(`Money cannot be negative: ${amountCents}`);
    }
    return new Money(amountCents, currency);
  }

  isZero(): boolean {
    return this.amountCents === 0;
  }

  equals(other: Money): boolean {
    return this.amountCents === other.amountCents && this.currency === other.currency;
  }
}
```

```typescript
// src/billing/core/domain/subscription.entity.ts
import {
  InvalidSubscriptionStateError,
  RenewalNotDueYetError,
} from './billing.errors';
import { Money, SupportedCurrency } from './money.vo';

export type SubscriptionStatus =
  | 'ACTIVE'
  | 'RENEWAL_PENDING'
  | 'PAST_DUE'
  | 'SUSPENDED'
  | 'CANCELED';

export interface SubscriptionProps {
  id: string;
  customerId: string;
  paymentMethodToken: string;
  planCode: string;
  price: Money;
  billingCycleDays: number;
  status: SubscriptionStatus;
  currentPeriodEnd: Date;
  failedAttempts: number;
  lastRenewalCycleKey: string | null;
  version: number;
}

export interface SubscriptionSnapshot {
  id: string;
  customerId: string;
  paymentMethodToken: string;
  planCode: string;
  amountCents: number;
  currency: SupportedCurrency;
  billingCycleDays: number;
  status: SubscriptionStatus;
  currentPeriodEnd: Date;
  failedAttempts: number;
  lastRenewalCycleKey: string | null;
  version: number;
}

const MAX_RETRY_ATTEMPTS = 3;
const MS_PER_DAY = 86_400_000;

export class Subscription {
  private constructor(private props: SubscriptionProps) {}

  static rehydrate(snapshot: SubscriptionSnapshot): Subscription {
    return new Subscription({
      id: snapshot.id,
      customerId: snapshot.customerId,
      paymentMethodToken: snapshot.paymentMethodToken,
      planCode: snapshot.planCode,
      price: Money.fromCents(snapshot.amountCents, snapshot.currency),
      billingCycleDays: snapshot.billingCycleDays,
      status: snapshot.status,
      currentPeriodEnd: new Date(snapshot.currentPeriodEnd.getTime()),
      failedAttempts: snapshot.failedAttempts,
      lastRenewalCycleKey: snapshot.lastRenewalCycleKey,
      version: snapshot.version,
    });
  }

  get id(): string {
    return this.props.id;
  }
  get customerId(): string {
    return this.props.customerId;
  }
  get paymentMethodToken(): string {
    return this.props.paymentMethodToken;
  }
  get price(): Money {
    return this.props.price;
  }
  get status(): SubscriptionStatus {
    return this.props.status;
  }
  get currentPeriodEnd(): Date {
    return new Date(this.props.currentPeriodEnd.getTime());
  }
  get failedAttempts(): number {
    return this.props.failedAttempts;
  }
  get version(): number {
    return this.props.version;
  }

  deriveCycleIdempotencyKey(): string {
    const periodDate = this.props.currentPeriodEnd.toISOString().slice(0, 10);
    return `ren_${this.props.id}_${periodDate}_${this.props.failedAttempts}`;
  }

  hasAlreadyCompletedCycle(cycleKey: string): boolean {
    return (
      this.props.lastRenewalCycleKey === cycleKey &&
      this.props.status === 'ACTIVE'
    );
  }

  beginRenewalAttempt(now: Date): string {
    if (this.props.status === 'CANCELED' || this.props.status === 'SUSPENDED') {
      throw new InvalidSubscriptionStateError(
        `Cannot renew subscription ${this.props.id} in terminal status ${this.props.status}`,
      );
    }
    if (this.props.status === 'RENEWAL_PENDING') {
      throw new InvalidSubscriptionStateError(
        `Subscription ${this.props.id} already has an in-flight renewal`,
      );
    }
    if (now.getTime() < this.props.currentPeriodEnd.getTime()) {
      throw new RenewalNotDueYetError(this.props.id, this.props.currentPeriodEnd);
    }

    const cycleKey = this.deriveCycleIdempotencyKey();
    this.props = {
      ...this.props,
      status: 'RENEWAL_PENDING',
    };
    return cycleKey;
  }

  markRenewalSucceeded(now: Date, cycleKey: string): void {
    if (this.props.status !== 'RENEWAL_PENDING') {
      throw new InvalidSubscriptionStateError(
        `Expected RENEWAL_PENDING before completing renewal, got ${this.props.status}`,
      );
    }
    const baseMs = Math.max(now.getTime(), this.props.currentPeriodEnd.getTime());
    const nextPeriodEnd = new Date(baseMs + this.props.billingCycleDays * MS_PER_DAY);

    this.props = {
      ...this.props,
      status: 'ACTIVE',
      currentPeriodEnd: nextPeriodEnd,
      failedAttempts: 0,
      lastRenewalCycleKey: cycleKey,
    };
  }

  markRenewalFailed(): void {
    if (this.props.status !== 'RENEWAL_PENDING') {
      throw new InvalidSubscriptionStateError(
        `Expected RENEWAL_PENDING before recording failure, got ${this.props.status}`,
      );
    }
    const nextAttempts = this.props.failedAttempts + 1;
    const nextStatus: SubscriptionStatus =
      nextAttempts >= MAX_RETRY_ATTEMPTS ? 'SUSPENDED' : 'PAST_DUE';

    this.props = {
      ...this.props,
      status: nextStatus,
      failedAttempts: nextAttempts,
    };
  }

  toSnapshot(): SubscriptionSnapshot {
    return {
      id: this.props.id,
      customerId: this.props.customerId,
      paymentMethodToken: this.props.paymentMethodToken,
      planCode: this.props.planCode,
      amountCents: this.props.price.amountCents,
      currency: this.props.price.currency,
      billingCycleDays: this.props.billingCycleDays,
      status: this.props.status,
      currentPeriodEnd: new Date(this.props.currentPeriodEnd.getTime()),
      failedAttempts: this.props.failedAttempts,
      lastRenewalCycleKey: this.props.lastRenewalCycleKey,
      version: this.props.version,
    };
  }
}
```

### 3.3 Driving & Driven Ports (Owned Strictly by Core)

Notice that every port signature imports **only domain types**—never `pg`, `stripe`, `@nestjs/common`, or `express`.

```typescript
// src/billing/core/ports/driving/process-renewal.use-case.ts
import { SubscriptionStatus } from '../../domain/subscription.entity';

export interface ProcessRenewalCommand {
  subscriptionId: string;
  correlationId: string;
}

export interface ProcessRenewalOutcome {
  subscriptionId: string;
  outcome: 'RENEWED' | 'PAYMENT_FAILED' | 'SUSPENDED' | 'ALREADY_RENEWED';
  resultingStatus: SubscriptionStatus;
  nextPeriodEnd: Date;
  providerTransactionId: string | null;
  declineReason: string | null;
}

export const PROCESS_RENEWAL_USE_CASE = Symbol('PROCESS_RENEWAL_USE_CASE');

export interface ProcessRenewalUseCase {
  execute(command: ProcessRenewalCommand): Promise<ProcessRenewalOutcome>;
}
```

```typescript
// src/billing/core/ports/driven/subscription-repository.port.ts
import { Subscription } from '../../domain/subscription.entity';

export const SUBSCRIPTION_REPOSITORY_PORT = Symbol('SUBSCRIPTION_REPOSITORY_PORT');

export interface SubscriptionRepositoryPort {
  findById(subscriptionId: string): Promise<Subscription | null>;
  findDueSubscriptionIds(asOf: Date, limit: number): Promise<string[]>;
  saveWithOptimisticLock(subscription: Subscription): Promise<void>;
}
```

```typescript
// src/billing/core/ports/driven/payment-gateway.port.ts
import { Money } from '../../domain/money.vo';

export interface ChargePaymentCommand {
  customerId: string;
  paymentMethodToken: string;
  amount: Money;
  idempotencyKey: string;
  correlationId: string;
}

export type ChargePaymentResult =
  | {
      status: 'SUCCEEDED';
      providerTransactionId: string;
    }
  | {
      status: 'DECLINED';
      declineCode: string;
      retryable: boolean;
    };

export const PAYMENT_GATEWAY_PORT = Symbol('PAYMENT_GATEWAY_PORT');

export interface PaymentGatewayPort {
  charge(command: ChargePaymentCommand): Promise<ChargePaymentResult>;
}
```

```typescript
// src/billing/core/ports/driven/clock.port.ts
export const CLOCK_PORT = Symbol('CLOCK_PORT');

export interface ClockPort {
  now(): Date;
}
```

```typescript
// src/billing/core/ports/driven/unit-of-work.port.ts
import { SubscriptionRepositoryPort } from './subscription-repository.port';

export const UNIT_OF_WORK_PORT = Symbol('UNIT_OF_WORK_PORT');

export interface BillingTransactionPorts {
  subscriptions: SubscriptionRepositoryPort;
}

export interface UnitOfWorkPort {
  runInTransaction<T>(work: (ports: BillingTransactionPorts) => Promise<T>): Promise<T>;
}
```

### 3.4 Application Core Service (Two-Phase State Reservation Pattern)

Observe a critical senior pattern here: **never hold a PostgreSQL transaction open while calling Stripe over HTTPS**. `ProcessRenewalService` splits persistence into two ultra-short atomic transactions using optimistic locking (`version`) + the `RENEWAL_PENDING` state, with an out-of-band external charge in between.

```typescript
// src/billing/core/application/process-renewal.service.ts
import { SubscriptionNotFoundError } from '../domain/billing.errors';
import { Subscription } from '../domain/subscription.entity';
import { ClockPort } from '../ports/driven/clock.port';
import { PaymentGatewayPort } from '../ports/driven/payment-gateway.port';
import { UnitOfWorkPort } from '../ports/driven/unit-of-work.port';
import {
  ProcessRenewalCommand,
  ProcessRenewalOutcome,
  ProcessRenewalUseCase,
} from '../ports/driving/process-renewal.use-case';

export class ProcessRenewalService implements ProcessRenewalUseCase {
  constructor(
    private readonly unitOfWork: UnitOfWorkPort,
    private readonly paymentGateway: PaymentGatewayPort,
    private readonly clock: ClockPort,
  ) {}

  async execute(command: ProcessRenewalCommand): Promise<ProcessRenewalOutcome> {
    const startedAt = this.clock.now();

    // Phase 1: Short atomic TX reserves RENEWAL_PENDING using optimistic version lock.
    // Releases DB connection before any external network call begins.
    const reservation = await this.unitOfWork.runInTransaction(async ({ subscriptions }) => {
      const subscription = await subscriptions.findById(command.subscriptionId);
      if (!subscription) {
        throw new SubscriptionNotFoundError(command.subscriptionId);
      }

      const cycleKey = subscription.deriveCycleIdempotencyKey();
      if (subscription.hasAlreadyCompletedCycle(cycleKey)) {
        return { alreadyCompleted: true as const, subscription, cycleKey };
      }

      const reservedCycleKey = subscription.beginRenewalAttempt(startedAt);
      await subscriptions.saveWithOptimisticLock(subscription);
      return { alreadyCompleted: false as const, subscription, cycleKey: reservedCycleKey };
    });

    if (reservation.alreadyCompleted) {
      return {
        subscriptionId: reservation.subscription.id,
        outcome: 'ALREADY_RENEWED',
        resultingStatus: reservation.subscription.status,
        nextPeriodEnd: reservation.subscription.currentPeriodEnd,
        providerTransactionId: null,
        declineReason: null,
      };
    }

    // Phase 2: External payment API call runs outside any PostgreSQL lock/transaction.
    const subSnapshot = reservation.subscription.toSnapshot();
    const chargeResult = await this.paymentGateway.charge({
      customerId: subSnapshot.customerId,
      paymentMethodToken: subSnapshot.paymentMethodToken,
      amount: reservation.subscription.price,
      idempotencyKey: reservation.cycleKey,
      correlationId: command.correlationId,
    });

    const finishedAt = this.clock.now();

    // Phase 3: Short atomic TX commits final state transition.
    const finalized = await this.unitOfWork.runInTransaction(async ({ subscriptions }) => {
      const fresh = await subscriptions.findById(command.subscriptionId);
      if (!fresh) {
        throw new SubscriptionNotFoundError(command.subscriptionId);
      }

      if (chargeResult.status === 'SUCCEEDED') {
        fresh.markRenewalSucceeded(finishedAt, reservation.cycleKey);
      } else {
        fresh.markRenewalFailed();
      }
      await subscriptions.saveWithOptimisticLock(fresh);
      return fresh;
    });

    if (chargeResult.status === 'SUCCEEDED') {
      return {
        subscriptionId: finalized.id,
        outcome: 'RENEWED',
        resultingStatus: finalized.status,
        nextPeriodEnd: finalized.currentPeriodEnd,
        providerTransactionId: chargeResult.providerTransactionId,
        declineReason: null,
      };
    }

    return {
      subscriptionId: finalized.id,
      outcome: finalized.status === 'SUSPENDED' ? 'SUSPENDED' : 'PAYMENT_FAILED',
      resultingStatus: finalized.status,
      nextPeriodEnd: finalized.currentPeriodEnd,
      providerTransactionId: null,
      declineReason: chargeResult.declineCode,
    };
  }
}
```

### 3.5 Primary (Driving) Adapters: HTTP Controller & Scheduled CLI Runner

Notice how both adapters drive the **exact same `ProcessRenewalUseCase` port**, handling only transport parsing, correlation ID extraction, and protocol-specific status mapping.

```typescript
// src/billing/adapters/primary/http/process-renewal.controller.ts
import {
  BadRequestException,
  ConflictException,
  Controller,
  Headers,
  HttpCode,
  Inject,
  NotFoundException,
  Param,
  Post,
  UnprocessableEntityException,
} from '@nestjs/common';
import { z } from 'zod';
import {
  InvalidSubscriptionStateError,
  OptimisticConcurrencyError,
  RenewalNotDueYetError,
  SubscriptionNotFoundError,
} from '../../../core/domain/billing.errors';
import {
  PROCESS_RENEWAL_USE_CASE,
  ProcessRenewalUseCase,
} from '../../../core/ports/driving/process-renewal.use-case';

const SubscriptionIdSchema = z.string().uuid();

@Controller('v1/subscriptions')
export class ProcessRenewalHttpController {
  constructor(
    @Inject(PROCESS_RENEWAL_USE_CASE)
    private readonly processRenewal: ProcessRenewalUseCase,
  ) {}

  @Post(':id/renewals')
  @HttpCode(200)
  async triggerRenewal(
    @Param('id') rawId: string,
    @Headers('x-correlation-id') correlationHeader?: string,
  ) {
    const parsed = SubscriptionIdSchema.safeParse(rawId);
    if (!parsed.success) {
      throw new BadRequestException('Invalid subscription UUID format');
    }

    const correlationId = correlationHeader?.trim() || `http_${Date.now()}`;

    try {
      const outcome = await this.processRenewal.execute({
        subscriptionId: parsed.data,
        correlationId,
      });

      return {
        subscriptionId: outcome.subscriptionId,
        outcome: outcome.outcome,
        status: outcome.resultingStatus,
        nextPeriodEnd: outcome.nextPeriodEnd.toISOString(),
        providerTransactionId: outcome.providerTransactionId,
        declineReason: outcome.declineReason,
      };
    } catch (err) {
      if (err instanceof SubscriptionNotFoundError) {
        throw new NotFoundException({ code: err.code, message: err.message });
      }
      if (
        err instanceof InvalidSubscriptionStateError ||
        err instanceof OptimisticConcurrencyError
      ) {
        throw new ConflictException({ code: err.code, message: err.message });
      }
      if (err instanceof RenewalNotDueYetError) {
        throw new UnprocessableEntityException({ code: err.code, message: err.message });
      }
      throw err;
    }
  }
}
```

```typescript
// src/billing/adapters/primary/cli/renewal-batch.runner.ts
import { ClockPort } from '../../../core/ports/driven/clock.port';
import { SubscriptionRepositoryPort } from '../../../core/ports/driven/subscription-repository.port';
import { ProcessRenewalUseCase } from '../../../core/ports/driving/process-renewal.use-case';

export interface BatchSummary {
  scanned: number;
  renewed: number;
  failed: number;
  skippedOrErrored: number;
}

export class RenewalBatchCliRunner {
  constructor(
    private readonly subscriptions: SubscriptionRepositoryPort,
    private readonly processRenewal: ProcessRenewalUseCase,
    private readonly clock: ClockPort,
  ) {}

  async runBatch(batchLimit = 100): Promise<BatchSummary> {
    const dueIds = await this.subscriptions.findDueSubscriptionIds(this.clock.now(), batchLimit);
    const summary: BatchSummary = {
      scanned: dueIds.length,
      renewed: 0,
      failed: 0,
      skippedOrErrored: 0,
    };

    for (const subscriptionId of dueIds) {
      try {
        const result = await this.processRenewal.execute({
          subscriptionId,
          correlationId: `cron_${subscriptionId}_${this.clock.now().getTime()}`,
        });

        if (result.outcome === 'RENEWED' || result.outcome === 'ALREADY_RENEWED') {
          summary.renewed += 1;
        } else {
          summary.failed += 1;
        }
      } catch (err) {
        summary.skippedOrErrored += 1;
        console.error(`Cron renewal failed for ${subscriptionId}:`, err);
      }
    }

    return summary;
  }
}
```

### 3.6 Secondary (Driven) Adapters: Postgres, Stripe ACL, Clock & Fakes

```typescript
// src/billing/adapters/secondary/postgres/pg-subscription.repository.ts
import { Pool, PoolClient } from 'pg';
import { OptimisticConcurrencyError } from '../../../core/domain/billing.errors';
import { SupportedCurrency } from '../../../core/domain/money.vo';
import {
  Subscription,
  SubscriptionStatus,
} from '../../../core/domain/subscription.entity';
import { SubscriptionRepositoryPort } from '../../../core/ports/driven/subscription-repository.port';

type Queryable = Pick<Pool | PoolClient, 'query'>;

interface SubscriptionRow {
  id: string;
  customer_id: string;
  payment_method_token: string;
  plan_code: string;
  amount_cents: number;
  currency: SupportedCurrency;
  billing_cycle_days: number;
  status: SubscriptionStatus;
  current_period_end: Date;
  failed_attempts: number;
  last_renewal_cycle_key: string | null;
  version: number;
}

export class PgSubscriptionRepository implements SubscriptionRepositoryPort {
  constructor(private readonly db: Queryable) {}

  async findById(subscriptionId: string): Promise<Subscription | null> {
    const { rows } = await this.db.query<SubscriptionRow>(
      `SELECT id, customer_id, payment_method_token, plan_code, amount_cents, currency,
              billing_cycle_days, status, current_period_end, failed_attempts,
              last_renewal_cycle_key, version
         FROM subscriptions
        WHERE id = $1`,
      [subscriptionId],
    );

    const row = rows[0];
    if (!row) return null;

    return Subscription.rehydrate({
      id: row.id,
      customerId: row.customer_id,
      paymentMethodToken: row.payment_method_token,
      planCode: row.plan_code,
      amountCents: Number(row.amount_cents),
      currency: row.currency,
      billingCycleDays: Number(row.billing_cycle_days),
      status: row.status,
      currentPeriodEnd: new Date(row.current_period_end),
      failedAttempts: Number(row.failed_attempts),
      lastRenewalCycleKey: row.last_renewal_cycle_key,
      version: Number(row.version),
    });
  }

  async findDueSubscriptionIds(asOf: Date, limit: number): Promise<string[]> {
    const { rows } = await this.db.query<{ id: string }>(
      `SELECT id
         FROM subscriptions
        WHERE status IN ('ACTIVE', 'PAST_DUE')
          AND current_period_end <= $1
        ORDER BY current_period_end ASC
        LIMIT $2`,
      [asOf, limit],
    );
    return rows.map((r) => r.id);
  }

  async saveWithOptimisticLock(subscription: Subscription): Promise<void> {
    const snap = subscription.toSnapshot();
    const res = await this.db.query(
      `UPDATE subscriptions
          SET status = $1,
              current_period_end = $2,
              failed_attempts = $3,
              last_renewal_cycle_key = $4,
              version = version + 1,
              updated_at = NOW()
        WHERE id = $5
          AND version = $6`,
      [
        snap.status,
        snap.currentPeriodEnd,
        snap.failedAttempts,
        snap.lastRenewalCycleKey,
        snap.id,
        snap.version,
      ],
    );

    if (res.rowCount === 0) {
      throw new OptimisticConcurrencyError(snap.id);
    }
  }
}
```

```typescript
// src/billing/adapters/secondary/postgres/pg-unit-of-work.ts
import { Pool } from 'pg';
import {
  BillingTransactionPorts,
  UnitOfWorkPort,
} from '../../../core/ports/driven/unit-of-work.port';
import { PgSubscriptionRepository } from './pg-subscription.repository';

export class PgUnitOfWork implements UnitOfWorkPort {
  constructor(private readonly pool: Pool) {}

  async runInTransaction<T>(
    work: (ports: BillingTransactionPorts) => Promise<T>,
  ): Promise<T> {
    const client = await this.pool.connect();
    try {
      await client.query('BEGIN');
      const ports: BillingTransactionPorts = {
        subscriptions: new PgSubscriptionRepository(client),
      };
      const result = await work(ports);
      await client.query('COMMIT');
      return result;
    } catch (err) {
      await client.query('ROLLBACK');
      throw err;
    } finally {
      client.release();
    }
  }
}
```

```typescript
// src/billing/adapters/secondary/stripe/stripe-payment-gateway.adapter.ts
import Stripe from 'stripe';
import {
  ChargePaymentCommand,
  ChargePaymentResult,
  PaymentGatewayPort,
} from '../../../core/ports/driven/payment-gateway.port';

export class StripePaymentGatewayAdapter implements PaymentGatewayPort {
  constructor(private readonly stripe: Stripe) {}

  async charge(command: ChargePaymentCommand): Promise<ChargePaymentResult> {
    try {
      const intent = await this.stripe.paymentIntents.create(
        {
          amount: command.amount.amountCents,
          currency: command.amount.currency.toLowerCase(),
          customer: command.customerId,
          payment_method: command.paymentMethodToken,
          confirm: true,
          off_session: true,
          metadata: {
            correlationId: command.correlationId,
          },
        },
        {
          idempotencyKey: command.idempotencyKey,
        },
      );

      if (intent.status === 'succeeded') {
        return {
          status: 'SUCCEEDED',
          providerTransactionId: intent.id,
        };
      }

      return {
        status: 'DECLINED',
        declineCode: `UNEXPECTED_INTENT_STATUS_${intent.status.toUpperCase()}`,
        retryable: intent.status === 'processing',
      };
    } catch (err) {
      if (err instanceof Stripe.errors.StripeCardError) {
        return {
          status: 'DECLINED',
          declineCode: err.decline_code ?? err.code ?? 'CARD_DECLINED',
          retryable: false,
        };
      }
      if (
        err instanceof Stripe.errors.StripeRateLimitError ||
        err instanceof Stripe.errors.StripeConnectionError
      ) {
        return {
          status: 'DECLINED',
          declineCode: 'GATEWAY_TRANSIENT_UNAVAILABLE',
          retryable: true,
        };
      }
      throw err;
    }
  }
}
```

```typescript
// src/billing/adapters/secondary/clock/system-clock.adapter.ts
import { ClockPort } from '../../../core/ports/driven/clock.port';

export class SystemClockAdapter implements ClockPort {
  now(): Date {
    return new Date();
  }
}
```

```typescript
// src/billing/adapters/secondary/fakes/in-memory-subscription.repository.ts
import { OptimisticConcurrencyError } from '../../../core/domain/billing.errors';
import {
  Subscription,
  SubscriptionSnapshot,
} from '../../../core/domain/subscription.entity';
import { SubscriptionRepositoryPort } from '../../../core/ports/driven/subscription-repository.port';
import {
  BillingTransactionPorts,
  UnitOfWorkPort,
} from '../../../core/ports/driven/unit-of-work.port';

export class InMemorySubscriptionRepository
  implements SubscriptionRepositoryPort, UnitOfWorkPort
{
  private readonly store = new Map<string, SubscriptionSnapshot>();

  seed(snapshot: SubscriptionSnapshot): void {
    this.store.set(snapshot.id, {
      ...snapshot,
      currentPeriodEnd: new Date(snapshot.currentPeriodEnd.getTime()),
    });
  }

  async findById(subscriptionId: string): Promise<Subscription | null> {
    const snap = this.store.get(subscriptionId);
    return snap ? Subscription.rehydrate(snap) : null;
  }

  async findDueSubscriptionIds(asOf: Date, limit: number): Promise<string[]> {
    return [...this.store.values()]
      .filter(
        (s) =>
          (s.status === 'ACTIVE' || s.status === 'PAST_DUE') &&
          s.currentPeriodEnd.getTime() <= asOf.getTime(),
      )
      .sort((a, b) => a.currentPeriodEnd.getTime() - b.currentPeriodEnd.getTime())
      .slice(0, limit)
      .map((s) => s.id);
  }

  async saveWithOptimisticLock(subscription: Subscription): Promise<void> {
    const next = subscription.toSnapshot();
    const existing = this.store.get(next.id);
    if (!existing || existing.version !== next.version) {
      throw new OptimisticConcurrencyError(next.id);
    }
    this.store.set(next.id, {
      ...next,
      version: existing.version + 1,
      currentPeriodEnd: new Date(next.currentPeriodEnd.getTime()),
    });
  }

  async runInTransaction<T>(
    work: (ports: BillingTransactionPorts) => Promise<T>,
  ): Promise<T> {
    // Deep snapshot clone provides atomic rollback semantics if work throws
    const backup = new Map<string, SubscriptionSnapshot>();
    for (const [key, val] of this.store.entries()) {
      backup.set(key, {
        ...val,
        currentPeriodEnd: new Date(val.currentPeriodEnd.getTime()),
      });
    }

    try {
      return await work({ subscriptions: this });
    } catch (err) {
      this.store.clear();
      for (const [key, val] of backup.entries()) {
        this.store.set(key, val);
      }
      throw err;
    }
  }
}
```

```typescript
// src/billing/adapters/secondary/fakes/fake-payment-gateway.adapter.ts
import {
  ChargePaymentCommand,
  ChargePaymentResult,
  PaymentGatewayPort,
} from '../../../core/ports/driven/payment-gateway.port';

export class FakePaymentGateway implements PaymentGatewayPort {
  readonly recordedCharges: ChargePaymentCommand[] = [];
  private nextResult: ChargePaymentResult = {
    status: 'SUCCEEDED',
    providerTransactionId: 'pi_fake_001',
  };

  willReturn(result: ChargePaymentResult): void {
    this.nextResult = result;
  }

  async charge(command: ChargePaymentCommand): Promise<ChargePaymentResult> {
    this.recordedCharges.push(command);
    return this.nextResult;
  }
}
```

```typescript
// src/billing/adapters/secondary/fakes/fixed-clock.adapter.ts
import { ClockPort } from '../../../core/ports/driven/clock.port';

export class FixedClock implements ClockPort {
  constructor(private current: Date) {}

  now(): Date {
    return new Date(this.current.getTime());
  }

  advanceMinutes(minutes: number): void {
    this.current = new Date(this.current.getTime() + minutes * 60_000);
  }
}
```

### 3.7 Composition Root: Pure DI vs NestJS Module

Both wiring models connect the exact same Core and Adapters cleanly without leaking NestJS decorators into `core/`.

```typescript
// src/billing/composition/pure-di.factory.ts
import { Pool } from 'pg';
import Stripe from 'stripe';
import { SystemClockAdapter } from '../adapters/secondary/clock/system-clock.adapter';
import { PgSubscriptionRepository } from '../adapters/secondary/postgres/pg-subscription.repository';
import { PgUnitOfWork } from '../adapters/secondary/postgres/pg-unit-of-work';
import { StripePaymentGatewayAdapter } from '../adapters/secondary/stripe/stripe-payment-gateway.adapter';
import { RenewalBatchCliRunner } from '../adapters/primary/cli/renewal-batch.runner';
import { ProcessRenewalService } from '../core/application/process-renewal.service';

export interface BillingRuntimeConfig {
  databaseUrl: string;
  stripeSecretKey: string;
}

export function createBillingRuntime(config: BillingRuntimeConfig) {
  const pool = new Pool({ connectionString: config.databaseUrl, max: 20 });
  const stripe = new Stripe(config.stripeSecretKey);

  const unitOfWork = new PgUnitOfWork(pool);
  const subscriptionRepo = new PgSubscriptionRepository(pool);
  const paymentGateway = new StripePaymentGatewayAdapter(stripe);
  const clock = new SystemClockAdapter();

  const processRenewalUseCase = new ProcessRenewalService(
    unitOfWork,
    paymentGateway,
    clock,
  );

  const cliRunner = new RenewalBatchCliRunner(
    subscriptionRepo,
    processRenewalUseCase,
    clock,
  );

  return {
    processRenewalUseCase,
    cliRunner,
    shutdown: () => pool.end(),
  };
}
```

```typescript
// src/billing/composition/billing.module.ts
import { Module } from '@nestjs/common';
import { Pool } from 'pg';
import Stripe from 'stripe';
import { ProcessRenewalHttpController } from '../adapters/primary/http/process-renewal.controller';
import { SystemClockAdapter } from '../adapters/secondary/clock/system-clock.adapter';
import { PgSubscriptionRepository } from '../adapters/secondary/postgres/pg-subscription.repository';
import { PgUnitOfWork } from '../adapters/secondary/postgres/pg-unit-of-work';
import { StripePaymentGatewayAdapter } from '../adapters/secondary/stripe/stripe-payment-gateway.adapter';
import { ProcessRenewalService } from '../core/application/process-renewal.service';
import { CLOCK_PORT, ClockPort } from '../core/ports/driven/clock.port';
import {
  PAYMENT_GATEWAY_PORT,
  PaymentGatewayPort,
} from '../core/ports/driven/payment-gateway.port';
import { SUBSCRIPTION_REPOSITORY_PORT } from '../core/ports/driven/subscription-repository.port';
import {
  UNIT_OF_WORK_PORT,
  UnitOfWorkPort,
} from '../core/ports/driven/unit-of-work.port';
import { PROCESS_RENEWAL_USE_CASE } from '../core/ports/driving/process-renewal.use-case';

const PG_POOL_TOKEN = Symbol('PG_POOL_TOKEN');
const STRIPE_CLIENT_TOKEN = Symbol('STRIPE_CLIENT_TOKEN');

@Module({
  controllers: [ProcessRenewalHttpController],
  providers: [
    {
      provide: PG_POOL_TOKEN,
      useFactory: () => new Pool({ connectionString: process.env.DATABASE_URL, max: 20 }),
    },
    {
      provide: STRIPE_CLIENT_TOKEN,
      useFactory: () => new Stripe(process.env.STRIPE_SECRET_KEY ?? ''),
    },
    {
      provide: CLOCK_PORT,
      useClass: SystemClockAdapter,
    },
    {
      provide: UNIT_OF_WORK_PORT,
      inject: [PG_POOL_TOKEN],
      useFactory: (pool: Pool) => new PgUnitOfWork(pool),
    },
    {
      provide: SUBSCRIPTION_REPOSITORY_PORT,
      inject: [PG_POOL_TOKEN],
      useFactory: (pool: Pool) => new PgSubscriptionRepository(pool),
    },
    {
      provide: PAYMENT_GATEWAY_PORT,
      inject: [STRIPE_CLIENT_TOKEN],
      useFactory: (stripe: Stripe) => new StripePaymentGatewayAdapter(stripe),
    },
    {
      provide: PROCESS_RENEWAL_USE_CASE,
      inject: [UNIT_OF_WORK_PORT, PAYMENT_GATEWAY_PORT, CLOCK_PORT],
      useFactory: (
        uow: UnitOfWorkPort,
        gateway: PaymentGatewayPort,
        clock: ClockPort,
      ) => new ProcessRenewalService(uow, gateway, clock),
    },
  ],
  exports: [PROCESS_RENEWAL_USE_CASE],
})
export class BillingHexagonalModule {}
```

### 3.8 Zero-Docker Unit Tests & Adapter Integration Tests

Because Core uses pure Driven Ports, unit tests wire stateful in-memory fakes directly in milliseconds while integration tests validate real SQL and Stripe ACL mapping separately.

```typescript
// src/billing/tests/process-renewal.unit.spec.ts
import { describe, expect, it, beforeEach } from 'vitest';
import { FakePaymentGateway } from '../adapters/secondary/fakes/fake-payment-gateway.adapter';
import { FixedClock } from '../adapters/secondary/fakes/fixed-clock.adapter';
import { InMemorySubscriptionRepository } from '../adapters/secondary/fakes/in-memory-subscription.repository';
import { ProcessRenewalService } from '../core/application/process-renewal.service';
import { RenewalNotDueYetError } from '../core/domain/billing.errors';

describe('ProcessRenewalService (Zero-Docker Hexagonal Core)', () => {
  let repo: InMemorySubscriptionRepository;
  let gateway: FakePaymentGateway;
  let clock: FixedClock;
  let useCase: ProcessRenewalService;

  const subId = '11111111-1111-4111-8111-111111111111';

  beforeEach(() => {
    repo = new InMemorySubscriptionRepository();
    gateway = new FakePaymentGateway();
    clock = new FixedClock(new Date('2026-11-13T10:00:00.000Z'));
    useCase = new ProcessRenewalService(repo, gateway, clock);

    repo.seed({
      id: subId,
      customerId: 'cus_999',
      paymentMethodToken: 'pm_card_visa',
      planCode: 'PRO_MONTHLY',
      amountCents: 4900,
      currency: 'USD',
      billingCycleDays: 30,
      status: 'ACTIVE',
      currentPeriodEnd: new Date('2026-11-13T09:00:00.000Z'),
      failedAttempts: 0,
      lastRenewalCycleKey: null,
      version: 1,
    });
  });

  it('renews a due subscription, advances period by 30 days, and passes deterministic idempotency key', async () => {
    const outcome = await useCase.execute({
      subscriptionId: subId,
      correlationId: 'corr_001',
    });

    expect(outcome.outcome).toBe('RENEWED');
    expect(outcome.resultingStatus).toBe('ACTIVE');
    expect(outcome.providerTransactionId).toBe('pi_fake_001');
    expect(gateway.recordedCharges).toHaveLength(1);
    expect(gateway.recordedCharges[0].idempotencyKey).toBe(
      `ren_${subId}_2026-11-13_0`,
    );

    const saved = await repo.findById(subId);
    expect(saved?.version).toBe(3); // Reserved (+1) then finalized (+1)
    expect(saved?.currentPeriodEnd.toISOString()).toBe('2026-12-13T10:00:00.000Z');
  });

  it('transitions from PAST_DUE to SUSPENDED on third failed charge attempt', async () => {
    repo.seed({
      id: subId,
      customerId: 'cus_999',
      paymentMethodToken: 'pm_card_declined',
      planCode: 'PRO_MONTHLY',
      amountCents: 4900,
      currency: 'USD',
      billingCycleDays: 30,
      status: 'PAST_DUE',
      currentPeriodEnd: new Date('2026-11-10T09:00:00.000Z'),
      failedAttempts: 2,
      lastRenewalCycleKey: null,
      version: 4,
    });
    gateway.willReturn({ status: 'DECLINED', declineCode: 'insufficient_funds', retryable: false });

    const outcome = await useCase.execute({ subscriptionId: subId, correlationId: 'corr_002' });

    expect(outcome.outcome).toBe('SUSPENDED');
    expect(outcome.resultingStatus).toBe('SUSPENDED');
    expect(outcome.declineReason).toBe('insufficient_funds');
    const saved = await repo.findById(subId);
    expect(saved?.failedAttempts).toBe(3);
  });

  it('rejects early renewal without calling payment gateway', async () => {
    clock = new FixedClock(new Date('2026-11-12T00:00:00.000Z'));
    useCase = new ProcessRenewalService(repo, gateway, clock);

    await expect(
      useCase.execute({ subscriptionId: subId, correlationId: 'corr_early' }),
    ).rejects.toThrow(RenewalNotDueYetError);

    expect(gateway.recordedCharges).toHaveLength(0);
    const saved = await repo.findById(subId);
    expect(saved?.status).toBe('ACTIVE');
  });
});
```

```typescript
// src/billing/tests/pg-subscription.integration.spec.ts
import { afterAll, beforeAll, describe, expect, it } from 'vitest';
import { Pool } from 'pg';
import Stripe from 'stripe';
import { PgSubscriptionRepository } from '../adapters/secondary/postgres/pg-subscription.repository';
import { StripePaymentGatewayAdapter } from '../adapters/secondary/stripe/stripe-payment-gateway.adapter';
import { OptimisticConcurrencyError } from '../core/domain/billing.errors';
import { Money } from '../core/domain/money.vo';

describe('Secondary Adapters Integration Verification', () => {
  let pool: Pool;
  let repo: PgSubscriptionRepository;

  beforeAll(async () => {
    pool = new Pool({
      connectionString: process.env.TEST_DATABASE_URL ?? 'postgres://app:app@localhost:5432/billing_test',
    });
    await pool.query(`
      CREATE TABLE IF NOT EXISTS subscriptions (
        id UUID PRIMARY KEY,
        customer_id TEXT NOT NULL,
        payment_method_token TEXT NOT NULL,
        plan_code TEXT NOT NULL,
        amount_cents INTEGER NOT NULL,
        currency TEXT NOT NULL,
        billing_cycle_days INTEGER NOT NULL,
        status TEXT NOT NULL,
        current_period_end TIMESTAMPTZ NOT NULL,
        failed_attempts INTEGER NOT NULL DEFAULT 0,
        last_renewal_cycle_key TEXT NULL,
        version INTEGER NOT NULL DEFAULT 1,
        updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
      );
    `);
    repo = new PgSubscriptionRepository(pool);
  });

  afterAll(async () => {
    await pool.end();
  });

  it('enforces optimistic concurrency conflict in real PostgreSQL SQL', async () => {
    const id = '22222222-2222-4222-8222-222222222222';
    await pool.query(
      `INSERT INTO subscriptions (
         id, customer_id, payment_method_token, plan_code, amount_cents, currency,
         billing_cycle_days, status, current_period_end, failed_attempts, version
       ) VALUES ($1, 'cus_pg', 'pm_pg', 'PRO', 2900, 'USD', 30, 'ACTIVE', NOW() - INTERVAL '1 day', 0, 1)
       ON CONFLICT (id) DO UPDATE SET status = 'ACTIVE', version = 1`,
      [id],
    );

    const workerA = await repo.findById(id);
    const workerB = await repo.findById(id);
    expect(workerA).not.toBeNull();
    expect(workerB).not.toBeNull();

    workerA!.beginRenewalAttempt(new Date());
    await repo.saveWithOptimisticLock(workerA!);

    workerB!.beginRenewalAttempt(new Date());
    await expect(repo.saveWithOptimisticLock(workerB!)).rejects.toThrow(
      OptimisticConcurrencyError,
    );
  });

  it('translates StripeCardError inside ACL into pure domain DECLINED contract', async () => {
    const fakeStripeClient = {
      paymentIntents: {
        create: async () => {
          throw new Stripe.errors.StripeCardError({
            type: 'card_error',
            code: 'card_declined',
            decline_code: 'expired_card',
            message: 'Your card has expired.',
          });
        },
      },
    } as unknown as Stripe;

    const adapter = new StripePaymentGatewayAdapter(fakeStripeClient);
    const result = await adapter.charge({
      customerId: 'cus_1',
      paymentMethodToken: 'pm_expired',
      amount: Money.fromCents(4900, 'USD'),
      idempotencyKey: 'ren_key_01',
      correlationId: 'corr_acl_01',
    });

    expect(result).toEqual({
      status: 'DECLINED',
      declineCode: 'expired_card',
      retryable: false,
    });
  });
});
```

---

## 4. Complete Framework / Construct Reference & Architectural Guardrails

### 4.1 Complete Adapter & Port Catalog

| Boundary Category | Adapter Examples | Critical Implementation Rule |
|---|---|---|
| **Driving: HTTP REST** | NestJS Controller, Fastify Route, Express Router | Parse Zod/DTO schemas, validate JWT claims, call 1 Driving Port, map Domain Errors (`NotFound`, `Conflict`) to HTTP codes (`404`, `409`) |
| **Driving: gRPC / GraphQL** | Protobuf gRPC Server, Apollo Resolver | Translate gRPC `Metadata` or GraphQL context into Domain Command + `correlationId`; map Domain Errors to `status.FAILED_PRECONDITION` or GraphQL union error types |
| **Driving: Async Worker / CLI** | BullMQ Worker, Kafka Consumer, K8s CronJob CLI | Deserialize message envelope, validate schema, extract idempotency token, invoke exact same Driving Port used by REST |
| **Driven: Persistence** | Raw `pg`, Drizzle, Prisma, TypeORM Adapter | Rehydrate pure Domain Entities via static `.rehydrate(...)` factory without firing domain creation validations on historical rows; never expose ORM models outside adapter |
| **Driven: Cache Decorator** | `CachedCatalogAdapter implements CatalogPort` | Wrap real SQL adapter via Decorator at Composition Root; Core remains oblivious to Redis TTLs and JSON serialization |
| **Driven: External API ACL** | Stripe, OpenAI, Twilio, SendGrid Adapter | Enforce explicit HTTP timeouts (`AbortSignal.timeout(5000)`), map vendor error taxonomies to retryable/terminal domain enums, pass idempotency keys |
| **Driven: Non-Determinism** | `ClockPort`, `IdGeneratorPort`, `RandomPort` | Never call `new Date()`, `Date.now()`, or `crypto.randomUUID()` inside Core state transitions; inject explicit Ports so time-travel tests remain deterministic |

### 4.2 Enforcing Boundaries Mechanically (`dependency-cruiser` + TypeScript Project References)

Conventions documented in README files erode under deadline pressure. Enforce the polygon boundary mechanically on every CI run using both **static dependency graphs** and **separate `tsconfig` compilation units**.

#### Static Enforcement with `dependency-cruiser`

```javascript
// .dependency-cruiser.cjs
module.exports = {
  forbidden: [
    {
      name: 'core-must-not-import-adapters-or-composition',
      severity: 'error',
      from: { path: '^src/[^/]+/core/' },
      to: { path: '^src/[^/]+/(adapters|composition)/' },
    },
    {
      name: 'core-must-not-import-infrastructure-packages',
      severity: 'error',
      from: { path: '^src/[^/]+/core/' },
      to: {
        path: 'node_modules/(pg|stripe|@prisma|typeorm|@nestjs|express|fastify|ioredis)',
      },
    },
    {
      name: 'domain-must-not-import-application-or-ports',
      severity: 'error',
      from: { path: '^src/[^/]+/core/domain/' },
      to: { path: '^src/[^/]+/core/(application|ports)/' },
    },
    {
      name: 'primary-adapters-must-not-bypass-to-secondary-adapters',
      severity: 'error',
      from: { path: '^src/[^/]+/adapters/primary/' },
      to: { path: '^src/[^/]+/adapters/secondary/' },
    },
    {
      name: 'no-circular-dependencies-in-core',
      severity: 'error',
      from: { path: '^src/[^/]+/core/' },
      to: { circular: true },
    },
  ],
};
```

#### Compile-Time Enforcement via TypeScript Project References

Split monorepo packages (`packages/billing-core` vs `packages/billing-adapters`) so `billing-core/package.json` contains **zero infrastructure dependencies** and `packages/billing-core/tsconfig.json` does not list `billing-adapters` in `references`:

```json
// packages/billing-core/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true
  },
  "include": ["src/**/*"]
}
```

```json
// packages/billing-adapters/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true
  },
  "references": [{ "path": "../billing-core" }],
  "include": ["src/**/*"]
}
```

Any developer writing `import { Pool } from 'pg'` inside `packages/billing-core` triggers immediate `tsc -b` compiler failure before tests even run.

### 4.3 Subtle Production Failure Modes & Security Traps

1. **Holding PostgreSQL Row Locks Across External Network Adapters**: Wrapping `SELECT ... FOR UPDATE` + `await paymentGateway.charge(...)` + `UPDATE` inside one `PgUnitOfWork.runInTransaction()` holds a database connection and row lock across unpredictable 2–30 second third-party HTTP latencies, exhausting `pg` pools during Stripe degradations. Always reserve a pending state in a short transaction, execute external I/O outside locks with an idempotency key, and finalize in a second short transaction.
2. **Stale `RENEWAL_PENDING` Zombie Rows on Process Crash**: If a Node pod crashes (`SIGKILL`) during Phase 2 after reserving `RENEWAL_PENDING` and before Phase 3 commits, the subscription remains stuck in `RENEWAL_PENDING`. Production billing cores require a **Reconciliation Sweeper** querying `WHERE status = 'RENEWAL_PENDING' AND updated_at < NOW() - INTERVAL '5 minutes'` and querying Stripe by the deterministic `cycleKey` idempotency token (`paymentIntents.search` or re-issuing with the exact `idempotencyKey`) to settle the outcome deterministically.
3. **Fake vs Real Repository Behavioral Drift**: An `InMemorySubscriptionRepository` using JavaScript `Map` omits SQL `UNIQUE` constraints, case-insensitive collations, timezone truncation, and `SERIALIZABLE`/`READ COMMITTED` lock blocking. Eliminate drift by running one **Shared Repository Contract Test Suite** against both the `InMemory` fake and real PostgreSQL (`Testcontainers`).
4. **Missing Signature Verification on Webhook Primary Adapters**: External webhook adapters (`POST /webhooks/stripe`) must authenticate raw request buffers via HMAC (`stripe.webhooks.constructEvent`) **inside the Primary Adapter** prior to dispatching commands into Core—never parse unauthenticated JSON into Driving Ports.

---

## 5. Production Debugging, Profiling & Operations Playbook

| Production Symptom | Structural Root Cause | Diagnostic Commands & Telemetry | Remediation |
|---|---|---|---|
| **API p99 latency spikes to 8s & `TimeoutError: pg pool exhausted`** | `UnitOfWork.runInTransaction()` opened before `PaymentGatewayPort.charge()`, holding `pg.PoolClient` open during Stripe slowdown | Inspect `SELECT state, wait_event_type, count(*) FROM pg_stat_activity GROUP BY 1,2;` (shows `idle in transaction` connections) + OpenTelemetry span showing DB TX wrapping HTTP span | Refactor into Two-Phase Reservation (`ACTIVE -> RENEWAL_PENDING`, release TX, call Stripe, finalize TX) as shown in Section 3.4 |
| **NestJS startup crash: `Nest can't resolve dependencies of ProcessRenewalHttpController (?)`** | Driven or Driving `Symbol` token mismatched or omitted from `@Module({ providers })` or missing `exports` across modules | Run `npx tsc --noEmit` + inspect NestJS bootstrap stack trace pointing to exact `@Inject(TOKEN)` index `[0]` | Export `PROCESS_RENEWAL_USE_CASE` from `BillingHexagonalModule`; verify exact same `Symbol` reference imported in controller and module |
| **Domain unit tests pass in 20ms, staging throws `column "amount_cents" type mismatch` or `OptimisticConcurrencyError`** | `InMemorySubscriptionRepository` diverged from SQL column types or failed to increment `version` identically to `PgSubscriptionRepository` | Run `npx vitest run src/billing/tests/pg-subscription.integration.spec.ts` against Testcontainers Postgres + enable `LOG_STATEMENT=all` locally | Extract a parameterized Contract Suite testing both `InMemory` and `Pg` adapters identically for `version`, `NULL` fields, and `UTC` timestamps |
| **Slow read endpoints (`450ms` for 200 dashboard rows) despite clean SQL indexes** | Controller invoked a rich Domain Repository hydrating 200 full Aggregate graphs + Value Objects only to serialize 4 summary fields | Run `node --cpu-prof dist/main.js` or inspect `pg` query timing vs total HTTP span; CPU flamegraph shows `Subscription.rehydrate` + `Date` allocations | Introduce a dedicated CQRS Read Port (`SubscriptionQueryPort`) executing one flat SQL projection straight to JSON DTO |
| **PR introduces `import { HttpException } from '@nestjs/common'` inside Core** | Developer threw framework HTTP errors from domain validation instead of pure `BillingDomainError` | Run `npx depcruise src --config .dependency-cruiser.cjs` in CI | Throw `BillingDomainError` subclass inside Core; translate to NestJS/HTTP errors inside `ProcessRenewalHttpController` or global exception filter |

---

## 6. Worked Exercises

### Exercise 1: Identifying Primary vs Secondary Adapter Leakage

Inspect the Go service below and identify **3 Hexagonal Architecture boundary leaks**, specifying which side (Primary or Secondary) each violates and how to fix them:

```go
// core/webhook_processor.go
package core

import (
	"net/http"
	"github.com/stripe/stripe-go/v76"
)

type BillingRepo interface {
	UpsertStripeCustomer(cust *stripe.Customer) error
}

type WebhookProcessor struct {
	repo BillingRepo
}

func (p *WebhookProcessor) Handle(w http.ResponseWriter, r *http.Request) {
	// parses r.Body, verifies Stripe-Signature header, calls p.repo.UpsertStripeCustomer(...)
	w.WriteHeader(http.StatusOK)
}
```

---

### Solution 1

#### 3 Hexagonal Boundary Leaks

1. **Primary Adapter Merged into Core (`Handle(w http.ResponseWriter, r *http.Request)`)**: `core.WebhookProcessor` imports `net/http` directly and manages HTTP response codes (`http.StatusOK`), tying domain billing synchronization to HTTP transport. It cannot be invoked from a dead-letter replay worker or CLI without fabricating `httptest.ResponseRecorder`.
2. **Third-Party SDK Leakage into Secondary Port (`*stripe.Customer`)**: `BillingRepo` accepts `*stripe.Customer` from `stripe-go/v76`. Upgrading to `stripe-go/v80` or adding PayPal/Adyen requires changing the Core Port signature and every repository adapter.
3. **Cryptographic Webhook Verification Mixed into Core**: Verifying `Stripe-Signature` HMAC headers is a transport-security concern belonging to the inbound Primary Adapter (`StripeWebhookHttpAdapter`), not core billing logic.

#### Clean Hexagonal Separation

```go
// core/ports.go (zero net/http, zero stripe-go)
package core

type SyncCustomerCommand struct {
	AccountID          string
	ExternalCustomerID string
	Email              string
}

type CustomerSyncPort interface {
	Sync(cmd SyncCustomerCommand) error
}

type BillingRepo interface {
	UpsertExternalCustomer(accountID, externalID, email string) error
}
```

- **Primary Adapter (`adapters/http/stripe_webhook.go`)**: Imports `net/http` + `stripe-go`, validates HMAC signature on raw bytes, translates `*stripe.Customer` into `core.SyncCustomerCommand`, and calls `CustomerSyncPort.Sync(...)`.
- **Secondary Adapter (`adapters/postgres/billing_repo.go`)**: Implements `core.BillingRepo` against PostgreSQL using only pure domain arguments.

---

### Exercise 2: Adding a Caching Layer Without Modifying the Core

A `CatalogService` inside a Hexagonal Core depends on `ProductCatalogPort.getBySku(sku: string): Promise<Product | null>`, currently implemented by `PostgresCatalogAdapter`. Under peak traffic, catalog reads hit 12,000 RPS and overwhelm Postgres CPU. How do you introduce a **Redis read-through cache with 60s TTL** strictly at the adapter level without editing a single line of `CatalogService` or `PostgresCatalogAdapter`?

---

### Solution 2

#### Adapter-Level Decorator Pattern

Implement a **Secondary Adapter Decorator** (`CachedCatalogAdapter`) implementing the exact same `ProductCatalogPort` interface, wrapping `PostgresCatalogAdapter`, and wire it inside the **Composition Root**:

```typescript
import Redis from 'ioredis';

export interface Product {
  sku: string;
  name: string;
  priceCents: number;
}

export interface ProductCatalogPort {
  getBySku(sku: string): Promise<Product | null>;
}

export class CachedCatalogAdapter implements ProductCatalogPort {
  constructor(
    private readonly inner: ProductCatalogPort,
    private readonly redis: Redis,
    private readonly ttlSeconds = 60,
  ) {}

  async getBySku(sku: string): Promise<Product | null> {
    const key = `catalog:sku:${sku}`;
    const cached = await this.redis.get(key);
    if (cached !== null) {
      return cached === '__NULL__' ? null : (JSON.parse(cached) as Product);
    }

    const product = await this.inner.getBySku(sku);
    if (product) {
      await this.redis.set(key, JSON.stringify(product), 'EX', this.ttlSeconds);
    } else {
      // Negative cache (short 10s TTL) protects Postgres from SKU enumeration scans
      await this.redis.set(key, '__NULL__', 'EX', 10);
    }
    return product;
  }
}

// Composition Root wiring:
// const catalogPort = new CachedCatalogAdapter(new PostgresCatalogAdapter(pgPool), redisClient);
// const catalogService = new CatalogService(catalogPort);
```

*Invariant preserved*: Neither `CatalogService` nor `PostgresCatalogAdapter` knows Redis exists, negative caching stops cache-penetration attacks on nonexistent SKUs, and unit tests still inject a clean in-memory adapter.

---

### Exercise 3: Fixing Dual-Adapter Transaction Boundary Failures

A developer wires a Hexagonal `TransferFundsService` with two separate Secondary Ports: `AccountRepository.debit(fromId, amt)` and `AccountRepository.credit(toId, amt)` plus `AuditLogPort.record(...)`. Each adapter method opens and commits its own standalone PostgreSQL query using `pool.query(...)`. During a pod crash after `debit` succeeds and before `credit` executes, funds vanish. How do you restore ACID atomicity across multiple Secondary Adapter calls without leaking `pg.PoolClient` or SQL transactions into the Hexagonal Core?

---

### Solution 3

#### Opaque Unit of Work Secondary Port

Define an **abstract `UnitOfWorkPort`** inside Core that hands the use case request-scoped repository/outbox ports bound to a shared opaque transaction scope, hiding `pg.PoolClient` completely inside the Postgres adapter layer:

```typescript
// Core Secondary Port (zero pg imports)
export interface AccountRepositoryPort {
  debit(accountId: string, amountCents: number): Promise<void>;
  credit(accountId: string, amountCents: number): Promise<void>;
}

export interface AuditOutboxPort {
  recordTransfer(fromId: string, toId: string, amountCents: number): Promise<void>;
}

export interface TransferTransactionPorts {
  accounts: AccountRepositoryPort;
  auditOutbox: AuditOutboxPort;
}

export interface TransferUnitOfWorkPort {
  inTransaction<T>(work: (ports: TransferTransactionPorts) => Promise<T>): Promise<T>;
}

// Core Service
export class TransferFundsService {
  constructor(private readonly uow: TransferUnitOfWorkPort) {}

  async transfer(fromId: string, toId: string, amountCents: number): Promise<void> {
    await this.uow.inTransaction(async ({ accounts, auditOutbox }) => {
      await accounts.debit(fromId, amountCents);
      await accounts.credit(toId, amountCents);
      await auditOutbox.recordTransfer(fromId, toId, amountCents);
    });
  }
}
```

Inside `PgTransferUnitOfWork`:
1. `inTransaction` checks out 1 connection (`client = await pool.connect()`), executes `BEGIN`, constructs request-scoped `new PgAccountRepository(client)` and `new PgAuditOutboxAdapter(client)`, and passes them into `work`.
2. Notice `AuditOutboxPort` writes to a transactional outbox table **using the exact same `client`** rather than emitting a non-transactional HTTP/Kafka call mid-DB-transaction.
3. Any exception rolls back all three writes atomically; `finally` releases `client` safely back to the pool.

---

### Exercise 4: Resolving Stuck `RENEWAL_PENDING` Subscriptions After Pod `SIGKILL`

Inspect Section 3.4's Two-Phase Reservation design. Suppose a Kubernetes worker pod executes Phase 1 (`status = 'RENEWAL_PENDING'`), calls `StripePaymentGatewayAdapter.charge()` which succeeds on Stripe's servers, and receives an OOMKill (`SIGKILL`) milliseconds before Phase 3 executes `markRenewalSucceeded`. The subscription is now permanently stuck in `RENEWAL_PENDING` and excluded from `findDueSubscriptionIds()`. Design and implement a Hexagonal **Reconciliation Sweeper** using Ports & Adapters without creating duplicate Stripe charges.

---

### Solution 4

#### Root Cause & Recovery Invariant

Because Phase 1 transitioned the database row to `RENEWAL_PENDING` and derived a deterministic `cycleKey = ren_${id}_${periodDate}_${failedAttempts}`, Stripe recorded that exact `idempotencyKey` when Phase 2 succeeded. Re-invoking `PaymentGatewayPort.charge()` with the identical `idempotencyKey` causes Stripe to return the cached `PaymentIntent` result **without charging the card twice**.

#### Complete Hexagonal Reconciliation Implementation

1. **Extend Driven Ports**:
   - Add `findStalePendingSubscriptionIds(olderThan: Date, limit: number): Promise<string[]>` on `SubscriptionRepositoryPort`.
   - Add `recoverFromPendingTimeout(now: Date)` or allow `Subscription` to compute `deriveCycleIdempotencyKey()` while in `RENEWAL_PENDING`.
2. **Add `ReconcileStaleRenewalsService` (Core)**:

```typescript
// src/billing/core/application/reconcile-stale-renewals.service.ts
import { ClockPort } from '../ports/driven/clock.port';
import { PaymentGatewayPort } from '../ports/driven/payment-gateway.port';
import { SubscriptionRepositoryPort } from '../ports/driven/subscription-repository.port';
import { UnitOfWorkPort } from '../ports/driven/unit-of-work.port';

export class ReconcileStaleRenewalsService {
  constructor(
    private readonly readRepo: SubscriptionRepositoryPort,
    private readonly unitOfWork: UnitOfWorkPort,
    private readonly paymentGateway: PaymentGatewayPort,
    private readonly clock: ClockPort,
  ) {}

  async reconcile(staleMinutes = 5, limit = 50): Promise<number> {
    const cutoff = new Date(this.clock.now().getTime() - staleMinutes * 60_000);
    const staleIds = await this.readRepo.findStalePendingSubscriptionIds(cutoff, limit);
    let reconciled = 0;

    for (const id of staleIds) {
      const sub = await this.readRepo.findById(id);
      if (!sub || sub.status !== 'RENEWAL_PENDING') continue;

      const snap = sub.toSnapshot();
      const cycleKey = sub.deriveCycleIdempotencyKey();

      // Stripe replays the original outcome safely because idempotencyKey matches Phase 2
      const chargeResult = await this.paymentGateway.charge({
        customerId: snap.customerId,
        paymentMethodToken: snap.paymentMethodToken,
        amount: sub.price,
        idempotencyKey: cycleKey,
        correlationId: `reconcile_${id}_${this.clock.now().getTime()}`,
      });

      const finishedAt = this.clock.now();
      await this.unitOfWork.runInTransaction(async ({ subscriptions }) => {
        const fresh = await subscriptions.findById(id);
        if (!fresh || fresh.status !== 'RENEWAL_PENDING') return;

        if (chargeResult.status === 'SUCCEEDED') {
          fresh.markRenewalSucceeded(finishedAt, cycleKey);
        } else {
          fresh.markRenewalFailed();
        }
        await subscriptions.saveWithOptimisticLock(fresh);
      });
      reconciled += 1;
    }

    return reconciled;
  }
}
```

Every stale pod crash recovers idempotently with zero double billing and zero SQL/Stripe coupling inside Core.

---

### Exercise 5: Eliminating In-Memory Fake vs PostgreSQL Behavioral Drift via Contract Tests

A team tests `OrderService` exclusively with `InMemoryOrderRepository`. All 140 tests pass locally. In production, saving an order with an identical `(tenant_id, external_reference)` throws `23505 unique_violation`, and querying orders modified after `2026-11-13T10:00:00.123456789Z` behaves differently because PostgreSQL `TIMESTAMPTZ` truncates nanoseconds to microseconds while JavaScript `Date` truncates to milliseconds. Design a **Shared Repository Contract Test Suite** proving behavioral parity between `InMemorySubscriptionRepository` and `PgSubscriptionRepository`.

---

### Solution 5

#### Pattern: Port Contract Verification Harness

Define a single exported test factory `verifySubscriptionRepositoryContract(createHarness)` that accepts either the In-Memory fake or the live PostgreSQL adapter and executes the exact same assertions against both:

```typescript
// src/billing/tests/subscription-repo.contract.ts
import { expect, it, describe } from 'vitest';
import { OptimisticConcurrencyError } from '../core/domain/billing.errors';
import { Subscription } from '../core/domain/subscription.entity';
import { SubscriptionRepositoryPort } from '../core/ports/driven/subscription-repository.port';

export interface RepoContractHarness {
  repo: SubscriptionRepositoryPort;
  seedActive(id: string, periodEnd: Date): Promise<void>;
  cleanup(): Promise<void>;
}

export function runSubscriptionRepositoryContract(
  label: string,
  createHarness: () => Promise<RepoContractHarness>,
) {
  describe(`SubscriptionRepositoryPort Contract [${label}]`, () => {
    it('returns null for missing subscription IDs', async () => {
      const h = await createHarness();
      try {
        const missing = await h.repo.findById('99999999-9999-4999-8999-999999999999');
        expect(missing).toBeNull();
      } finally {
        await h.cleanup();
      }
    });

    it('rejects stale version writes identically with OptimisticConcurrencyError', async () => {
      const h = await createHarness();
      const id = '33333333-3333-4333-8333-333333333333';
      try {
        await h.seedActive(id, new Date('2026-11-01T00:00:00.000Z'));
        const copyA = await h.repo.findById(id);
        const copyB = await h.repo.findById(id);

        copyA!.beginRenewalAttempt(new Date('2026-11-02T00:00:00.000Z'));
        await h.repo.saveWithOptimisticLock(copyA!);

        copyB!.beginRenewalAttempt(new Date('2026-11-02T00:00:00.000Z'));
        await expect(h.repo.saveWithOptimisticLock(copyB!)).rejects.toThrow(
          OptimisticConcurrencyError,
        );
      } finally {
        await h.cleanup();
      }
    });

    it('returns due subscriptions ordered ascending by currentPeriodEnd', async () => {
      const h = await createHarness();
      const idLater = '44444444-4444-4444-8444-444444444444';
      const idEarlier = '55555555-5555-4555-8555-555555555555';
      try {
        await h.seedActive(idLater, new Date('2026-11-10T00:00:00.000Z'));
        await h.seedActive(idEarlier, new Date('2026-11-05T00:00:00.000Z'));

        const due = await h.repo.findDueSubscriptionIds(
          new Date('2026-11-12T00:00:00.000Z'),
          10,
        );
        expect(due.indexOf(idEarlier)).toBeLessThan(due.indexOf(idLater));
      } finally {
        await h.cleanup();
      }
    });
  });
}
```

Whenever a production bug exposes a discrepancy (e.g., millisecond truncation or unique constraints), adding a test to `runSubscriptionRepositoryContract` guarantees **both** adapters must match semantics before CI passes.

---

### Exercise 6: Refactoring an Over-Mapped Hexagonal Reporting Endpoint Without Breaking Boundaries

A dashboard endpoint `GET /v1/tenants/:tenantId/subscriptions` suffers 650ms p95 latency and 80MB per-request heap churn because the controller invokes `ListSubscriptionsUseCase`, which calls `SubscriptionRepositoryPort.findAllByTenant()`, hydrating 5,000 full `Subscription` Aggregates and `Money` Value Objects only to render `{ id, planCode, status, amountCents }` in a paginated table. Refactor this safely using a **CQRS Read-Port Bypass** while preserving Hexagonal boundaries and `dependency-cruiser` compliance.

---

### Solution 6

#### Diagnosis

Forcing read-only tabular projections through transactional Write Aggregates creates unnecessary object allocation, N+1 joins, and artificial mapper boilerplate without protecting any state-transition invariant.

#### CQRS Read-Port Solution

1. **Define a Narrow Read Port inside Core (`src/billing/core/ports/driven/subscription-read-query.port.ts`)**:

```typescript
export interface SubscriptionListItemView {
  id: string;
  planCode: string;
  status: string;
  amountCents: number;
  currency: string;
  currentPeriodEndIso: string;
}

export interface ListTenantSubscriptionsFilter {
  tenantId: string;
  cursorPeriodEndIso?: string;
  limit: number;
}

export const SUBSCRIPTION_READ_QUERY_PORT = Symbol('SUBSCRIPTION_READ_QUERY_PORT');

export interface SubscriptionReadQueryPort {
  listByTenant(filter: ListTenantSubscriptionsFilter): Promise<SubscriptionListItemView[]>;
}
```

2. **Implement Flat Keyset-Paginated SQL Read Adapter (`src/billing/adapters/secondary/postgres/pg-subscription-read.adapter.ts`)**:

```typescript
import { Pool } from 'pg';
import {
  ListTenantSubscriptionsFilter,
  SubscriptionListItemView,
  SubscriptionReadQueryPort,
} from '../../../core/ports/driven/subscription-read-query.port';

export class PgSubscriptionReadAdapter implements SubscriptionReadQueryPort {
  constructor(private readonly pool: Pool) {}

  async listByTenant(
    filter: ListTenantSubscriptionsFilter,
  ): Promise<SubscriptionListItemView[]> {
    const safeLimit = Math.min(Math.max(filter.limit, 1), 100);
    const { rows } = await this.pool.query<{
      id: string;
      plan_code: string;
      status: string;
      amount_cents: number;
      currency: string;
      current_period_end: Date;
    }>(
      `SELECT id, plan_code, status, amount_cents, currency, current_period_end
         FROM subscriptions
        WHERE tenant_id = $1
          AND ($2::timestamptz IS NULL OR current_period_end > $2::timestamptz)
        ORDER BY current_period_end ASC
        LIMIT $3`,
      [filter.tenantId, filter.cursorPeriodEndIso ?? null, safeLimit],
    );

    return rows.map((r) => ({
      id: r.id,
      planCode: r.plan_code,
      status: r.status,
      amountCents: Number(r.amount_cents),
      currency: r.currency,
      currentPeriodEndIso: r.current_period_end.toISOString(),
    }));
  }
}
```

3. **Bind Through a Driving Query Port (`ListTenantSubscriptionsQueryUseCase`)**:
   - The Primary HTTP Controller still invokes a **Driving Port** (satisfying `primary-adapters-must-not-bypass-to-secondary-adapters`), while the Query Use Case delegates straight to `SubscriptionReadQueryPort` with zero Aggregate hydration and zero transaction locks.
   - Latency drops from ~650ms to `<12ms` using index-only keyset pagination (`(tenant_id, current_period_end)`), while all state mutations continue passing safely through `Subscription` Aggregates.
