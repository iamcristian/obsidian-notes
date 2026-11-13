---
tags:
  - software-engineering
  - devops
  - logging
  - structured-logging
  - loki
  - security
created: 2026-11-13
updated: 2026-11-13
status: 🟢
level: mid-to-senior
---
# Structured Logging & Log Pipelines

Logs are immutable, timestamped discrete event records capturing rich per-request context that bounded-cardinality metrics must discard. Production logging demands single-line JSON schemas, **Canonical Log Lines (Wide Events)**, non-blocking buffered `stdout` I/O, automatic asynchronous context propagation across HTTP and message queues, DaemonSet log shipping, defense-in-depth recursive PII redaction, and strict storage label discipline to prevent ingestion outages and runaway cloud bills. Related foundations: [[Programming/Software Engineering/DevOps/DevOps Complete Guide|DevOps Complete Guide]], [[Programming/Software Engineering/DevOps/Monitoring|Monitoring]], [[Programming/Software Engineering/DevOps/Observability|Observability]], [[Programming/Software Engineering/DevOps/Kubernetes|Kubernetes]], [[Programming/Software Engineering/System Design/Message Queues|Message Queues]].

---

## 1. I/O Physics, Serialization Overhead & Canonical Wide Events

### 1.1 Why Blocking I/O Stalls Event Loops

Application logging executes on the hot request path. Naive logging routines introduce severe tail-latency stalls through two mechanisms: synchronous filesystem/pipe syscalls and synchronous CPU-bound JSON/string formatting.

```mermaid
flowchart LR
    Req["HTTP Request (Event Loop)"] -->|"JSON.stringify + Buffer"| Ring["Sonic-Boom Async Buffer (4 KB)"]
    Ring -->|"Non-blocking write(1)"| Pipe["POSIX Pipe (64 KB Kernel Buffer)"]
    Pipe --> CRI["containerd CRI File Writer"]
    CRI --> Disk["/var/log/pods (Rotated by kubelet)"]
```

1. **Blocking `stdout` & `fs.writeFileSync` Mechanics**:
   - In Node.js (`libuv`), `fs.writeFileSync` executes `write(fd, buf)` directly on the V8 main thread, halting JavaScript execution, socket `epoll` reads, and timer callbacks until the Linux VFS page cache or NVMe block queue returns.
   - Even `console.log()` / `process.stdout.write()` can turn synchronous under POSIX pipe backpressure! When containerized PID 1 writes to `stdout` (`fd 1`), `fd 1` connects to an anonymous Linux kernel pipe (`64 KB` default capacity) drained by `containerd-shim`. In Node.js on Linux, writes to pipes/TTYs via `process.stdout` historically block or accumulate unbounded internal `Writable` buffers whenever `containerd-shim` falls behind during bursty traffic, driving GC heap spikes or synchronous syscall stalls.
   - In Python, the standard `logging.StreamHandler` acquires a global thread lock (`RLock`) per log record, formats strings synchronously, and calls `.flush()` immediately on every line—creating severe lock contention under multi-threaded WSGI workers or starving the `asyncio` event loop.
2. **Non-Blocking Ring-Buffered Writes (`pino` / `sonic-boom`)**:
   - High-throughput Node.js loggers (`pino`) eliminate `util.format` reflection, pre-compile fixed base metadata (`service`, `env`, `version`) as serialized JSON prefixes at boot, and write bytes into an off-path **`sonic-boom`** memory buffer (`minLength: 4096`, `sync: false`) flushed asynchronously via non-blocking `fs.write` when the `4 KB` buffer fills or an explicit interval ticks.
   - **POSIX `PIPE_BUF` Atomicity (`4,096 bytes`)**: POSIX guarantees that `write()` calls $\le 4,096$ bytes to a pipe never interleave bytes with concurrent writers. Keeping canonical log records concise (`1–3 KB`) and flushing from a single `sonic-boom` writer preserves valid newline-delimited JSON even at `20,000+ lines/sec`. Always attach `SIGTERM` and `uncaughtException` handlers calling `logger.flush()` (`flushSync()`) so the last `4 KB` buffered batch is never lost during Kubernetes pod termination.

### 1.2 Fragmented Logs vs Canonical Log Lines (Wide Events)

Conventional microservice code emits `15–25` separate log statements per HTTP request (`request started`, `jwt decoded`, `db query start`, `cache miss`, `payment authorized`, `response sent`). At `3,000 RPS`, that creates `60,000+` fragmented JSON lines/sec that scramble across concurrent async coroutines, multiply serialization CPU by `15x`, and require expensive multi-line joins at query time.

**Canonical Log Lines (Wide Events)** invert this pattern:
- Allocate a single mutable request context object when the HTTP request or queue message enters the process.
- Deep inside domain services, repositories, and HTTP clients, mutate/append telemetry counters and attributes onto that request context (`db_queries += 1`, `cache_hit = false`, `tenant_tier = "enterprise"`).
- Inside `res.once("finish")` (or `finally`), serialize **one single, wide structured JSON record** capturing the entire execution summary.

| Attribute Dimension | Fragmented Chatty Logging (`20 lines/req`) | Canonical Wide Event (`1 line/req`) |
| :--- | :--- | :--- |
| **Serialization CPU & Pipe I/O** | `20x` `JSON.stringify` + `20x` timestamp calls per request | `1x` `JSON.stringify` on request completion |
| **Cross-Field Correlation** | Requires external `WHERE request_id="x"` joins across 20 rows | Every dimension sits inside the same JSON row (`tenant_id` + `duration_ms` + `db_queries`) |
| **Debuggability of Slow 200 OKs** | Must manually reconstruct timeline from scattered timestamps | Single row immediately reveals `duration_ms=1840`, `db_queries=47`, `cache_hits=0` |
| **Ingestion & Storage Volume** | Massive boilerplate header repetition (`service`, `pod`, `timestamp` repeated 20x) | Reduces raw JSON bytes per request by **`70%–85%`** |

### 1.3 Production Wide Event JSON Schema

Every emitted line must serialize as a single valid UTF-8 JSON object terminated by `\n` (no unescaped literal newlines):

| Field | Type & Format | Cardinality / Purpose |
| :--- | :--- | :--- |
| `timestamp` | RFC 3339 / ISO 8601 UTC (`2026-11-13T21:14:05.812Z`) | Monotonic chronological ordering independent of collector ingest lag |
| `level` | Enum (`debug`, `info`, `warn`, `error`, `fatal`) | Routing, severity filtering, and tail-buffered sampling policies |
| `service` / `env` / `version` | Bounded strings (`order-api`, `production`, git SHA) | Release regression correlation and bounded Loki stream labels |
| `event` | Stable dot-delimited identifier (`http.request.completed`) | Schema-invariant filtering unaffected by human prose wording edits |
| `trace_id` / `span_id` | 32-hex / 16-hex W3C identifiers | Direct correlation with [[Programming/Software Engineering/DevOps/Observability|Distributed Traces & Exemplars]] |
| `request_id` | UUIDv7 / ULID edge identifier | Request tracing across customer support tickets and edge gateways |
| `http` | `{ method, route, status_code, duration_ms, bytes_out }` | Typed numeric filtering (`duration_ms > 500`) without regex parsing |
| `actor` | `{ tenant_id, tenant_tier, auth_method }` | High-cardinality slicing (`GROUP BY tenant_id`) prohibited in metrics |
| `deps` | `{ db_queries, db_ms, cache_hits, upstream_calls }` | Immediate isolation of N+1 queries or external API stalls |
| `error` | `{ code, type, message, retryable, stack }` | Structured failure taxonomy (stack truncated/sanitized cleanly) |

---

## 2. Asynchronous Context Propagation & Cross-Boundary Correlation

