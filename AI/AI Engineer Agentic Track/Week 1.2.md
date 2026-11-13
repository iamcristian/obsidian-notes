---
tags:
  - ai
  - ai-agents
  - engineering-principles
  - best-practices
created: 2026-11-06
course: "AI Engineer Agentic Track: The Complete Agent & MCP Course"
module: "Week 1.2 - Agent Traps, Risks & Engineering Mindset"
---

# Agent Traps, Risks & Engineering Mindset

> [!info] Navigation
> - Previous: [[Week 1.1|Week 1.1 - Concepts & Architectural Patterns]]
> - Next: [[Week 1.3|Week 1.3 - Orchestrating LLMs & Model Ecosystem]]

> [!tip] Companion Engineering Reference
> Production guardrails, tracing, doom-loop hashing, and security controls: [[AI/AI Engineering/Agentic Systems, Tool Calling & MCP|Agentic Systems, Tool Calling & MCP]] · [[AI/AI Engineering/LLM Evaluation, Guardrails & Security|LLM Evaluation, Guardrails & Security]].

---

## Risks of Agent Frameworks

- **Unpredictable path:** Execution flow is dynamic and non-deterministic.
- **Unpredictable output:** Variance in generation and tool arguments.
- **Unpredictable costs:** Token spikes due to loops or bloated context windows.

### Ways of Mitigating
- **Observability & Monitoring:** Tracing calls, token consumption, latency, and automated evaluations (Evals).
- **Guardrails:** Deterministic constraints to ensure agents behave safely, consistently, and within intended system boundaries.

---

## First Agent Trap: Solution Looking for a Problem

> [!quote] Client conversation trap
> **Client:** "I need an Agent for my business."  
> **Engineer:** "Ok — and what business problem is that solving?"  
> **Client:** "I want a strategy agent to give me advice for my org."  
> **Engineer:** "Right... and what specific business problem does that solve?"

> [!tip] Golden Rule
> Always start with a **concrete business problem**, never with the technology.

---

## Second Agent Trap: Anthropomorphizing

**Anthropomorphizing:** Designing agent architectures that assign human-like organizational roles to AI agents:
- *Trading manager agent*
- *Market research agent*
- *Trader agent*
- *Risk manager agent*

> [!IMPORTANT]
> **They are not people.** They are token generation engines.  
> **Always start simple:** 1 LLM, 1 prompt, 1 concrete objective. Add multi-agent complexity *only* when empirical performance data justifies it.

---

## Core Takeaways

1. **Measurable Value:** Build agent systems to solve real problems in a measurable, verifiable way.
2. **Performance-Driven Architecture:** Decide your agent architecture based on performance, cost, and reliability metrics—not hype.
3. **Plausibility vs. Accuracy:** LLMs generate *plausible* outputs, not necessarily *accurate* outputs. Aligning outputs with business results and deterministic guarantees is the engineer's responsibility.

---

## The Agentic Engineer

> Is an Agentic Engineer:
> 1. Someone who uses agents to engineer software?
> 2. Someone who engineers agentic systems?

- **Current Course Goal:** Leveraging software engineering skills to build, debug, and orchestrate robust, production-grade AI agents and MCP tools.
