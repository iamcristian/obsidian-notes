---
tags:
  - flashcards/business-tech
  - english/vocabulary
  - level/C1
  - review
created: 2026-11-12
total_cards: 250
---
# 💻 Business, Meetings & Engineering English — Mazo Anki C1/C2

> [!tip] Mazo de Repetición Espaciada (Anki)
> Mazo práctico de **250 expresiones reales** para reuniones, diseño de arquitectura, code reviews, entrevistas técnicas y liderazgo.
> Presiona `Alt + R` para repasar el mazo `#flashcards/business-tech`.

---

### Graceful degradation
?
- **Part of Speech:** Architecture Noun Phrase [Engineering]
- **Español:** degradación controlada / degradación progresiva
- **Definition:** The ability of a system to keep serving core features at reduced quality or capacity rather than crashing outright when a dependency fails or traffic spikes.
- **Example:** *"If the personalization service times out, the homepage should fall back to cached defaults for graceful degradation."*
- **Collocations:** *design for graceful degradation · ensure graceful degradation · degrade gracefully under load*
- **Nuance & Exam Tip:** Preferred over 'partial failure' in architecture reviews because it signals intentional resilience engineering rather than accidental breakage.
- **Word Family:** degrade gracefully (v.) · fault tolerance · circuit breaker · load shedding

---

### Single point of failure (SPOF)
?
- **Part of Speech:** Reliability Noun Phrase [Engineering]
- **Español:** punto único de fallo
- **Definition:** Any component whose malfunction stops an entire system from working because no redundancy or failover exists around it.
- **Example:** *"Running all auth checks through a single unclustered Redis node turns our cache into a single point of failure."*
- **Collocations:** *eliminate a SPOF · introduce a single point of failure · mitigate SPOFs*
- **Nuance & Exam Tip:** Pronounced letter-by-letter ('S-P-O-F') or as 'spoff' informally; avoid saying 'only point of failure,' which sounds non-idiomatic.
- **Word Family:** high availability (HA) · redundancy · failover · active-active

---

### Technical debt
?
- **Part of Speech:** Engineering Noun Phrase [Neutral]
- **Español:** deuda técnica
- **Definition:** The implied future cost of rework caused by choosing a quick or expedient implementation now instead of a cleaner, more maintainable architecture.
- **Example:** *"We can hardcode the billing tiers to hit Friday's launch, as long as we budget next sprint to pay down the technical debt."*
- **Collocations:** *accrue technical debt · pay down technical debt · service technical debt · carry tech debt*
- **Nuance & Exam Tip:** Non-natives often say 'pay technical debt'; native engineers almost always say 'pay down' or 'address' tech debt, treating it like financial interest.
- **Word Family:** tech debt (informal) · refactor · quick-and-dirty workaround · maintainability

---

### Blast radius
?
- **Part of Speech:** Reliability Idiom [Engineering]
- **Español:** radio de impacto / alcance del fallo
- **Definition:** The maximum scope of users, services, or data affected if a deployment, configuration change, or component goes wrong.
- **Example:** *"Deploying the schema change to one tenant shard first keeps the blast radius minimal if something breaks."*
- **Collocations:** *limit the blast radius · contain the blast radius · assess the blast radius*
- **Nuance & Exam Tip:** Originally a military/explosives metaphor, now standard SRE vocabulary; never use it casually about personal blame—apply it strictly to systems.
- **Word Family:** fault isolation · bulkhead pattern · canary release · containment

---

### Idempotency
?
- **Part of Speech:** Distributed Systems Noun [Formal]
- **Español:** idempotencia
- **Definition:** The property of an operation or API endpoint whereby applying it multiple times with the same input produces the exact same state as applying it once.
- **Example:** *"Every payment webhook handler must enforce idempotency using an idempotency key so retries never double-charge a customer."*
- **Collocations:** *guarantee idempotency · enforce idempotency · idempotency key · idempotent retry*
- **Nuance & Exam Tip:** Crucial in distributed systems interviews; remember that PUT and DELETE are intended to be idempotent in HTTP semantics, whereas POST usually is not.
- **Word Family:** idempotent (adj.) · at-least-once delivery · deduplication · retry-safe

---

### Eventual consistency
?
- **Part of Speech:** Distributed Systems Noun Phrase [Technical]
- **Español:** consistencia eventual / coherencia eventual
- **Definition:** A consistency model guaranteeing that, if no new updates occur, all replicas of a distributed data store will converge to the same value over time.
- **Example:** *"Our analytics dashboard can tolerate eventual consistency, whereas account balances require strong consistency."*
- **Collocations:** *trade strong for eventual consistency · converge eventually · embrace eventual consistency*
- **Nuance & Exam Tip:** Avoid saying 'final consistency'; the established computer science term is strictly 'eventual consistency' (contrasted with 'strong consistency').
- **Word Family:** strong consistency · CAP theorem · replication lag · read-your-writes

---

### Backwards compatibility
?
- **Part of Speech:** Architecture Noun Phrase [Technical]
- **Español:** retrocompatibilidad / compatibilidad hacia atrás
- **Definition:** The assurance that a newer version of a library, API, or data schema continues to work smoothly with older clients, payloads, or integrations.
- **Example:** *"Adding an optional field preserves backwards compatibility, whereas renaming an existing JSON key breaks older mobile apps."*
- **Collocations:** *maintain backwards compatibility · break backwards compatibility · backwards-compatible change*
- **Nuance & Exam Tip:** Both 'backward compatibility' (US) and 'backwards compatibility' (UK/common speech) are accepted; hyphenate before a noun ('backwards-compatible API').
- **Word Family:** forwards compatibility · breaking change · API versioning · contract testing

---

### Tight coupling
?
- **Part of Speech:** Architecture Noun Phrase [Technical]
- **Español:** acoplamiento fuerte / acoplamiento estrecho
- **Definition:** A design flaw in which modules or services depend heavily on each other's internal details, making independent changes or deployments risky.
- **Example:** *"Because the reporting worker reads directly from the checkout database, tight coupling blocks us from splitting the monolith."*
- **Collocations:** *reduce tight coupling · suffer from tight coupling · decouple services · loosely coupled*
- **Nuance & Exam Tip:** Notice the antonym pair: 'tightly coupled' (undesirable dependency) versus 'loosely coupled' (desired modularity); avoid 'strong coupling'.
- **Word Family:** loose coupling · decouple (v.) · distributed monolith · dependency inversion

---

### Separation of concerns
?
- **Part of Speech:** Design Principle Noun Phrase [Formal]
- **Español:** separación de responsabilidades / separación de incumbencias
- **Definition:** An architectural principle that divides a system into distinct layers or modules, each responsible for a single well-defined aspect of behavior.
- **Example:** *"Keeping SQL queries inside repository classes rather than HTTP controllers preserves a clean separation of concerns."*
- **Collocations:** *enforce separation of concerns · violate separation of concerns · clean separation of concerns*
- **Nuance & Exam Tip:** Often paired with Single Responsibility Principle (SRP), though 'separation of concerns' applies more broadly across architectural boundaries.
- **Word Family:** single responsibility principle · modularity · cohesion · layered architecture

---

### Bottleneck
?
- **Part of Speech:** Performance Noun / Metaphor [Neutral]
- **Español:** cuello de botella
- **Definition:** The narrowest or slowest stage in a pipeline, query path, or workflow that caps overall system throughput.
- **Example:** *"Profiling showed that synchronous image resizing—not the database—was the primary bottleneck during peak uploads."*
- **Collocations:** *identify a bottleneck · eliminate a bottleneck · I/O bottleneck · bottlenecked on*
- **Nuance & Exam Tip:** Can be used as a verb ('we are bottlenecked by disk I/O') in engineering standups; avoid 'neck of the bottle'.
- **Word Family:** throughput · contention · hot spot · critical path

---

### Edge case
?
- **Part of Speech:** Engineering Noun Phrase [Neutral]
- **Español:** caso límite / caso extremo
- **Definition:** A rare or extreme input, timing condition, or state transition that sits at the boundary of expected parameters and often exposes hidden bugs.
- **Example:** *"Make sure the date parser handles the leap-year edge case at midnight UTC before we sign off on the release."*
- **Collocations:** *handle an edge case · uncover an edge case · account for edge cases · corner case*
- **Nuance & Exam Tip:** 'Edge case' hits one boundary parameter; 'corner case' involves multiple extreme parameters colliding simultaneously, though teams often use them loosely.
- **Word Family:** corner case · happy path · boundary condition · defensive programming

---

### Sanity check
?
- **Part of Speech:** Workplace Idiom [Informal / Neutral]
- **Español:** comprobación rápida de cordura / verificación básica
- **Definition:** A quick, lightweight verification to confirm that a number, configuration, or assumption makes basic sense before deeper work or deployment.
- **Example:** *"Could you do a five-minute sanity check on my Terraform plan before I apply it to staging?"*
- **Collocations:** *run a sanity check · do a quick sanity check · sanity-check the numbers*
- **Nuance & Exam Tip:** Common in tech, though some inclusive-language guides suggest alternatives like 'confidence check' or 'quick gut check' in formal docs.
- **Word Family:** smoke test · gut check · dry run · pre-flight check

---

### Root cause analysis (RCA)
?
- **Part of Speech:** Incident Management Noun Phrase [Formal]
- **Español:** análisis de causa raíz
- **Definition:** A structured investigation following an outage or defect to uncover the underlying systemic trigger rather than merely patching the visible symptom.
- **Example:** *"After stabilizing production, the on-call engineer led a root cause analysis tracing the outage to an expired TLS certificate."*
- **Collocations:** *conduct a root cause analysis · identify the root cause · draft the RCA*
- **Nuance & Exam Tip:** Modern SRE culture emphasizes 'contributing factors' alongside 'root cause' because complex outages rarely stem from a single isolated cause.
- **Word Family:** contributing factor · five whys · blameless postmortem · incident review

---

### Blameless postmortem
?
- **Part of Speech:** SRE Culture Noun Phrase [Formal]
- **Español:** análisis post mortem sin culpables / retrospectiva de incidente sin reproches
- **Definition:** A post-incident review assuming everyone acted with good intent given the information they had, focusing on fixing tooling and safeguards instead of punishing individuals.
- **Example:** *"Because we practice blameless postmortems, engineers report near-misses openly instead of hiding manual mistakes."*
- **Collocations:** *hold a blameless postmortem · publish a blameless postmortem · postmortem action items*
- **Nuance & Exam Tip:** Using 'postmortem' without 'blameless' can feel punitive in non-SRE orgs; some teams prefer 'incident retrospective' or 'learning review'.
- **Word Family:** psychological safety · root cause analysis · near-miss · corrective action

---

### Canary release
?
- **Part of Speech:** Deployment Noun Phrase [Engineering]
- **Español:** despliegue canario / lanzamiento gradual
- **Definition:** A deployment strategy that routes a tiny percentage of live traffic to a new software version first to detect regressions before rolling out broadly.
- **Example:** *"Our canary release flagged elevated 502 errors at 2 percent traffic, triggering an automatic halt within three minutes."*
- **Collocations:** *roll out a canary release · promote the canary · canary analysis · canary deployment*
- **Nuance & Exam Tip:** Named after 'canary in a coal mine'; distinct from 'blue-green deployment' because traffic shifts incrementally rather than all at once.
- **Word Family:** progressive delivery · blue-green deployment · blast radius · feature flag

---

### Zero-downtime migration
?
- **Part of Speech:** Operations Noun Phrase [Engineering]
- **Español:** migración sin tiempo de inactividad / sin interrupción del servicio
- **Definition:** A database, schema, or infrastructure transition executed in carefully sequenced steps so active users experience no service interruption.
- **Example:** *"Using the expand-and-contract pattern enabled a zero-downtime migration across our two-billion-row orders table."*
- **Collocations:** *execute a zero-downtime migration · design for zero-downtime · expand-and-contract pattern*
- **Nuance & Exam Tip:** Hyphenate 'zero-downtime' before a noun; saying 'migration with zero downtime' requires no hyphen.
- **Word Family:** expand-and-contract · dual-write · backwards compatibility · maintenance window

---

### Rollback plan
?
- **Part of Speech:** Operations Noun Phrase [Neutral]
- **Español:** plan de reversión / plan de marcha atrás
- **Definition:** A pre-tested procedure to safely restore a system or database to its prior healthy state if a release misbehaves in production.
- **Example:** *"No database change gets approved in change management without a verified rollback plan attached to the ticket."*
- **Collocations:** *have a rollback plan · execute a rollback · test the rollback plan · roll back a release*
- **Nuance & Exam Tip:** Remember the spacing rule: 'rollback' is one word as a noun/adjective ('rollback plan'), while 'roll back' is two words as a verb ('roll back the deploy').
- **Word Family:** roll back (v.) · roll forward (v.) · forward-fix · kill switch

---

### Latency spike
?
- **Part of Speech:** Observability Noun Phrase [Engineering]
- **Español:** pico de latencia
- **Definition:** A sudden, sharp temporary increase in request response time, often visible in tail percentiles like p95 or p99 before full timeouts appear.
- **Example:** *"Whenever garbage collection pauses exceed 400 ms, we observe a pronounced p99 latency spike across checkout APIs."*
- **Collocations:** *trigger a latency spike · diagnose a latency spike · p99 latency · tail latency*
- **Nuance & Exam Tip:** Senior engineers distinguish average latency from 'tail latency' (p95/p99), because averages hide severe latency spikes affecting power users.
- **Word Family:** tail latency · jitter · timeout · cold start

---

### Throughput
?
- **Part of Speech:** Performance Noun [Technical]
- **Español:** rendimiento / capacidad de procesamiento / caudal
- **Definition:** The rate at which a system processes units of work—such as requests per second, messages per minute, or megabytes transferred—over a given period.
- **Example:** *"Batching writes boosted ingestion throughput threefold without increasing end-to-end user latency."*
- **Collocations:** *maximize throughput · sustained throughput · throughput bottleneck · requests per second*
- **Nuance & Exam Tip:** Do not confuse 'throughput' (volume over time) with 'latency' (time per single operation); optimizing one often trades off against the other.
- **Word Family:** latency · bandwidth · concurrency · saturation

---

### Deprecate
?
- **Part of Speech:** Engineering Verb [Formal]
- **Español:** declarar obsoleto / desaconsejar el uso de (previo a retirar)
- **Definition:** To formally mark an API endpoint, library method, or feature as outdated and discouraged from new use while still supporting it during a transition window.
- **Example:** *"We plan to deprecate the v1 search endpoint in Q2 and remove it completely six months later."*
- **Collocations:** *deprecate an API · deprecation warning · deprecation timeline · mark as deprecated*
- **Nuance & Exam Tip:** Crucial pitfall: 'deprecate' does NOT mean 'delete immediately'—it means warn users that removal is upcoming. Also never confuse with 'depreciate' (lose financial value).
- **Word Family:** deprecation notice · sunset (v.) · end-of-life (EOL) · migration path

---

### Instrumentation
?
- **Part of Speech:** Observability Noun [Engineering]
- **Español:** instrumentación (de métricas y trazas)
- **Definition:** The practice of embedding code, agents, or hooks inside an application to emit logs, metrics, and distributed traces for performance monitoring.
- **Example:** *"Without proper instrumentation around the third-party payment client, we are flying blind during slowdowns."*
- **Collocations:** *add instrumentation · auto-instrumentation · instrument a service · telemetry data*
- **Nuance & Exam Tip:** Used both as an uncountable noun ('missing instrumentation') and verb form ('instrument the worker'); precedes observability.
- **Word Family:** instrument (v.) · observability · OpenTelemetry · distributed tracing

---

### Observability
?
- **Part of Speech:** Architecture Noun [Formal]
- **Español:** observabilidad
- **Definition:** The degree to which engineers can understand a system's internal state and diagnose novel failure modes solely by inspecting its external telemetry outputs.
- **Example:** *"Transitioning to microservices forced us to invest heavily in observability so we could trace requests across twelve services."*
- **Collocations:** *improve observability · three pillars of observability · observability stack · observable system*
- **Nuance & Exam Tip:** Distinguish 'monitoring' (watching known dashboards/alerts for expected failures) from 'observability' (querying rich telemetry to debug unknown unknowns).
- **Word Family:** monitoring · telemetry · logs, metrics, and traces · instrumentation

---

### Deterministic
?
- **Part of Speech:** Computer Science Adjective [Formal]
- **Español:** determinista
- **Definition:** Producing the exact same output or state sequence every single time when given identical inputs and initial state, without randomness or timing variance.
- **Example:** *"If our build pipeline isn't deterministic, passing staging tests gives us false confidence about production."*
- **Collocations:** *deterministic behavior · deterministic build · non-deterministic bug · reproducible state*
- **Nuance & Exam Tip:** Great C1 word for debugging; 'non-deterministic' sounds far more precise and credible in incident reviews than 'random' or 'weird'.
- **Word Family:** non-deterministic (adj.) · idempotent · reproducible · flaky test

---

### Race condition
?
- **Part of Speech:** Concurrency Noun Phrase [Engineering]
- **Español:** condición de carrera
- **Definition:** A subtle bug where a system's outcome depends unexpectedly on the relative timing or interleaving of concurrent threads, processes, or network calls.
- **Example:** *"Two parallel checkout requests hit the inventory row simultaneously, triggering a race condition that oversold the last item."*
- **Collocations:** *trigger a race condition · eliminate a race condition · classic TOCTOU race condition*
- **Nuance & Exam Tip:** Frequently shortened verbally to 'a race' ('that looks like a race between the cache write and DB commit'); solved via locks, atomic ops, or isolation levels.
- **Word Family:** concurrency · deadlock · atomic operation · optimistic locking

---

### Dry run
?
- **Part of Speech:** Operations Idiom [Neutral]
- **Español:** ensayo general / simulación previa (en seco)
- **Definition:** A full rehearsal or non-destructive test execution of a migration, deployment, or presentation without applying real irreversible changes.
- **Example:** *"Let's run the cleanup script with the dry-run flag first so we can verify which records it would archive."*
- **Collocations:** *perform a dry run · dry-run mode · run a dry run ahead of launch*
- **Nuance & Exam Tip:** Hyphenate when modifying a noun ('a dry-run flag', 'a dry-run migration'); leave unhyphenated as a noun ('do a dry run').
- **Word Family:** dress rehearsal · sandbox · no-op mode · sanity check

---

### Proof of concept (PoC)
?
- **Part of Speech:** Engineering Noun Phrase [Neutral]
- **Español:** prueba de concepto
- **Definition:** A quick, throwaway prototype built solely to validate technical feasibility before committing engineering resources to a full production design.
- **Example:** *"Before rewriting the search engine in Rust, let's build a two-day proof of concept to verify memory usage."*
- **Collocations:** *build a PoC · validate via a proof of concept · throwaway PoC · graduate from PoC to production*
- **Nuance & Exam Tip:** Unlike an MVP (which validates market/user value with real customers), a PoC validates technical feasibility internally and should usually be thrown away.
- **Word Family:** spike · prototype · minimum viable product (MVP) · feasibility study

---

### Circuit breaker
?
- **Part of Speech:** Architecture Pattern [Engineering]
- **Español:** disyuntor / interruptor de circuito (patrón de resiliencia)
- **Definition:** A resilience pattern that detects repeated failures in a downstream dependency and temporarily short-circuits calls to prevent cascading timeouts.
- **Example:** *"Once the fraud API failed five requests in a row, the circuit breaker tripped open and routed orders to manual review."*
- **Collocations:** *trip a circuit breaker · open the circuit · half-open state · wrap calls in a circuit breaker*
- **Nuance & Exam Tip:** Uses electrical terminology: 'closed' means traffic flows normally; 'open' means requests fail fast without hitting the struggling dependency.
- **Word Family:** cascading failure · fail-fast · graceful degradation · bulkhead pattern

---

### Cascading failure
?
- **Part of Speech:** Reliability Noun Phrase [Engineering]
- **Español:** fallo en cascada
- **Definition:** A chain reaction in distributed systems where one degraded service exhausts thread pools, queues, or retries in upstream callers until the entire platform collapses.
- **Example:** *"Aggressive client retries without jitter amplified a minor DNS blip into a full cascading failure across all regions."*
- **Collocations:** *prevent a cascading failure · trigger a cascading failure · retry storm*
- **Nuance & Exam Tip:** Often caused by retry storms and missing timeouts; pairing this phrase with 'blast radius' shows strong distributed-systems fluency.
- **Word Family:** retry storm · circuit breaker · load shedding · backpressure

---

### Exponential backoff
?
- **Part of Speech:** Systems Algorithm Phrase [Engineering]
- **Español:** retroceso exponencial / espera exponencial
- **Definition:** A retry strategy that multiplies the wait interval between failed attempts—usually combined with random jitter—so clients do not overwhelm a recovering server.
- **Example:** *"Make sure the SDK uses exponential backoff with jitter rather than hammering the rate-limited endpoint every second."*
- **Collocations:** *retry with exponential backoff · exponential backoff with jitter · cap the backoff*
- **Nuance & Exam Tip:** Always mention 'with jitter' in senior system design discussions; bare exponential backoff without jitter still causes synchronized 'thundering herd' spikes.
- **Word Family:** jitter · thundering herd · retry policy · rate limiting

---

### Rate limiting
?
- **Part of Speech:** Traffic Control Noun Phrase [Engineering]
- **Español:** limitación de tasa / control de frecuencia de peticiones
- **Definition:** Controlling the rate of incoming requests a user, IP, or client token can make within a time window to protect availability and prevent abuse.
- **Example:** *"We enforced per-tenant rate limiting at the API gateway so one runaway scraper cannot starve other customers."*
- **Collocations:** *apply rate limiting · hit a rate limit · token bucket algorithm · 429 Too Many Requests*
- **Nuance & Exam Tip:** Distinguish 'rate limiting' (hard cap rejecting excess traffic with HTTP 429) from 'throttling' (slowing down or queuing traffic smoothly).
- **Word Family:** throttling · token bucket · noisy neighbor · load shedding

---

### Dead-letter queue (DLQ)
?
- **Part of Speech:** Messaging Noun Phrase [Engineering]
- **Español:** cola de mensajes fallidos / cola de mensajes muertos
- **Definition:** A dedicated holding queue where asynchronous messages that repeatedly fail processing are routed for inspection without blocking healthy events.
- **Example:** *"Poison-pill payloads get moved to the dead-letter queue after three retries so the consumer group keeps moving."*
- **Collocations:** *route to a DLQ · inspect the dead-letter queue · replay messages from the DLQ · poison pill*
- **Nuance & Exam Tip:** Never leave a DLQ unmonitored—always clarify in design docs how alerts trigger and how engineers replay or purge DLQ items.
- **Word Family:** poison pill · message broker · consumer lag · idempotency