Passing `logger` and `trace_id` parameters manually through every service, repository, and utility signature couples business logic to telemetry plumbing. Modern runtimes maintain implicit async-safe execution context stores while enforcing strict propagation boundaries across queues.

| Runtime | Propagation Primitive | Internal Execution Mechanism | Common Context-Loss Trap |
| :--- | :--- | :--- | :--- |
| **Node.js / TS** | `AsyncLocalStorage` (`node:async_hooks`) | V8 `AsyncContextFrame` binds promise/timer continuations automatically | Legacy `EventEmitter`s or connection pools initialized outside request scope require `AsyncLocalStorage.bind(fn)` |
| **Python** | `contextvars.ContextVar` + `structlog` | Copy-on-write mapping copied automatically onto `asyncio.create_task` | Long-lived worker loops leak prior request keys unless `clear_contextvars()` runs at entry; `ThreadPoolExecutor` needs `copy_context().run` |
| **Java / JVM** | SLF4J `MDC` + Micrometer Context Propagation | `ThreadLocal` map attached to OS/carrier threads | Drops across `CompletableFuture.supplyAsync`, reactive Reactor schedulers, or thread pools without ContextSnapshot decorators |
| **Go** | Explicit `ctx context.Context` (`slog.InfoContext`) | Immutable linked value chain passed explicitly as first parameter | Goroutines spawned with `go func()` using parent `ctx` inherit cancellation + values; background workers require `context.WithoutCancel(ctx)` |

### 2.1 Node.js `AsyncLocalStorage` Propagation Pitfalls

`AsyncLocalStorage.run(store, callback)` guarantees that every `Promise` `.then()`, `async`/`await` continuation, `setTimeout`, and `nextTick` created within `callback` sees `store` via `als.getStore()`. However, **three real-world boundaries drop `AsyncLocalStorage`**:

1. **Pre-Initialized Event Emitters & Driver Queues**: If a database driver or custom connection pool opens a TCP socket at application startup and dispatches query completion callbacks from the socket's original initialization context without wrapping callbacks in `AsyncResource`, `als.getStore()` returns `undefined` inside callbacks. Fix by wrapping legacy callbacks with `AsyncLocalStorage.snapshot()` or `AsyncLocalStorage.bind(fn)`.
2. **Producer-to-Consumer Message Brokers (Kafka, BullMQ, SQS)**: `AsyncLocalStorage` lives strictly inside one OS process's V8 isolate. When an HTTP handler pushes a job to BullMQ or publishes a Kafka record, the worker process consumes that message inside a completely unrelated `AsyncLocalStorage` context. Producers **must** serialize `trace_id`, `span_id`, `request_id`, and `tenant_id` into message headers/metadata, and workers **must** extract them into a fresh `als.run(...)` scope.
3. **Mutating Shared Store Across Concurrent `Promise.all` Sub-Tasks**: While `als.getStore()` isolates *different* HTTP requests, sibling `Promise.all` tasks spawned inside *one* request share the same mutable `WideEventContext` reference. Increment numeric counters atomically in single-threaded JS (`ctx.db_queries += 1`), and avoid overwriting shared scalar keys destructively from parallel branches.

### 2.2 Python `contextvars` & `structlog` Mechanics

Python's `contextvars` module provides native task-local storage across `asyncio` coroutines (`create_task` snapshots the caller's context automatically) and WSGI threads. Unlike Node's scoped `.run()` callback, Python's `structlog.contextvars.bind_contextvars()` mutates the active thread/task context persistently until cleared:
- Always invoke `structlog.contextvars.clear_contextvars()` at the **beginning** of every ASGI/WSGI middleware request and Celery task handler—otherwise a reused worker thread retains `tenant_id` and `trace_id` from the previous failed job!
- When offloading CPU work to `concurrent.futures.ThreadPoolExecutor` inside `asyncio`, capture the context explicitly via `ctx = contextvars.copy_context()` and execute `loop.run_in_executor(pool, ctx.run, worker_fn, arg)`.

---

## 3. Step-by-Step Practical Application: TypeScript, Python & Vector Pipeline

Below is a complete, production-ready logging architecture implementing non-blocking `pino` + `sonic-boom` wide events, `AsyncLocalStorage` HTTP + BullMQ propagation, deep recursive PII/CRLF sanitization, tail-buffered error debug breadcrumbs, Python `structlog` parity, and unit tests.

### 3.1 Production Project Layout

```text
logging-reference/
├── apps/
│   ├── order-api/
│   │   ├── src/
│   │   │   ├── logging/
│   │   │   │   ├── context.ts              # AsyncLocalStorage WideEvent store + BullMQ extraction helpers
│   │   │   │   ├── redactor.ts             # Deep recursive PII/secret masker + CRLF log-injection guard
│   │   │   │   └── logger.ts               # Non-blocking pino/sonic-boom engine + graceful SIGTERM flush
│   │   │   ├── middleware/
│   │   │   │   └── wideEventMiddleware.ts  # 1 canonical log line/req + tail-buffered DEBUG ring buffer
│   │   │   └── workers/
│   │   │       └── bullmqLogContext.ts     # Producer header injection + consumer AsyncLocalStorage re-hydration
│   │   └── test/
│   │       └── logging.test.ts             # Verification tests for redaction, CRLF, ALS & debug sampling
│   └── billing-py/
│       └── logging_setup.py                # Python FastAPI + structlog + contextvars + recursive scrubber
└── infra/
    ├── vector/
    │   └── vector.toml                     # DaemonSet CRI partial reassembly, VRL scrub, health drop, Loki sink
    └── k8s/
        └── kubelet-log-rotation.yaml       # Node kubelet ephemeral disk protection bounds
```

### 3.2 Complete Runnable TypeScript Implementation

#### `apps/order-api/src/logging/context.ts`

```typescript
import { AsyncLocalStorage } from "node:async_hooks";
import crypto from "node:crypto";

export interface DebugBreadcrumb {
  ts: string;
  msg: string;
  attrs?: Record<string, unknown>;
}

export interface WideEventContext {
  trace_id: string;
  span_id: string;
  request_id: string;
  tenant_id?: string;
  tenant_tier?: "enterprise" | "standard" | "free";
  route?: string;
  method?: string;
  db_queries: number;
  db_duration_ms: number;
  cache_hits: number;
  cache_misses: number;
  error_code?: string;
  custom_attrs: Record<string, unknown>;
  // Ring buffer holding last N debug events flushed ONLY on >=400 or slow requests
  debug_buffer: DebugBreadcrumb[];
}

export const wideEventStore = new AsyncLocalStorage<WideEventContext>();

const MAX_DEBUG_BREADCRUMBS = 25;

export function createInitialWideContext(partial: Partial<WideEventContext> = {}): WideEventContext {
  return {
    trace_id: partial.trace_id ?? crypto.randomBytes(16).toString("hex"),
    span_id: partial.span_id ?? crypto.randomBytes(8).toString("hex"),
    request_id: partial.request_id ?? crypto.randomUUID(),
    tenant_id: partial.tenant_id,
    tenant_tier: partial.tenant_tier,
    route: partial.route,
    method: partial.method,
    db_queries: 0,
    db_duration_ms: 0,
    cache_hits: 0,
    cache_misses: 0,
    error_code: partial.error_code,
    custom_attrs: partial.custom_attrs ?? {},
    debug_buffer: [],
  };
}

export function enrichWideEvent(attrs: Record<string, unknown>): void {
  const store = wideEventStore.getStore();
  if (!store) return;
  Object.assign(store.custom_attrs, attrs);
}

export function recordDbTiming(durationMs: number): void {
  const store = wideEventStore.getStore();
  if (!store) return;
  store.db_queries += 1;
  store.db_duration_ms = Number((store.db_duration_ms + durationMs).toFixed(2));
}

export function recordDebugBreadcrumb(msg: string, attrs?: Record<string, unknown>): void {
  const store = wideEventStore.getStore();
  if (!store) return;
  if (store.debug_buffer.length >= MAX_DEBUG_BREADCRUMBS) {
    store.debug_buffer.shift();
  }
  store.debug_buffer.push({
    ts: new Date().toISOString(),
    msg,
    attrs,
  });
}
```

