---
tags:
  - ai
  - ai-agents
  - mcp
  - workflows
  - architecture
created: 2026-11-06
course: "AI Engineer Agentic Track: The Complete Agent & MCP Course"
module: "Week 1.1 - Concepts & Architectural Patterns"
---

# Concepts & Agentic Architectures

> [!info] Navigation
> Next lesson: [[Week 1.2|Week 1.2 - Agent Traps, Risks & Engineering Mindset]].

---

## AI Agents - 3 Primary Definitions

1. **Autonomous execution:** AI systems capable of performing tasks independently.
2. **LLM-guided orchestration:** A system where an LLM guides the workflow dynamically based on user prompts.
3. **Tool-use loop:** An LLM that utilizes external tools in an iterative loop to achieve specific goals.

---

## Agentic Systems - Anthropic Distinction

Anthropic categorizes agentic architectures into two fundamental types:

- **Workflows:** Systems where LLMs and tools are orchestrated through **predefined code paths** (deterministic routing, explicit branching).
- **Agents:** Dynamic systems where LLMs direct their own processes and tool usage, maintaining autonomous control over how they accomplish tasks.

---

## Workflow Design Patterns

### 1. Prompt Chaining
Decomposes a complex task into a sequential pipeline of fixed sub-tasks, passing intermediate results with optional validation gates.

```mermaid
%%{init: {'flowchart': {'nodeSpacing': 12, 'rankSpacing': 20}}}%%
flowchart LR
    Input([Input]) --> LLM1["LLM 1"]
    LLM1 --> Gate{"Gate"}
    Gate -->|Pass| LLM2["LLM 2"]
    Gate -->|Fail| Exit["Exit / Fallback"]
    LLM2 --> LLM3["LLM 3"]
    LLM3 --> Out([Output])
```

---

### 2. Routing
Directs an input into a specialized sub-task or domain-specific prompt/model, ensuring clear separation of concerns.

```mermaid
%%{init: {'flowchart': {'nodeSpacing': 12, 'rankSpacing': 20}}}%%
flowchart TD
    Input([Input]) --> Router["LLM Router"]
    Router -->|Type A| LLM1["LLM 1"]
    Router -->|Type B| LLM2["LLM 2"]
    Router -->|Type C| LLM3["LLM 3"]
    LLM1 --> Out([Output])
    LLM2 --> Out
    LLM3 --> Out
```

---

### 3. Parallelization
Breaks down tasks to run multiple subtasks concurrently (either sectioning independent tasks or ensembling/voting for higher reliability).

```mermaid
%%{init: {'flowchart': {'nodeSpacing': 12, 'rankSpacing': 20}}}%%
flowchart TD
    Input([Input]) --> Coord["Coordinator"]
    Coord --> LLM1["LLM 1"]
    Coord --> LLM2["LLM 2"]
    Coord --> LLM3["LLM 3"]
    LLM1 --> Agg["Aggregator"]
    LLM2 --> Agg
    LLM3 --> Agg
    Agg --> Out([Output])
```

---

### 4. Orchestrator-Worker
A central orchestrator LLM dynamically breaks down a complex problem into subtasks on the fly, delegates them to worker LLMs, and synthesizes the outputs.

```mermaid
%%{init: {'flowchart': {'nodeSpacing': 12, 'rankSpacing': 20}}}%%
flowchart TD
    Input([Input]) --> Orch["Orchestrator"]
    Orch -->|Task 1| W1["Worker 1"]
    Orch -->|Task 2| W2["Worker 2"]
    Orch -->|Task 3| W3["Worker 3"]
    W1 --> Synth["Synthesizer"]
    W2 --> Synth
    W3 --> Synth
    Synth --> Out([Output])
```

---

### 5. Evaluator-Optimizer
One LLM generates candidate solutions while another evaluates/critiques them, creating an iterative refinement loop until quality criteria are met.

```mermaid
%%{init: {'flowchart': {'nodeSpacing': 12, 'rankSpacing': 20}}}%%
flowchart LR
    Input([Input]) --> Gen["LLM Generator"]
    Gen --> Eval{"LLM Evaluator"}
    Eval -->|Feedback| Gen
    Eval -->|Pass| Out([Output])
```

---

## Agents Design Patterns

Autonomous agents operate in open-ended execution loops without a rigid, predefined path:

1. **Open-ended problem solving:** The model decides which actions to take next.
2. **Feedback loops:** Results from actions/environment inform subsequent thoughts.
3. **No fixed path:** Trajectories emerge dynamically based on tool responses.

```mermaid
%%{init: {'flowchart': {'nodeSpacing': 12, 'rankSpacing': 20}}}%%
flowchart TD
    User([User Prompt]) --> Agent["LLM Agent"]
    Agent -->|Tool Call| Env["Environment / Tools"]
    Env -->|Observation| Agent
    Agent -->|Stop| Out([Result])
```
