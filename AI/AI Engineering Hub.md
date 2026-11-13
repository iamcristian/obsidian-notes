---
tags:
  - ai
  - ai-engineering
  - hub
  - architecture
  - moc
created: 2026-11-13
updated: 2026-11-13
status: 🟢
level: mid-to-senior
---
# AI Engineering Hub

> [!abstract] Scope & Engineering Standard
> Master technical index for designing, evaluating, securing, and optimizing production LLM and agentic systems. Every guide targets **Mid-to-Senior engineering mastery**: wire-level protocols, memory/latency economics, deterministic verification invariants, failure modes, and worked diagnostic exercises.

**Prerequisites & Cross-Domain Anchors:** [[Programming/Machine Learning/Deep Learning Guide|Deep Learning Guide]] · [[Programming/Software Engineering/DevOps/Observability|Observability]]

---

## 1. System Architecture & Dependency Map

```mermaid
flowchart LR
    Inf["1. LLM Internals & Economics"] --> Ctx["2. Context & Structured Outputs"]
    Ctx --> RAG["3. Production RAG"]
    Ctx --> Ag["4. Agents, Tools & MCP"]
    RAG --> Ag
    Ag --> Eval["5. Evals, Guardrails & Security"]
    Eval --> FT["6. Fine-Tuning, LoRA & Alignment"]
```

---

## 2. Core Curriculum Matrix (6 Deep Technical Guides)

| Phase | Guide | Core Mechanics & Invariants | Primary Production Failure Modes Addressed |
| :--- | :--- | :--- | :--- |
| **1. Foundations** | [[AI/AI Engineering/LLM Internals, Inference & Economics|LLM Internals, Inference & Economics]] | Prefill vs. decode arithmetic intensity, KV-cache VRAM sizing, PagedAttention, continuous batching, prefix caching, quantization, cost/TTFT/ITL math | OOM under concurrency spikes, cache invalidation on dynamic prefixes, unbudgeted long-context latency |
| **2. Conditioning** | [[AI/AI Engineering/Context Engineering & Structured Outputs|Context Engineering & Structured Outputs]] | Token budgets, lost-in-the-middle mitigation, KV-prefix stability, CFG/FSM constrained decoding, Pydantic/JSON Schema strict modes | Fragile regex parsing, silent schema drift, non-deterministic tool argument failures |
| **3. Retrieval** | [[AI/AI Engineering/Production RAG Architecture|Production RAG Architecture]] | Semantic/structure-aware chunking, hybrid BM25 + dense vector + RRF, cross-encoder reranking, metadata ACL pre-filtering, RAGAS metrics | Lost context across split chunks, stale indexes, cross-tenant data leakage, false precision |
| **4. Execution** | [[AI/AI Engineering/Agentic Systems, Tool Calling & MCP|Agentic Systems, Tool Calling & MCP]] | Workflows vs. agents decision boundary, ACI design, bounded ReAct loops, doom-loop hashing, idempotency keys, MCP JSON-RPC/Streamable HTTP/OAuth 2.1 | Multiplicative accuracy collapse, $O(T^2)$ ping-pong bills, duplicate mutations, MCP confused deputy & `stdout` framing bugs |
| **5. Assurance** | [[AI/AI Engineering/LLM Evaluation, Guardrails & Security|LLM Evaluation, Guardrails & Security]] | 3-tier eval pyramid, $\text{pass}@k$, bias-mitigated LLM-as-a-Judge + Cohen's $\kappa$, OpenTelemetry GenAI traces, Lethal Trifecta, OWASP LLM Top 10 | Uncalibrated judge drift, silent model-swap regressions, indirect prompt injection, Markdown XSS / Text-to-SQL injection |
| **6. Adaptation** | [[AI/AI Engineering/Fine-Tuning, LoRA & Alignment|Fine-Tuning, LoRA & Alignment]] | Build-vs-buy tree, `-100` assistant-only loss masking, LoRA/QLoRA (`NF4`, DQ, Paged AdamW) memory math, PPO vs. DPO vs. GRPO | Memorizing mutable facts, cross-sample packing bleed, likelihood displacement, catastrophic forgetting |

---

## 3. Architectural Decision Matrix: Choosing the Right Lever

Always apply the **lowest-complexity lever** that satisfies accuracy, latency, privacy, and cost SLAs before escalating to heavier architectures.