#### `apps/order-api/src/logging/redactor.ts`

```typescript
const SENSITIVE_KEY_PATTERN =
  /^(authorization|proxy-authorization|cookie|set-cookie|password|passwd|secret|token|access_token|refresh_token|api_?key|private_?key|credit_?card|card_?number|cvv|cvc|ssn)$/i;

const JWT_PATTERN = /\beyJ[A-Za-z0-9_-]{8,}\.[A-Za-z0-9_-]{8,}\.[A-Za-z0-9_-]{8,}\b/g;
const BEARER_PATTERN = /\b(Bearer|Basic)\s+[A-Za-z0-9\-._~+/]+=*/gi;
const DSN_CRED_PATTERN = /\b(postgres(?:ql)?|mysql|redis|mongodb(?:\+srv)?|amqp):\/\/([^:\s/]+):([^@\s]+)@/gi;
const SSN_PATTERN = /\b\d{3}-\d{2}-\d{4}\b/g;
const PAN_CANDIDATE_PATTERN = /\b(?:\d[ -]?){13,19}\b/g;

function passesLuhn(digits: string): boolean {
  let sum = 0;
  let shouldDouble = false;
  for (let i = digits.length - 1; i >= 0; i -= 1) {
    let digit = digits.charCodeAt(i) - 48;
    if (shouldDouble) {
      digit *= 2;
      if (digit > 9) digit -= 9;
    }
    sum += digit;
    shouldDouble = !shouldDouble;
  }
  return sum % 10 === 0;
}

/**
 * Neutralizes CRLF / control-character log injection so untrusted request headers or user
 * inputs cannot forge synthetic JSON log lines or corrupt terminal escapes.
 */
export function sanitizeString(input: string): string {
  // Strip CR, LF, NUL, and ANSI/C0 control bytes first
  let out = input.replace(/[\r\n\x00-\x1f\x7f]/g, " ");
  out = out.replace(BEARER_PATTERN, "$1 [REDACTED]");
  out = out.replace(JWT_PATTERN, "[REDACTED_JWT]");
  out = out.replace(DSN_CRED_PATTERN, "$1://$2:[REDACTED]@");
  out = out.replace(SSN_PATTERN, "[REDACTED_SSN]");
  out = out.replace(PAN_CANDIDATE_PATTERN, (match) => {
    const digitsOnly = match.replace(/[^0-9]/g, "");
    if (digitsOnly.length >= 13 && digitsOnly.length <= 19 && passesLuhn(digitsOnly)) {
      return `[REDACTED_PAN_LAST4_${digitsOnly.slice(-4)}]`;
    }
    return match;
  });
  return out;
}

/**
 * Recursively scrubs arbitrary nested objects, arrays, and Error instances while defending
 * against circular references via WeakSet. Never rely strictly on top-level key paths.
 */
export function redactRecursive(value: unknown, seen = new WeakSet<object>(), depth = 0): unknown {
  if (value === null || value === undefined) return value;
  if (typeof value === "string") return sanitizeString(value);
  if (typeof value !== "object") return value;
  if (depth > 12) return "[MAX_DEPTH_TRUNCATED]";

  if (seen.has(value)) return "[CIRCULAR]";
  seen.add(value);

  if (value instanceof Date) return value.toISOString();

  if (value instanceof Error) {
    const errObj = value as Error & { code?: unknown; statusCode?: unknown; config?: unknown };
    return {
      type: value.name,
      message: sanitizeString(value.message),
      code: errObj.code,
      status_code: errObj.statusCode,
      // Explicitly omit Axios/Got err.config.headers where Authorization headers commonly leak
      stack: value.stack ? sanitizeString(value.stack).slice(0, 2048) : undefined,
    };
  }

  if (Array.isArray(value)) {
    return value.map((item) => redactRecursive(item, seen, depth + 1));
  }

  const sanitized: Record<string, unknown> = {};
  for (const [key, nested] of Object.entries(value as Record<string, unknown>)) {
    if (SENSITIVE_KEY_PATTERN.test(key)) {
      sanitized[key] = "[REDACTED]";
    } else {
      sanitized[key] = redactRecursive(nested, seen, depth + 1);
    }
  }
  return sanitized;
}
```

#### `apps/order-api/src/logging/logger.ts`

```typescript
import pino from "pino";
import { wideEventStore } from "./context.js";
import { redactRecursive } from "./redactor.js";

// Non-blocking sonic-boom destination: buffers up to 4 KB to keep POSIX write() non-blocking & atomic
export const logDestination = pino.destination({
  fd: 1,
  minLength: Number(process.env.LOG_BUFFER_BYTES ?? 4096),
  sync: false,
});

export const logger = pino(
  {
    level: process.env.LOG_LEVEL ?? "info",
    timestamp: pino.stdTimeFunctions.isoTime,
    base: {
      service: process.env.SERVICE_NAME ?? "order-api",
      env: process.env.NODE_ENV ?? "production",
      version: process.env.GIT_SHA ?? "dev",
    },
    formatters: {
      level(label) {
        return { level: label };
      },
      // Defense-in-depth: recursively scrub arbitrary nested log payloads before serialization
      log(object) {
        return redactRecursive(object) as Record<string, unknown>;
      },
    },
    serializers: {
      err(error: unknown) {
        return redactRecursive(error);
      },
    },
    mixin() {
      const store = wideEventStore.getStore();
      if (!store) return {};
      return {
        trace_id: store.trace_id,
        span_id: store.span_id,
        request_id: store.request_id,
        ...(store.tenant_id ? { tenant_id: store.tenant_id } : {}),
      };
    },
  },
  logDestination
);

// Guarantee buffered log records flush before pod exit on SIGTERM or crash
export function registerGracefulLogFlush(): void {
  const flushAndExit = (signal: string, code: number) => {
    logger.info({ event: "process.shutdown", signal }, "flushing buffered logs before exit");
    logDestination.flushSync();
    process.exit(code);
  };

  process.once("SIGTERM", () => flushAndExit("SIGTERM", 0));
  process.once("SIGINT", () => flushAndExit("SIGINT", 0));
}
```

#### `apps/order-api/src/middleware/wideEventMiddleware.ts`

