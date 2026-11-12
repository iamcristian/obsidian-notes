---
tags:
  - ai
  - ai-agents
  - llm
  - orchestration
  - providers
  - open-source
  - python
  - lab
created: 2026-11-07
course: "AI Engineer Agentic Track: The Complete Agent & MCP Course"
module: "Week 1.3 - Orchestrating LLMs & Model Ecosystem"
---

# Orchestrating LLMs & The Model Ecosystem

> [!info] Navigation
> Previous: [[Week 1.2|Week 1.2 - Agent Traps, Risks & Engineering Mindset]].

---

## 🏢 The Big Three Model Families (Tiered Sizing)

Frontier providers structure their offerings into **three distinct tiers**:

| Tier                                   | Role & Characteristics                                          | OpenAI                        | Anthropic         | Google                |
| :------------------------------------- | :-------------------------------------------------------------- | :---------------------------- | :---------------- | :-------------------- |
| **Small (Fast & Cheap)**               | High throughput, classification, routing, fast guardrails.      | **GPT Nano** (e.g., 5.4 Nano) | **Claude Haiku**  | **Gemini Flash Lite** |
| **Medium (Balanced Workhorse)**        | Default for agent loops, function calling, tool use, reasoning. | **GPT Mini** (e.g., 5.4 Mini) | **Claude Sonnet** | **Gemini Flash**      |
| **Large (Flagship / Heavy Reasoning)** | Deep planning, complex code architecture, final evaluation.     | **GPT** (e.g., 5.5)           | **Claude Opus**   | **Gemini Pro**        |