| Engineering Requirement | Primary Mechanism | Why Higher-Complexity Alternatives Fail | Reference Guide |
| :--- | :--- | :--- | :--- |
| **Format, schema, or tone control** | Context engineering + `strict` JSON Schema / CFG | Fine-tuning adds training/maintenance overhead before prompt iteration stabilizes | [[AI/AI Engineering/Context Engineering & Structured Outputs|Context Engineering & Structured Outputs]] |
| **Mutable knowledge & strict ACLs** | Hybrid RAG + DB/vector pre-filter ACLs | Weights cannot enforce per-user permissions or atomic knowledge updates | [[AI/AI Engineering/Production RAG Architecture|Production RAG Architecture]] |
| **Multi-step predictable pipeline** | Deterministic Workflow (Routing / Chaining / Parallel) | Open-ended agent loops burn tokens and compound per-step failure rates ($p^k$) | [[AI/AI Engineering/Agentic Systems, Tool Calling & MCP|Agentic Systems, Tool Calling & MCP]] |
| **Dynamic external state inspection** | Bounded Single Agent + hardened ACI / MCP tools | Multi-agent swarms introduce serial latency and "telephone game" state loss | [[AI/AI Engineering/Agentic Systems, Tool Calling & MCP|Agentic Systems, Tool Calling & MCP]] |
| **Preventing silent regressions & exploits** | Layered deterministic assertions + calibrated Judge + sandboxing | System prompts alone are bypassable; vibes-based testing misses slice regressions | [[AI/AI Engineering/LLM Evaluation, Guardrails & Security|LLM Evaluation, Guardrails & Security]] |
| **10–30$\times$ cost/latency cut on fixed skill** | Frontier-to-8B LoRA SFT distillation | Frontier prompts carry high TTFT and token spend at scale | [[AI/AI Engineering/Fine-Tuning, LoRA & Alignment|Fine-Tuning, LoRA & Alignment]] |
| **Verifiable math, SQL, or code reasoning** | GRPO (RLVR with programmatic test/compiler oracle) | Pure SFT lacks negative boundary signal; PPO requires 4 models in VRAM | [[AI/AI Engineering/Fine-Tuning, LoRA & Alignment|Fine-Tuning, LoRA & Alignment]] |

---

## 4. Study Progression Path & Course Integration

### Recommended Deep-Dive Sequence

1. **Stage 1 — Inference Physics & Economics:** Master [[AI/AI Engineering/LLM Internals, Inference & Economics|LLM Internals, Inference & Economics]] (backed by [[Programming/Machine Learning/Deep Learning Guide|Deep Learning Guide]]) so latency, KV-cache memory, and token billing math govern every downstream architectural trade-off.
2. **Stage 2 — Conditioning & Grounding:** Study [[AI/AI Engineering/Context Engineering & Structured Outputs|Context Engineering & Structured Outputs]] followed by [[AI/AI Engineering/Production RAG Architecture|Production RAG Architecture]] to stabilize single-turn accuracy and deterministic provenance before adding loops.
3. **Stage 3 — Agentic Control & MCP:** Build through [[AI/AI Engineering/Agentic Systems, Tool Calling & MCP|Agentic Systems, Tool Calling & MCP]] with hard step/cost budgets, idempotency, and standardized MCP transports.
4. **Stage 4 — Verification, Security & Adaptation:** Pair [[AI/AI Engineering/LLM Evaluation, Guardrails & Security|LLM Evaluation, Guardrails & Security]] (plus [[Programming/Software Engineering/DevOps/Observability|Observability]]) with [[AI/AI Engineering/Fine-Tuning, LoRA & Alignment|Fine-Tuning, LoRA & Alignment]] so model customization is always backed by quantitative CI regression gates.

### Mapping to `AI/AI Engineer Agentic Track/` Course Notes

| Course Note | Course Topic | Corresponding Deep Engineering Guide |
| :--- | :--- | :--- |
| [[AI/AI Engineer Agentic Track/Week 1.1|Week 1.1]] | Anthropic Workflows vs. Agents & 5 Core Workflow Patterns | [[AI/AI Engineering/Agentic Systems, Tool Calling & MCP|Agentic Systems, Tool Calling & MCP]] (Section 1) |
| [[AI/AI Engineer Agentic Track/Week 1.2|Week 1.2]] | Agent Traps (Anthropomorphizing, Risk Mitigation, Mindset) | [[AI/AI Engineering/Agentic Systems, Tool Calling & MCP|Agentic Systems, Tool Calling & MCP]] (Sections 3 & 5) & [[AI/AI Engineering/LLM Evaluation, Guardrails & Security|LLM Evaluation, Guardrails & Security]] |
| [[AI/AI Engineer Agentic Track/Week 1.3|Week 1.3]] | Tiered Model Sizing (Small / Medium / Large Ecosystems) | [[AI/AI Engineering/LLM Internals, Inference & Economics|LLM Internals, Inference & Economics]] & [[AI/AI Engineering/Agentic Systems, Tool Calling & MCP|Agentic Systems, Tool Calling & MCP]] |
| `Week 1.4.md` *(stub transcript note)* | Pending course lecture capture in `AI/AI Engineer Agentic Track/` | Expand into the relevant production guide once captured |