```typescript
import type { NextFunction, Request, Response } from "express";
import { trace } from "@opentelemetry/api";
import { createInitialWideContext, wideEventStore } from "../logging/context.js";
import { logger } from "../logging/logger.js";
import { sanitizeString } from "../logging/redactor.js";

const HEALTH_PATHS = new Set(["/healthz", "/readyz", "/metrics"]);

/**
 * Deterministic hash-based tail sampling for healthy 2xx requests so 100% of errors,
 * slow requests (>500ms), and enterprise tenants emit full telemetry while baseline 200 OK
 * noise can be sampled cleanly if configured.
 */
function shouldEmitHealthyEvent(traceId: string, sampleRate: number): boolean {
  if (sampleRate >= 1) return true;
  if (sampleRate <= 0) return false;
  const tailHex = traceId.slice(-8);
  const bucket = parseInt(tailHex, 16) / 0xffffffff;
  return bucket < sampleRate;
}

export function wideEventMiddleware(
  req: Request,
  res: Response,
  next: NextFunction,
  healthySampleRate = Number(process.env.LOG_HEALTHY_SAMPLE_RATE ?? "1.0")
): void {
  if (HEALTH_PATHS.has(req.path)) {
    next();
    return;
  }

  const activeSpanCtx = trace.getActiveSpan()?.spanContext();
  const rawReqId = req.header("x-request-id");

  const ctx = createInitialWideContext({
    trace_id: activeSpanCtx?.traceId,
    span_id: activeSpanCtx?.spanId,
    request_id: rawReqId ? sanitizeString(rawReqId).slice(0, 64) : undefined,
    method: req.method,
    route: req.path,
  });

  const startNs = process.hrtime.bigint();

  wideEventStore.run(ctx, () => {
    res.once("finish", () => {
      const durationMs = Number(process.hrtime.bigint() - startNs) / 1e6;
      const statusCode = res.statusCode;
      const isFailure = statusCode >= 400;
      const isSlow = durationMs >= 500;
      const matchedRoute = req.route?.path ? `${req.baseUrl ?? ""}${req.route.path}` : "/:unmatched";

      if (!isFailure && !isSlow && !shouldEmitHealthyEvent(ctx.trace_id, healthySampleRate)) {
        return;
      }

      const wideRecord: Record<string, unknown> = {
        event: "http.request.completed",
        http: {
          method: req.method,
          route: matchedRoute,
          status_code: statusCode,
          duration_ms: Number(durationMs.toFixed(2)),
          user_agent: req.header("user-agent") ? sanitizeString(req.header("user-agent")!).slice(0, 160) : undefined,
        },
        deps: {
          db_queries: ctx.db_queries,
          db_duration_ms: ctx.db_duration_ms,
          cache_hits: ctx.cache_hits,
          cache_misses: ctx.cache_misses,
        },
        tenant_tier: ctx.tenant_tier,
        error_code: ctx.error_code,
        context: ctx.custom_attrs,
      };

      // Tail-Triggered Debug Buffering: include granular debug breadcrumbs ONLY on failures or slow outliers
      if ((statusCode >= 500 || isSlow) && ctx.debug_buffer.length > 0) {
        wideRecord.debug_breadcrumbs = ctx.debug_buffer;
      }

      if (statusCode >= 500) {
        logger.error(wideRecord, "request failed");
      } else if (statusCode >= 400 || isSlow) {
        logger.warn(wideRecord, "request degraded or client error");
      } else {
        logger.info(wideRecord, "request completed");
      }
    });

    next();
  });
}
```

#### `apps/order-api/src/workers/bullmqLogContext.ts`

```typescript
import { createInitialWideContext, wideEventStore } from "../logging/context.js";
import { logger } from "../logging/logger.js";

export interface PropagatedJobEnvelope<T> {
  payload: T;
  telemetry: {
    trace_id: string;
    span_id: string;
    request_id: string;
    tenant_id?: string;
  };
}

/**
 * Producer helper: serializes active AsyncLocalStorage correlation fields into job payload
 * before enqueuing onto Redis/BullMQ or Kafka because ALS cannot traverse process/TCP boundaries.
 */
export function wrapJobWithLogContext<T>(payload: T): PropagatedJobEnvelope<T> {
  const store = wideEventStore.getStore();
  const fallback = createInitialWideContext();
  return {
    payload,
    telemetry: {
      trace_id: store?.trace_id ?? fallback.trace_id,
      span_id: store?.span_id ?? fallback.span_id,
      request_id: store?.request_id ?? fallback.request_id,
      tenant_id: store?.tenant_id,
    },
  };
}

/**
 * Worker helper: restores correlation context inside a fresh AsyncLocalStorage.run() scope
 * and emits exactly one canonical wide event per background job execution.
 */
export async function runJobWithWideLogging<T>(
  jobId: string,
  jobName: string,
  envelope: PropagatedJobEnvelope<T>,
  handler: (payload: T) => Promise<void>
): Promise<void> {
  const ctx = createInitialWideContext({
    trace_id: envelope.telemetry.trace_id,
    span_id: envelope.telemetry.span_id,
    request_id: envelope.telemetry.request_id,
    tenant_id: envelope.telemetry.tenant_id,
  });

  const startNs = process.hrtime.bigint();

  await wideEventStore.run(ctx, async () => {
    try {
      await handler(envelope.payload);
      const durationMs = Number(process.hrtime.bigint() - startNs) / 1e6;
      logger.info(
        {
          event: "worker.job.completed",
          job_id: jobId,
          job_name: jobName,
          duration_ms: Number(durationMs.toFixed(2)),
          deps: { db_queries: ctx.db_queries, db_duration_ms: ctx.db_duration_ms },
          context: ctx.custom_attrs,
        },
        "worker job completed"
      );
    } catch (err) {
      const durationMs = Number(process.hrtime.bigint() - startNs) / 1e6;
      logger.error(
        {
          event: "worker.job.failed",
          job_id: jobId,
          job_name: jobName,
          duration_ms: Number(durationMs.toFixed(2)),
          debug_breadcrumbs: ctx.debug_buffer,
          err,
        },
        "worker job failed"
      );
      throw err;
    }
  });
}
```

#### `apps/order-api/test/logging.test.ts`

```typescript
import { describe, it } from "node:test";
import assert from "node:assert/strict";
import { redactRecursive, sanitizeString } from "../src/logging/redactor.js";
import {
  createInitialWideContext,
  enrichWideEvent,
  recordDbTiming,
  wideEventStore,
} from "../src/logging/context.js";
import { runJobWithWideLogging, wrapJobWithLogContext } from "../src/workers/bullmqLogContext.js";

describe("Structured Logging Redaction, Sanitization & Async Context", () => {
  it("recursively redacts nested secrets, Axios error headers, JWTs, DSNs, and circular objects", () => {
    const circular: Record<string, unknown> = { ok: true };
    circular.self = circular;

    const axiosLikeErr = Object.assign(new Error("Upstream 401 Bearer secret.jwt.token"), {
      code: "ERR_BAD_REQUEST",
      statusCode: 401,
      config: { headers: { Authorization: "Bearer super_secret_token_value" } },
    });

    const scrubbed = redactRecursive({
      nested: {
        Authorization: "Bearer eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjMifQ.signature1234",
        dsn: "postgres://svc_user:myPlainPass@db.internal:5432/orders",
        ssn: "123-45-6789",
      },
      err: axiosLikeErr,
      circular,
    }) as Record<string, unknown>;

    const nested = scrubbed.nested as Record<string, unknown>;
    assert.equal(nested.Authorization, "[REDACTED]");
    assert.equal(nested.dsn, "postgres://svc_user:[REDACTED]@db.internal:5432/orders");
    assert.equal(nested.ssn, "[REDACTED]");
    assert.equal((scrubbed.circular as Record<string, unknown>).self, "[CIRCULAR]");

    const errOut = scrubbed.err as Record<string, unknown>;
    assert.equal("config" in errOut, false);
    assert.match(String(errOut.message), /Bearer \[REDACTED\]/);
  });

  it("neutralizes CRLF log injection attempts in untrusted strings", () => {
    const forged = 'guest\n{"level":"info","event":"auth.bypass","admin":true}\r\n';
    const clean = sanitizeString(forged);
    assert.equal(clean.includes("\n"), false);
    assert.equal(clean.includes("\r"), false);
  });

  it("propagates correlation IDs across async awaits and BullMQ job boundaries", async () => {
    const rootCtx = createInitialWideContext({
      trace_id: "4bf92f3577b34da6a3ce929d0e0e4736",
      tenant_id: "tenant_acme",
    });

    let capturedEnvelope!: ReturnType<typeof wrapJobWithLogContext<{ orderId: string }>>;

    await wideEventStore.run(rootCtx, async () => {
      await new Promise((resolve) => setTimeout(resolve, 5));
      recordDbTiming(12.4);
      enrichWideEvent({ checkout_step: "payment_authorized" });
      capturedEnvelope = wrapJobWithLogContext({ orderId: "ord_77" });
    });

    assert.equal(capturedEnvelope.telemetry.trace_id, "4bf92f3577b34da6a3ce929d0e0e4736");
    assert.equal(capturedEnvelope.telemetry.tenant_id, "tenant_acme");

    let workerObservedTraceId = "";
    await runJobWithWideLogging("job-99", "invoice.pdf", capturedEnvelope, async () => {
      workerObservedTraceId = wideEventStore.getStore()?.trace_id ?? "";
    });
    assert.equal(workerObservedTraceId, "4bf92f3577b34da6a3ce929d0e0e4736");
  });
});
```

