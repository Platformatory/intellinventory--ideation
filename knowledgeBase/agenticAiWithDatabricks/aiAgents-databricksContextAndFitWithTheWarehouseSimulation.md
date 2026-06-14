<h1>AI Agents: Databricks Context & Fit with the Warehouse Simulation</h1>

> **AI-Use Disclaimer**: This documented was generated using Claude Sonnet 4.6, with the following context and prompt.
> 
> **Context**: [`implementation`](../../implementation/); specifically:
>
> - [`implementation/docs/simulationDevelopmentApproach.md`](../../implementation/docs/simulationDevelopmentApproach.md)
> - [`pranavg-platformatory/dbx-agentic-ai-project-sim-package`, `devlog.md` (**github.com**)](https://github.com/pranavg-platformatory/dbx-agentic-ai-project-sim-package/blob/main/devlog.md)
> 
> **Prompt**: 

```
I want to explore 2 questions, but with clear references for each statement, and a comprehensive overview.

1. What can an "agent" (AI agent) mean w.r.t. Databricks
2. How would an agentic AI system fit in our context (as given by the attached files)
```


---

**Contents**:

- [1. What Can an "Agent" Mean w.r.t. Databricks?](#1-what-can-an-agent-mean-wrt-databricks)
  - [1a. Mosaic AI Agent Framework (the core product meaning)](#1a-mosaic-ai-agent-framework-the-core-product-meaning)
  - [1b. ReAct / Tool-calling Loop Agents](#1b-react--tool-calling-loop-agents)
  - [1c. Unity Catalog Functions as Agent Tools](#1c-unity-catalog-functions-as-agent-tools)
  - [1d. Agent Evaluation (the "quality" meaning)](#1d-agent-evaluation-the-quality-meaning)
  - [1e. Summary of Meanings](#1e-summary-of-meanings)
- [2. How Would an Agentic AI System Fit in Our Context?](#2-how-would-an-agentic-ai-system-fit-in-our-context)
  - [2a. The Contract Is Already in Place](#2a-the-contract-is-already-in-place)
  - [2b. What `AgentContext` Gives the LLM](#2b-what-agentcontext-gives-the-llm)
  - [2c. The Databricks-Native Implementation Path](#2c-the-databricks-native-implementation-path)
  - [2d. Evaluation: The Rule-Based Agent as Baseline](#2d-evaluation-the-rule-based-agent-as-baseline)
  - [2e. Where the Interesting Problems Are](#2e-where-the-interesting-problems-are)
  - [2f. Summary](#2f-summary)

---


# 1. What Can an "Agent" Mean w.r.t. Databricks?

Databricks uses the term "agent" across several overlapping but distinct meanings. The following covers the primary interpretations, from narrowest to broadest.

---

## 1a. Mosaic AI Agent Framework (the core product meaning)

Databricks' primary definition of an agent is a **compound AI system that combines an LLM with tools, memory, and an orchestration loop** — all built, evaluated, and deployed within the Databricks platform. This is what Databricks means when it says "agents" in its product documentation (Mosaic AI docs, 2024–2025).

The canonical components are:

- **LLM backbone** — typically a Foundation Model served via Databricks Model Serving (e.g. DBRX, Llama 3, or an external model like Claude/GPT-4 via the AI Gateway).
- **Tools** — Python functions the LLM can call. These can be Unity Catalog functions (registered SQL or Python UDFs), Vector Search retrievals, or arbitrary REST calls.
- **Retrieval / RAG** — Databricks Vector Search provides an embedding store the agent can query for context before generating a response. The retrieved chunks are injected into the LLM's prompt.
- **Orchestration** — either LangChain/LangGraph (explicitly first-class in Databricks' docs) or PyFunc wrappers defining a custom `predict()` method. The orchestrator controls the loop: tool selection → call → observe → next step.
- **Memory** — short-term (within a session) via prompt context; longer-term via Delta tables or Vector Search if the developer implements it.
- **MLflow tracing** — every agent invocation can emit spans (tool calls, LLM calls, retrieved chunks) to MLflow, giving an audit trail of reasoning steps.

> **Reference**: Databricks documentation — *"Build your first agent"*, Mosaic AI Agent Framework overview. The framework is designed so that agents are deployed as Model Serving endpoints, which means they are versioned, governed, and observable just like any other ML model.

---

## 1b. ReAct / Tool-calling Loop Agents

Within the Agent Framework, Databricks specifically endorses the **ReAct pattern** (Reason + Act): the LLM iterates between generating a thought, calling a tool, observing the result, and deciding whether to continue or return a final answer.

The practical unit here is:

```
[User Input]
    → LLM thinks: "I need to check current stock for item X"
    → Tool call: query_warehouse_state(item_id="X")
    → Observe result: {stock_on_hand: 12, reorder_point: 20}
    → LLM thinks: "Stock is below reorder point; I should recommend a reorder"
    → Final answer: ReorderDecision(item_id="X", qty=100)
```

This is a **single-turn agent** (one user input → one final output after N internal steps). Databricks also supports multi-turn (chat-style) agents via the same serving infrastructure.

> **Reference**: Databricks docs — *"ReAct agents"*, LangGraph integration guide. LangGraph is the preferred orchestration library for stateful, multi-step agents on Databricks as of 2024.

---

## 1c. Unity Catalog Functions as Agent Tools

One of the most Databricks-specific interpretations of "agent tooling" is the use of **Unity Catalog (UC) as a tool registry**. A Python or SQL function registered in UC can be called by an agent at runtime. This gives tools:

- **Governance** — access control, lineage, and audit are inherited from Unity Catalog.
- **Discoverability** — tools are catalogued alongside the data they operate on.
- **Versioning** — function signatures and implementations are versioned in the Metastore.

For a data-heavy simulation like the warehouse system, this is relevant: a function like `get_warehouse_state(sim_id, tick)` registered in UC becomes a first-class tool an LLM agent can call without any additional wrapping.

> **Reference**: Databricks docs — *"Use Unity Catalog tools with agents"*. This pattern is specifically called out as the production-grade way to expose operational data to an agent.

---

## 1d. Agent Evaluation (the "quality" meaning)

Databricks also uses "agent" in the context of **Agent Evaluation** — a framework within MLflow for assessing agent output quality. An evaluator agent (sometimes called a "judge LLM") scores another agent's responses for correctness, groundedness, relevance, and safety. This is a meta-agent: an AI system whose job is to evaluate AI systems.

This matters for production deployment: before an agent goes live, Databricks recommends running it through the evaluation harness with a golden dataset and human review via the review app.

> **Reference**: MLflow docs — *"Agent Evaluation"*, Databricks docs — *"Evaluate your agent"*.

---

## 1e. Summary of Meanings

| Meaning | What it refers to | Databricks surface |
|---|---|---|
| Mosaic AI Agent | LLM + tools + orchestration loop | Agent Framework, Model Serving |
| ReAct/tool-calling agent | Iterative reason-act-observe loop | LangGraph, PyFunc |
| UC-registered tool agent | Agent backed by UC functions as tools | Unity Catalog, AI Gateway |
| Judge/evaluator agent | LLM that scores another agent's output | MLflow Agent Evaluation |
| Any `BaseAgent` subclass | Project-internal: any class implementing `decide()` | `warehouse_sim/agent/` |

The last row is the project's own internal meaning — discussed in the next section.

---

# 2. How Would an Agentic AI System Fit in Our Context?

The warehouse simulation was explicitly designed for this. The architecture documents make the intent clear: *"The final use-case is the LLM agent"* and *"treat the agent as a plugin"* (`simulationDevelopmentApproach.md`, Overview). What follows traces how an agentic AI system slots into the existing stack, what it would need to do, and where the natural extension points are.

---

## 2a. The Contract Is Already in Place

The single most important architectural fact is that the agent boundary is already clean and complete. From `agent/base.py`:

```python
class BaseAgent(ABC):
    @abstractmethod
    def decide(self, context: AgentContext) -> list[ReorderDecision]:
        ...
```

This contract — `AgentContext` in, `list[ReorderDecision]` out — is the only interface the engine ever sees. The engine does not import any concrete agent; it receives one at startup via dependency injection (`simulationDevelopmentApproach.md`, Agent Contract; `devlog.md`, Stage 4 Pre-Development Notes: *"Agent is injected. runner.py takes a BaseAgent instance as a parameter — it calls agent.decide(context) at sub-step 4."*).

An LLM agent built on Databricks' Mosaic AI Agent Framework would simply be a new subclass:

```python
class LLMAgent(BaseAgent):
    def decide(self, context: AgentContext) -> list[ReorderDecision]:
        # call LLM (via Model Serving endpoint or AI Gateway)
        # parse response into list[ReorderDecision]
        ...
```

No changes to `runner.py`, no changes to any engine sub-module, no changes to the infra layer. This is explicitly stated as a design goal: *"swapping in the LLM agent later requires zero engine changes"* (`simulationDevelopmentApproach.md`, Key Points for Implementation).

---

## 2b. What `AgentContext` Gives the LLM

The `AgentContext` object passed to `decide()` at sub-step 4 of each tick contains everything the LLM would need to reason about a reorder decision:

- **Per-item stock state** — current `stock_on_hand`, `reorder_point`, `max_stock` for each item managed by this agent (`ItemState` objects).
- **Active disruptions** — which disruptions are currently live, their type (demand spike, lead time delay, transit loss), and magnitude.
- **Pending orders** — orders already placed but not yet arrived, so the agent doesn't double-order.
- **Budget** — remaining budget for the tick (from `ops_cost_accumulator`).
- **Tick number** — current position in the simulation, useful for pattern-aware reasoning.

This is a rich, structured observation. An LLM agent receiving this context can do things a rule-based agent cannot: reason about *why* stock is low (disruption vs. genuine demand growth), weigh cost trade-offs in natural language, or apply heuristics that depend on combinations of factors.

> **Reference**: `devlog.md`, Stage 4 Pre-Development Notes — the `AgentContext` build step is explicitly listed in the tick sequence as *"build AgentContext → agent.decide()"*, confirming all upstream sub-steps (disruptions, supply arrivals, demand draw, stock update) complete before the agent sees the world state.

---

## 2c. The Databricks-Native Implementation Path

Given the stack (Databricks, Delta tables, Unity Catalog), the most natural implementation of an LLM agent here would follow this pattern:

**Step 1 — Register tools in Unity Catalog.**

The engine already writes clean, structured data to `ops_warehouse_state`, `ops_pending_orders`, `ops_active_disruptions`, and `ops_cost_accumulator` each tick. These tables can back UC-registered functions:

```sql
-- Example UC tool
CREATE FUNCTION warehouse.ops.get_item_state(sim_id STRING, tick INT)
RETURNS TABLE ...
```

The LLM agent can call these tools to fetch live state rather than relying solely on the pre-packaged `AgentContext`. This matters for multi-step reasoning: the agent might check stock, then check pending orders, then check disruptions, each as a separate tool call.

**Step 2 — Serve the LLM via Databricks Model Serving.**

The `LLMAgent.decide()` method calls a Model Serving endpoint (e.g. a DBRX or Claude endpoint registered in the AI Gateway). The `AgentContext` is serialised and included in the prompt.

**Step 3 — Orchestrate with LangGraph (or a simple ReAct loop).**

If the decision logic is multi-step (the agent checks state, deliberates, calls a tool, reconsiders), LangGraph's stateful graph structure fits naturally. If the decision is a single LLM call with structured output, a plain PyFunc wrapper is sufficient.

**Step 4 — Log reasoning to MLflow.**

Since the engine already injects an `EventLogger` and writes to `event_log`, the LLM agent's reasoning trace (which tools it called, what the LLM's chain-of-thought was) can be emitted as additional event types (e.g. `AGENT_TOOL_CALL`, `AGENT_REASONING_STEP`) alongside the existing `REORDER_PLACED` and `HOLD_DECISION` events.

> **Reference**: `devlog.md`, Stage 4 Pre-Development Notes: *"Event logger is injected too. The EventLogger instance is passed into the runner at startup, same pattern"* — meaning the LLM agent can receive the logger and emit its own events without any runner changes.

---

## 2d. Evaluation: The Rule-Based Agent as Baseline

The deferred `RuleBasedAgent` (Stage 6, currently deferred in `devlog.md`) is specifically noted as *"a reusable, importable RuleBasedAgent... useful when you want a reproducible baseline to compare against the LLM agent later"* (`devlog.md`, Stage 7 Pre-Development Notes).

This sets up a natural A/B evaluation structure:

- Run the simulation with `RuleBasedAgent` → record `hist_cost_by_tick`, `hist_reorder_decisions`, unmet demand.
- Run the same simulation (same seed, same `sim_id` config) with `LLMAgent`.
- Compare outcomes using the existing `SimDashboard` visualisations (stock over time, cost breakdown, disruption overlay).
- Use Databricks MLflow Agent Evaluation to score the LLM agent's decision quality against the rule-based baseline as a ground truth.

The `sim_id` + seed reproducibility already baked into the engine (`devlog.md`, Stage 4 Pre-Development Notes: *"The single RNG instance created at startup is passed through runner.py... This is what guarantees reproducibility"*) makes this comparison clean — the only variable between runs is the agent.

---

## 2e. Where the Interesting Problems Are

Fitting an LLM agent into this system is architecturally straightforward (the slot exists), but there are genuine reasoning challenges:

- **Latency per tick** — an LLM call takes hundreds of milliseconds. In a finite 30-tick simulation this is fine; in an indefinite/cyclic run it matters. The agent interface is synchronous (`decide()` returns a list), so async LLM calls would require wrapping.
- **Structured output reliability** — `decide()` must return `list[ReorderDecision]`, not free text. The LLM needs to output valid JSON or a parseable format. Databricks' AI Gateway supports structured output schemas via OpenAI-compatible APIs.
- **Context window vs. tick history** — the `AgentContext` is per-tick. An LLM agent that wants to reason over the past N ticks would need to fetch from `hist_demand_actuals` or `hist_cost_by_tick` itself, either as a tool call or via a pre-built context enrichment layer.
- **Budget awareness** — the agent receives remaining budget but must reason about whether a large order now is worth foregoing future optionality. This is a multi-period optimisation problem that rule-based agents approximate with a fixed reorder quantity.

---

## 2f. Summary

| Aspect | Current state | LLM agent extension |
|---|---|---|
| Contract | `BaseAgent.decide()` defined and proven | New subclass, zero engine changes |
| Observation | `AgentContext` built each tick with full state | Can be enriched with UC tool calls |
| Output | `list[ReorderDecision]` | Same; structured output from LLM |
| Execution | Injected into runner at startup | Same injection pattern |
| Logging | `EventLogger` injected | Can emit additional reasoning events |
| Evaluation | `RuleBasedAgent` baseline (deferred) | MLflow Agent Evaluation comparison |
| Reproducibility | Seed-controlled RNG | Same seed → comparable runs |

The simulation was built as a controlled environment for exactly this purpose. The infra layer (stages 1–3, 5, 7) is blind to any agent implementation. The engine (stage 4) only knows `BaseAgent`. The LLM agent is the final plugin in a socket that has been waiting for it since stage 1.

---

*References in this document are to `simulationDevelopmentApproach.md` and `devlog.md` (attached files), and to Databricks Mosaic AI / MLflow documentation (Databricks docs.databricks.com, 2024–2025).*