---

### Fail-safe vs. fail-fast
?
- **Part of Speech:** Design Contrast Phrase [Engineering]
- **Español:** fallo seguro frente a fallo rápido
- **Definition:** Contrasting philosophies: fail-safe defaults to a harmless fallback state (e.g., denying access or serving cached data), while fail-fast aborts immediately to expose bugs early.
- **Example:** *"Our permission checker must be fail-safe—deny by default on error—whereas startup config validation should fail-fast."*
- **Collocations:** *adopt a fail-fast approach · fail-safe default · fail open vs fail closed*
- **Nuance & Exam Tip:** In security contexts, engineers sharpen 'fail-safe' into 'fail closed' (deny access if auth is down) versus 'fail open' (allow through, used only for non-critical gates).
- **Word Family:** fail closed · fail open · defensive programming · circuit breaker

---

### Horizontal scaling
?
- **Part of Speech:** Infrastructure Noun Phrase [Engineering]
- **Español:** escalado horizontal
- **Definition:** Increasing system capacity by adding more parallel machine instances or containers behind a load balancer rather than upgrading a single server's hardware.
- **Example:** *"Stateless API servers allow seamless horizontal scaling during Black Friday, whereas our primary SQL DB hits vertical limits."*
- **Collocations:** *scale horizontally · horizontal scaling vs vertical scaling · autoscaling group · stateless tier*
- **Nuance & Exam Tip:** Informally called 'scaling out' (horizontal) versus 'scaling up' (vertical); mixing up 'scale out' and 'scale up' is a common interview slip.
- **Word Family:** vertical scaling · scale out · stateless · load balancer

---

### Sharding
?
- **Part of Speech:** Database Architecture Noun [Engineering]
- **Español:** fragmentación horizontal (de bases de datos)
- **Definition:** Horizontal partitioning of a large dataset across multiple independent database nodes using a shard key so no single machine holds all rows or write traffic.
- **Example:** *"Sharding customer tables by organization ID distributed write load evenly, though cross-tenant analytics became harder."*
- **Collocations:** *choose a shard key · horizontal sharding · hot shard · resharding*
- **Nuance & Exam Tip:** Reserve 'sharding' for splitting data across separate servers/instances; splitting tables inside one database instance is usually called 'partitioning'.
- **Word Family:** partitioning · shard key · hot spot · consistent hashing

---

### Cache invalidation
?
- **Part of Speech:** Systems Noun Phrase [Engineering]
- **Español:** invalidación de caché
- **Definition:** The process of removing or refreshing stale cached entries when underlying source-of-truth data changes.
- **Example:** *"Stale pricing bugs usually trace back to missing cache invalidation hooks in the admin bulk-update endpoint."*
- **Collocations:** *cache invalidation strategy · invalidate the cache · stale data · TTL expiration*
- **Nuance & Exam Tip:** Famously cited via Phil Karlton's adage ('There are only two hard things in Computer Science: cache invalidation and naming things').
- **Word Family:** time-to-live (TTL) · cache stampede · write-through cache · eventual consistency

---

### Cold start
?
- **Part of Speech:** Serverless / Runtime Noun Phrase [Engineering]
- **Español:** arranque en frío / inicio en frío
- **Definition:** The extra latency incurred when a serverless function, container, or cache must initialize runtime environments and dependencies from scratch before serving its first request.
- **Example:** *"Trimming heavyweight ORM imports cut our Lambda cold starts from 2.4 seconds down to 280 milliseconds."*
- **Collocations:** *mitigate cold starts · cold-start penalty · provisioned concurrency · warm instance*
- **Nuance & Exam Tip:** Also used metaphorically in product/ML ('the cold-start problem' when a new user has zero recommendation history yet).
- **Word Family:** warm-up · provisioned concurrency · latency spike · runtime overhead

---

### Memory leak
?
- **Part of Speech:** Runtime Defect Noun Phrase [Engineering]
- **Español:** fuga de memoria
- **Definition:** A bug in which allocated memory is never released back to the runtime or OS because stale references linger, causing gradual RAM exhaustion and crashes.
- **Example:** *"Unremoved event listeners inside long-lived WebSocket handlers caused a slow memory leak that OOM-killed pods every 18 hours."*
- **Collocations:** *hunt down a memory leak · heap snapshot · out-of-memory (OOM) crash · plug a memory leak*
- **Nuance & Exam Tip:** Note the verb collocation: engineers 'track down,' 'diagnose,' or 'plug/fix' a memory leak; don't say 'lose memory' for this bug.
- **Word Family:** garbage collection · heap dump · OOM killer · resource exhaustion

---

### Concurrency vs. parallelism
?
- **Part of Speech:** Systems Contrast Phrase [Formal]
- **Español:** concurrencia frente a paralelismo
- **Definition:** Concurrency is structuring a program to interleave and manage multiple independent tasks at once, whereas parallelism is physically executing multiple computations simultaneously across cores.
- **Example:** *"Single-threaded async Node achieves high concurrency for I/O-bound workloads without true CPU parallelism."*
- **Collocations:** *concurrent execution · true parallelism · I/O-bound vs CPU-bound · worker threads*
- **Nuance & Exam Tip:** Rob Pike's classic formulation—'Concurrency is about dealing with lots of things at once; parallelism is about doing lots of things at once'—wins interviews.
- **Word Family:** asynchronous · thread safety · race condition · context switching

---

### Immutable infrastructure
?
- **Part of Speech:** DevOps Noun Phrase [Formal]
- **Español:** infraestructura inmutable
- **Definition:** An operational model where servers or containers are never modified or patched in place after deployment; instead, any change builds and replaces them with fresh images.
- **Example:** *"Adopting immutable infrastructure wiped out configuration drift across our staging and production clusters."*
- **Collocations:** *immutable container images · prevent configuration drift · replace rather than patch*
- **Nuance & Exam Tip:** Directly contrasts with 'mutable' pet servers patched via SSH ('pets vs. cattle' analogy, though 'ephemeral instances' is more formal).
- **Word Family:** configuration drift · infrastructure as code (IaC) · ephemeral · golden image

---

### Infrastructure as code (IaC)
?
- **Part of Speech:** DevOps Noun Phrase [Engineering]
- **Español:** infraestructura como código
- **Definition:** Managing and provisioning cloud networks, databases, and compute resources through version-controlled declarative configuration files rather than manual console clicks.
- **Example:** *"Moving our VPC rules into Terraform gave us infrastructure as code with peer-reviewed pull requests before any network change."*
- **Collocations:** *manage via IaC · declarative configuration · drift detection · IaC pipeline*
- **Nuance & Exam Tip:** Saying 'ClickOps' sarcastically refers to the anti-pattern of configuring cloud dashboards manually instead of using IaC.
- **Word Family:** declarative · Terraform · immutable infrastructure · GitOps

---

### Blue-green deployment
?
- **Part of Speech:** Release Engineering Noun Phrase [Engineering]
- **Español:** despliegue azul-verde
- **Definition:** A deployment technique maintaining two identical production environments—one live ('blue') and one idle/staged ('green')—and switching router traffic atomically after verification.
- **Example:** *"With blue-green deployment, rolling back takes two seconds at the load balancer if smoke tests fail on green."*
- **Collocations:** *cut over to green · blue-green cutover · instant rollback · smoke test*
- **Nuance & Exam Tip:** Costs double compute momentarily during cutover and requires schema backwards compatibility while both versions briefly coexist.
- **Word Family:** canary release · zero-downtime migration · cutover · rollback plan

---

### Feature flag
?
- **Part of Speech:** Release Engineering Noun Phrase [Neutral]
- **Español:** indicador de funcionalidad / interruptor de función
- **Definition:** A runtime configuration toggle that decouples code deployment from feature release, allowing teams to turn capabilities on or off for specific cohorts without redeploying.
- **Example:** *"Let's merge the unfinished checkout flow behind a feature flag so we avoid a painful long-lived branch."*
- **Collocations:** *wrap behind a feature flag · toggle a flag · progressive rollout · stale feature flags*
- **Nuance & Exam Tip:** Also called 'feature toggle'; remember that forgotten, permanent flags quickly become technical debt requiring scheduled cleanup.
- **Word Family:** kill switch · dark launch · trunk-based development · canary release

---

### Chaos engineering
?
- **Part of Speech:** Reliability Discipline [Engineering]
- **Español:** ingeniería del caos
- **Definition:** The discipline of intentionally injecting controlled faults—like latency, packet loss, or node termination—into systems to verify resilience before real outages occur.
- **Example:** *"Monthly chaos engineering game days proved our failover scripts actually worked under regional packet loss."*
- **Collocations:** *chaos experiment · game day · fault injection · steady-state hypothesis*
- **Nuance & Exam Tip:** Not random destruction: emphasize that real chaos engineering starts with a hypothesis and strict blast-radius guardrails.
- **Word Family:** fault injection · game day · resilience · blast radius

---

### Load shedding
?
- **Part of Speech:** Reliability Noun Phrase [Engineering]
- **Español:** descarte selectivo de carga / alivio de carga
- **Definition:** Deliberately dropping low-priority requests early when a service nears saturation so that critical workloads remain fast and healthy instead of everything timing out.
- **Example:** *"During traffic surges, our edge proxy performs load shedding on background analytics while preserving checkout calls."*
- **Collocations:** *shed excess load · priority-based load shedding · prevent overload collapse*
- **Nuance & Exam Tip:** Often paired with 'graceful degradation'; unlike client-side rate limiting, load shedding protects overall server health regardless of who sends requests.
- **Word Family:** backpressure · rate limiting · graceful degradation · brownout

---

### Backpressure
?
- **Part of Speech:** Systems Noun [Engineering]
- **Español:** contrapresión / regulación de flujo inverso
- **Definition:** A flow-control mechanism where a slow downstream consumer signals upstream producers to pause or slow down emission rather than overflowing buffers and crashing.
- **Example:** *"Without backpressure between Kafka consumers and the database writer, memory queues ballooned until the workers crashed."*
- **Collocations:** *apply backpressure · propagate backpressure · bounded queue · flow control*
- **Nuance & Exam Tip:** Always written as one word in software engineering ('backpressure'); signals high systems maturity in streaming and queue architectures.
- **Word Family:** bounded buffer · load shedding · reactive streams · consumer lag

---

### SLA vs. SLO vs. SLI
?
- **Part of Speech:** SRE Framework Triad [Formal]
- **Español:** SLA (acuerdo contractual), SLO (objetivo interno) e SLI (indicador medido)
- **Definition:** SLI is the raw measured metric (e.g., success rate), SLO is the internal reliability target (e.g., 99.95%), and SLA is the external legal contract with penalties if breached.
- **Example:** *"We set our internal SLO tighter than our customer-facing SLA so engineers get alerted well before breaching contracts."*
- **Collocations:** *breach an SLA · miss an SLO · define meaningful SLIs · 99.9% uptime SLA*
- **Nuance & Exam Tip:** Never confuse SLO and SLA in customer or leadership meetings—promising an internal SLO as a legal SLA creates financial liability.
- **Word Family:** error budget · uptime · reliability target · compliance

---

### Error budget
?
- **Part of Speech:** SRE Governance Noun Phrase [Engineering]
- **Español:** presupuesto de error / margen de indisponibilidad
- **Definition:** The allowable margin of unreliability derived from an SLO (e.g., 0.1% downtime per quarter) used to balance feature velocity against stability work.
- **Example:** *"After two major outages burned through our monthly error budget, we froze feature launches to focus purely on reliability."*
- **Collocations:** *burn through the error budget · exhaust the error budget · error budget policy*
- **Nuance & Exam Tip:** Turns emotional arguments between Product ('ship faster!') and Ops ('stop changing things!') into objective, data-driven policy.
- **Word Family:** SLA vs. SLO vs. SLI · release freeze · toil · reliability

---

### Mean time to recovery (MTTR)
?
- **Part of Speech:** SRE Metric Noun Phrase [Formal]
- **Español:** tiempo medio de recuperación
- **Definition:** The average elapsed time from detecting a production incident until service is fully restored for users.
- **Example:** *"Automated rollbacks and clear runbooks helped cut our MTTR from 47 minutes to under eight."*
- **Collocations:** *drive down MTTR · track MTTR · DORA metrics · mean time to detect (MTTD)*
- **Nuance & Exam Tip:** The 'R' can mean Recovery, Resolution, or Repair depending on org definitions—clarify whether detection time is included when comparing benchmarks.
- **Word Family:** MTTD · DORA metrics · incident response · runbook

---