### 3.3 Python `structlog` + `contextvars` FastAPI Parity

#### `apps/billing-py/logging_setup.py`

```python
import logging
import re
import sys
import time
import uuid
import structlog
from fastapi import Request, Response
from structlog.contextvars import bind_contextvars, clear_contextvars

SENSITIVE_KEYS = re.compile(
    r"^(authorization|cookie|password|secret|token|api_?key|ssn|credit_card)$",
    re.IGNORECASE,
)
JWT_RE = re.compile(r"\beyJ[A-Za-z0-9_-]{8,}\.[A-Za-z0-9_-]{8,}\.[A-Za-z0-9_-]{8,}\b")


def _scrub_value(value):
    if isinstance(value, str):
        sanitized = re.sub(r"[\r\n\x00-\x1f\x7f]", " ", value)
        return JWT_RE.sub("[REDACTED_JWT]", sanitized)
    if isinstance(value, dict):
        return {
            k: "[REDACTED]" if SENSITIVE_KEYS.match(str(k)) else _scrub_value(v)
            for k, v in value.items()
        }
    if isinstance(value, list):
        return [_scrub_value(item) for item in value]
    return value


def recursive_pii_processor(_logger, _method_name: str, event_dict: dict) -> dict:
    return _scrub_value(event_dict)


def configure_logging(service_name: str = "billing-py") -> None:
    structlog.configure(
        processors=[
            structlog.contextvars.merge_contextvars,
            structlog.processors.add_log_level,
            structlog.processors.TimeStamper(fmt="iso", utc=True, key="timestamp"),
            structlog.processors.format_exc_info,
            recursive_pii_processor,
            structlog.processors.JSONRenderer(),
        ],
        wrapper_class=structlog.make_filtering_bound_logger(logging.INFO),
        context_class=dict,
        logger_factory=structlog.PrintLoggerFactory(file=sys.stdout),
        cache_logger_on_first_use=True,
    )
    bind_contextvars(service=service_name)


log = structlog.get_logger()


async def canonical_logging_middleware(request: Request, call_next) -> Response:
    if request.url.path in {"/healthz", "/metrics"}:
        return await call_next(request)

    # Mandatory in worker loops: clear any leftover contextvars from prior tasks
    clear_contextvars()
    trace_id = request.headers.get("traceparent", "").split("-")[1] if "traceparent" in request.headers else uuid.uuid4().hex
    bind_contextvars(
        service="billing-py",
        trace_id=trace_id,
        request_id=request.headers.get("x-request-id", str(uuid.uuid4())),
    )

    start = time.perf_counter()
    status_code = 500
    try:
        response = await call_next(request)
        status_code = response.status_code
        return response
    finally:
        duration_ms = round((time.perf_counter() - start) * 1000, 2)
        log_fn = log.error if status_code >= 500 else (log.warning if status_code >= 400 else log.info)
        log_fn(
            "http.request.completed",
            method=request.method,
            path=request.url.path,
            status_code=status_code,
            duration_ms=duration_ms,
        )
```

---

## 4. Log Shipping Pipeline, Storage Engines & Security Hardening

### 4.1 Container `stdout` to Object Storage Pipeline

```mermaid
flowchart LR
    App["App Pod (stdout JSON)"] --> CRI["containerd (/var/log/pods)"]
    CRI -->|"Tail + Reassemble P/F"| Vec["Vector DaemonSet"]
    Vec -->|"VRL Scrub + Drop Healthz"| Buf["Disk Buffer (2 GB/Node)"]
    Buf -->|"Batch Snappy HTTP"| Dist["Loki Distributor"]
    Dist --> Ing["Loki Ingester -> S3 Chunks"]
```

1. **Kubernetes CRI Log Format & `16 KB` Line Splitting**:
   - Container runtimes (`containerd` / `CRI-O`) wrap every `stdout`/`stderr` line into `/var/log/pods/<ns>_<pod>_<uid>/<container>/0.log` with this exact prefix:
     `2026-11-13T21:14:05.812441901Z stdout F {"timestamp":"...","level":"info",...}`
   - Notice the flag byte: `F` denotes a complete line ending in `\n`, whereas **`P`** denotes a **partial line** split when a giant stack trace or payload exceeds CRI's `16 KB` internal buffer! If your collector lacks CRI partial-line reassembly (`auto_partial_merge: true`), a `22 KB` JSON stack trace breaks into two malformed non-JSON fragments that silently fail JSON parsing downstream.
2. **Never Write Log Files Inside Container Rootfs**:
   - Writing `/app/logs/app.log` inside a container fills node `overlay2` ephemeral storage, starving the node and triggering Kubernetes `DiskPressure` Pod evictions.
   - Configure `kubelet` explicitly so rotated `stdout` logs cannot exhaust node NVMe space (`infra/k8s/kubelet-log-rotation.yaml`):

```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
containerLogMaxSize: "50Mi"
containerLogMaxFiles: 5
```

### 4.2 Production Vector DaemonSet Configuration (`vector.toml`)

Vector (written in Rust) tails `/var/log/pods`, merges partial `P` CRI lines automatically, parses JSON, drops high-volume `200 OK` probe logs, scrubs leaked credentials via Vector Remap Language (VRL), buffers onto node NVMe during downstream outages, and ships bounded stream labels plus **Loki Structured Metadata**:

```toml
# infra/vector/vector.toml
[sources.kubernetes_logs]
type = "kubernetes_logs"
auto_partial_merge = true
exclude_paths_glob_patterns = ["/var/log/pods/kube-system_*/*/*.log"]

[transforms.parse_and_filter]
type = "remap"
inputs = ["kubernetes_logs"]
source = '''
  parsed, err = parse_json(.message)
  if err == null && is_object(parsed) {
    . = merge(., parsed)
  } else {
    .raw_message = .message
    .level = "info"
  }

  # Drop noisy 200 OK liveness probes before paying network/storage costs
  route = string(.http.route) ?? ""
  status = int(.http.status_code) ?? 0
  if (route == "/healthz" || route == "/readyz" || route == "/metrics") && status == 200 {
    abort
  }

  # Normalize bounded stream metadata
  .cluster = "${CLUSTER_NAME:-prod-us-east-1}"
  .namespace = .kubernetes.pod_namespace
  .app = .kubernetes.pod_labels."app.kubernetes.io/name" ?? .kubernetes.container_name
  .level = downcase(string(.level) ?? "info")
'''

[transforms.scrub_pii_vrl]
type = "remap"
inputs = ["parse_and_filter"]
source = '''
  # Layer 2 Defense: Catch secrets emitted by third-party native libs or stdout warnings
  raw = encode_json(.)
  raw = replace(raw, r'Bearer\s+[A-Za-z0-9\-._~+/]+=*', "Bearer [REDACTED]")
  raw = replace(raw, r'eyJ[A-Za-z0-9_-]{8,}\.[A-Za-z0-9_-]{8,}\.[A-Za-z0-9_-]{8,}', "[REDACTED_JWT]")
  raw = replace(raw, r'(?i)(postgres(?:ql)?|mysql|redis|amqp)://([^:\s/]+):([^@\s]+)@', "$1://$2:[REDACTED]@")
  raw = replace(raw, r'\b\d{3}-\d{2}-\d{4}\b', "[REDACTED_SSN]")
  . = parse_json!(raw)
'''

[transforms.sample_debug_events]
type = "sample"
inputs = ["scrub_pii_vrl"]
# Keep 100% of info/warn/error/fatal, sample stray debug records down to 5%
exclude = '.level != "debug"'
rate = 20

[sinks.loki_prod]
type = "loki"
inputs = ["sample_debug_events"]
endpoint = "http://loki-gateway.observability.svc.cluster.local:80"
compression = "snappy"
out_of_order_action = "accept"

[sinks.loki_prod.buffer]
type = "disk"
max_size = 2147483648   # 2 GiB node-local NVMe ring buffer
when_full = "drop_newest"

[sinks.loki_prod.encoding]
codec = "json"

# Strictly bounded low-cardinality Loki stream labels (<= 1,000 total streams cluster-wide)
[sinks.loki_prod.labels]
cluster = "{{ cluster }}"
namespace = "{{ namespace }}"
app = "{{ app }}"
level = "{{ level }}"

# Loki 3.0+ Structured Metadata: high-cardinality trace/request IDs indexed cleanly WITHOUT stream splits!
[sinks.loki_prod.structured_metadata]
trace_id = "{{ trace_id }}"
request_id = "{{ request_id }}"
pod = "{{ kubernetes.pod_name }}"
```

### 4.3 Storage Engines & Grafana Loki Label vs Structured Metadata Internals

| Storage Engine | Indexing Architecture | Storage Footprint | Query Latency Model | Best Architectural Fit |
| :--- | :--- | :--- | :--- | :--- |
| **Grafana Loki** | Inverted index over **stream labels only**; ZSTD-compressed raw log chunks + structured metadata on S3/GCS | Very Low (`0.2x–0.35x` raw volume) | Fast within bounded streams + parallelized S3 chunk scans (`\| json`) | Cloud-native K8s logs (`5x–10x` cheaper than Elasticsearch) |
| **Elasticsearch / OpenSearch** | Full Lucene inverted index + doc values across every JSON property | High (`1.5x–3.0x` raw volume) | Sub-second global facet counts & arbitrary free-text regex | Security SIEM / compliance teams demanding instant ad-hoc multi-TB text facets |
| **ClickHouse** | Columnar ZSTD blocks + min-max / Bloom-filter data-skipping indexes | Very Low (`0.15x–0.3x` raw volume) | Fast analytical SQL `GROUP BY` across structured wide events | Unified high-volume Wide Events + Traces + Logs analytics |

#### Why Dynamic IDs Destroy Loki Stream Indexes (and How Structured Metadata Solves It)

Inside Grafana Loki, each unique combination of **Stream Labels** opens an isolated append stream inside **Ingester** memory (`head chunk`), builds an independent `1–2 MB` ZSTD block, and writes a distinct chunk object + TSDB index entry to S3.
- **Fatal Mistake**: Placing `trace_id`, `request_id`, `user_id`, or `client_ip` into Loki **Stream Labels**. At `3,000 RPS`, Loki tries to open `3,000` new streams every second (`10.8M` streams/hour), fracturing chunks into tiny `800-byte` blocks with near-zero compression, exhausting ingester RAM, throttling S3 `PutObject` IOPS, and returning **`429 TooManyStreams`**.
- **Loki 3.0+ Structured Metadata**: Attaches high-cardinality metadata (`trace_id`, `pod`, `tenant_id`) as key-value attributes stored *inside the shared chunk block alongside the log line*, NOT in the stream index! Streams remain bounded by `(cluster, namespace, app, level)`, keeping chunks large (`~1.5 MB`) with `4:1+` compression while allowing sub-second pushdown filtering via `{app="order-api"} | trace_id="4bf92f3577b34da6a3ce929d0e0e4736"` (or Bloom-accelerated lookups in Loki 3.x).

#### Essential Production LogQL Queries

```logql
# 1. Instant trace lookup via Loki Structured Metadata (zero stream explosion)
{namespace="checkout", app="order-api"} | trace_id=`4bf92f3577b34da6a3ce929d0e0e4736`

# 2. Extract JSON payload fields and filter slow enterprise tenant errors
{namespace="checkout", app="order-api", level="error"}
  | json
  | tenant_tier="enterprise"
  | http_duration_ms > 500
  | line_format "{{.timestamp}} trace={{.trace_id}} route={{.http_route}} code={{.error_code}}"

# 3. Compute p99 latency directly from Wide Event JSON logs using unwrap metric query
quantile_over_time(0.99,
  {namespace="checkout", app="order-api"}
    | json
    | http_route="/v1/orders"
    | unwrap http_duration_ms [5m]
) by (app)

# 4. Top error codes per minute over rolling 5m window
topk(10, sum by (error_code) (
  count_over_time({namespace="checkout", level="error"} | json | error_code != "" [5m])
))
```

### 4.4 Security, PII & Compliance Guardrails

1. **Log Injection (CRLF & JSON Forging) Defense**: If an API logs unsanitized `req.headers["user-agent"]` or username inputs into plaintext or naive string-interpolated logs, attackers inject `\r\n{"level":"info","event":"admin.login.ok"}` to fabricate audit trails or inject ANSI terminal escape sequences (`\x1b]2;...`) targeting engineer terminals. Strictly serialize via JSON object encoders (`JSON.stringify`) AND strip `[\r\n\x00-\x1f\x7f]` control bytes.
2. **Three-Layer Secret Redaction**:
   - **Layer 1 (Application)**: Recursive object traversal + safe Error serializer (never top-level key paths alone!).
   - **Layer 2 (DaemonSet Collector)**: Vector VRL regex scrubbing (`Bearer`, JWT `eyJ...`, DB connection URIs, PANs, SSNs) catching leaked native stderr/third-party library output before leaving cluster nodes.
   - **Layer 3 (Storage & Retention Tiering)**: Separate security audit streams (`auth.audit.*`) into an immutable S3 bucket with **Object Lock (WORM)** for 1–7 year compliance retention while expiring noisy operational debug/info logs after `7–30 days`.

---

## 5. Production Debugging & Operations Playbook

| Symptom | Root Cause | Exact Diagnostic Commands / Queries | Fix |
| :--- | :--- | :--- | :--- |
| **Loki `429 Maximum active stream limit exceeded`** | Unbounded label (`trace_id`, `user_id`, `pod`) placed in Loki `labels` instead of `structured_metadata` | `topk(10, count by (app, namespace) (loki_ingester_memory_streams))` or `logcli series '{namespace="checkout"}'` | Move dynamic IDs out of `[sinks.loki.labels]` into `structured_metadata` or JSON payload |
| **Broken JSON Lines on Giant Stack Traces** | `containerd` split a `> 16 KB` line into `P` partial chunks without collector merge | `kubectl logs <pod> \| grep -v '^{'` or inspect `/var/log/pods/.../0.log` for `stdout P` flags | Enable `auto_partial_merge = true` in Vector (`Parser cri` + `Multiline` in Fluent Bit) |
| **Node `DiskPressure` Evictions Despite Small App Image** | App writes rotating log files inside container writable `overlay2` FS or unconfigured kubelet rotation | `kubectl get --raw /api/v1/nodes/<node>/proxy/stats/summary \| jq '.pods[].ephemeral-storage'` | Switch app strictly to `stdout` (`fd 1`), remove local log files, set `containerLogMaxSize: 50Mi` |
| **Missing `trace_id` Inside Queue Consumers** | Worker callbacks execute outside HTTP `AsyncLocalStorage` context | Inspect emitted worker JSON via `kubectl logs -l app=worker \| head -n 5` | Serialize `trace_id`/`request_id` into Kafka/BullMQ envelope and wrap handler in `wideEventStore.run()` |
| **Missing Final Crash Logs on `OOM` / `SIGTERM`** | Async `sonic-boom` `4 KB` buffer did not flush before `process.exit()` | Check last 10 lines before restart: `kubectl logs <pod> --previous --tail=10` | Register `SIGTERM`/`SIGINT` calling `logDestination.flushSync()` before process exit |
| **Event Loop Lag Spikes (`> 200 ms`) Under Burst Traffic** | `console.log` / `fs.writeFileSync` blocking main V8 thread or slow synchronous stringification | Check `nodejs_eventloop_lag_p99_seconds` against log write spikes; `vector top` throughput | Replace `console.log` with buffered `pino` (`minLength: 4096`, `sync: false`) and single Wide Event per request |