### Architecture decision record (ADR)
?
- **Part of Speech:** Documentation Noun Phrase [Formal]
- **Español:** registro de decisión arquitectónica (ADR)
- **Definition:** A short, immutable markdown document capturing the context, considered options, chosen architecture decision, and trade-offs at the time it was made.
- **Example:** *"Check the ADR folder in the repo—we documented exactly why we chose Postgres over DynamoDB two years ago."*
- **Collocations:** *author an ADR · supersede an ADR · ADR log · immutable decision record*
- **Nuance & Exam Tip:** Invaluable for stopping recurring debates ('why didn't we use GraphQL?') by preserving historical constraints and rationale.
- **Word Family:** request for comments (RFC) · rationale · trade-off · institutional knowledge

---

### Request for comments (RFC)
?
- **Part of Speech:** Engineering Process Noun Phrase [Formal]
- **Español:** solicitud de comentarios / propuesta técnica (RFC)
- **Definition:** A structured written proposal circulated across engineering teams before coding major changes so peers can surface edge cases, risks, and alternatives early.
- **Example:** *"Before opening a 3,000-line PR, please circulate an RFC outlining the migration strategy and data model."*
- **Collocations:** *circulate an RFC · review an RFC · RFC sign-off · design doc*
- **Nuance & Exam Tip:** Saying 'let's write an RFC' is a diplomatic way to pause an overly hasty implementation and invite cross-team architectural alignment.
- **Word Family:** design doc · architecture decision record (ADR) · get buy-in · peer review

---

### Breaking change
?
- **Part of Speech:** Versioning Noun Phrase [Engineering]
- **Español:** cambio incompatible / cambio disruptivo (que rompe compatibilidad)
- **Definition:** A modification to an API, contract, CLI, or schema that causes existing consumers or integrations to fail unless they update their code.
- **Example:** *"Changing the timestamp format from epoch seconds to ISO-8601 is a breaking change requiring a major version bump."*
- **Collocations:** *introduce a breaking change · avoid breaking changes · semver major bump · breaking API change*
- **Nuance & Exam Tip:** Under Semantic Versioning (SemVer), any breaking change mandates incrementing the MAJOR version (e.g., 2.0.0 to 3.0.0).
- **Word Family:** backwards compatibility · semantic versioning · deprecation · contract testing

---

### Contract testing
?
- **Part of Speech:** Testing Noun Phrase [Engineering]
- **Español:** pruebas de contrato
- **Definition:** An automated testing technique verifying that an API provider and its consumers adhere to an agreed request/response schema independently without full end-to-end staging environments.
- **Example:** *"Consumer-driven contract testing caught the removed enum field in CI before the billing service ever deployed."*
- **Collocations:** *consumer-driven contract testing · Pact tests · verify API contracts · schema drift*
- **Nuance & Exam Tip:** Bridges the gap between brittle end-to-end integration tests and isolated unit tests that mock away real API changes.
- **Word Family:** API contract · schema drift · breaking change · integration testing

---

### Single source of truth (SSOT)
?
- **Part of Speech:** Data Architecture Phrase [Formal]
- **Español:** fuente única de verdad
- **Definition:** The practice of designating one authoritative system, table, or document as the canonical origin for a piece of data so conflicting copies do not diverge.
- **Example:** *"Billing state should live exclusively in Stripe as our single source of truth rather than being duplicated across three user tables."*
- **Collocations:** *act as the single source of truth · canonical data store · eliminate conflicting copies*
- **Nuance & Exam Tip:** Applies equally to documentation (one canonical Notion/repo doc) and databases; prevents silent data drift.
- **Word Family:** source of truth · canonical · data drift · synchronization

---

### Vendor lock-in
?
- **Part of Speech:** Architecture Risk Noun Phrase [Neutral]
- **Español:** dependencia tecnológica de un proveedor / cautividad del proveedor
- **Definition:** A situation where proprietary APIs, pricing, or data formats make switching cloud or software providers prohibitively expensive or technically painful.
- **Example:** *"Standardizing on open Kubernetes manifests reduces vendor lock-in compared to building exclusively on proprietary serverless glue."*
- **Collocations:** *avoid vendor lock-in · mitigate lock-in · proprietary extensions · portability*
- **Nuance & Exam Tip:** Senior engineers weigh vendor lock-in realistically against time-to-market—avoiding lock-in at all costs can lead to over-engineering.
- **Word Family:** portability · abstraction layer · trade-off · open standards

---

### Over-engineering
?
- **Part of Speech:** Critique Noun [Neutral]
- **Español:** sobreingeniería / exceso de complejidad innecesaria
- **Definition:** Designing a system with far more abstraction, scale, or flexibility than current or foreseeable business requirements justify.
- **Example:** *"Spinning up a multi-region Kafka cluster for 200 daily users is classic over-engineering when a simple cron job suffices."*
- **Collocations:** *avoid over-engineering · risk over-engineering · over-engineered solution · YAGNI*
- **Nuance & Exam Tip:** In code reviews or design docs, soften the critique by asking 'Do we need this flexibility today, or could a simpler path work for now?'
- **Word Family:** over-engineer (v.) · premature optimization · YAGNI · gold-plating

---

### Touch base
?
- **Part of Speech:** Workplace Idiom [Neutral / Informal]
- **Español:** ponerse al día / contactar brevemente
- **Definition:** To make brief contact with a colleague or stakeholder to check status, exchange quick updates, or reconnect on an ongoing topic.
- **Example:** *"Let's touch base for ten minutes after standup to coordinate the staging release."*
- **Collocations:** *touch base briefly · touch base next week · touch base with legal*
- **Nuance & Exam Tip:** Extremely common in North American workplaces, though overused by some managers; never say 'touch the base' (no article).
- **Word Family:** sync up · check in · catch up · circle back

---

### Circle back
?
- **Part of Speech:** Workplace Phrasal Verb [Neutral]
- **Español:** retomar el tema / volver sobre algo más adelante
- **Definition:** To return to an unanswered question or paused topic later once additional context, metrics, or stakeholders are available.
- **Example:** *"Let's circle back to the caching trade-offs once we review the load test metrics."*
- **Collocations:** *circle back to a topic · circle back on Friday · circle back with the team*
- **Nuance & Exam Tip:** Great for politely parking tangents in meetings, though overuse without an explicit owner or date can sound evasive.
- **Word Family:** revisit · follow up · put a pin in · touch base

---

### Loop someone in
?
- **Part of Speech:** Workplace Phrasal Verb [Neutral]
- **Español:** mantener informado a alguien / incluir a alguien en el hilo o reunión
- **Definition:** To include a colleague or stakeholder in an email thread, Slack channel, or meeting so they have visibility or can weigh in.
- **Example:** *"Before finalizing the API contract, let's loop in the mobile lead so we don't miss client constraints."*
- **Collocations:** *loop someone in on · keep someone in the loop · out of the loop*
- **Nuance & Exam Tip:** Often paired with 'cc' in email ('looping in Maria from Security'); its opposite is 'leave someone out of the loop'.
- **Word Family:** keep in the loop · pull someone in · bring up to speed · cc

---

### Take something offline
?
- **Part of Speech:** Meeting Idiom [Neutral]
- **Español:** tratar algo aparte / hablarlo fuera de la reunión
- **Definition:** To pause a detailed, contentious, or narrowly relevant discussion during a group meeting and continue it privately afterward among only the needed participants.
- **Example:** *"This IAM role syntax won't apply to everyone here—let's take it offline right after this call."*
- **Collocations:** *take this offline · take the debate offline · discuss offline*
- **Nuance & Exam Tip:** A polite, face-saving way to rescue a derailed meeting; ironically, 'offline' usually just means a smaller Slack huddle or 1:1 call.
- **Word Family:** table an agenda item · timebox a discussion · put a pin in · parking lot

---

### Push back on
?
- **Part of Speech:** Workplace Phrasal Verb [Neutral]
- **Español:** rebatir / expresar desacuerdo o resistencia ante algo
- **Definition:** To raise reasoned objections, challenge an unrealistic request, or resist a decision constructively rather than passively agreeing.
- **Example:** *"We should push back on the two-week deadline if security audits alone take ten days."*
- **Collocations:** *push back politely · push back on scope · face pushback · gentle pushback*
- **Nuance & Exam Tip:** Notice the noun is one word ('pushback'—uncountable, e.g., 'expect some pushback') while the phrasal verb is two words ('push back on').
- **Word Family:** pushback (n.) · have reservations about · voice a concern · respectfully disagree

---

### Play devil's advocate
?
- **Part of Speech:** Diplomatic Idiom [Neutral]
- **Español:** hacer de abogado del diablo
- **Definition:** To deliberately argue the opposing side or probe weaknesses in a popular proposal—not out of hostility, but to stress-test the team's reasoning.
- **Example:** *"Just to play devil's advocate: what happens to our unit economics if cloud egress fees double next year?"*
- **Collocations:** *play devil's advocate for a moment · stress-test an idea · poke holes in*
- **Nuance & Exam Tip:** Signaling 'just to play devil's advocate' gives you psychological cover to criticize a senior leader's idea without sounding confrontational.
- **Word Family:** stress-test · steelman an argument · red-team · challenge assumptions

---

### Table an agenda item
?
- **Part of Speech:** Meeting Phrase [Formal]
- **Español:** posponer un punto (EE. UU.) / poner un tema sobre la mesa (Reino Unido)
- **Definition:** In US business English, to postpone or suspend discussion of a topic until a later date; in UK English, to formally introduce a topic for immediate discussion.
- **Example:** *"Given the late hour, let's table the vendor renewal until next Tuesday's steering committee."*
- **Collocations:** *table a motion · table the discussion · shelve for later*
- **Nuance & Exam Tip:** Famous transatlantic trap! With mixed US/UK teams, avoid ambiguity by saying 'postpone until next week' (US sense) or 'put on the agenda' (UK sense).
- **Word Family:** shelve · put a pin in · put on the back burner · bring to the table

---

### Get buy-in
?
- **Part of Speech:** Leadership Collocation [Neutral]
- **Español:** conseguir respaldo / lograr la aceptación o apoyo de los implicados
- **Definition:** To persuade key stakeholders or teammates to genuinely support and commit to a proposal before execution begins.
- **Example:** *"Spending two days socializing the RFC early helped us get buy-in from both Platform and Product."*
- **Collocations:** *get executive buy-in · secure buy-in · build buy-in early · lack of buy-in*
- **Nuance & Exam Tip:** 'Buy-in' (always hyphenated as a noun) means genuine commitment, not mere grudging compliance; 'buy into' is the verb ('they bought into the vision').
- **Word Family:** buy into (v.) · consensus · get on board · socialize an idea

---

### Get on the same page
?
- **Part of Speech:** Alignment Idiom [Neutral]
- **Español:** ponerse de acuerdo / alinear criterios
- **Definition:** To ensure everyone shares an identical understanding of goals, assumptions, definitions, or next steps before proceeding.
- **Example:** *"Before writing code, let's get on the same page about what 'active user' means in the analytics spec."*
- **Collocations:** *make sure we're on the same page · get everyone on the same page · stay on the same page*
- **Nuance & Exam Tip:** Never say 'be in the same page'—English strictly requires the preposition 'on' ('on the same page').
- **Word Family:** align on priorities · shared understanding · crystal clear · cross-functional alignment

---

### Align on priorities
?
- **Part of Speech:** Workplace Collocation [Formal]
- **Español:** alinear prioridades / acordar el orden de importancia
- **Definition:** To reach explicit joint agreement across teams or leaders on which initiatives rank highest and which must wait.
- **Example:** *"Product and Infrastructure need to align on Q3 priorities before locking engineering headcount."*
- **Collocations:** *align on goals · achieve alignment · misaligned incentives · out of alignment*
- **Nuance & Exam Tip:** More executive and formal than 'get on the same page'; 'align on' works cleanly with abstract nouns ('align on scope, timeline, metrics').
- **Word Family:** alignment (n.) · get on the same page · reach consensus · trade-off

---

### Ballpark estimate
?
- **Part of Speech:** Estimation Idiom [Informal / Neutral]
- **Español:** estimación aproximada / cálculo a grandes rasgos
- **Definition:** A rough, order-of-magnitude guess of cost, effort, or duration given early before detailed discovery or scoping is done.
- **Example:** *"Even as a ballpark estimate, migrating off legacy auth looks like a six-to-eight-week effort."*
- **Collocations:** *give a ballpark figure · in the ballpark of · rough ballpark · order of magnitude*
- **Nuance & Exam Tip:** Always hedge a ballpark estimate explicitly ('rough ballpark, not a commitment') so business stakeholders don't freeze it into a hard deadline.
- **Word Family:** order of magnitude · rough cut · t-shirt sizing · back-of-the-envelope

---

### Moving the goalposts
?
- **Part of Speech:** Negotiation Idiom [Informal / Critical]
- **Español:** cambiar las reglas a mitad de partido / mover la meta
- **Definition:** Unfairly changing success criteria, scope, or acceptance bars after work has already started or targets were nearly met.
- **Example:** *"Adding GDPR export compliance three days before sign-off feels like moving the goalposts."*
- **Collocations:** *keep moving the goalposts · accuse someone of shifting the goalposts*
- **Nuance & Exam Tip:** Carries a sharp accusatory tone; in diplomatic meetings, reframe softly as 'It looks like our success criteria have evolved since kickoff.'
- **Word Family:** scope creep · shifting requirements · lock down scope · acceptance criteria

---

### Scope creep
?
- **Part of Speech:** Project Management Noun Phrase [Neutral]
- **Español:** corrupción del alcance / expansión descontrolada del alcance
- **Definition:** The gradual, unauthorized expansion of project features or requirements beyond original plans, threatening timelines and quality.
- **Example:** *"Unreviewed 'tiny' UI tweaks during QA caused serious scope creep that pushed our launch back by three weeks."*
- **Collocations:** *prevent scope creep · suffer from scope creep · guard against scope creep*
- **Nuance & Exam Tip:** Naming 'scope creep' objectively focuses criticism on the process rather than blaming the colleague requesting a new feature.
- **Word Family:** scope down · out of scope · moving the goalposts · feature creep

---

### Bandwidth
?
- **Part of Speech:** Workplace Metaphor [Informal / Neutral]
- **Español:** capacidad disponible / tiempo y energía para asumir trabajo
- **Definition:** A person's or team's available time, cognitive capacity, and workload margin to take on additional tasks or meetings.
- **Example:** *"I'd love to review the design doc, but I don't have the bandwidth until after Thursday's release."*
- **Collocations:** *have the bandwidth · lack bandwidth · free up bandwidth · mental bandwidth*
- **Nuance & Exam Tip:** Polite, professional way to say 'I am too busy' without sounding overwhelmed or dismissive—just propose when your bandwidth opens up.
- **Word Family:** capacity · workload · on my plate · stretched thin

---

### Hard stop
?
- **Part of Speech:** Meeting Idiom [Neutral]
- **Español:** límite horario estricto / corte inaplazable
- **Definition:** A fixed, non-negotiable time when a participant must exit a meeting because of another urgent commitment.
- **Example:** *"Quick heads-up: I have a hard stop at 11:00 for a customer call, so let's decide the API owner first."*
- **Collocations:** *have a hard stop at · announce a hard stop · soft stop*
- **Nuance & Exam Tip:** State your hard stop right at the start of a meeting (not at 10:59!) so the facilitator can prioritize critical decisions early.
- **Word Family:** timebox · run over · wrap up · cut off

---

### Ping someone
?
- **Part of Speech:** Tech Slang Verb [Informal]
- **Español:** dar un toque / escribir un mensaje rápido a alguien
- **Definition:** To send someone a quick message on Slack, Teams, or email to nudge them or ask a short question.
- **Example:** *"Ping me on Slack once CI turns green and I'll merge the PR right away."*
- **Collocations:** *ping someone on Slack · give someone a quick ping · drop someone a line*
- **Nuance & Exam Tip:** Standard across tech companies horizontally and vertically, though avoid 'ping' in very formal external client or C-suite emails ('reach out to' is safer).
- **Word Family:** nudge · follow up · reach out · DM

---

### Flag a blocker
?
- **Part of Speech:** Agile Collocation [Neutral]
- **Español:** avisar de un bloqueo / señalar un impedimento
- **Definition:** To proactively call attention to an obstacle, missing dependency, or unresolved decision that prevents progress.
- **Example:** *"I want to flag a blocker early: our staging cluster still lacks valid OAuth credentials from the partner team."*
- **Collocations:** *flag a blocker early · raise a blocker · remove a blocker · unblocked*
- **Nuance & Exam Tip:** 'Flag' sounds proactive and constructive; waiting silently until the deadline passes violates engineering ownership norms.
- **Word Family:** unblock · escalate · raise a red flag · dependency

---

### Double-click on
?
- **Part of Speech:** Executive Idiom [Informal / Trendy]
- **Español:** profundizar en / ampliar el detalle de
- **Definition:** To zoom into a specific sub-topic, metric, or slide to examine its details more closely during a conversation.
- **Example:** *"Before we move to Q4 hiring, can we double-click on why enterprise churn ticked up in August?"*
- **Collocations:** *double-click on a metric · double-click into that point · zoom in on*
- **Nuance & Exam Tip:** Huge in Silicon Valley executive speak, though some engineers find it overly buzzwordy—'zoom in on' or 'examine closer' feels crisper.
- **Word Family:** drill down into · unpack · zoom in on · take a closer look

---

### Drill down into
?
- **Part of Speech:** Analytical Phrasal Verb [Neutral / Formal]
- **Español:** ahondar en / desglosar en detalle
- **Definition:** To investigate progressively deeper layers of data, root causes, or operational mechanics beneath a high-level summary.
- **Example:** *"Let's drill down into the regional latency numbers to see whether the slowdown hits EU users exclusively."*
- **Collocations:** *drill down into the data · drill down by cohort · high-level vs granular*
- **Nuance & Exam Tip:** Pairs naturally with 'high-level': executives start high-level and drill down only when anomalies appear.
- **Word Family:** double-click on · unpack · granular · root cause analysis

---

### Flesh out
?
- **Part of Speech:** Workplace Phrasal Verb [Neutral]
- **Español:** desarrollar en detalle / dar forma completa a
- **Definition:** To add concrete details, edge cases, examples, or implementation steps to an initial rough outline or draft idea.
- **Example:** *"The product brief looks promising—let's spend tomorrow afternoon fleshing out the error states and edge cases."*
- **Collocations:** *flesh out a proposal · flesh out the spec · flesh out the details*
- **Nuance & Exam Tip:** Common mistake: writing 'flush out' instead of 'flesh out'. 'Flush out' means drive something out of hiding; 'flesh out' means add substance.
- **Word Family:** iron out · spec out · draft · elaborate on

---

### Put a pin in
?
- **Part of Speech:** Meeting Idiom [Informal / Neutral]
- **Español:** dejar en pausa por un momento / aparcar temporalmente
- **Definition:** To temporarily set aside a specific point during a meeting with the clear intention of returning to it later.
- **Example:** *"Great point on pricing tiers—let's put a pin in that for ten minutes while we finish the technical timeline."*
- **Collocations:** *put a pin in that · circle back to · revisit later*
- **Nuance & Exam Tip:** Warmer and less bureaucratic than 'table that item'; works great when you want to validate someone's idea while keeping the meeting moving.
- **Word Family:** circle back · park an idea · take offline · table an agenda item

---

### Unpack
?
- **Part of Speech:** Analytical / Meeting Verb [Neutral]
- **Español:** desmenuzar / analizar paso a paso
- **Definition:** To break down a dense statement, complex problem, or loaded assumption into smaller, understandable components.
- **Example:** *"Saying 'the API feels slow' covers several issues—let's unpack whether that means cold starts, payload size, or UI rendering."*
- **Collocations:** *unpack a statement · unpack the complexity · lot to unpack here*
- **Nuance & Exam Tip:** Useful diplomatic move when someone drops a vague criticism—asking 'Can we unpack what you mean by fragile?' turns heat into clarity.
- **Word Family:** break down · dissect · drill down into · clarify

---

### Chime in
?
- **Part of Speech:** Meeting Phrasal Verb [Informal / Neutral]
- **Español:** intervenir / aportar un comentario
- **Definition:** To join an active discussion or thread smoothly to contribute a brief perspective, answer, or agreement.
- **Example:** *"Feel free to chime in anytime if the proposed rollout schedule conflicts with your team's freezes."*
- **Collocations:** *chime in with a thought · jump in · weigh in*
- **Nuance & Exam Tip:** Softer and friendlier than 'interrupt'; in remote meetings, saying 'Jumping in quickly...' or 'I'd like to chime in here' signals courtesy.
- **Word Family:** weigh in · jump in · pipe up · add color

---

### Piggyback on
?
- **Part of Speech:** Collaborative Phrasal Verb [Informal / Neutral]
- **Español:** aprovechar / sumarse al hilo de (una idea o proyecto existente)
- **Definition:** To build directly onto a teammate's suggestion in conversation, or leverage an existing launch or infrastructure channel for another task.
- **Example:** *"Piggybacking on Elena's security point, we should also rotate the webhook secrets during Tuesday's deploy."*
- **Collocations:** *piggyback on someone's point · piggyback on an existing migration · build on*
- **Nuance & Exam Tip:** Gives explicit credit to the original speaker ('Piggybacking on Raj's point...'), which fosters goodwill in cross-functional meetings.
- **Word Family:** build on · echo someone's point · leverage · ride on the coattails of

---

### Echo someone's point
?
- **Part of Speech:** Diplomatic Collocation [Formal / Neutral]
- **Español:** secundar / reforzar la idea de alguien
- **Definition:** To express strong agreement with and reinforce a colleague's earlier argument, especially to amplify a voice that risked being overlooked.
- **Example:** *"I want to echo Priya's concern about test coverage—shipping without integration tests burned us last quarter."*
- **Collocations:** *echo someone's sentiment · echo that concern · second a point*
- **Nuance & Exam Tip:** Powerful allyship technique in meetings: echoing an idea while naming its originator prevents credit theft and builds consensus.
- **Word Family:** second · amplify · piggyback on · align with

---

### Play it by ear
?
- **Part of Speech:** Decision Idiom [Informal]
- **Español:** improvisar sobre la marcha / decidir según evolucionen las cosas
- **Definition:** To decide how to proceed dynamically based on unfolding events rather than locking into a rigid upfront plan.
- **Example:** *"We'll ship to 5 percent of beta users on Monday and play it by ear before opening general availability."*
- **Collocations:** *play it by ear for now · stay flexible · adapt as we go*
- **Nuance & Exam Tip:** Fine for tactical day-to-day calls, but avoid telling senior executives 'we'll play it by ear' on strategic commitments—say 'iterate based on telemetry'.
- **Word Family:** improvise · iterate · adaptive plan · course-correct

---

### Err on the side of caution
?
- **Part of Speech:** Decision Idiom [Formal / Neutral]
- **Español:** pecar de prudente / optar por la opción más segura
- **Definition:** To deliberately choose the safer, more conservative option when facing uncertainty or asymmetric risk.
- **Example:** *"Given how painful a billing mistake would be, let's err on the side of caution and keep manual approval for invoices over $10k."*
- **Collocations:** *err on the side of caution · err on the side of transparency · take a conservative stance*
- **Nuance & Exam Tip:** Also adapts flexibly to engineering values: 'err on the side of clarity', 'err on the side of over-communicating'.
- **Word Family:** play it safe · conservative estimate · risk-averse · safeguard

---

### Hedge one's bets
?
- **Part of Speech:** Strategy Idiom [Neutral]
- **Español:** cubrirse las espaldas / no jugárselo todo a una sola carta
- **Definition:** To protect against risk or uncertainty by keeping multiple options open or qualifying a prediction instead of committing fully to one path.
- **Example:** *"By keeping our containers cloud-agnostic, we're hedging our bets in case vendor pricing shifts."*
- **Collocations:** *hedge our bets · hedge a forecast · keep options open*
- **Nuance & Exam Tip:** In communication, 'hedging language' ('perhaps', 'tentatively', 'to some extent') softens pushback—too little sounds blunt, too much sounds indecisive.
- **Word Family:** diversify · contingency plan · mitigate risk · keep a foot in both camps

---

### Steelman an argument
?
- **Part of Speech:** Rhetorical Verb [C2 / Intellectual]
- **Español:** reforzar el argumento contrario en su versión más sólida (antónimo de hombre de paja)
- **Definition:** To articulate the strongest, most charitable version of an opponent's viewpoint before critiquing it—the opposite of attacking a weak strawman.
- **Example:** *"Before voting against GraphQL, let's steelman the mobile team's argument for why over-fetching hurts them."*
- **Collocations:** *steelman the opposing view · steelman vs strawman · charitable interpretation*
- **Nuance & Exam Tip:** High-status C2 tech-culture term; immediately earns trust in heated architecture debates because people feel genuinely understood.
- **Word Family:** strawman · play devil's advocate · intellectual honesty · empathetic debate

---

### Talk past each other
?
- **Part of Speech:** Communication Idiom [Neutral]
- **Español:** hablar sin entenderse / debatir en planos distintos
- **Definition:** To argue or debate while unintentionally addressing completely different meanings, assumptions, or timeframes without realizing it.
- **Example:** *"Half the meeting felt tense because Security and Product were talking past each other on what 'anonymous data' means."*
- **Collocations:** *we're talking past each other · cross-purposes · shared definitions*
- **Nuance & Exam Tip:** Pointing out 'I think we might be talking past each other' is a neutral, de-escalating reset button during circular arguments.
- **Word Family:** at cross-purposes · misalignment · unpack · get on the same page

---

### Bike-shedding
?
- **Part of Speech:** Tech Culture Noun [Informal]
- **Español:** perder el tiempo debatiendo detalles triviales (ley de la trivialidad)
- **Definition:** Spending disproportionate time debating trivial, easy-to-understand details (like button colors or bracket formatting) while neglecting complex high-stakes design issues.
- **Example:** *"Let's stop bike-shedding variable names in this RFC and focus on the data-retention architecture."*
- **Collocations:** *avoid bike-shedding · devolve into bike-shedding · Parkinson's Law of Triviality*
- **Nuance & Exam Tip:** Derived from C. Northcote Parkinson's nuclear-reactor bike-shed story; use lightly with humor so you don't dismiss genuinely helpful nitpicks harshly.
- **Word Family:** nitpicking · missing the forest for the trees · pedantic · yak shaving

---

### Split the difference
?
- **Part of Speech:** Negotiation Idiom [Neutral]
- **Español:** partir la diferencia / buscar un punto medio equitativo
- **Definition:** To settle a disagreement over numbers, timelines, or scope by choosing a compromise halfway between two competing proposals.
- **Example:** *"Design wants four weeks and Sales wants two—could we split the difference with a phased beta in three?"*
- **Collocations:** *split the difference · compromise · meet in the middle*
- **Nuance & Exam Tip:** Works best for quantitative disagreements (dates, budgets, SLAs); avoid splitting the difference on architectural integrity or security.
- **Word Family:** meet someone halfway · middle ground · compromise · give-and-take

---

### Meet someone halfway
?
- **Part of Speech:** Negotiation Idiom [Neutral]
- **Español:** ceder a mitad de camino / hacer una concesión mutua
- **Definition:** To make a reasonable concession toward another party's position in expectation that they will reciprocate so both sides can reach agreement.
- **Example:** *"If Operations automates the staging resets, Dev will gladly meet them halfway by writing stricter smoke tests."*
- **Collocations:** *willing to meet halfway · find common ground · mutual concession*
- **Nuance & Exam Tip:** Signals collaborative flexibility without looking weak; pairs nicely with explicit reciprocity ('If you X, we can meet you halfway on Y').
- **Word Family:** split the difference · compromise · concede a point · quid pro quo

---

### Read the room
?
- **Part of Speech:** Emotional Intelligence Idiom [Informal / Neutral]
- **Español:** captar el ambiente / leer el clima de la sala
- **Definition:** To accurately sense the unspoken mood, energy, hierarchy, or political sensitivities of participants in a meeting and adjust your tone accordingly.
- **Example:** *"After three hours of outage triage, reading the room meant cutting our roadmap slide deck down to two minutes."*
- **Collocations:** *fail to read the room · read the room well · situational awareness*
- **Nuance & Exam Tip:** Applies equally on Zoom/video calls—watch camera energy, long silences, and terse chat replies before pitching ambitious new scope.
- **Word Family:** situational awareness · emotional intelligence · executive presence · tact

---

### Give someone the floor
?
- **Part of Speech:** Facilitation Idiom [Formal]
- **Español:** ceder la palabra a alguien
- **Definition:** To formally yield speaking time and attention to another person so they can present or respond uninterrupted.
- **Example:** *"Before I wrap up, I want to give Sam the floor to share the customer support perspective."*
- **Collocations:** *yield the floor · take the floor · hold the floor · open the floor to questions*
- **Nuance & Exam Tip:** Great for meeting chairs ensuring quieter contributors get heard; 'hold the floor' implies dominating conversation, often negatively.
- **Word Family:** open the floor · hand over to · pass the mic · facilitate

---

### Keep things on track
?
- **Part of Speech:** Facilitation Idiom [Neutral]
- **Español:** mantener la reunión (o proyecto) encarrilada
- **Definition:** To steer a meeting or project away from tangents and delays so it stays aligned with its agenda, schedule, and objectives.
- **Example:** *"To keep things on track for our 11:30 hard stop, let's capture UI questions in the doc for async review."*
- **Collocations:** *stay on track · get back on track · derail the meeting · off track*
- **Nuance & Exam Tip:** Tactful framing ('In the interest of keeping us on track...') lets even junior engineers gently redirect senior colleagues who ramble.
- **Word Family:** stay on course · derail · timebox · shepherd

---

### Timebox a discussion
?
- **Part of Speech:** Agile Facilitation Verb [Neutral]
- **Español:** acotar el tiempo de un debate / fijar un límite de tiempo
- **Definition:** To allocate a strict, predefined block of minutes to a conversation or spike so open-ended exploration does not consume the whole schedule.
- **Example:** *"Let's timebox the schema debate to ten minutes; if we aren't aligned by then, two leads will decide offline."*
- **Collocations:** *timebox to 15 minutes · strict timebox · timeboxed spike*
- **Nuance & Exam Tip:** Invaluable when debating subjective preferences—announcing the timebox before starting makes interrupting at the buzzer feel fair.
- **Word Family:** hard stop · spike · cap · keep things on track

---

### Recap action items
?
- **Part of Speech:** Meeting Hygiene Phrase [Neutral]
- **Español:** recapitular las tareas pendientes / resumir los acuerdos
- **Definition:** To briefly summarize concrete next steps, assigned owners, and due dates at the end of a meeting so nothing falls through the cracks.
- **Example:** *"In the last three minutes, let's recap action items so everyone knows who owns each follow-up."*
- **Collocations:** *recap key takeaways · assign action items · action item with owner and ETA*
- **Nuance & Exam Tip:** An action item without a single named owner and a target date is just a wish; use 'DRI' (Directly Responsible Individual) when clarity matters.
- **Word Family:** next steps · owner · follow up · minutes

---

### Follow up with
?
- **Part of Speech:** Workplace Phrasal Verb [Neutral]
- **Español:** hacer seguimiento con / volver a contactar con
- **Definition:** To reconnect with someone after an earlier conversation or deliverable to get an answer, confirm status, or complete an agreed step.
- **Example:** *"I'll follow up with Legal tomorrow morning if they haven't reviewed the data-processing addendum."*
- **Collocations:** *follow up on a thread · follow-up email · timely follow-up · circle back*
- **Nuance & Exam Tip:** Two words as a verb ('I will follow up'); hyphenated as a noun or adjective ('send a follow-up', 'a follow-up meeting').
- **Word Family:** follow-up (n.) · circle back · nudge · close the loop

---

### Hold off on
?
- **Part of Speech:** Workplace Phrasal Verb [Neutral]
- **Español:** posponer / esperar antes de hacer algo
- **Definition:** To intentionally delay starting or releasing something until a dependency clears, risks subside, or stakeholders align.
- **Example:** *"Let's hold off on announcing the deprecation date until the v2 migration guide is published."*
- **Collocations:** *hold off on merging · hold off until Q3 · put on hold*
- **Nuance & Exam Tip:** Politer and smoother than commanding 'Don't deploy yet'; 'Let's hold off on X until Y' invites agreement naturally.
- **Word Family:** postpone · pause · defer · put on ice

---

### Walk someone through
?
- **Part of Speech:** Collaborative Phrasal Verb [Neutral]
- **Español:** explicar paso a paso / guiar a alguien por
- **Definition:** To guide a teammate or stakeholder step by step through a codebase, dashboard, architecture diagram, or proposal.
- **Example:** *"Could you walk me through the retry logic in this PR? I want to make sure I'm reading the state machine right."*
- **Collocations:** *walk through the deck · quick walkthrough · walk me through your thinking*
- **Nuance & Exam Tip:** One word as a noun ('a code walkthrough'), two words as a verb ('walk us through it'); non-intimidating way to review dense logic.
- **Word Family:** walkthrough (n.) · demo · talk through · unpack

---

### Run something by someone
?
- **Part of Speech:** Workplace Idiom [Neutral / Informal]
- **Español:** consultar algo con alguien / pedirle su opinión previa
- **Definition:** To share a draft idea, decision, or message with a colleague or manager first to get feedback or quick sanity-check approval.
- **Example:** *"Before posting the incident summary externally, let me run the wording by our VP of Engineering."*
- **Collocations:** *run an idea by you · run it past legal · quick gut check*
- **Nuance & Exam Tip:** 'Run it by you' and 'run it past you' are completely interchangeable; both show collaborative respect without sounding submissive.
- **Word Family:** sound someone out · bounce an idea off · sanity check · get sign-off

---

### Sound someone out
?
- **Part of Speech:** Diplomatic Phrasal Verb [C1 / Formal]
- **Español:** sondear a alguien / tantear la postura de alguien
- **Definition:** To discreetly gauge a stakeholder's likely reaction, concerns, or support in private before formally presenting a controversial proposal to a wider group.
- **Example:** *"I sounded out the Finance lead before the all-hands so the headcount request wouldn't catch her off guard."*
- **Collocations:** *sound out stakeholders · socialize privately · test the waters*
- **Nuance & Exam Tip:** Essential executive skill ('no surprises' rule): sounding out key decision-makers privately prevents public ambushes in steering meetings.
- **Word Family:** test the waters · socialize · pre-align · run something by

---

### Voice a concern
?
- **Part of Speech:** Diplomatic Collocation [Formal]
- **Español:** expresar una inquietud / manifestar una preocupación
- **Definition:** To state a worry, risk, or reservation clearly and constructively in a meeting or document.
- **Example:** *"Several staff engineers voiced concerns about maintaining two authentication pipelines indefinitely."*
- **Collocations:** *voice concerns about · raise a concern · address a concern · legitimate concern*
- **Nuance & Exam Tip:** Notice the verb choices: native speakers 'voice,' 'raise,' or 'flag' a concern—never 'say a concern' or 'make a concern'.
- **Word Family:** raise a red flag · have reservations about · flag · speak up

---

### Have reservations about
?
- **Part of Speech:** Diplomatic Phrase [Formal]
- **Español:** tener reservas o reparos respecto a
- **Definition:** To feel polite doubt or hesitation about fully endorsing a proposal due to unresolved risks or trade-offs.
- **Example:** *"I support the broader redesign, though I have serious reservations about shipping without a fallback toggle."*
- **Collocations:** *have reservations about · express mild reservations · without reservation*
- **Nuance & Exam Tip:** More nuanced and conciliatory than 'I disagree'; shows you have weighed the merits while highlighting specific caveats.
- **Word Family:** caveat · voice a concern · misgivings · push back on

---

### Concede a point
?
- **Part of Speech:** Debate / Negotiation Collocation [Formal]
- **Español:** conceder un punto / reconocer que la otra parte tiene razón en algo
- **Definition:** To gracefully acknowledge that another person's counterargument or piece of evidence is valid, even if you maintain your overall stance.
- **Example:** *"I'll concede that cold starts look worse in Python, though our ecosystem velocity still outweighs the latency gap."*
- **Collocations:** *concede the point · grant someone that · fair enough · make a concession*
- **Nuance & Exam Tip:** Conceding valid minor points actually increases your credibility and makes others far more receptive when you defend your core position.
- **Word Family:** grant · acknowledge · fair point · meet halfway

---

### Reach consensus
?
- **Part of Speech:** Decision-Making Collocation [Formal]
- **Español:** alcanzar un consenso
- **Definition:** To arrive at a collective decision that all key group members can accept and support, even if it was not everyone's first preference.
- **Example:** *"After reviewing three database benchmarks, the architecture guild reached consensus on PostgreSQL."*
- **Collocations:** *reach consensus · build consensus · consensus-driven · lack of consensus*
- **Nuance & Exam Tip:** Remember 'consensus' does NOT mean 100% unanimity—it means general agreement without active blocking objections; avoid 'consensus of opinion' (redundant).
- **Word Family:** disagree and commit · align on · buy-in · unanimous

---

### Disagree and commit
?
- **Part of Speech:** Leadership Principle [Neutral / Tech Culture]
- **Español:** discrepar y comprometerse
- **Definition:** A management principle where team members debate openly before a decision, but once the decider chooses a direction, everyone executes wholeheartedly without passive sabotage.
- **Example:** *"I voiced my preference for an event-driven architecture, but now that we chose REST, I'm ready to disagree and commit."*
- **Collocations:** *practice disagree and commit · voice dissent then commit · unified execution*
- **Nuance & Exam Tip:** Popularized by Intel and Amazon; signals high emotional maturity when you accept a lost debate without sulking or saying 'I told you so' later.
- **Word Family:** reach consensus · ownership · alignment · single-threaded owner

---

### Draw a line in the sand
?
- **Part of Speech:** Negotiation Idiom [Emphatic]
- **Español:** trazar una línea roja / fijar un límite infranqueable
- **Definition:** To establish a firm, non-negotiable boundary beyond which you or your team will not compromise.
- **Example:** *"Security drew a line in the sand on unencrypted PII exports—no customer deal justifies bypassing KMS encryption."*
- **Collocations:** *draw a line in the sand · red line · hard boundary · non-negotiable*
- **Nuance & Exam Tip:** Strong, dramatic idiom—save it for genuine non-negotiables like safety, compliance, or burnout prevention rather than minor preferences.
- **Word Family:** non-negotiable · hard boundary · put one's foot down · stand firm

---

### Nip something in the bud
?
- **Part of Speech:** Proactive Idiom [Neutral]
- **Español:** cortar de raíz / atajar a tiempo
- **Definition:** To address and resolve a small emerging problem, misunderstanding, or bad habit immediately before it escalates into a crisis.
- **Example:** *"Sharing explicit schema guidelines early helped nip inconsistent naming conventions in the bud."*
- **Collocations:** *nip a habit in the bud · nip scope creep in the bud · catch early*
- **Nuance & Exam Tip:** Botanical metaphor (pinching off a flower bud before it blooms); always use 'bud' singular, never 'buds'.
- **Word Family:** head off · preempt · catch early · root out

---

### Elephant in the room
?
- **Part of Speech:** Candor Idiom [Neutral]
- **Español:** el elefante en la habitación / el problema obvio del que nadie habla
- **Definition:** An obvious, sensitive, or uncomfortable issue that everyone is privately thinking about but avoiding discussing openly.
- **Example:** *"Let's address the elephant in the room: even with overtime, our current team cannot support four legacy codebases."*
- **Collocations:** *address the elephant in the room · ignore the elephant in the room · name the unspoken issue*
- **Nuance & Exam Tip:** Great phrasing for courageous leadership: prefacing with 'I'll name the elephant in the room constructively...' lowers defensiveness.
- **Word Family:** unspoken tension · candid · radical candor · call out

---

### Lose sight of the big picture
?
- **Part of Speech:** Strategic Idiom [Neutral]
- **Español:** perder la perspectiva general / perder de vista el panorama completo
- **Definition:** To become so absorbed in granular details, edge cases, or departmental turf wars that you forget the primary user or business goal.
- **Example:** *"We've debated ORM micro-benchmarks for an hour and lost sight of the big picture: shipping onboarding fixes before churn spikes."*
- **Collocations:** *keep the big picture in mind · lose sight of the goal · see the forest for the trees*
- **Nuance & Exam Tip:** Helps redirect overly tactical meetings tactfully by reminding colleagues of the shared outcome you are both optimizing for.
- **Word Family:** big picture · miss the forest for the trees · north star · zoom out

---

### Beat around the bush
?
- **Part of Speech:** Communication Idiom [Informal]
- **Español:** andarse por las ramas / dar rodeos
- **Definition:** To talk evasively around an uncomfortable topic instead of stating the core issue directly.
- **Example:** *"I won't beat around the bush—our current deploy pipeline is too brittle to support weekly enterprise releases."*
- **Collocations:** *don't beat around the bush · get straight to the point · stop beating around the bush*
- **Nuance & Exam Tip:** Saying 'I won't beat around the bush' is a crisp C1 transition before delivering bad news; telling someone else 'Stop beating around the bush' sounds blunt.
- **Word Family:** cut to the chase · mince words · get straight to the point · speak candidly

---

### Cut to the chase
?
- **Part of Speech:** Communication Idiom [Informal / Neutral]
- **Español:** ir al grano / ir directo al asunto
- **Definition:** To skip unnecessary preamble and get straight to the central point, decision, or conclusion.
- **Example:** *"Since everyone read the pre-read memo, let's cut to the chase and review the two migration options."*
- **Collocations:** *cut right to the chase · skip the preamble · get to the bottom line*
- **Nuance & Exam Tip:** Originates from early silent-film editing (cutting straight to the exciting chase scene); matches 'bottom line up front' style.
- **Word Family:** get straight to the point · bottom line · executive summary · brass tacks

---

### Put on the back burner
?
- **Part of Speech:** Prioritization Idiom [Neutral]
- **Español:** dejar en segundo plano / aparcar temporalmente
- **Definition:** To deliberately assign a valid task or project lower priority for now so urgent initiatives get full focus, without canceling it permanently.
- **Example:** *"We're putting the dark-mode redesign on the back burner until our Q1 reliability milestones stabilize."*
- **Collocations:** *put on the back burner · move to the front burner · deprioritize for now*
- **Nuance & Exam Tip:** Stove metaphor; softer than 'kill' or 'cancel' because it honors the idea's value while clearly signaling it is not active right now.
- **Word Family:** deprioritize · shelve · put on ice · front burner

---

### Deliverable
?
- **Part of Speech:** Project Management Noun [Formal / Neutral]
- **Español:** entregable
- **Definition:** A tangible, verifiable work output—such as a deployed service, signed design spec, API schema, or report—expected at the end of a phase.
- **Example:** *"Our key Q2 deliverable is a multi-tenant billing module backed by automated reconciliation tests."*
- **Collocations:** *concrete deliverable · final deliverable · hit every deliverable · scope of deliverables*
- **Nuance & Exam Tip:** Used almost exclusively as a countable noun in business ('three key deliverables'); avoid vague deliverables like 'improve quality'—they must be verifiable.
- **Word Family:** deliver (v.) · milestone · artifact · output vs outcome

---

### Stakeholder
?
- **Part of Speech:** Business Noun [Formal / Neutral]
- **Español:** parte interesada / interlocutor clave
- **Definition:** Any person, team, executive, or customer group affected by, investing in, or holding decision influence over a project's outcome.
- **Example:** *"Because billing changes affect Finance, Legal, and Support, we need all three stakeholders represented at kickoff."*
- **Collocations:** *key stakeholders · external stakeholders · align stakeholders · stakeholder buy-in*
- **Nuance & Exam Tip:** Do not confuse 'stakeholder' (anyone with a stake/interest in the project) with 'shareholder' (someone owning company stock).
- **Word Family:** sponsor · decision-maker · end user · cross-functional partner

---

### Roadmap
?
- **Part of Speech:** Product Strategy Noun [Neutral]
- **Español:** hoja de ruta
- **Definition:** A high-level visual and strategic plan communicating product direction, sequenced themes, and target horizons over quarters rather than daily tasks.
- **Example:** *"Multi-region failover isn't committed for Q3, though it sits high on our H2 infrastructure roadmap."*
- **Collocations:** *product roadmap · roadmap item · align on the roadmap · slide on the roadmap*
- **Nuance & Exam Tip:** Executive pitfall: a roadmap communicates strategic intent and sequencing, not rigid day-level delivery promises—distinguish 'committed' vs 'exploratory'.
- **Word Family:** milestone · backlog · north star · prioritization

---

### Milestone
?
- **Part of Speech:** Delivery Noun [Neutral]
- **Español:** hito
- **Definition:** A significant checkpoint or inflection event on a project timeline marking the completion of a major phase rather than routine progress.
- **Example:** *"Passing the external penetration test was the final milestone blocking our enterprise beta launch."*
- **Collocations:** *hit a milestone · miss a milestone · major milestone · milestone review*
- **Nuance & Exam Tip:** Notice the verb pairing: teams 'hit,' 'reach,' or 'miss' milestones (never 'do a milestone').
- **Word Family:** deliverable · checkpoint · go-live · critical path

---

### Low-hanging fruit
?
- **Part of Speech:** Prioritization Idiom [Neutral]
- **Español:** fruta al alcance de la mano / victorias fáciles e inmediatas
- **Definition:** High-value tasks, bug fixes, or optimizations that require minimal effort and risk to capture quickly.
- **Example:** *"Before rewriting the query engine, let's grab the low-hanging fruit by adding missing composite indexes."*
- **Collocations:** *pick the low-hanging fruit · go after low-hanging fruit · obvious low-hanging fruit*
- **Nuance & Exam Tip:** Always uncountable ('low-hanging fruit', NEVER 'fruits'). Good for quick early momentum, though over-focusing on it starves deeper structural bets.
- **Word Family:** quick win · easy win · bang for your buck · diminishing returns

---

### Minimum viable product (MVP)
?
- **Part of Speech:** Product Noun Phrase [Neutral]
- **Español:** producto mínimo viable
- **Definition:** The leanest version of a new product containing just enough core functionality to ship to early users and validate real-world value hypotheses.
- **Example:** *"For the MVP, let's support CSV uploads only and postpone live Salesforce syncing until customers prove demand."*
- **Collocations:** *scope an MVP · launch an MVP · MVP criteria · iterate beyond the MVP*
- **Nuance & Exam Tip:** Common pitfall: treating MVP as an excuse for broken, sloppy software. An MVP must still be reliable and 'viable,' just narrow in feature breadth.
- **Word Family:** minimum lovable product (MLP) · scope down · proof of concept · iterate

---

### Ship it
?
- **Part of Speech:** Engineering Slang / Verb [Informal]
- **Español:** lanzarlo a producción / publicarlo ya
- **Definition:** To release software or a feature to real users; as an exclamation, a stamp of enthusiastic approval to deploy.
- **Example:** *"The checkout flow passed all canary checks and metrics look clean—let's ship it!"*
- **Collocations:** *ship to production · ship fast and iterate · ready to ship · shipping cadence*
- **Nuance & Exam Tip:** Casual and energizing inside engineering ('Real artists ship'), but use 'release' or 'deploy to general availability' in formal client contracts.
- **Word Family:** deploy · launch · general availability (GA) · bias for action

---

### Sign off on
?
- **Part of Speech:** Approval Phrasal Verb [Formal / Neutral]
- **Español:** dar el visto bueno a / aprobar formalmente
- **Definition:** To grant formal approval or authorization so a proposal, design, budget, or release can proceed.
- **Example:** *"Legal and InfoSec both need to sign off on the data-retention policy before we enable automated archiving."*
- **Collocations:** *get sign-off from · sign off on the spec · final sign-off · executive sign-off*
- **Nuance & Exam Tip:** Verb takes 'on' ('sign off on X'); noun is hyphenated and uncountable ('waiting on Legal sign-off', not 'a sign-off' usually).
- **Word Family:** sign-off (n.) · rubber-stamp · greenlight · approve

---

### Escalate
?
- **Part of Speech:** Operations / Management Verb [Formal]
- **Español:** escalar (un problema o bloqueo a un nivel superior)
- **Definition:** To raise an unresolved blocker, outage, or priority conflict to higher leadership or specialized teams when local resolution stalls.
- **Example:** *"After waiting 48 hours on the payment vendor, we decided to escalate the ticket to their VP of Customer Success."*
- **Collocations:** *escalate an issue · escalation path · escalate early · de-escalate*
- **Nuance & Exam Tip:** In healthy engineering cultures, escalating is NOT tattling—it is responsible risk management when authority or SLA leverage exceeds your level.
- **Word Family:** escalation path · flag a blocker · page on-call · de-escalate

---

### Ownership
?
- **Part of Speech:** Workplace Value Noun [Neutral / Formal]
- **Español:** sentido de responsabilidad integral / apropiación del resultado
- **Definition:** Taking end-to-end responsibility for a system, feature, or problem—from design and deployment through monitoring and fixing unexpected fallout.
- **Example:** *"True ownership means not throwing code over the wall to QA, but checking production dashboards after deploy."*
- **Collocations:** *take ownership of · extreme ownership · clear ownership · end-to-end ownership*
- **Nuance & Exam Tip:** Highly prized in tech interviews ('demonstrated strong ownership'); contrasts with 'not my job' or 'throwing work over the wall'.
- **Word Family:** accountability · autonomous execution · DRI (Directly Responsible Individual) · steward

---

### Accountability
?
- **Part of Speech:** Governance Noun [Formal]
- **Español:** rendición de cuentas / responsabilidad exigible
- **Definition:** The explicit obligation of a person or team to answer for outcomes, decisions, and commitments against agreed expectations.
- **Example:** *"Shared ownership across four squads led to zero accountability when the migration slipped twice."*
- **Collocations:** *hold someone accountable · single point of accountability · foster accountability*
- **Nuance & Exam Tip:** Subtle distinction from 'responsibility': you can share responsibility across helpers, but ultimate accountability rests on one clear owner.
- **Word Family:** ownership · answerable · DRI · governance

---

### Leverage
?
- **Part of Speech:** Business Verb / Noun [Formal]
- **Español:** aprovechar estratégicamente / apalancar / multiplicar el impacto
- **Definition:** To utilize an existing asset, relationship, platform, or insight strategically to achieve disproportionate results with limited extra effort.
- **Example:** *"Rather than building a custom SSO parser, we can leverage our existing identity gateway across all internal tools."*
- **Collocations:** *leverage existing infrastructure · negotiating leverage · high-leverage activity*
- **Nuance & Exam Tip:** Overused in corporate jargon ('leverage synergies'); keep it concrete by pairing it with technical assets ('leverage Redis', 'negotiating leverage').
- **Word Family:** capitalize on · force multiplier · reuse · bargaining chip

---

### Mitigate risk
?
- **Part of Speech:** Risk Management Collocation [Formal]
- **Español:** mitigar el riesgo / reducir el impacto probable
- **Definition:** To take proactive technical or operational steps that reduce either the likelihood or potential damage of an adverse event.
- **Example:** *"We can mitigate migration risk by running dual-writes in shadow mode for a full week before switching reads."*
- **Collocations:** *mitigate the risk of · risk mitigation strategy · mitigate downside · residual risk*
- **Nuance & Exam Tip:** 'Mitigate' means reduce severity or likelihood—NEVER use 'mitigate' to mean 'eliminate 100%'.
- **Word Family:** de-risk · contingency plan · safeguard · blast radius

---

### De-risk
?
- **Part of Speech:** Product / Engineering Verb [Neutral]
- **Español:** reducir la incertidumbre o el riesgo de un proyecto
- **Definition:** To proactively tackle the riskiest technical unknowns, customer assumptions, or compliance hurdles early so a project does not fail late.
- **Example:** *"Building the payment integration spike in sprint one will de-risk the whole Q4 launch."*
- **Collocations:** *de-risk the launch · de-risk an assumption · front-load risk*
- **Nuance & Exam Tip:** Punchier and more active than 'mitigate risk'; implies tackling the scariest unknowns FIRST instead of polishing easy UI work early.
- **Word Family:** mitigate risk · front-load uncertainty · proof of concept · spike

---

### Prioritize ruthlessly
?
- **Part of Speech:** Executive Collocation [Emphatic]
- **Español:** priorizar sin contemplaciones / descartar con rigor lo secundario
- **Definition:** To make tough, disciplined trade-offs—explicitly cutting or deferring good ideas so scarce time and talent go strictly to the highest-impact work.
- **Example:** *"With two engineers out on leave, we need to prioritize ruthlessly and drop all non-essential dashboard widgets."*
- **Collocations:** *ruthless prioritization · cut scope · stack-rank priorities*
- **Nuance & Exam Tip:** Saying 'everything is P0' means nothing is prioritized; executive fluency requires naming what you are deliberately NOT doing.
- **Word Family:** stack-rank · scope down · opportunity cost · trade-off

---

### Trade-off
?
- **Part of Speech:** Decision Noun [Neutral / Formal]
- **Español:** compromiso / contrapartida / equilibrio entre ventajas e inconvenientes
- **Definition:** A balanced exchange where gaining one desirable property (speed, simplicity, consistency) requires sacrificing or compromising another.
- **Example:** *"Every caching layer involves a classic trade-off between lower read latency and stale-data complexity."*
- **Collocations:** *weigh the trade-offs · make a conscious trade-off · acceptable trade-off · latency-consistency trade-off*
- **Nuance & Exam Tip:** Always hyphenate as a noun ('a trade-off'), but two words as a verb ('trade off memory for speed'). Seniority shines in explaining trade-offs explicitly.
- **Word Family:** trade off (v.) · opportunity cost · compromise · cost-benefit

---

### ROI (return on investment)
?
- **Part of Speech:** Finance / Product Acronym [Formal]
- **Español:** retorno de la inversión
- **Definition:** A ratio or practical measure comparing the net value gained from an initiative against the engineering time, money, and maintenance cost spent.
- **Example:** *"Automating that weekly manual CSV report costs three engineering weeks for ten minutes saved—the ROI just isn't there."*
- **Collocations:** *high ROI · negative ROI · justify the ROI · time-to-ROI*
- **Nuance & Exam Tip:** Essential vocabulary when justifying refactoring or platform tooling to product managers: frame tech work in terms of future velocity ROI.
- **Word Family:** cost-benefit ratio · payback period · value proposition · opportunity cost

---

### Cost-benefit ratio
?
- **Part of Speech:** Analytical Noun Phrase [Formal]
- **Español:** relación coste-beneficio
- **Definition:** A structured comparison weighing the total implementation, operational, and maintenance costs of an option against its projected gains.
- **Example:** *"Running our own Kubernetes control plane had a poor cost-benefit ratio compared to managed EKS for a twenty-person team."*
- **Collocations:** *favorable cost-benefit ratio · weigh the cost-benefit · cost-benefit analysis*
- **Nuance & Exam Tip:** Remember to factor in ongoing maintenance ('Day 2 ops'), not merely upfront coding cost, when presenting a cost-benefit analysis.
- **Word Family:** ROI · trade-off · total cost of ownership (TCO) · build vs buy

---

### Greenfield vs. brownfield
?
- **Part of Speech:** Architecture Context Contrast [Engineering]
- **Español:** proyecto desde cero (greenfield) frente a entorno heredado (brownfield)
- **Definition:** Greenfield refers to building a new system from scratch with no legacy constraints; brownfield means building within or integrating around existing legacy code and live traffic.
- **Example:** *"The analytics microservice was a clean greenfield build, whereas checkout remains a complex brownfield integration."*
- **Collocations:** *greenfield project · brownfield modernization · legacy constraints*
- **Nuance & Exam Tip:** Borrowed from real estate/construction; interviewers love candidates who recognize that real production engineering is 90% brownfield.
- **Word Family:** legacy system · clean slate · strangler fig pattern · technical debt

---

### Ramp up
?
- **Part of Speech:** Workplace Phrasal Verb [Neutral]
- **Español:** ponerse al ritmo / coger velocidad (en un rol o proyecto)
- **Definition:** To gradually increase familiarity, productivity, or throughput—used for new hires learning a codebase or systems scaling toward target traffic.
- **Example:** *"Good architecture docs help new backend engineers ramp up in three weeks instead of two months."*
- **Collocations:** *ramp-up time · ramp up quickly · ramp up traffic · steep ramp-up*
- **Nuance & Exam Tip:** Two words as verb ('ramp up fast'); hyphenated as noun/adjective ('a short ramp-up period'). Opposite is 'ramp down' or 'wind down'.
- **Word Family:** ramp-up (n.) · onboard · get up to speed · learning curve

---

### Onboard
?
- **Part of Speech:** HR / Engineering Verb [Neutral]
- **Español:** incorporar / integrar (a un nuevo empleado o cliente)
- **Definition:** To formally integrate and equip a new employee, team, or enterprise customer with access, context, documentation, and workflows.
- **Example:** *"We updated our local dev container so new hires can onboard and merge their first PR on day two."*
- **Collocations:** *onboard a new hire · customer onboarding · onboarding checklist · seamless onboarding*
- **Nuance & Exam Tip:** Applies both to employees ('employee onboarding') and B2B SaaS customers ('client onboarding'); 'offboard' is the clean antonym for departing users.
- **Word Family:** onboarding (n.) · offboard · ramp up · knowledge transfer

---

### Handoff
?
- **Part of Speech:** Collaboration Noun [Neutral]
- **Español:** traspaso / entrega (de turno, proyecto o diseño)
- **Definition:** The structured transfer of ownership, context, and next actions from one role, team, or on-call shift to another.
- **Example:** *"Smooth design-to-engineering handoff requires annotated states for loading, empty views, and errors."*
- **Collocations:** *clean handoff · on-call handoff · drop the ball at handoff · seamless handoff*
- **Nuance & Exam Tip:** One word as a noun ('the shift handoff'); two words as a verb ('hand off the incident to EMEA'). Also spelled 'hand-off' in UK usage.
- **Word Family:** hand off (v.) · baton pass · knowledge transfer · transition

---

### Backlog grooming
?
- **Part of Speech:** Agile Process Noun Phrase [Neutral]
- **Español:** refinamiento del backlog / priorización y depuración de tareas pendientes
- **Definition:** A recurring team session to review, clarify, estimate, split, and reprioritize upcoming tickets so sprints start with well-defined work.
- **Example:** *"Ten minutes of weekly backlog grooming saved us from picking up vague, unestimated tickets during sprint planning."*
- **Collocations:** *backlog refinement · groom the backlog · stale backlog tickets*
- **Nuance & Exam Tip:** Important cultural shift: many organizations now prefer 'backlog refinement' over 'backlog grooming' because 'grooming' carries negative connotations elsewhere.
- **Word Family:** backlog refinement · sprint planning · story points · triage

---

### Unblock
?
- **Part of Speech:** Agile / Leadership Verb [Neutral]
- **Español:** desbloquear / eliminar un impedimento para que alguien avance
- **Definition:** To actively remove a technical, organizational, or decision obstacle preventing a teammate or downstream team from progressing.
- **Example:** *"My top priority as tech lead this morning is reviewing those three PRs to unblock the frontend launch."*
- **Collocations:** *unblock a teammate · unblock the release · get unblocked*
- **Nuance & Exam Tip:** One of the highest-signal words for servant leadership; asking 'What do you need from me to get unblocked?' shows proactive teamwork.
- **Word Family:** blocker · flag a blocker · clear the path · facilitate

---

### Sprint velocity
?
- **Part of Speech:** Agile Metric Noun Phrase [Neutral]
- **Español:** velocidad de sprint
- **Definition:** The average number of story points or tasks a Scrum team completes per sprint, used purely for internal capacity forecasting.
- **Example:** *"Our sprint velocity dipped in December due to holiday PTO and unplanned incident support."*
- **Collocations:** *measure sprint velocity · predictable velocity · game velocity metrics*
- **Nuance & Exam Tip:** Warning: Goodhart's Law applies! Never compare velocity across different teams or use it as an individual productivity KPI—teams will simply inflate points.
- **Word Family:** story points · capacity planning · burndown · throughput

---

### Critical path
?
- **Part of Speech:** Project Planning Noun Phrase [Formal]
- **Español:** ruta crítica / camino crítico
- **Definition:** The longest sequence of dependent tasks in a project schedule where any delay to a single step directly pushes back the final delivery date.
- **Example:** *"Icon polish isn't on the critical path, whereas provisioning SOC-2 audit logs directly gates our launch date."*
- **Collocations:** *on the critical path · off the critical path · shorten the critical path*
- **Nuance & Exam Tip:** Saying 'that's not on the critical path' is a crisp, objective way to reject distractions without dismissing a colleague's idea.
- **Word Family:** dependency · bottleneck · milestone · slack time

---

### Go-to-market (GTM)
?
- **Part of Speech:** Business Strategy Phrase [Formal]
- **Español:** estrategia de salida al mercado / lanzamiento comercial
- **Definition:** A coordinated cross-functional plan uniting Product, Sales, Marketing, and Support to price, position, sell, and deliver a new offering to customers.
- **Example:** *"Engineering shouldn't lock API pricing tiers without syncing with the go-to-market team on enterprise packaging."*
- **Collocations:** *GTM strategy · GTM readiness · align with GTM · GTM motion*
- **Nuance & Exam Tip:** Often abbreviated GTM ('our GTM motion'); engineers who understand GTM constraints earn massive credibility with business executives.
- **Word Family:** launch readiness · positioning · product-market fit · value proposition

---

### Product-market fit (PMF)
?
- **Part of Speech:** Startup / Product Noun Phrase [Neutral]
- **Español:** encaje producto-mercado
- **Definition:** The point at which a product satisfies a genuine market demand strongly enough that users adopt, retain, and organically recommend it.
- **Example:** *"Before investing in multi-region active-active clusters, we need to validate basic product-market fit."*
- **Collocations:** *find product-market fit · iterate toward PMF · pre-PMF vs post-PMF*
- **Nuance & Exam Tip:** Coined by Marc Andreessen/Andy Rachleff; pre-PMF startups optimize for speed of learning, while post-PMF orgs optimize for scale and reliability.
- **Word Family:** value proposition · pivot · retention · validation

---

### Value proposition
?
- **Part of Speech:** Product / Strategy Noun Phrase [Formal]
- **Español:** propuesta de valor
- **Definition:** A clear statement explaining the concrete benefit a product or feature delivers to users, why it beats alternatives, and why they should adopt it.
- **Example:** *"Cutting onboarding setup from two days to ten minutes is the core value proposition of our hosted SDK."*
- **Collocations:** *core value proposition · sharpen the value prop · customer value proposition*
- **Nuance & Exam Tip:** Shortened informally to 'value prop'; avoid listing technical features alone—translate features into user time saved, money earned, or risk reduced.
- **Word Family:** product-market fit · differentiator · ROI · positioning

---

### Opportunity cost
?
- **Part of Speech:** Economics / Strategy Term [Formal]
- **Español:** coste de oportunidad
- **Definition:** The hidden value of the next-best alternative you forfeit whenever you commit scarce time, money, or headcount to a chosen option.
- **Example:** *"Even if building an in-house feature-flag engine looks cheap, the opportunity cost is delaying core checkout improvements by a month."*
- **Collocations:** *factor in opportunity cost · high opportunity cost · hidden opportunity cost*
- **Nuance & Exam Tip:** One of the strongest executive arguments against 'free' internal builds—engineer hours spent on undifferentiated tooling cannot build customer value.
- **Word Family:** trade-off · build vs buy · prioritize ruthlessly · sunk cost fallacy

---

### Sunk cost fallacy
?
- **Part of Speech:** Cognitive Bias Noun Phrase [Formal]
- **Español:** falacia del coste hundido
- **Definition:** The irrational tendency to continue pouring resources into a failing project merely because significant time or money was already spent on it.
- **Example:** *"Let's resist the sunk cost fallacy: six months spent on that brittle prototype shouldn't trap us into maintaining it for years."*
- **Collocations:** *fall prey to the sunk cost fallacy · avoid sunk-cost thinking · cut our losses*
- **Nuance & Exam Tip:** Diplomatic phrasing tip: frame decisions around 'forward-looking value' rather than calling prior colleagues' work 'a failure'.
- **Word Family:** cut losses · pivot · opportunity cost · sunset

---

### Diminishing returns
?
- **Part of Speech:** Economics / Engineering Idiom [Formal]
- **Español:** rendimientos decrecientes
- **Definition:** The point beyond which investing additional effort, tuning, or capital yields progressively smaller incremental improvements.
- **Example:** *"Pushing unit-test line coverage from 85 percent to 100 percent quickly hits diminishing returns compared to adding integration tests."*
- **Collocations:** *hit diminishing returns · point of diminishing returns · law of diminishing returns*
- **Nuance & Exam Tip:** Notice 'returns' is always plural; classic framing for knowing when 'good enough' beats perfectionism.
- **Word Family:** 80/20 rule (Pareto) · gold-plating · cost-benefit ratio · good enough

---

### Move the needle
?
- **Part of Speech:** Business Impact Idiom [Informal / Neutral]
- **Español:** marcar una diferencia real / tener un impacto perceptible
- **Definition:** To produce a measurable, meaningful shift in a key business, user, or reliability metric rather than merely generating busywork.
- **Example:** *"Tweaking button borders won't move the needle on conversion, whereas fixing our three-second checkout latency will."*
- **Collocations:** *genuinely move the needle · fail to move the needle · needle-mover*
- **Nuance & Exam Tip:** Like a dashboard gauge needle moving visibly; great filter question in sprint planning: 'Will this actually move the needle for users?'
- **Word Family:** high-impact · north star metric · outcomes over outputs · leverage

---

### Quick win
?
- **Part of Speech:** Strategy Noun Phrase [Neutral]
- **Español:** logro rápido / victoria rápida
- **Definition:** An improvement deliverable rapidly with modest effort that demonstrates tangible progress and builds stakeholder trust early.
- **Example:** *"Fixing the top five customer-reported export bugs gives us an immediate quick win while the v2 rewrite runs in parallel."*
- **Collocations:** *deliver a quick win · score an early quick win · quick-win backlog*
- **Nuance & Exam Tip:** Valuable in the first 30–60 days of a new role or turnaround project; just don't mistake a string of quick wins for a long-term strategy.
- **Word Family:** low-hanging fruit · momentum · phased rollout · early traction

---

### Silver bullet
?
- **Part of Speech:** Realism Idiom [Neutral]
- **Español:** solución mágica / panacea
- **Definition:** A single, effortless, universal fix wrongly assumed to solve a complex structural or organizational problem overnight.
- **Example:** *"Microservices are no silver bullet—without clear domain boundaries, you just trade a monolith for network latency."*
- **Collocations:** *no silver bullet · look for a silver bullet · magic bullet*
- **Nuance & Exam Tip:** Almost always used in the negative ('There is no silver bullet for X'); references Fred Brooks' landmark software engineering essay.
- **Word Family:** panacea · cure-all · golden hammer · trade-off

---

### Dogfooding
?
- **Part of Speech:** Tech Culture Gerund / Verb [Informal]
- **Español:** probar internamente el propio producto (consumir lo que se fabrica)
- **Definition:** Using your own company's software internally in daily production workflows before external launch to catch friction and quality issues early.
- **Example:** *"Three weeks of internal dogfooding uncovered annoying permission quirks no automated test suite noticed."*
- **Collocations:** *dogfood our own tools · internal dogfooding phase · eat our own dog food*
- **Nuance & Exam Tip:** Short for 'eating your own dog food'; some teams prefer the crisper alternative 'drinking our own champagne', though 'dogfooding' dominates tech.
- **Word Family:** internal beta · alpha test · user empathy · early feedback

---

### Pivot
?
- **Part of Speech:** Strategy Verb / Noun [Neutral]
- **Español:** pivotar / dar un giro estratégico
- **Definition:** To deliberately change product direction, target market, or technical approach based on hard evidence while preserving validated core learnings.
- **Example:** *"When SMB self-serve churned heavily, the company pivoted toward mid-market compliance workflows."*
- **Collocations:** *pivot away from · strategic pivot · pivot toward enterprise*
- **Nuance & Exam Tip:** A true pivot changes strategy intentionally based on evidence—randomly thrashing priorities every week is 'whiplash', not pivoting.
- **Word Family:** course-correct · product-market fit · adapt · strategic shift

---

### Sunset
?
- **Part of Speech:** Product Lifecycle Verb [Formal]
- **Español:** retirar gradualmente / dar de baja definitiva (un producto o API)
- **Definition:** To formally retire, phase out, and eventually shut down an aging product, legacy service, or API version along a communicated timeline.
- **Example:** *"After migrating the final twelve enterprise accounts, we can finally sunset the SOAP API in October."*
- **Collocations:** *sunset a feature · sunsetting plan · sunset timeline · end-of-life (EOL)*
- **Nuance & Exam Tip:** Notice the distinction: you 'deprecate' an interface early to warn against new use, and 'sunset' it when you permanently switch it off.
- **Word Family:** deprecate · end-of-life (EOL) · wind down · decommission

---

### Scope down
?
- **Part of Speech:** Delivery Phrasal Verb [Neutral]
- **Español:** recortar el alcance / acotar a lo esencial
- **Definition:** To intentionally trim non-critical features or edge cases from a project so the core value ships reliably on schedule.
- **Example:** *"To hit the conference demo without burnout, let's scope down batch exports to v1.1 and focus on single-record downloads."*
- **Collocations:** *scope down the MVP · aggressively scope down · trim scope*
- **Nuance & Exam Tip:** Much healthier than 'cut corners': scoping down trims visible features openly while preserving code quality and testing standards.
- **Word Family:** descope (v.) · trim scope · prioritize ruthlessly · MVP

---

### Cut corners
?
- **Part of Speech:** Quality Idiom [Critical / Informal]
- **Español:** tomar atajos chapuceros / escatimar en calidad o seguridad
- **Definition:** To skip essential quality, testing, or security safeguards simply to save time or money, risking defects or outages later.
- **Example:** *"Cutting corners on input validation just to beat a demo date invariably triggers costly security hotfixes later."*
- **Collocations:** *refuse to cut corners · cut corners on testing · pay the price for cutting corners*
- **Nuance & Exam Tip:** Always carries negative judgment—contrasts sharply with smart 'scoping down' (reducing feature breadth while keeping engineering rigor high).
- **Word Family:** take shortcuts · technical debt · scope down · compromise quality

---

### Gold-plating
?
- **Part of Speech:** Engineering Anti-Pattern [Informal / Critical]
- **Español:** perfeccionismo excesivo / añadir adornos innecesarios fuera de requisito
- **Definition:** Spending extra engineering time adding unrequested polish, speculative abstractions, or niche customization after core requirements are already met.
- **Example:** *"Building a custom theme engine for an internal admin form is pure gold-plating—ship the standard component."*
- **Collocations:** *stop gold-plating · guilty of gold-plating · over-polish*
- **Nuance & Exam Tip:** Great tactful check-in on yourself or peers during late sprint days: 'Are we shipping core value or gold-plating right now?'
- **Word Family:** over-engineering · YAGNI · diminishing returns · scope creep

---

### Slip a deadline
?
- **Part of Speech:** Delivery Collocation [Neutral]
- **Español:** retrasar una fecha límite / incumplir un plazo
- **Definition:** To miss or push back a scheduled target delivery date; also used intransitively ('the launch slipped by two weeks').
- **Example:** *"If QA uncovers migration data mismatches tomorrow, the release date may slip into next sprint."*
- **Collocations:** *schedule slippage · let a deadline slip · slip by two weeks · slip the date*
- **Nuance & Exam Tip:** Very idiomatic intransitive use ('the timeline slipped'); communicate potential slippage early with updated forecasts rather than on deadline day.
- **Word Family:** schedule slippage · miss a milestone · push back · reset expectations

---

### Crunch time
?
- **Part of Speech:** Workplace Noun Phrase [Informal]
- **Español:** recta final intensa / periodo de máxima presión antes de una entrega
- **Definition:** A high-pressure period immediately preceding a fixed launch or hard deadline when teams work intensely to finish critical tasks.
- **Example:** *"Strong automated testing kept crunch time manageable right before our annual customer summit."*
- **Collocations:** *hit crunch time · crunch mode · avoid chronic crunch*
- **Nuance & Exam Tip:** Originally from gaming/tech; chronic 'crunch culture' signals poor planning or leadership failure rather than heroism in modern engineering.
- **Word Family:** burnout · critical path · sprint to the finish · sustainable pace

---

### Capacity planning
?
- **Part of Speech:** Operations / Management Noun Phrase [Formal]
- **Español:** planificación de capacidad
- **Definition:** Forecasting upcoming engineering bandwidth or infrastructure compute needs against projected roadmap work, PTO, on-call load, and traffic growth.
- **Example:** *"Realistic capacity planning allocates 20 percent of sprint bandwidth to maintenance and bug triage."*
- **Collocations:** *quarterly capacity planning · headcount capacity · infra capacity planning*
- **Nuance & Exam Tip:** Applies equally to human teams (sprint capacity minus holidays/on-call) and cloud infrastructure (forecasting peak Q4 traffic).
- **Word Family:** resource allocation · bandwidth · sprint velocity · headroom

---

### Resource allocation
?
- **Part of Speech:** Management Noun Phrase [Formal]
- **Español:** asignación de recursos
- **Definition:** Deciding how to distribute budget, engineering headcount, and cloud compute across competing initiatives to maximize strategic goals.
- **Example:** *"Shifting two backend engineers onto Search reflects a deliberate resource allocation toward enterprise retention."*
- **Collocations:** *optimize resource allocation · reallocate resources · staffing mix*
- **Nuance & Exam Tip:** In humane leadership, avoid referring to individual engineers as 'resources' in 1:1s ('people' or 'engineers' builds respect); save 'resource allocation' for budgets and staffing models.
- **Word Family:** capacity planning · staffing · trade-off · prioritize ruthlessly

---

### Non-negotiable
?
- **Part of Speech:** Negotiation Adjective / Noun [Formal]
- **Español:** innegociable / condición irrenunciable
- **Definition:** A requirement, standard, or boundary that cannot be traded away or compromised under any schedule or commercial pressure.
- **Example:** *"Encryption at rest and audit logging are non-negotiables for any healthcare integration we ship."*
- **Collocations:** *hard non-negotiable · treat security as non-negotiable · compromise on*
- **Nuance & Exam Tip:** Can function both as an adjective ('non-negotiable terms') and countable noun ('our three non-negotiables'). Keep the list short so it stays credible.
- **Word Family:** draw a line in the sand · red line · table stakes · hard requirement

---

### Bargaining chip
?
- **Part of Speech:** Negotiation Idiom [Neutral]
- **Español:** baza negociadora / moneda de cambio
- **Definition:** A concession, capability, or advantage held by one party in a negotiation that can be traded for something valuable from the other side.
- **Example:** *"Offering a free 60-day onboarding migration served as a useful bargaining chip to lock in a three-year enterprise contract."*
- **Collocations:** *use as a bargaining chip · lose bargaining power · give-and-take*
- **Nuance & Exam Tip:** Always frame concessions as conditional trades ('If you commit to X, we can include Y') rather than giving away bargaining chips for free.
- **Word Family:** leverage · quid pro quo · concession · walk-away point

---

### Walk-away point
?
- **Part of Speech:** Negotiation Noun Phrase [Formal]
- **Español:** límite de ruptura / punto a partir del cual se abandona la negociación
- **Definition:** The firm threshold beyond which a vendor deal, salary offer, or partnership becomes worse than having no deal at all.
- **Example:** *"Before entering vendor renewal talks, Finance and Engineering agreed on $120k/year as our strict walk-away point."*
- **Collocations:** *establish a walk-away point · BATNA · bottom line · walk away from the table*
- **Nuance & Exam Tip:** Paired closely in formal negotiation theory with BATNA (Best Alternative To a Negotiated Agreement)—knowing your walk-away point prevents bad concessions.
- **Word Family:** BATNA · bottom line · non-negotiable · bargaining chip

---

### Phased rollout
?
- **Part of Speech:** Delivery Strategy Noun Phrase [Formal / Neutral]
- **Español:** despliegue por fases / lanzamiento escalonado
- **Definition:** Introducing a product change, migration, or org policy incrementally across user cohorts or regions rather than flipping everyone at once.
- **Example:** *"A phased rollout—starting with internal teams, then 10 percent beta, then GA—keeps support volume predictable."*
- **Collocations:** *execute a phased rollout · staggered rollout · cohort-based rollout*
- **Nuance & Exam Tip:** Broader than 'canary release': a canary tests server stability for minutes/hours, while a phased rollout spans days/weeks across business cohorts.
- **Word Family:** canary release · progressive delivery · blast radius · feature flag

---

### Acceptance criteria
?
- **Part of Speech:** Product / QA Noun Phrase [Formal]
- **Español:** criterios de aceptación
- **Definition:** Explicit, testable conditions listed on a ticket or story that software must satisfy before product owners consider it complete.
- **Example:** *"Before picking up the export ticket, make sure the acceptance criteria specify max file size and CSV encoding."*
- **Collocations:** *define acceptance criteria · meet all acceptance criteria · Given-When-Then format*
- **Nuance & Exam Tip:** Grammatically plural ('criteria', singular is 'criterion')—avoid the frequent error 'an acceptance criteria' (say 'an acceptance criterion' or 'the acceptance criteria are...').
- **Word Family:** definition of done · edge case · deliverable · spec

---

### Nitpick / nit
?
- **Part of Speech:** PR Review Noun / Verb [Informal]
- **Español:** detalle menor / observación puntual sin peso bloqueante
- **Definition:** A minor, non-blocking observation in a code review or document—such as formatting, phrasing, or a tiny stylistic tweak.
- **Example:** *"Nit: could we rename 'tmp' to 'normalizedEmail' here? Leaving approval either way so you aren't blocked."*
- **Collocations:** *minor nit · prefix with 'nit:' · nitpicky comment · non-blocking nit*
- **Nuance & Exam Tip:** Always mark tiny suggestions with 'nit:' (or 'nit (non-blocking):') so authors know they can merge immediately without endless back-and-forth.
- **Word Family:** non-blocking suggestion · pedantic · bike-shedding · LGTM

---

### Constructive feedback
?
- **Part of Speech:** Feedback Collocation [Formal / Neutral]
- **Español:** crítica constructiva / retroalimentación constructiva
- **Definition:** Specific, actionable, and respectful critique focused on improving code, design, or behavior rather than attacking the author personally.
- **Example:** *"Instead of writing 'this logic makes no sense,' constructive feedback explains the failing race condition and proposes an alternative."*
- **Collocations:** *offer constructive feedback · actionable feedback · open to feedback*
- **Nuance & Exam Tip:** Never say 'a constructive feedback'—'feedback' is strictly uncountable in English ('some feedback' or 'a piece of feedback').
- **Word Family:** actionable critique · radical candor · soften the blow · peer review

---

### Clarity over cleverness
?
- **Part of Speech:** Engineering Maxim [Neutral]
- **Español:** primar la claridad sobre el ingenio críptico
- **Definition:** The principle of writing straightforward, readable code that teammates can debug at 3 a.m. rather than overly terse, show-off one-liners.
- **Example:** *"That nested bitwise reducer is impressive, but let's favor clarity over cleverness and expand it into a readable loop."*
- **Collocations:** *prioritize clarity over cleverness · readable over clever · debuggable at 3 a.m.*
- **Nuance & Exam Tip:** A diplomatic PR framing phrase because it compliments the author's intelligence ('clever') while nudging toward team maintainability.
- **Word Family:** readability · maintainability · cognitive load · idiomatic code

---

### Premature optimization
?
- **Part of Speech:** Engineering Anti-Pattern [Formal]
- **Español:** optimización prematura
- **Definition:** Complicating code for micro-performance gains before profiling demonstrates a real bottleneck in production.
- **Example:** *"Hand-rolling an object pool before benchmarking seems like premature optimization; let's keep the straightforward array map first."*
- **Collocations:** *avoid premature optimization · root of all evil · profile before optimizing*
- **Nuance & Exam Tip:** References Donald Knuth's famous quote ('Premature optimization is the root of all evil'); pair with 'let's benchmark first' to keep feedback objective.
- **Word Family:** premature abstraction · over-engineering · YAGNI · bottleneck

---

### Defensive programming
?
- **Part of Speech:** Coding Discipline Noun Phrase [Engineering]
- **Español:** programación defensiva
- **Definition:** Writing code that anticipates invalid inputs, missing environment variables, null payloads, or partial failures and validates boundaries explicitly.
- **Example:** *"Good defensive programming here: validating the webhook payload schema upfront prevents undefined crashes three layers deep."*
- **Collocations:** *practice defensive programming · defensive check · guard clause · input validation*
- **Nuance & Exam Tip:** Avoid overdoing defensive checks internally inside trusted boundaries where failing loudly helps catch bugs faster than silent null fallbacks.
- **Word Family:** guard clause · fail-fast · edge case · silent failure

---

### Code smell
?
- **Part of Speech:** Code Quality Idiom [Informal / Technical]
- **Español:** mal olor de código / indicio de diseño problemático
- **Definition:** A surface pattern in source code—like giant functions, duplicated switches, or deep nesting—that is not necessarily a bug yet but signals deeper design flaws.
- **Example:** *"Passing seven boolean flags into one billing helper is a classic code smell pointing toward missing polymorphism."*
- **Collocations:** *classic code smell · flag a code smell · refactor away a code smell*
- **Nuance & Exam Tip:** Coined by Kent Beck and Martin Fowler; phrase it gently in PRs ('feels like a small smell around coupling') rather than attacking the author.
- **Word Family:** anti-pattern · technical debt · refactor · maintainability

---

### Boilerplate
?
- **Part of Speech:** Software Noun [Neutral]
- **Español:** código repetitivo estándar / plantilla base
- **Definition:** Verbose, repetitive sections of setup or configuration code required by a language or framework across many files with minimal variation.
- **Example:** *"Migrating from Redux actions to React Query cut nearly 60 percent of our API-fetching boilerplate."*
- **Collocations:** *reduce boilerplate · boilerplate code · eliminate boilerplate · scaffolding*
- **Nuance & Exam Tip:** Always uncountable ('too much boilerplate', not 'many boilerplates'); originates from standardized newspaper metal printing plates.
- **Word Family:** scaffolding · DRY · abstraction · syntactic sugar

---

### Refactor
?
- **Part of Speech:** Engineering Verb / Noun [Neutral]
- **Español:** refactorizar (reestructurar código sin cambiar su comportamiento externo)
- **Definition:** Restructuring existing internal code to improve clarity, modularity, or extensibility while keeping observable external behavior unchanged.
- **Example:** *"Let's extract the retry loop in a dedicated refactor PR rather than mixing structural cleanups with this hotfix."*
- **Collocations:** *refactor into helpers · clean refactor · separate refactor PR · behavioral parity*
- **Nuance & Exam Tip:** Strict engineering rule: 'refactoring' changes structure WITHOUT changing behavior; if behavior changes, call it a 'rewrite' or 'feature change'.
- **Word Family:** refactoring (n.) · clean up · extract · Boy Scout Rule

---

### Regression
?
- **Part of Speech:** QA / Engineering Noun [Technical]
- **Español:** regresión (reaparición de un fallo o rotura de una función que antes funcionaba)
- **Definition:** A defect introduced by a recent code change that breaks previously working functionality or degrades latency and accuracy.
- **Example:** *"Adding dark-mode CSS inadvertently caused a layout regression on Safari mobile checkout buttons."*
- **Collocations:** *introduce a regression · catch a regression · regression suite · performance regression*
- **Nuance & Exam Tip:** Don't confuse with statistical 'linear regression' in ML contexts; in QA/PRs, 'regression' always means backwards breakage of working behavior.
- **Word Family:** regression test · breaking change · canary release · rollback plan

---

### Flaky test
?
- **Part of Speech:** Testing Noun Phrase [Informal / Technical]
- **Español:** prueba inestable / test intermitente (que pasa y falla sin cambios en el código)
- **Definition:** An automated test that passes and fails unpredictably across identical runs due to timing races, shared state, or external network dependencies.
- **Example:** *"Quarantining that flaky Playwright test stopped developers from blindly clicking 're-run CI' until real bugs slipped through."*
- **Collocations:** *quarantine a flaky test · flake rate · deflake the suite · intermittent failure*
- **Nuance & Exam Tip:** Teams use 'flake' as a verb too ('that timezone spec flaked again at midnight'); ignoring flaky tests erodes all trust in CI.
- **Word Family:** non-deterministic · race condition · CI hygiene · test harness

---

### Edge-case coverage
?
- **Part of Speech:** Testing Noun Phrase [Engineering]
- **Español:** cobertura de casos límite
- **Definition:** The degree to which automated tests explicitly exercise boundary inputs, empty sets, concurrency limits, and failure states rather than just normal paths.
- **Example:** *"Line coverage reads 92 percent, yet edge-case coverage around empty arrays and expired tokens is still thin."*
- **Collocations:** *improve edge-case coverage · thin edge-case coverage · boundary tests*
- **Nuance & Exam Tip:** Asking 'Could we add edge-case coverage for null and empty inputs?' is constructive and specific—much better than vague 'add more tests'.
- **Word Family:** edge case · happy path · unit test · defensive programming

---

### Naming convention
?
- **Part of Speech:** Style Standard Noun Phrase [Neutral]
- **Español:** convención de nomenclatura
- **Definition:** An agreed team rule governing casing, prefixes, suffixes, and semantic clarity for variables, files, endpoints, metrics, and branches.
- **Example:** *"Following our repo's naming convention, boolean flags start with 'is' or 'has' and event names use past tense."*
- **Collocations:** *adhere to naming conventions · violate naming conventions · consistent naming*
- **Nuance & Exam Tip:** Automate naming conventions with linters (ESLint, Ruff) whenever possible so human reviewers save energy for architecture and correctness.
- **Word Family:** linting · style guide · idiomatic code · bike-shedding

---

### Readability
?
- **Part of Speech:** Code Quality Noun [Neutral / Formal]
- **Español:** legibilidad
- **Definition:** How easily another engineer—or your future self six months later—can scan, parse, and mentally trace a piece of code without guesswork.
- **Example:** *"Replacing deeply nested ternaries with early returns dramatically improves readability in the permission middleware."*
- **Collocations:** *improve readability · sacrifice readability · optimize for readability*
- **Nuance & Exam Tip:** Google famously certifies senior engineers in 'Readability' for specific languages—emphasizing that code is read ten times more often than written.
- **Word Family:** clarity over cleverness · cognitive load · maintainability · self-documenting code

---

### Maintainability
?
- **Part of Speech:** Software Quality Noun [Formal]
- **Español:** mantenibilidad / facilidad de mantenimiento
- **Definition:** The ease with which a codebase can be safely understood, modified, extended, debugged, and onboarded onto over months and years.
- **Example:** *"Duplicating a five-line helper across two isolated modules actually boosts long-term maintainability over a tangled shared dependency."*
- **Collocations:** *long-term maintainability · hurt maintainability · maintainable architecture*
- **Nuance & Exam Tip:** Strong C1 noun for design-doc justifications: executives care about maintainability when framed as sustained delivery speed and lower outage risk.
- **Word Family:** readability · technical debt · modularity · separation of concerns

---

### Blocking comment
?
- **Part of Speech:** PR Etiquette Noun Phrase [Engineering]
- **Español:** comentario bloqueante (que requiere cambio antes de aprobar)
- **Definition:** A code-review remark pointing out a correctness bug, security flaw, or architectural violation that must be resolved before the PR can merge.
- **Example:** *"I'm leaving one blocking comment on the SQL string interpolation; everything else looks solid."*
- **Collocations:** *mark as blocking · non-blocking vs blocking · resolve blocking comments*
- **Nuance & Exam Tip:** Be sparing and explicit with blocking comments—pair them with clear reasoning and proposed alternatives so reviews never feel like power trips.
- **Word Family:** non-blocking suggestion · nitpick / nit · request changes · PR etiquette

---

### Non-blocking suggestion
?
- **Part of Speech:** PR Etiquette Noun Phrase [Diplomatic]
- **Español:** sugerencia no bloqueante
- **Definition:** An optional improvement idea offered in a code review accompanied by explicit approval so the author may adopt it now, ticket it later, or skip it.
- **Example:** *"Approved! Left two non-blocking suggestions on helper extraction—feel free to merge as-is or address if convenient."*
- **Collocations:** *leave a non-blocking suggestion · approve with comments · optional polish*
- **Nuance & Exam Tip:** Game-changer for team velocity and morale: adding '(non-blocking)' removes ambiguity across time zones and avoids unnecessary 24-hour review loops.
- **Word Family:** nitpick / nit · blocking comment · LGTM · approve with nits

---

### Looks good to me (LGTM)
?
- **Part of Speech:** PR Approval Acronym [Informal]
- **Español:** me parece bien / visto bueno (LGTM)
- **Definition:** Standard engineering shorthand signaling that you reviewed the pull request, doc, or plan and approve moving forward.
- **Example:** *"Nice cleanup on the query builder and solid edge-case tests—LGTM!"*
- **Collocations:** *LGTM with one tiny nit · rubber-stamp LGTM · give an LGTM*
- **Nuance & Exam Tip:** Avoid dropping a bare 'LGTM' on a 900-line risky change unless you genuinely verified it—otherwise teammates suspect a careless rubber-stamp.
- **Word Family:** rubber-stamp · approve with nits · sign off on · ship it

---

### Caveat
?
- **Part of Speech:** Formal Qualification Noun [C1 / Formal]
- **Español:** salvedad / matiz / advertencia condicionante
- **Definition:** An explicit warning, limitation, or condition attached to an approval, benchmark, or agreement to prevent misinterpretation.
- **Example:** *"LGTM, with one caveat: we should watch Redis memory metrics closely during the first hour after rollout."*
- **Collocations:** *add a caveat · major caveat · with the caveat that · caveat emptor*
- **Nuance & Exam Tip:** Pronounced 'KAV-ee-at' (or 'KAH-vee-at'); super versatile C1 word in design docs, metrics reports, and PR approvals ('with one minor caveat').
- **Word Family:** qualification · disclaimer · reservation · stipulation

---

### Rubber-stamp
?
- **Part of Speech:** Critique Verb / Noun [Informal / Critical]
- **Español:** aprobar automáticamente sin revisar a fondo
- **Definition:** To approve a pull request, RFC, or expense routinely without reading carefully or applying critical scrutiny.
- **Example:** *"Sending 1,500-line diffs encourages reviewers to rubber-stamp PRs just to clear their inbox."*
- **Collocations:** *rubber-stamp a PR · avoid rubber-stamping · mere rubber stamp*
- **Nuance & Exam Tip:** Always critical in tone; if you need a tiny config fix merged fast without deep architecture review, ask for a 'quick sanity check' rather than asking peers to 'rubber-stamp'.
- **Word Family:** LGTM · self-contained PR · due diligence · sign off on

---

### Self-contained PR
?
- **Part of Speech:** PR Hygiene Noun Phrase [Engineering]
- **Español:** pull request autocontenido / independiente
- **Definition:** A focused pull request that compiles, passes tests, and makes complete logical sense on its own without hidden dependencies on unmerged branches.
- **Example:** *"Splitting the schema addition into a small, self-contained PR let us merge and verify it cleanly before the UI landed."*
- **Collocations:** *keep PRs self-contained · stacked PRs · isolated diff*
- **Nuance & Exam Tip:** Small self-contained PRs cut review latency dramatically and simplify git bisect and rollbacks if production breaks.
- **Word Family:** tightly scoped · atomic commit · stacked PRs · blast radius

---

### Atomic commit
?
- **Part of Speech:** Git Hygiene Noun Phrase [Engineering]
- **Español:** commit atómico (cambio único indivisible y coherente)
- **Definition:** A single version-control commit encapsulating one cohesive logical change—leaving tests green—so it can be reviewed, cherry-picked, or reverted cleanly.
- **Example:** *"Keeping dependency bumps in separate atomic commits saved us when we had to revert the auth refactor cleanly."*
- **Collocations:** *craft atomic commits · revert cleanly · cherry-pick · logical unit*
- **Nuance & Exam Tip:** Pairs neatly with Conventional Commits and interactive rebase; prevents tangled 'WIP fix everything' commits that obscure git history.
- **Word Family:** self-contained PR · revert · cherry-pick · git hygiene

---

### Tightly scoped
?
- **Part of Speech:** PR / Project Adjective Phrase [Neutral]
- **Español:** bien acotado / de alcance muy limitado
- **Definition:** Deliberately restricted in size, surface area, and responsibility so a PR, spike, or ticket stays fast to review and safe to deploy.
- **Example:** *"Thanks for keeping this PR tightly scoped to pagination—it took five minutes to review with full confidence."*
- **Collocations:** *tightly scoped diff · keep the ticket tightly scoped · narrow scope*
- **Nuance & Exam Tip:** Praiseworthy complement to 'self-contained'; if someone mixes unrelated lint formatting into a bug fix, gently request keeping it tightly scoped.
- **Word Family:** self-contained PR · scope down · orthogonal · atomic commit

---

### Cognitive load
?
- **Part of Speech:** Engineering Psychology Noun Phrase [Formal]
- **Español:** carga cognitiva
- **Definition:** The amount of mental effort and working memory required for a developer to understand, modify, or operate a piece of code or architecture.
- **Example:** *"Five layers of indirection around a single database lookup create unnecessary cognitive load for new maintainers."*
- **Collocations:** *reduce cognitive load · high cognitive load · minimize mental overhead*
- **Nuance & Exam Tip:** Popularized by Team Topologies; using 'cognitive load' makes critiques about overly complex code sound empirical rather than subjective.
- **Word Family:** readability · clarity over cleverness · simplicity · mental model

---

### Footgun
?
- **Part of Speech:** Engineering Slang Noun [Informal]
- **Español:** trampa de diseño propensa a errores graves (arma para dispararse en el pie)
- **Definition:** An API, configuration default, or language quirk so deceptive or easy to misuse that developers routinely injure production with it.
- **Example:** *"Making 'deleteAll: true' the default flag on the CLI cleanup script is a serious footgun—let's require '--force'."*
- **Collocations:** *hidden footgun · API footgun · eliminate footguns · pit of success*
- **Nuance & Exam Tip:** Humorous blend of 'shoot yourself in the foot' + 'gun'; opposite design philosophy is the 'pit of success' (making the safe path easiest).
- **Word Family:** pit of success · defensive programming · UX trap · gotcha

---

### Magic number
?
- **Part of Speech:** Code Quality Noun Phrase [Engineering]
- **Español:** número mágico (constante numérica sin explicar incrustada en el código)
- **Definition:** An unexplained raw numeric literal embedded directly in code logic instead of being named as a descriptive constant.
- **Example:** *"Could we extract '86400' into a named constant like 'SECONDS_PER_DAY' instead of leaving a magic number in the cache TTL?"*
- **Collocations:** *replace magic numbers · hard-coded constant · magic string*
- **Nuance & Exam Tip:** Easy, universally accepted code-review feedback; applies equally to 'magic strings' like arbitrary status codes.
- **Word Family:** named constant · readability · naming convention · self-documenting code

---

### Dead code
?
- **Part of Speech:** Code Quality Noun Phrase [Engineering]
- **Español:** código muerto / código inalcanzable o en desuso
- **Definition:** Unreachable branches, unused functions, or obsolete feature paths left in a repository that confuse readers and inflate maintenance burden.
- **Example:** *"Now that the v1 banner experiment ended, let's prune the dead code rather than leaving commented-out handlers behind."*
- **Collocations:** *prune dead code · dead-code elimination · unreachable branch · zombie code*
- **Nuance & Exam Tip:** Remind junior teammates gently that Git preserves history—there is rarely any reason to keep commented-out dead code in main.
- **Word Family:** tech debt · tree-shaking · clean up · stale feature flag

---

### Minimal reproducible example (MRE)
?
- **Part of Speech:** Debugging Noun Phrase [Formal / Technical]
- **Español:** ejemplo mínimo reproducible
- **Definition:** The smallest possible self-contained code snippet, payload, and environment details that reliably reproduce a bug without unrelated noise.
- **Example:** *"Before escalating to the compiler team, please attach a minimal reproducible example so anyone can trigger the crash locally."*
- **Collocations:** *provide an MRE · minimal repro · reproduce reliably · isolate the bug*
- **Nuance & Exam Tip:** Often shortened to 'repro' ('share a minimal repro'); creates huge goodwill when reporting bugs in open-source or cross-team Slack channels.
- **Word Family:** repro · deterministic · isolate · bug report

---

### Orthogonal
?
- **Part of Speech:** Mathematics / Engineering Adjective [C2 / Formal]
- **Español:** ortogonal / independiente entre sí
- **Definition:** Statistically, architecturally, or editorially independent—meaning one change or concern does not intersect or interfere with another.
- **Example:** *"Upgrading our linter rules is orthogonal to fixing the payment timeout, so let's handle it in a separate PR."*
- **Collocations:** *completely orthogonal · orthogonal concerns · orthogonal to this PR*
- **Nuance & Exam Tip:** Signature C2 engineering word! Saying 'That refactoring is orthogonal to this bugfix' politely defers unrelated changes without arguing their merit.
- **Word Family:** independent · decoupled · out of scope · tightly scoped

---

### YAGNI (You Aren't Gonna Need It)
?
- **Part of Speech:** XP Engineering Principle [Informal]
- **Español:** no lo vas a necesitar (principio de no implementar funcionalidad especulativa)
- **Definition:** An Extreme Programming principle warning engineers against building speculative abstractions or features today for hypothetical future requirements.
- **Example:** *"Supporting multi-currency invoicing before we even have non-US customers feels like classic YAGNI—let's keep it USD-only for now."*
- **Collocations:** *apply YAGNI · violate YAGNI · build for today's requirements*
- **Nuance & Exam Tip:** Say 'YAGNI' warmly, not dismissively; explain the concrete cost of carrying unused hooks now versus adding them cleanly when real demand arrives.
- **Word Family:** KISS principle · over-engineering · premature abstraction · MVP

---

### DRY (Don't Repeat Yourself)
?
- **Part of Speech:** Engineering Principle [Neutral]
- **Español:** no te repitas (principio de evitar duplicación de conocimiento)
- **Definition:** A software design principle stating that every piece of business knowledge or logic should have one authoritative representation rather than copy-pasted variants.
- **Example:** *"Three controllers compute tax discounts slightly differently—let's apply DRY and centralize that rule in a domain service."*
- **Collocations:** *keep it DRY · DRY up the code · over-DRYing · wrong abstraction*
- **Nuance & Exam Tip:** Crucial C2 nuance: blindly chasing DRY across superficially similar modules creates 'wrong abstractions'; Sandi Metz wisely noted 'duplication is far cheaper than the wrong abstraction'.
- **Word Family:** WET (Write Everything Twice) · premature abstraction · single source of truth · refactor

---

### KISS principle
?
- **Part of Speech:** Design Principle [Informal / Neutral]
- **Español:** principio KISS (mantenlo simple)
- **Definition:** Short for 'Keep It Simple, Stupid' (or politely 'Keep It Simple and Straightforward'): favoring minimal, transparent designs over needlessly clever complexity.
- **Example:** *"Following the KISS principle, a plain Postgres cron job beats maintaining a whole Airflow cluster for one nightly task."*
- **Collocations:** *apply the KISS principle · keep it simple · avoid unnecessary moving parts*
- **Nuance & Exam Tip:** Because the final 'S' originally stands for 'Stupid', cite 'KISS principle' or 'keep it simple and straightforward' rather than spelling it out toward a colleague.
- **Word Family:** YAGNI · simplicity · over-engineering · clarity over cleverness

---

### Boy Scout Rule
?
- **Part of Speech:** Engineering Culture Maxim [Neutral]
- **Español:** regla del boy scout (dejar el código mejor de como lo encontraste)
- **Definition:** The habit of leaving surrounding code slightly cleaner—better named, better typed, or better tested—than you found it during normal work.
- **Example:** *"In the spirit of the Boy Scout Rule, I added missing TypeScript types to the helper I touched while fixing the bug."*
- **Collocations:** *apply the Boy Scout Rule · leave the codebase cleaner · incremental cleanup*
- **Nuance & Exam Tip:** Balance this carefully in PRs: tiny opportunistic cleanups are praised, whereas massive unrelated rewrites hijack diffs and obscure bug fixes.
- **Word Family:** incremental refactoring · technical debt · tightly scoped · code stewardship

---

### Side effect
?
- **Part of Speech:** Programming Noun Phrase [Technical]
- **Español:** efecto secundario / efecto colateral (mutación de estado o E/S externa)
- **Definition:** Any observable state mutation or external interaction (network call, disk write, global variable update) performed by a function beyond returning its value.
- **Example:** *"Putting an email-sending side effect inside a getter method surprised callers and triggered duplicate notices during tests."*
- **Collocations:** *unintended side effect · free of side effects · isolate side effects*
- **Nuance & Exam Tip:** In code reviews, always check for 'hidden side effects' inside read-only getters, constructors, or React render functions.
- **Word Family:** pure function · idempotency · state mutation · referential transparency

---

### Pure function
?
- **Part of Speech:** Functional Programming Noun Phrase [Technical]
- **Español:** función pura
- **Definition:** A function whose output depends solely on its input arguments and which produces zero side effects—making it deterministic, cacheable, and trivial to unit test.
- **Example:** *"Extracting pricing math into pure functions let us cover twenty discount permutations without mocking a database."*
- **Collocations:** *extract into a pure function · pure core, imperative shell · deterministic output*
- **Nuance & Exam Tip:** Pairs wonderfully with Gary Bernhardt's 'Functional Core, Imperative Shell' pattern—even in object-oriented codebases like Java or Python.
- **Word Family:** side effect · deterministic · referential transparency · immutability

---

### Silent failure
?
- **Part of Speech:** Reliability / Code Review Noun Phrase [Technical]
- **Español:** fallo silencioso
- **Definition:** An error condition caught and swallowed without logging, alerting, or propagating—leaving systems subtly corrupted while appearing green.
- **Example:** *"That empty catch block causes a dangerous silent failure when webhook verification throws a parsing error."*
- **Collocations:** *swallow errors · silent failure · fail silently · surface errors loudly*
- **Nuance & Exam Tip:** Swallowing exceptions silently is one of the strongest reasons to leave a blocking comment in a PR review.
- **Word Family:** fail loudly · fail-fast · error handling · observability

---

### Error handling
?
- **Part of Speech:** Engineering Noun Phrase [Neutral]
- **Español:** gestión de errores / manejo de excepciones
- **Definition:** The deliberate design of how software catches, logs, retries, recovers from, or propagates unexpected failures across boundaries.
- **Example:** *"Before approving the integration, let's verify error handling for 429 rate limits and malformed upstream JSON."*
- **Collocations:** *robust error handling · graceful error handling · error-handling path · swallow exceptions*
- **Nuance & Exam Tip:** Happy paths are easy; senior reviewers inspect the error-handling paths first because that is where production surprises happen.
- **Word Family:** silent failure · happy path · fallback · exception propagation

---

### Explicit over implicit
?
- **Part of Speech:** Design Principle [Formal]
- **Español:** lo explícito frente a lo implícito
- **Definition:** A design guideline favoring clear, visible parameters, defaults, and control flow rather than magical hidden behaviors or implicit global conventions.
- **Example:** *"Passing timezone explicitly instead of relying on server locale follows 'explicit over implicit' and avoids subtle UTC bugs."*
- **Collocations:** *prefer explicit over implicit · Zen of Python · implicit magic · explicit contract*
- **Nuance & Exam Tip:** Famous line from the Zen of Python; directly addresses why 'clever magic' in frameworks often slows down onboarding and debugging.
- **Word Family:** clarity over cleverness · magic behavior · self-documenting code · defensive programming

---

### Self-documenting code
?
- **Part of Speech:** Code Quality Noun Phrase [Neutral]
- **Español:** código autodocumentado
- **Definition:** Code written with expressive function names, clear types, and structured flow so readers understand WHAT it does without redundant comments.
- **Example:** *"Splitting that condition into 'isEligibleForTrialRenewal()' turns dense logic into self-documenting code."*
- **Collocations:** *write self-documenting code · explain 'why' not 'what' · expressive naming*
- **Nuance & Exam Tip:** Golden rule in reviews: self-documenting code explains *what* and *how*, while comments are still essential to explain *why* a non-obvious constraint exists.
- **Word Family:** readability · naming convention · inline comment · rationale

---

### Assume good intent
?
- **Part of Speech:** Team Culture Maxim [Neutral]
- **Español:** asumir buena intención
- **Definition:** The communication habit of interpreting terse or blunt feedback charitably rather than reading hostility into text-only PR reviews or Slack messages.
- **Example:** *"Written comments lack vocal tone, so let's assume good intent in code reviews and hop on a quick call if tension rises."*
- **Collocations:** *assume positive intent · benefit of the doubt · low-context text*
- **Nuance & Exam Tip:** Works in both directions: reviewers must strive to write kind feedback, while authors read terse comments charitably before feeling attacked.
- **Word Family:** psychological safety · blameless postmortem · constructive feedback · benefit of the doubt

---

### Soften the blow
?
- **Part of Speech:** Diplomatic Idiom [Neutral]
- **Español:** suavizar el golpe / amortiguar una mala noticia o crítica
- **Definition:** To phrase tough critique, rejection, or unwelcome scope cuts empathetically so the recipient hears the message without feeling attacked.
- **Example:** *"Highlighting what worked well in the prototype helped soften the blow before asking for a full architecture rework."*
- **Collocations:** *soften the blow · cushion a critique · constructive framing*
- **Nuance & Exam Tip:** Avoid fake 'compliment sandwiches' that confuse people; better to pair clear directness with appreciation of the effort and shared goals.
- **Word Family:** tact · constructive feedback · radical candor · diplomatic pushback

---

### Strong opinions, loosely held
?
- **Part of Speech:** Decision Culture Maxim [Neutral]
- **Español:** opiniones firmes, pero abiertas a cambiar con nueva evidencia
- **Definition:** An engineering mindset combining clear, researched conviction when proposing ideas with total readiness to abandon them immediately when stronger evidence emerges.
- **Example:** *"I advocate 'strong opinions, loosely held' in RFCs—bring a clear recommendation, yet drop it fast if load tests prove otherwise."*
- **Collocations:** *strong opinions, loosely held · weakly held · intellectual humility · update priors*
- **Nuance & Exam Tip:** Coined by Paul Saffo (sometimes cited as 'strong opinions, weakly held'); separates decisive leadership from stubborn ego attachment.
- **Word Family:** intellectual humility · steelman an argument · disagree and commit · ego-free code

---

### Pedantic
?
- **Part of Speech:** Critique Adjective [Formal / Critical]
- **Español:** pedante / excesivamente quisquilloso con detalles formales
- **Definition:** Excessively concerned with trivial rule-following, exact terminology, or minor formatting at the expense of practical communication or progress.
- **Example:** *"Not to sound pedantic, but calling an asynchronous queue an 'RPC' here might confuse readers of the design doc."*
- **Collocations:** *sound pedantic · overly pedantic · minor terminology point*
- **Nuance & Exam Tip:** Softener hack: saying 'Not to be pedantic, but...' acknowledges that your precision check is minor while still clarifying a genuine ambiguity.
- **Word Family:** nitpick / nit · bike-shedding · stickler · semantic

---

### Premature abstraction
?
- **Part of Speech:** Architecture Anti-Pattern [Engineering]
- **Español:** abstracción prematura
- **Definition:** Extracting generic base classes, factories, or shared plugins after seeing only one or two concrete use cases before real variation patterns emerge.
- **Example:** *"Creating a pluggable StorageAdapter interface for a single Postgres table feels like premature abstraction right now."*
- **Collocations:** *avoid premature abstraction · wrong abstraction · Rule of Three*
- **Nuance & Exam Tip:** Mention the 'Rule of Three' (wait until three genuinely similar use cases appear before abstracting)—a respected, concrete counterargument in reviews.
- **Word Family:** premature optimization · over-engineering · YAGNI · Rule of Three

---

### Idiomatic code
?
- **Part of Speech:** Language Style Noun Phrase [Engineering]
- **Español:** código idiomático (fiel al estilo propio del lenguaje)
- **Definition:** Code written naturally using a programming language's standard conventions, idioms, and built-in patterns rather than translating habits from another language.
- **Example:** *"Using list comprehensions and context managers makes the script feel like clean, idiomatic Python instead of ported Java."*
- **Collocations:** *idiomatic Python · Pythonic · idiomatic Go · language idioms*
- **Nuance & Exam Tip:** Different from 'idiomatic English'—in software, 'idiomatic' is high praise meaning native fluency in a specific stack (e.g., 'Pythonic', 'Rustacean').
- **Word Family:** naming convention · readability · best practice · convention over configuration

---

### Mock vs. stub
?
- **Part of Speech:** Testing Contrast Phrase [Engineering]
- **Español:** mock (objeto simulado que verifica interacciones) frente a stub (respuesta fija)
- **Definition:** A stub supplies canned return values so tests run without real dependencies, while a mock additionally records and asserts exact calls or arguments invoked.
- **Example:** *"Let's use a simple stub for the currency feed instead of brittle mocks that assert exact internal method call counts."*
- **Collocations:** *over-mocking · test double · stub an external API · mock verification*
- **Nuance & Exam Tip:** Over-mocking couples tests tightly to internal implementation details, causing false failures during safe refactors; mention 'test doubles' as the umbrella term.
- **Word Family:** test double · contract testing · flaky test · test harness

---

### Articulate a rationale
?
- **Part of Speech:** Executive Communication Collocation [Formal]
- **Español:** exponer con claridad la fundamentación / argumentar el porqué de una decisión
- **Definition:** To clearly and logically explain the underlying reasoning, constraints, and evidence behind an architectural or business decision.
- **Example:** *"In staff-level interviews, candidates must articulate a clear rationale for why they chose eventual consistency over strong consistency."*
- **Collocations:** *articulate the rationale behind · compelling rationale · data-backed rationale*
- **Nuance & Exam Tip:** Strong C1 verb ('articulate' beats plain 'explain' when referring to persuasive structured thinking); avoid confusing 'rationale' (noun) with 'rational' (adj.).
- **Word Family:** rationale (n.) · justify · narrative · trade-off analysis

---

### Behavioral STAR framework
?
- **Part of Speech:** Interview Methodology Phrase [Formal]
- **Español:** metodología conductual STAR (Situación, Tarea, Acción, Resultado)
- **Definition:** A structured storytelling method—Situation, Task, Action, Result—used in behavioral interviews to give crisp, evidence-backed examples of past impact.
- **Example:** *"Spend less than 20 percent of your STAR story on the Situation so you leave ample time for your specific Action and measurable Result."*
- **Collocations:** *structure using STAR · STAR response · behavioral round · quantify the Result*
- **Nuance & Exam Tip:** Biggest non-native interview trap: saying 'we' throughout the Action section. Interviewers need to know what *you* personally owned ('I initiated...', 'I negotiated...').
- **Word Family:** quantify impact · behavioral interview · storytelling · ownership

---

### Mentor vs. sponsor
?
- **Part of Speech:** Career Development Contrast [Formal]
- **Español:** mentor (quien aconseja) frente a patrocinador (quien aboga por tu ascenso)
- **Definition:** A mentor advises and coaches you privately, whereas a sponsor actively uses their political capital behind closed doors to advocate for your promotions and high-visibility projects.
- **Example:** *"My mentor helped me sharpen my system-design talks, while my sponsor fought for my staff promotion during calibration."*
- **Collocations:** *executive sponsor · career sponsorship · seek mentorship · advocate in the room*
- **Nuance & Exam Tip:** Carla Harris's vital distinction: you earn mentors through chemistry and curiosity, while you earn sponsors through consistent high-trust execution.
- **Word Family:** executive sponsorship · mentorship · calibration · allyship

---

### Cross-functional collaboration
?
- **Part of Speech:** Workplace Noun Phrase [Formal]
- **Español:** colaboración transversal / trabajo conjunto entre distintas áreas
- **Definition:** Coordinated execution across distinct disciplines—such as Engineering, Product, Design, Security, Sales, and Legal—toward a shared outcome.
- **Example:** *"Shipping self-serve GDPR deletion required tight cross-functional collaboration across Backend, Legal, and Customer Support."*
- **Collocations:** *foster cross-functional collaboration · cross-functional alignment · XFN partner*
- **Nuance & Exam Tip:** Frequently abbreviated 'XFN' in written Slack messages and Google/Meta docs ('syncing with XFN partners on launch').
- **Word Family:** XFN · stakeholder · siloed · cross-team alignment

---

### Ambiguity tolerance
?
- **Part of Speech:** Leadership Competency Noun Phrase [Formal]
- **Español:** tolerancia a la ambigüedad
- **Definition:** The composure and judgment to keep making progress, framing hypotheses, and structuring clarity when goals, specs, or markets remain undefined.
- **Example:** *"Seniority isn't executing crisp Jira tickets faster—it's showing high ambiguity tolerance when nobody knows the right problem yet."*
- **Collocations:** *thrive in ambiguity · navigate ambiguity · ambiguity tolerance · bring clarity out of chaos*
- **Nuance & Exam Tip:** One of the primary differentiators between mid-level engineers (who execute defined specs) and staff+ engineers (who define the problem space).
- **Word Family:** navigate uncertainty · autonomous execution · hypothesis-driven · clarity

---

### Bias for action
?
- **Part of Speech:** Leadership Principle Phrase [Neutral / Formal]
- **Español:** orientación a la acción / iniciativa resolutiva
- **Definition:** An instinct to take decisive, reversible steps and test ideas in reality rather than stalling in endless analysis or waiting for permission.
- **Example:** *"Showing real bias for action, she mocked the failing endpoint overnight so QA could resume testing by 9 a.m."*
- **Collocations:** *demonstrate bias for action · calculated risk · two-way door decision*
- **Nuance & Exam Tip:** Famous Amazon leadership principle; pair it with Bezos's 'two-way door' vs 'one-way door' framing so bias for action doesn't turn into reckless haste.
- **Word Family:** two-way door · ship it · initiative · analysis paralysis

---

### Post-hoc rationalization
?
- **Part of Speech:** Critical Thinking Noun Phrase [C2 / Formal]
- **Español:** racionalización a posteriori / justificación retrospectiva
- **Definition:** Inventing logical-sounding reasons after an emotional, accidental, or politically driven decision has already been made to make it look strategic.
- **Example:** *"Claiming we skipped load testing to stay agile sounds like post-hoc rationalization for missed planning."*
- **Collocations:** *guard against post-hoc rationalization · retroactive justification · cognitive bias*
- **Nuance & Exam Tip:** High-register C2 phrase ideal for retrospectives and strategy reviews; use gently on collective reasoning rather than accusing one teammate directly.
- **Word Family:** hindsight bias · confirmation bias · motivated reasoning · intellectual honesty

---

### Strategic vision
?
- **Part of Speech:** Executive Noun Phrase [Formal]
- **Español:** visión estratégica
- **Definition:** A coherent, multi-quarter picture of where a product, platform, or organization is heading and why those bets position it for long-term advantage.
- **Example:** *"The CTO's strategic vision connected our tedious database cleanup directly to unlocking enterprise multi-region deals next year."*
- **Collocations:** *articulate a strategic vision · align with the vision · long-term vision*
- **Nuance & Exam Tip:** Without concrete milestones and metrics, 'vision' rings hollow ('all vision, no execution'); great leaders tie vision down to next quarter's steps.
- **Word Family:** north star · roadmap · mission · executive framing

---

### Executive summary
?
- **Part of Speech:** Document Structure Noun Phrase [Formal]
- **Español:** resumen ejecutivo
- **Definition:** A brief, front-loaded opening section of a lengthy RFC, proposal, or report summarizing the core decision, business impact, cost, and recommendation in under one minute.
- **Example:** *"Start your 12-page architecture RFC with a five-line executive summary so VPs can grasp the recommendation immediately."*
- **Collocations:** *lead with an executive summary · concise executive summary · TL;DR*
- **Nuance & Exam Tip:** Think of it as the formal cousin of 'TL;DR': executives read top-down and drill into technical appendices only if the summary triggers questions.
- **Word Family:** TL;DR · bottom line · key takeaway · BLUF (Bottom Line Up Front)

---

### Key takeaway
?
- **Part of Speech:** Presentation Noun Phrase [Neutral / Formal]
- **Español:** conclusión clave / punto principal para recordar
- **Definition:** The essential insight, lesson, or decision an audience should retain after reading a report, attending a meeting, or reviewing an incident.
- **Example:** *"The key takeaway from this load test is that Postgres connections—not CPU—cap our checkout throughput."*
- **Collocations:** *three key takeaways · main takeaway · actionable takeaway*
- **Nuance & Exam Tip:** Always written as one word ('takeaway') as a noun; note that in UK casual English, 'a takeaway' also means takeout restaurant food, though context makes business meaning clear.
- **Word Family:** bottom line · executive summary · insight · synthesis

---

### Bottom line
?
- **Part of Speech:** Executive Idiom [Neutral]
- **Español:** en resumidas cuentas / el punto esencial / resultado neto
- **Definition:** The ultimate conclusion, net financial impact, or non-negotiable truth once all supporting details and caveats are stripped away.
- **Example:** *"Bottom line: unless we upgrade the message broker before November, holiday traffic will saturate our queues."*
- **Collocations:** *the bottom line is · impact the bottom line · BLUF (Bottom Line Up Front)*
- **Nuance & Exam Tip:** Originates from the net profit line at the bottom of an income statement; saying 'Bottom line:' signals crisp, decisive executive synthesis.
- **Word Family:** key takeaway · executive summary · cut to the chase · net impact

---

### Actionable next step
?
- **Part of Speech:** Execution Noun Phrase [Neutral / Formal]
- **Español:** siguiente paso concreto y accionable
- **Definition:** A concrete, specific, owner-assigned task that a team can immediately act upon rather than an abstract observation or vague wish.
- **Example:** *"Let's close the retrospective by turning these frustrations into three actionable next steps with owners and due dates."*
- **Collocations:** *define actionable next steps · clear action items · concrete next step*
- **Nuance & Exam Tip:** 'Actionable' is a prized C1 adjective: 'improve monitoring' is vague; 'add a p99 alert for /checkout by Friday' is actionable.
- **Word Family:** action item · recap action items · DRI · follow-through

---

### Proactive transparency
?
- **Part of Speech:** Leadership Value Phrase [Formal]
- **Español:** transparencia proactiva (comunicar avances y riesgos antes de que pregunten)
- **Definition:** Voluntarily sharing delays, risks, trade-offs, and context early—before stakeholders discover problems or have to chase status updates.
- **Example:** *"Practicing proactive transparency when the vendor API slipped earned trust from leadership instead of defensive scrutiny."*
- **Collocations:** *practice proactive transparency · no-surprises culture · communicate risks early*
- **Nuance & Exam Tip:** Follows the executive 'no surprises' rule: bad news ages terribly in silence, whereas early transparency paired with a mitigation plan builds immense credibility.
- **Word Family:** no surprises · flag early · earn trust · candid

---

### Autonomous execution
?
- **Part of Speech:** Career / Leadership Phrase [Formal]
- **Español:** ejecución autónoma (sin necesidad de microgestión)
- **Definition:** The ability to drive complex, multi-step initiatives from ambiguous inception through production delivery independently while knowing when to escalate or align.
- **Example:** *"Hiring committees look for evidence of autonomous execution—someone who navigates blockers independently yet checks in at key milestones."*
- **Collocations:** *drive autonomous execution · operate autonomously · self-directed delivery*
- **Nuance & Exam Tip:** Crucial nuance: 'autonomous' does NOT mean working in isolation ('lone wolf')—it means driving progress without hand-holding while proactively aligning stakeholders.
- **Word Family:** ownership · empower autonomy · self-starter · proactive transparency

---

### Force multiplier
?
- **Part of Speech:** Staff Engineering Idiom [Formal / Tech Culture]
- **Español:** multiplicador de fuerza / catalizador del rendimiento de todo el equipo
- **Definition:** A person, tool, or practice that dramatically boosts the effectiveness, speed, and output of everyone around them rather than merely increasing individual output.
- **Example:** *"Building shared CI templates and mentoring three mid-level devs made Maya a true force multiplier across the whole org."*
- **Collocations:** *act as a force multiplier · platform force multiplier · leverage across teams*
- **Nuance & Exam Tip:** Classic distinction in staff+ promotion packets: an individual hero closes tickets alone; a force multiplier raises the baseline output of twenty peers.
- **Word Family:** elevate the team · high-leverage · platform engineering · enablement

---

### North star metric
?
- **Part of Speech:** Product Strategy Noun Phrase [Neutral]
- **Español:** métrica estrella polar (indicador guía principal del éxito)
- **Definition:** The single overarching metric that best captures the core value a product delivers to users and guides cross-functional alignment.
- **Example:** *"Setting 'weekly active workspaces collaborating on docs' as our north star metric stopped teams from chasing vanity sign-up numbers."*
- **Collocations:** *define a north star metric · vanity metric vs north star · align around a north star*
- **Nuance & Exam Tip:** Contrast directly with 'vanity metrics' (raw downloads or page views that look flattering on slides without reflecting sustained user value).
- **Word Family:** vanity metric · KPI · OKR · move the needle

---

### Sphere of influence
?
- **Part of Speech:** Leadership Concept Noun Phrase [Formal]
- **Español:** esfera de influencia
- **Definition:** The range of teams, technical decisions, and organizational outcomes you can meaningfully shape through credibility and persuasion rather than direct reporting authority.
- **Example:** *"As a principal engineer without direct reports, your sphere of influence grows through clear RFCs and trusted technical judgment."*
- **Collocations:** *expand your sphere of influence · circle of control vs influence · lead without authority*
- **Nuance & Exam Tip:** Derived from Stephen Covey's model; essential mindset for senior IC (Individual Contributor) engineers who lead through influence rather than managerial hierarchy.
- **Word Family:** leading without authority · force multiplier · stakeholder alignment · credibility

---

### Manage up
?
- **Part of Speech:** Career Strategy Phrasal Verb [Informal / Neutral]
- **Español:** gestionar hacia arriba (facilitar la labor y alinear expectativas con los superiores)
- **Definition:** Proactively communicating progress, surfacing blockers, and tailoring concise context to make your manager or executives effective while protecting your team's autonomy.
- **Example:** *"Sending a crisp Friday bullet summary with risks and asks is one of the highest-ROI ways to manage up."*
- **Collocations:** *manage up effectively · tailor updates · keep leadership aligned*
- **Nuance & Exam Tip:** Not sycophancy or flattery! Managing up means adapting to your manager's communication needs so they can unblock resources and shield your team.
- **Word Family:** proactive transparency · executive summary · no surprises · upward communication

---

### Executive presence
?
- **Part of Speech:** Leadership Attribute Noun Phrase [Formal]
- **Español:** presencia ejecutiva / solvencia comunicativa ante dirección
- **Definition:** A combination of composed poise, clarity under pressure, strategic framing, and credible brevity that inspires confidence in senior rooms.
- **Example:** *"Answering tough board questions calmly with 'Here is what we know, what we don't, and our next step' radiates executive presence."*
- **Collocations:** *project executive presence · develop executive presence · poise under pressure*
- **Nuance & Exam Tip:** For non-native English speakers, executive presence comes NOT from fancy vocabulary or fast speech, but from deliberate pacing, concise headlines, and calm answers.
- **Word Family:** bottom line · read the room · concise framing · gravitas

---

### Succession planning
?
- **Part of Speech:** Talent Strategy Noun Phrase [Formal]
- **Español:** planificación de sucesión / relevo de liderazgo
- **Definition:** Proactively identifying and developing internal engineers or managers capable of stepping into critical leadership or architectural roles before transitions occur.
- **Example:** *"True succession planning meant letting our senior engineer shadow architecture reviews six months before I moved to another org."*
- **Collocations:** *invest in succession planning · build a talent bench · single point of failure*
- **Nuance & Exam Tip:** If you cannot take a two-week vacation or get promoted because nobody else knows how your system works, you have failed succession planning.
- **Word Family:** bus factor · institutional knowledge · delegate effectively · elevate the team

---

### Psychological safety
?
- **Part of Speech:** Org Psychology Noun Phrase [Formal]
- **Español:** seguridad psicológica
- **Definition:** A shared team climate where members feel safe taking interpersonal risks—asking questions, admitting mistakes, or dissenting—without fear of humiliation or retaliation.
- **Example:** *"Google's Project Aristotle found psychological safety—not raw individual IQ—was the strongest predictor of high-performing engineering teams."*
- **Collocations:** *foster psychological safety · erode psychological safety · safe to fail*
- **Nuance & Exam Tip:** Often misunderstood as 'being soft' or avoiding conflict; true psychological safety pairs high candor and high standards with zero personal shaming.
- **Word Family:** blameless postmortem · radical candor · assume good intent · speak up

---

### Radical candor
?
- **Part of Speech:** Feedback Framework Noun Phrase [Neutral]
- **Español:** franqueza radical (cuidar personalmente mientras se cuestiona de frente)
- **Definition:** Kim Scott's management framework combining genuine personal care with direct, unambiguous feedback—avoiding both harsh aggression and toxic artificial politeness.
- **Example:** *"Radical candor isn't blunt rudeness; it's caring enough about a colleague's growth to tell them directly when a presentation missed the mark."*
- **Collocations:** *practice radical candor · challenge directly · ruinous empathy*
- **Nuance & Exam Tip:** Beware confusing 'radical candor' with 'obnoxious aggression'; without the 'care personally' half, blunt critique just destroys trust.
- **Word Family:** ruinous empathy · constructive feedback · psychological safety · soften the blow

---

### Imposter syndrome
?
- **Part of Speech:** Psychology Noun Phrase [Neutral]
- **Español:** síndrome del impostor
- **Definition:** The persistent internal self-doubt—common even among accomplished senior engineers—of feeling like a fraud about to be exposed despite objective evidence of competence.
- **Example:** *"Stepping into a principal role across an unfamiliar stack triggered imposter syndrome until I realized everyone asks basic questions early on."*
- **Collocations:** *struggle with imposter syndrome · overcome imposter syndrome · normalize self-doubt*
- **Nuance & Exam Tip:** Also spelled 'impostor syndrome' (both spellings accepted; 'imposter' dominates everyday tech writing). Naming it openly builds mentorship rapport.
- **Word Family:** intellectual humility · growth mindset · psychological safety · self-doubt

---

### Career ladder / leveling
?
- **Part of Speech:** HR / Engineering Governance Phrase [Neutral]
- **Español:** escala profesional / matriz de niveles (junior, senior, staff, principal)
- **Definition:** A structured rubric defining expectations for scope, technical autonomy, leadership, and impact across engineering levels (e.g., L4, Senior, Staff, Principal).
- **Example:** *"Look closely at the engineering career ladder: advancing from Senior to Staff shifts expectations from local coding output to org-wide impact."*
- **Collocations:** *engineering career ladder · leveling rubric · dual track (IC vs EM) · down-level*
- **Nuance & Exam Tip:** Learn 'dual track': senior ICs (Individual Contributors) can advance to Staff/Principal/Distinguished without becoming Engineering Managers (EM).
- **Word Family:** IC vs EM · performance calibration · scope · down-leveling

---

### Skip-level meeting
?
- **Part of Speech:** Management Ritual Noun Phrase [Neutral]
- **Español:** reunión con el superior de tu jefe (skip-level)
- **Definition:** A recurring 1:1 conversation between an individual contributor and their manager's manager to share ground-level pulse, career goals, and organizational feedback.
- **Example:** *"During our quarterly skip-level meeting, the engineering director asked what slows down day-to-day deployments across teams."*
- **Collocations:** *hold a skip-level · prepare for a skip-level · upward feedback*
- **Nuance & Exam Tip:** Never use a skip-level to ambush your direct manager unfairly—raise immediate issues with your manager first, and use skip-levels for systemic context and career growth.
- **Word Family:** one-on-one (1:1) · upward feedback · alignment · manage up

---

### Brag document
?
- **Part of Speech:** Career Strategy Noun Phrase [Informal]
- **Español:** documento de logros / registro personal de hitos
- **Definition:** A living personal log updated regularly throughout the year capturing shipped projects, quantified metrics, design reviews, and mentoring wins for performance reviews.
- **Example:** *"Keeping a biweekly brag document saved me ten hours of digging through old Slack threads before performance calibration."*
- **Collocations:** *maintain a brag document · hype doc · document your impact · self-review*
- **Nuance & Exam Tip:** Popularized by Julia Evans; essential for non-native professionals who were raised to stay overly modest—if you don't document invisible work, managers forget it.
- **Word Family:** quantify impact · performance calibration · self-evaluation · glue work

---

### Glue work
?
- **Part of Speech:** Engineering Career Idiom [Informal]
- **Español:** trabajo invisible de cohesión (coordinación, documentación, mentoría)
- **Definition:** Essential but often unglamorous coordination tasks—onboarding hires, triaging bugs, updating docs, aligning teams—that keep projects healthy yet risk under-recognition in promotions.
- **Example:** *"Be mindful of uncredited glue work: documenting processes helps the team, but make sure you also ship high-visibility technical deliverables."*
- **Collocations:** *uncredited glue work · Tanya Reilly's 'Being Glue' · balance glue work*
- **Nuance & Exam Tip:** Coined by Tanya Reilly; critical career warning—if you do glue work at Senior/Staff level, tie it explicitly to measurable team velocity in your brag doc.
- **Word Family:** brag document · force multiplier · uncredited labor · career ladder

---

### Bus factor
?
- **Part of Speech:** Risk Idiom [Informal / Technical]
- **Español:** factor autobús (riesgo por concentración de conocimiento en una sola persona)
- **Definition:** The minimum number of key team members whose sudden absence (vacation, resignation, illness) would stall critical system maintenance because knowledge isn't shared.
- **Example:** *"Having only one developer understand the custom tax engine gives us a terrifying bus factor of one."*
- **Collocations:** *bus factor of one · increase the bus factor · mitigate bus-factor risk*
- **Nuance & Exam Tip:** Also called 'lottery factor' in gentler workplaces ('What happens if Alex wins the lottery tomorrow?'). Higher bus factor = healthier resilience.
- **Word Family:** institutional knowledge · single point of failure · succession planning · documentation

---

### Institutional knowledge
?
- **Part of Speech:** Organizational Noun Phrase [Formal]
- **Español:** conocimiento institucional / memoria organizativa
- **Definition:** The accumulated context, historical constraints, undocumented edge cases, and lessons learned residing inside an organization's people, ADRs, and postmortems.
- **Example:** *"Without written postmortems and ADRs, layoffs wipe out years of hard-won institutional knowledge overnight."*
- **Collocations:** *preserve institutional knowledge · lose institutional knowledge · tribal knowledge*
- **Nuance & Exam Tip:** Contrast with 'tribal knowledge': 'tribal knowledge' is unwritten lore trapped in veterans' heads, whereas 'institutional knowledge' can be captured in docs and systems.
- **Word Family:** tribal knowledge · bus factor · ADR · knowledge transfer

---

### Thought leadership
?
- **Part of Speech:** Branding / Career Noun Phrase [Formal]
- **Español:** liderazgo intelectual / referencia de opinión en el sector
- **Definition:** Sharing authoritative, original insights via engineering blogs, conference talks, open-source RFCs, or industry articles that shape how peers solve problems.
- **Example:** *"Publishing our distributed-tracing case study built genuine thought leadership and attracted strong senior applicants."*
- **Collocations:** *establish thought leadership · engineering blog · external visibility*
- **Nuance & Exam Tip:** Watch out for empty LinkedIn fluff! Real engineering thought leadership shares hard numbers, honest trade-offs, and post-incident lessons—never buzzwords.
- **Word Family:** external visibility · engineering brand · evangelism · community

---

### Elevate the team
?
- **Part of Speech:** Leadership Collocation [Formal / Neutral]
- **Español:** elevar el nivel del equipo / potenciar al grupo
- **Definition:** To intentionally raise collective technical standards, mentorship quality, and confidence so teammates grow faster alongside you.
- **Example:** *"Pair-programming through complex debugging sessions lets staff engineers elevate the team rather than hoarding hard problems."*
- **Collocations:** *elevate those around you · raise the bar · level up the team*
- **Nuance & Exam Tip:** Top signal in senior/staff behavioral interviews: explain not just what *you* built, but how the team became stronger because you were in the room.
- **Word Family:** force multiplier · mentor · level up · raise the bar

---

### Delegate effectively
?
- **Part of Speech:** Management Collocation [Formal]
- **Español:** delegar eficazmente
- **Definition:** To hand off real decision authority and stretch projects to teammates with clear success criteria, context, and support—rather than dumping tedious chores or hovering.
- **Example:** *"Learning to delegate effectively meant handing full RFC ownership to my senior dev while staying available for design check-ins."*
- **Collocations:** *delegate outcomes, not tasks · empower through delegation · avoid abdication*
- **Nuance & Exam Tip:** Key contrast: 'delegation' pairs clear guardrails and periodic checkpoints with autonomy; simply tossing a task with zero support is 'abdication'.
- **Word Family:** micromanage · empower autonomy · ownership · stretch assignment

---

### Micromanage
?
- **Part of Speech:** Management Verb [Critical]
- **Español:** microgestionar / controlar hasta el último detalle
- **Definition:** To supervise and dictate every minor tactical step of a subordinate's or teammate's work rather than trusting them with clear outcomes and autonomy.
- **Example:** *"Dictating variable names and daily ticket order makes experienced engineers feel micromanaged and kills intrinsic motivation."*
- **Collocations:** *tendency to micromanage · feel micromanaged · step back from micromanaging*
- **Nuance & Exam Tip:** Instead of accusing someone of 'micromanaging' to their face, align upward by asking: 'Would you prefer daily check-ins right now, or milestone demos on Friday?'
- **Word Family:** micromanagement (n.) · empower autonomy · delegate effectively · hover

---

### Empower autonomy
?
- **Part of Speech:** Leadership Collocation [Formal]
- **Español:** fomentar la autonomía / dar capacidad de decisión real
- **Definition:** To equip teams with clear business context, guardrails, and decision rights so they can solve problems independently without bottlenecks.
- **Example:** *"Giving product squads independent deploy pipelines did more to empower autonomy than any leadership memo."*
- **Collocations:** *aligned autonomy · empower teams · guardrails over gates*
- **Nuance & Exam Tip:** Best paired with 'alignment' ('aligned autonomy'): autonomy without shared direction produces chaos; alignment without autonomy produces bureaucracy.
- **Word Family:** autonomous execution · delegate effectively · micromanage · guardrails

---

### Course-correct
?
- **Part of Speech:** Execution Verb [Neutral]
- **Español:** corregir el rumbo / reorientar a tiempo
- **Definition:** To make timely, evidence-based adjustments to an active plan, architecture, or behavior when telemetry or feedback reveals drift.
- **Example:** *"Weekly metrics reviews let us course-correct the onboarding flow before wasting another month on unused features."*
- **Collocations:** *course-correct early · mid-flight course correction · agile feedback loop*
- **Nuance & Exam Tip:** Aviation/navigation metaphor; framed positively as adaptive strength rather than admitting failure.
- **Word Family:** pivot · iterate · continuous feedback loop · adapt

---

### Hindsight bias
?
- **Part of Speech:** Cognitive Bias Noun Phrase [Formal]
- **Español:** sesgo retrospectivo / ilusión de que «ya se veía venir»
- **Definition:** The cognitive illusion after an outage or failure that the trigger was obvious and easily predictable beforehand, ignoring the uncertainty engineers faced in real time.
- **Example:** *"Blameless postmortems guard against hindsight bias by reconstructing exactly what alerts and graphs were visible at 2:14 a.m."*
- **Collocations:** *beware hindsight bias · counterfactual thinking · knew-it-all-along effect*
- **Nuance & Exam Tip:** Foundational SRE concept (Sidney Dekker/John Allspaw): asking 'Why didn't they check X?' in postmortems usually reflects pure hindsight bias.
- **Word Family:** blameless postmortem · post-hoc rationalization · counterfactual · cognitive bias

---

### Second-order thinking
?
- **Part of Speech:** Decision Science Noun Phrase [C2 / Formal]
- **Español:** pensamiento de segundo orden (anticipar las consecuencias de las consecuencias)
- **Definition:** Looking beyond the immediate, obvious result of an action ('first-order effect') to anticipate downstream ripple effects, incentives, and unintended consequences.
- **Example:** *"Second-order thinking asks: if we reward engineers solely by PRs merged, won't people game metrics with tiny trivial commits?"*
- **Collocations:** *apply second-order thinking · downstream ripple effects · unintended consequences*
- **Nuance & Exam Tip:** Signature executive and staff-engineer trait: 'And then what happens next?' separates thoughtful systems thinkers from reactive problem-solvers.
- **Word Family:** systems thinking · Goodhart's Law · unintended consequences · first-principles thinking

---

### First-principles thinking
?
- **Part of Speech:** Problem-Solving Noun Phrase [Formal]
- **Español:** pensamiento desde primeros principios
- **Definition:** Deconstructing a problem down to its irreducible physical, mathematical, or economic truths and reasoning upward instead of copying industry conventions blindly.
- **Example:** *"Applying first-principles thinking showed our hardware budget came from polling frequency, not dataset size—so push events cut costs 80 percent."*
- **Collocations:** *reason from first principles · challenge conventional wisdom · fundamental constraints*
- **Nuance & Exam Tip:** Powerful in architecture reviews, though watch out for 'reinventing the wheel'—combine first-principles clarity with respect for proven standards.
- **Word Family:** second-order thinking · root cause · cargo culting · lateral thinking

---

### Intellectual humility
?
- **Part of Speech:** Character Trait Noun Phrase [Formal]
- **Español:** humildad intelectual
- **Definition:** Recognizing the limits of your own knowledge, staying genuinely curious about conflicting viewpoints, and admitting errors promptly without ego defense.
- **Example:** *"Senior interviewers probe for intellectual humility—candidates who say 'Actually, looking back, my initial indexing design was flawed' stand out immediately."*
- **Collocations:** *show intellectual humility · ego-free engineering · update your beliefs*
- **Nuance & Exam Tip:** Far from signaling weakness, intellectual humility at senior levels conveys quiet confidence and safety to collaborate with.
- **Word Family:** strong opinions, loosely held · ego-free · growth mindset · self-awareness

---

### Quantify impact
?
- **Part of Speech:** Resume / Review Collocation [Formal]
- **Español:** cuantificar el impacto
- **Definition:** To express technical work in concrete, verifiable numbers—latency saved, revenue unlocked, error rate dropped, or hours reclaimed—instead of vague task descriptions.
- **Example:** *"Don't write 'optimized SQL queries' on your resume; quantify impact by noting 'cut p95 query latency by 62 percent, saving $4k/month in RDS compute.'"*
- **Collocations:** *quantify your impact · measurable outcome · before-and-after metrics*
- **Nuance & Exam Tip:** Even if exact revenue is unknown, use engineering proxies: percentage speedup, incidents prevented, hours saved per week, or adoption rate.
- **Word Family:** STAR framework · brag document · outcomes over outputs · ROI

---

### Performance calibration
?
- **Part of Speech:** HR / Management Process [Formal]
- **Español:** calibración de desempeño (reunión de homogeneización de evaluaciones)
- **Definition:** A cross-team leadership meeting where managers compare promotion packets and ratings side by side to ensure fair, consistent standards across the organization.
- **Example:** *"Having concrete metrics and peer quotes in your packet helps your manager defend your promotion during performance calibration."*
- **Collocations:** *calibration session · promotion packet · consistent bar · calibrated rating*
- **Nuance & Exam Tip:** Knowing how calibration works demystifies promotions: your manager isn't the sole judge—your written evidence has to persuade peer directors who don't know you.
- **Word Family:** career ladder · brag document · mentor vs. sponsor · promotion packet

---

### Earn trust
?
- **Part of Speech:** Leadership Principle Collocation [Neutral]
- **Español:** ganarse la confianza
- **Definition:** Building sustained credibility through consistent follow-through, admitting mistakes openly, listening attentively, and treating teammates with respect.
- **Example:** *"Admitting our migration missed an edge case—and publishing the fix proactively—helped us earn trust with our largest enterprise client."*
- **Collocations:** *earn trust over time · erode trust · high-trust environment · trust bank*
- **Nuance & Exam Tip:** Note the verb: English says 'earn trust' or 'build trust' (not 'win trust' in workplace contexts). Trust builds slowly in drops and drains fast in buckets.
- **Word Family:** proactive transparency · credibility · psychological safety · integrity

---

### Raise the bar
?
- **Part of Speech:** Quality / Hiring Idiom [Neutral]
- **Español:** subir el listón / elevar el estándar de exigencia
- **Definition:** To elevate expected standards of engineering quality, hiring excellence, or operational rigor across a team or company.
- **Example:** *"Every new senior hire should raise the bar on code quality, mentorship, or systems judgment—not merely fill an empty seat."*
- **Collocations:** *raise the engineering bar · bar raiser · set a high bar · lower the bar*
- **Nuance & Exam Tip:** Amazon famously calls independent hiring reviewers 'Bar Raisers'; in daily engineering, focus on raising the bar systematically via tooling rather than shaming peers.
- **Word Family:** elevate the team · high standards · bar raiser · excellence

---

### Culture add vs. culture fit
?
- **Part of Speech:** Hiring Philosophy Contrast [Formal]
- **Español:** aporte cultural (cultura que suma diversidad) frente a encaje cultural (homogeneidad)
- **Definition:** Shifting hiring evaluation from 'culture fit'—which risks cloning existing backgrounds and biases—to 'culture add', valuing fresh perspectives aligned with shared core values.
- **Example:** *"Moving from 'culture fit' to 'culture add' helped our platform team hire engineers who brought much-needed accessibility expertise."*
- **Collocations:** *hire for culture add · shared values · avoid cloning bias · diverse perspectives*
- **Nuance & Exam Tip:** Important modern hiring nuance: rejecting a candidate for vague 'bad culture fit' triggers bias flags; specify exact behaviors and values instead.
- **Word Family:** inclusive hiring · diversity of thought · core values · rubric

---

### Continuous feedback loop
?
- **Part of Speech:** Systems / Management Noun Phrase [Formal]
- **Español:** bucle de retroalimentación continua
- **Definition:** An ongoing cycle in which telemetry, user behavior, or peer critique feeds directly back into rapid product, code, or leadership improvements.
- **Example:** *"Tightening our continuous feedback loop between customer support tags and weekly sprint triage halved repeat UX bugs."*
- **Collocations:** *tighten the feedback loop · shorten the feedback loop · close the feedback loop*
- **Nuance & Exam Tip:** Applies universally across CI/CD speed (fast test loops), product discovery, and 1:1 coaching ('no surprises at annual review time').
- **Word Family:** tight feedback loop · iterate · course-correct · retrospective

---