### Collector & Pipeline Inspection Commands

```bash
# 1. Live inspect Vector component throughput, errors, and disk buffer utilization inside DaemonSet pod
kubectl exec -n observability ds/vector -- vector top

# 2. Validate VRL scrubbing transformations deterministically with unit tests in CI
vector test infra/vector/vector_tests.toml

# 3. Query Grafana Loki directly from CLI across structured metadata
logcli query --addr=https://loki.internal --since=1h \
  '{namespace="checkout", app="order-api"} | json | http_status_code >= 500'

# 4. Inspect raw CRI partial-line flags on a Kubernetes worker node (via debug pod)
head -n 5 /var/log/pods/checkout_order-api-7f8b_*/order-api/0.log
```

---

## 6. Worked Exercises

### Exercise 1: Fixing a Grafana Loki `429 TooManyStreams` Outage

A platform team configures Fluent Bit / Vector to forward Kubernetes JSON logs to Grafana Loki using these stream labels:

```ini
Labels job=fluentbit, namespace=$kubernetes['namespace_name'], pod=$kubernetes['pod_name'], trace_id=$trace_id, client_ip=$client_ip
```

During a `2,000 RPS` traffic spike, Loki rejects writes with `429 Maximum active stream limit exceeded`, ingester RAM saturates, and LogQL queries time out.
1. Explain why this label configuration broke Loki's chunk storage engine.
2. Provide the exact corrected Vector/Loki label + structured-metadata mapping and the corresponding LogQL query to locate all `error` events for `trace_id = "9b1deb4d3b7d4bad9bdd2b0d7b3dcb6d"`.

### Solution 1

1. **Root Cause**: `trace_id` is unique per request (`2,000` new streams/sec = `7.2M` streams/hour) and `client_ip` spans tens of thousands of unique clients, multiplied further by ephemeral `pod_name`. Loki allocates an in-memory chunk buffer and TSDB inverted index entry per unique label combination—opening millions of uncompressed `1-line` micro-chunks, exhausting ingester RAM, thrashing object storage writes, and triggering `429 TooManyStreams`.
2. **Corrected Mapping**: Keep stream labels strictly bounded (`cluster`, `namespace`, `app`, `level`) and push `trace_id`, `client_ip`, and `pod` into **`structured_metadata`** (or extract them inside JSON log payloads):

```toml
[sinks.loki_prod.labels]
cluster = "{{ cluster }}"
namespace = "{{ namespace }}"
app = "{{ app }}"
level = "{{ level }}"

[sinks.loki_prod.structured_metadata]
trace_id = "{{ trace_id }}"
client_ip = "{{ client_ip }}"
pod = "{{ kubernetes.pod_name }}"
```

**Corrected LogQL Query**:

```logql
{namespace="checkout", app="payment-api", level="error"} | trace_id="9b1deb4d3b7d4bad9bdd2b0d7b3dcb6d"
```

---

### Exercise 2: Debugging Node.js `AsyncLocalStorage` Context Loss & Secret Leak

In an Express + `pino` microservice, developers observe two production failures:
1. Logs emitted inside a legacy RabbitMQ `channel.consume(queue, (msg) => ...)` callback and a legacy callback-based payment SDK (`gateway.charge(req, (err, res) => ...)`) are **missing `trace_id`**, breaking trace correlation.
2. Failed upstream Axios requests dump `err.config.headers.Authorization: "Bearer eyJhbG..."` into log storage despite `redact: ["req.headers.authorization"]`.

Diagnose both failures and provide the exact code fixes.

### Solution 2

1. **Context Loss Root Cause & Fix**:
   - **RabbitMQ Worker Boundary**: `channel.consume` callbacks fire inside the AMQP socket's startup async context, outside any HTTP `AsyncLocalStorage.run()` scope. Extract `traceparent` / `trace_id` from `msg.properties.headers` and wrap message execution inside `wideEventStore.run(ctx, async () => { ... })`.
   - **Legacy Callback SDK (`gateway.charge`)**: If the legacy SDK invokes callbacks via internal timers or connection sockets initialized at boot, bind the active context explicitly using **`AsyncLocalStorage.bind(cb)`** or `AsyncLocalStorage.snapshot()` at the call site:
   ```typescript
   const boundCallback = AsyncLocalStorage.bind((err: Error | null, result: unknown) => {
     logger.info({ err, result }, "gateway charge callback");
   });
   gateway.charge(chargePayload, boundCallback);
   ```
2. **Secret Leak Root Cause & Fix**:
   - Top-level `pino` `redact` paths such as `"req.headers.authorization"` are **case-sensitive and path-anchored**. Axios attaches outbound headers under `err.config.headers.Authorization` (capital `A`).
   - **Fix**: Replace fragile top-level path lists with **deep recursive scrubbing** (`redactRecursive`) + explicit safe `err` serializer stripping `err.config` completely, paired with Layer-2 Vector VRL regex scrubbing (`Bearer\s+...`, `eyJ...`).

---

### Exercise 3: Log Volume & Storage Cost Sizing

A Kubernetes cluster processes **`3,000 business RPS`** plus **`200 RPS` `/healthz` probes**. Currently, each business request emits **`6` verbose JSON log lines averaging `1.2 KB` each**, and `/healthz` emits `1` line (`1.2 KB`).
- Managed SaaS Log/SIEM provider bills **`$1.50 / GB` ingested** + **`$0.20 / GB-month` 30-day retention**.
- Self-hosted Grafana Loki (S3 + EKS) achieves `4:1` ZSTD compression (`0.25x` raw volume) at `$0.023 / GB-month` S3 storage + `$0.10 / GB` raw ingest compute.

Calculate:
1. Current daily raw log volume (`GB/day`) and monthly Managed SaaS cost (`30 days`).
2. Optimized monthly Loki cost after dropping `/healthz` `200 OK` probes at Vector and consolidating the `6` per-request lines into **1 Canonical Wide Log Line** (`1.8 KB` per business request).

### Solution 3

1. **Current Daily Volume & Managed SaaS Cost** (`1 GB = 10^9 B`, `86,400 s/day`):
   - Business log rate: $3,000 \times 6 \times 1,200\text{ B} = 21.6\text{ MB/s}$
   - Health check rate: $200 \times 1,200\text{ B} = 0.24\text{ MB/s}$
   - Total raw ingest rate: $21.84\text{ MB/s}$
   - Daily raw volume: $21.84\text{ MB/s} \times 86,400\text{ s} = 1,886,976\text{ MB} \approx \mathbf{1,887\text{ GB/day}}$ (`56,610 GB / 30 days`)
   - Managed SaaS Monthly Cost:
     - Ingest: $56,610 \times \$1.50 = \$84,915$
     - Retention: $56,610 \times \$0.20 = \$11,322$
     - **Total Current Cost**: **`$96,237 / month`**
2. **Optimized Volume & Loki Cost**:
   - New raw rate (drop `/healthz` + 1 Wide Event `1.8 KB` per business req): $3,000 \times 1,800\text{ B} = 5.4\text{ MB/s}$
   - Daily volume: $5.4\text{ MB/s} \times 86,400\text{ s} = \mathbf{466.56\text{ GB/day}}$ (`13,996.8 GB / 30 days` — **`75.3%` volume cut**).
   - Loki Monthly Cost:
     - Ingest compute: $13,996.8 \times \$0.10 = \$1,399.68$
     - S3 Compressed Storage (`0.25x` = `3,499.2 GB`): $3,499.2 \times \$0.023 = \$80.48$
     - **Total Optimized Cost**: **`~$1,480 / month`** (saving **`~$94,757 / month`**, **`98.5%`** reduction).

---

### Exercise 4: Event-Loop Stalls Under `console.log` & Truncated Stack Traces

A Fastify API pod handling `1,500 RPS` suddenly experiences `800 ms` `nodejs_eventloop_lag_p99_seconds` spikes whenever a downstream validation bug logs `24 KB` JSON error objects via `console.log(JSON.stringify(err))`. Concurrently, Loki fails to parse those error records (`json_parse_error`), showing raw text fragments starting halfway through the JSON object.

1. Explain BOTH physical I/O causes behind the event-loop stall and the broken JSON fragments.
2. Provide the concrete code and pipeline fixes.

### Solution 4

1. **Dual Root Cause**:
   - **Event-Loop Block (`console.log` + synchronous pipe backpressure)**: Formatting `24 KB` payloads synchronously on `console.log()` stalls the V8 main thread on `JSON.stringify` serialization and synchronous `stdout` pipe writes (`64 KB` pipe fills while `containerd` drains `24 KB * 1,500 RPS = 36 MB/s`).
   - **CRI `16 KB` Line Split (`P` vs `F`)**: `containerd` splits any single `stdout` line exceeding `16 KB` into two physical log records tagged with CRI partial flag `P` (first `16 KB`) and `F` (remaining `8 KB`). Without collector partial-line reassembly (`auto_partial_merge`), Loki receives two unparseable half-JSON lines!
2. **Fixes**:
   - **Application**: Replace `console.log` with buffered **`pino` + `sonic-boom` (`sync: false`, `minLength: 4096`)**, enforce payload caps (`stack.slice(0, 2048)`), and emit 1 concise Wide Event (`< 4 KB`) so lines never exceed `PIPE_BUF` (`4 KB`) or CRI `16 KB` split thresholds!
   - **Collector**: Enable `auto_partial_merge = true` in Vector (`kubernetes_logs` source) so rare oversized lines still merge cleanly before JSON parsing.

---

### Exercise 5: Designing a Zero-Trust Recursive Redactor & Log-Injection Sanitizer

A security audit reports two logging defects in a legacy Node.js payments microservice:
1. Passing `username = 'alice\n{"level":"info","event":"admin.impersonate","ok":true}'` during login writes a spoofed valid JSON record into plaintext log parsers.
2. Customer support notes sometimes contain pasted `4111-1111-1111-1111` (Luhn-valid card numbers) inside deep arrays (`req.body.items[2].notes`), bypassing top-level key redaction (`"*.credit_card"`).

Write a TypeScript sanitization function that neutralizes BOTH attacks simultaneously without crashing on circular objects.

### Solution 5

1. **CRLF Neutralization**: Replace `[\r\n\x00-\x1f\x7f]` with spaces inside every nested string before serialization AND emit exclusively through JSON serializer (`pino`), preventing forged newline-delimited JSON rows.
2. **Recursive Traversal + Luhn PAN Verification**: Walk every array element and nested object key (tracking visited references via `WeakSet` to eliminate circular-reference `RangeError`), redacting sensitive keys case-insensitively AND running a Luhn checksum check over `13–19`-digit sequences inside arbitrary string values (`notes`) so valid PANs redact while normal order IDs (`1234567890123`) remain untouched:

```typescript
function sanitizeUserField(input: unknown, seen = new WeakSet<object>()): unknown {
  if (typeof input === "string") {
    const noCrlf = input.replace(/[\r\n\x00-\x1f\x7f]/g, " ");
    return noCrlf.replace(/\b(?:\d[ -]?){13,19}\b/g, (candidate) => {
      const digits = candidate.replace(/[^0-9]/g, "");
      let sum = 0;
      let dbl = false;
      for (let i = digits.length - 1; i >= 0; i -= 1) {
        let d = digits.charCodeAt(i) - 48;
        if (dbl) { d *= 2; if (d > 9) d -= 9; }
        sum += d;
        dbl = !dbl;
      }
      return sum % 10 === 0 ? `[REDACTED_PAN_${digits.slice(-4)}]` : candidate;
    });
  }
  if (!input || typeof input !== "object") return input;
  if (seen.has(input)) return "[CIRCULAR]";
  seen.add(input);
  if (Array.isArray(input)) return input.map((v) => sanitizeUserField(v, seen));
  const out: Record<string, unknown> = {};
  for (const [k, v] of Object.entries(input)) {
    out[k] = /password|secret|token|card|ssn/i.test(k) ? "[REDACTED]" : sanitizeUserField(v, seen);
  }
  return out;
}
```

---

### Exercise 6: Analytical LogQL Unwrap Queries & Error-Triggered Debug Buffering

1. An engineer wants to track `p95` database query duration (`deps.db_duration_ms`) strictly for enterprise tenants hitting `/v1/checkout` directly from Grafana Loki Wide Events (`{namespace="checkout", app="order-api"}`), without creating Prometheus cardinality issues. Write the exact **LogQL metric query**.
2. Explain why running production at `LOG_LEVEL=debug` destroys cost budgets, and describe the exact in-memory ring-buffer mechanism that preserves granular `DEBUG` diagnostics for failed requests at **zero baseline volume overhead**.

### Solution 6

1. **LogQL `unwrap` Quantile Metric Query**:
   - Parse JSON payloads (`| json` produces flattened keys like `deps_db_duration_ms` and `http_route`), filter by `tenant_tier` and `http_route`, unwrap the numeric field cleanly (dropping parsing errors via `__error__=""`), and compute `quantile_over_time`:

```logql
quantile_over_time(0.95,
  {namespace="checkout", app="order-api"}
    | json
    | tenant_tier="enterprise"
    | http_route="/v1/checkout"
    | unwrap deps_db_duration_ms
    | __error__="" [5m]
) by (app)
```

2. **Tail-Triggered Debug Ring Buffering**:
   - Running global `DEBUG` logs emits `10x–30x` extra lines on healthy `200 OK` traffic, saturating pipe I/O and S3/SaaS budgets while 99.9% of debug lines are never read.
   - **Mechanism**: Store the last `N` (`e.g., 25`) debug breadcrumbs inside a bounded circular array (`ctx.debug_buffer`) attached directly to the request's `AsyncLocalStorage` `WideEventContext`. On `200 OK` fast requests, discard `debug_buffer` when the request scope closes (`GC` reclaims it immediately). On `status >= 500` or `duration_ms > 500`, serialize `debug_buffer` inline inside the single failed Wide Event JSON line—providing 100% granular debugging breadcrumbs for failures at `< 1%` steady-state volume!

---

## Related Notes

- [[Programming/Software Engineering/DevOps/Monitoring|Monitoring]]
- [[Programming/Software Engineering/DevOps/Observability|Observability]]
- [[Programming/Software Engineering/DevOps/Kubernetes|Kubernetes]]
- [[Programming/Software Engineering/DevOps/Cloud Fundamentals|Cloud Fundamentals]]
- [[Programming/Software Engineering/DevOps/DevOps Complete Guide|Back to DevOps Hub]]
