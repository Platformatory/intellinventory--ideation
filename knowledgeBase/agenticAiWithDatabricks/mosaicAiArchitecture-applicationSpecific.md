<h1>Databricks Mosaic AI - Agentic Architecture Reference</h1>

> **AI-Use Disclaimer**: This document was generated using Claude Sonnet 4.6:
>
> Chat link: [Confirming architectural context understanding](https://claude.ai/share/6d17ff81-3c0e-4bf6-a75a-033f92df1a79) (accessible to Platformatory members only)

---

> **Documents that serve as key context for this document**:
>
> Application-specific documents:
>
> - [`implementation/docs/simulationSpecs.md`](../../implementation/docs/simulationSpecs.md)
> - [`implementation/docs/simulationDevelopmentApproach.md`](../../implementation/docs/simulationDevelopmentApproach.md)
> - [Repo (**github.com**): `pranavg-platformatory/dbx-agentic-ai-project-sim-package`, File: `devlog.md`](https://github.com/pranavg-platformatory/dbx-agentic-ai-project-sim-package/blob/main/devlog.md)
>
> Conceptual exploration document: [`knowledgeBase/agenticAiWithDatabricks/mosaicAiArchitecture.md`](./mosaicAiArchitecture.md)

---

<h2>Application-Specific Annotation: Warehouse Reorder Agent</h2>

> **Document purpose**: This document annotates and extends the base Mosaic AI architecture reference from the perspective of a specific application: an LLM-based reorder agent that plugs into the `BaseAgent` contract of a warehouse reorder simulation, and is designed to extend naturally to a real-world warehouse or inventory management system. It should be read alongside the base reference document.
>
> **Agent boundary**: The agent under consideration is the `decide(context: AgentContext) -> list[ReorderDecision]` interface. The simulation engine, world setup, event logger, and visualisation layers are fixed infrastructure. This document addresses only the agent layer and its surrounding Databricks tooling.
>
> **Design principle**: Architectural choices are evaluated on two axes: (1) fitness for the simulation context as it currently stands, and (2) forward-compatibility with a real-world equivalent where data flows from live ERP/WMS systems rather than simulated Delta tables.
>
> **Claim notation**: Claims are annotated as **[verified]** where they are supported by official Databricks, MLflow, or LangFuse documentation, or **[inferred]** where they follow by reasonable extrapolation from known product behaviour. Inferred claims note the inference basis briefly.

---

**Contents**:

- [Reference Note](#reference-note)
- [1. Tool Relevance Assessment](#1-tool-relevance-assessment)
  - [1.1 Unity Catalog - Relevant](#11-unity-catalog---relevant)
  - [1.2 MLflow - Relevant (core)](#12-mlflow---relevant-core)
  - [1.3 Agent Bricks - Conditionally Relevant](#13-agent-bricks---conditionally-relevant)
  - [1.4 LangChain and LangGraph - Relevant (approach-dependent)](#14-langchain-and-langgraph---relevant-approach-dependent)
  - [1.5 LangFuse - Relevant (primary observability layer)](#15-langfuse---relevant-primary-observability-layer)
  - [1.6 Lakebase - Marginally Relevant in Simulation; More Relevant in Production](#16-lakebase---marginally-relevant-in-simulation-more-relevant-in-production)
  - [1.7 Model Serving Endpoints - Relevant (deployment target)](#17-model-serving-endpoints---relevant-deployment-target)
  - [1.8 Databricks Apps - Marginally Relevant](#18-databricks-apps---marginally-relevant)
  - [1.9 AI Playground - Relevant (early-stage prototyping only)](#19-ai-playground---relevant-early-stage-prototyping-only)
  - [1.10 AI Gateway - Relevant (production path)](#110-ai-gateway---relevant-production-path)
  - [1.11 Managed OAuth MCP Connectors - Not Relevant (simulation); Potentially Relevant (production)](#111-managed-oauth-mcp-connectors---not-relevant-simulation-potentially-relevant-production)
- [2. Component Shaping by Application Requirements](#2-component-shaping-by-application-requirements)
  - [2.1 Unity Catalog - Shaped by the Simulation's Tool Contract](#21-unity-catalog---shaped-by-the-simulations-tool-contract)
  - [2.2 MLflow - Shaped by the Need for Reproducible, Comparable Runs](#22-mlflow---shaped-by-the-need-for-reproducible-comparable-runs)
  - [2.3 Agent Bricks - Shaped by the Simplicity of the Decision Loop](#23-agent-bricks---shaped-by-the-simplicity-of-the-decision-loop)
  - [2.4 LangChain / LangGraph - Shaped by Whether the Agent Needs Internal State](#24-langchain--langgraph---shaped-by-whether-the-agent-needs-internal-state)
  - [2.5 LangFuse - Shaped by the Tick-Level Observability Requirement](#25-langfuse---shaped-by-the-tick-level-observability-requirement)
  - [2.6 Lakebase - Shaped by Agent Memory Requirements](#26-lakebase---shaped-by-agent-memory-requirements)
  - [2.7 Model Serving Endpoints - Shaped by the Agent Contract](#27-model-serving-endpoints---shaped-by-the-agent-contract)
  - [2.8 AI Gateway - Shaped by Cost and Safety Constraints](#28-ai-gateway---shaped-by-cost-and-safety-constraints)
- [3. Testing and Deployment Approaches](#3-testing-and-deployment-approaches)
  - [3.1 Testing the Agent in Isolation](#31-testing-the-agent-in-isolation)
  - [3.2 Integration Testing via the Simulation](#32-integration-testing-via-the-simulation)
  - [3.3 Deployment Approaches](#33-deployment-approaches)
- [4. Agent Evaluation](#4-agent-evaluation)
  - [4.1 What Evaluation Means for This Agent](#41-what-evaluation-means-for-this-agent)
  - [4.2 Outcome-Level Evaluation (Simulation Metrics)](#42-outcome-level-evaluation-simulation-metrics)
  - [4.3 Decision-Quality Evaluation (Per-Tick Reasoning)](#43-decision-quality-evaluation-per-tick-reasoning)
  - [4.4 Robustness Evaluation (Disruption Scenarios)](#44-robustness-evaluation-disruption-scenarios)
  - [4.5 Comparison Against Baselines](#45-comparison-against-baselines)
  - [4.6 MLflow as Evaluation Infrastructure](#46-mlflow-as-evaluation-infrastructure)
  - [4.7 Production-Readiness Evaluation (Forward-Looking)](#47-production-readiness-evaluation-forward-looking)
- [5. LangFuse Integration - Deep Dive](#5-langfuse-integration---deep-dive)
  - [5.1 What LangFuse Is and Is Not](#51-what-langfuse-is-and-is-not)
  - [5.2 Integration Points in the Simulation Architecture](#52-integration-points-in-the-simulation-architecture)
  - [5.3 Approach A - Thin SDK Integration at the Agent Boundary](#53-approach-a---thin-sdk-integration-at-the-agent-boundary)
  - [5.4 Approach B - Span-per-Subtool Tracing](#54-approach-b---span-per-subtool-tracing)
  - [5.5 Approach C - LangFuse as Evaluation Dataset Builder](#55-approach-c---langfuse-as-evaluation-dataset-builder)
  - [5.6 LangFuse in Development vs. Production](#56-langfuse-in-development-vs-production)
  - [5.7 LangFuse + MLflow: Complementary, Not Redundant](#57-langfuse--mlflow-complementary-not-redundant)
  - [5.8 Forward Compatibility: LangFuse in the Real-World Extension](#58-forward-compatibility-langfuse-in-the-real-world-extension)
- [6. Limitations of Tick-by-Tick LLM Reasoning and Scalable Alternatives](#6-limitations-of-tick-by-tick-llm-reasoning-and-scalable-alternatives)
  - [6.1 What Tick-by-Tick LLM Reasoning Means in This Context](#61-what-tick-by-tick-llm-reasoning-means-in-this-context)
  - [6.2 Core Limitations](#62-core-limitations)
  - [6.3 Scalable Alternative Approaches](#63-scalable-alternative-approaches)
    - [Alternative A - Sparse LLM Calling (Trigger-Based Reasoning)](#alternative-a---sparse-llm-calling-trigger-based-reasoning)
    - [Alternative B - Fine-Tuned Small Model](#alternative-b---fine-tuned-small-model)
    - [Alternative C - LLM for Policy Generation, Not Decision Execution](#alternative-c---llm-for-policy-generation-not-decision-execution)
    - [Alternative D - Reinforcement Learning from Simulation](#alternative-d---reinforcement-learning-from-simulation)
    - [Alternative E - Hybrid: LLM + Structured Optimiser](#alternative-e---hybrid-llm--structured-optimiser)
  - [6.4 Cross-Reference: Where These Alternatives Appear in Earlier Sections](#64-cross-reference-where-these-alternatives-appear-in-earlier-sections)

---

# Reference Note

Primary references from the base architecture document are carried forward:

- **[R1]** Databricks Blog: *Agent Bricks - Governed Enterprise Agent Platform* - https://www.databricks.com/blog/agent-bricks-governed-enterprise-agent-platform
- **[R2]** Medium (AI on Databricks): *Building an Investment Assistant with the Databricks Mosaic AI Agent Framework* - https://medium.com/@AI-on-Databricks/building-an-investment-assistant-with-the-databricks-mosaic-ai-agent-framework-d2ff276a61d2
- **[R3]** Databricks Unity Catalog documentation - https://docs.databricks.com/en/data-governance/unity-catalog/index.html
- **[R4]** MLflow documentation - https://mlflow.org/docs/latest/index.html
- **[R5]** MLflow `mlflow.evaluate()` API docs - https://mlflow.org/docs/latest/python_api/mlflow.html#mlflow.evaluate
- **[R6]** Databricks Model Serving documentation - https://docs.databricks.com/en/machine-learning/model-serving/index.html
- **[R7]** Databricks AI Playground documentation - https://docs.databricks.com/en/large-language-models/ai-playground.html
- **[R8]** LangGraph documentation - https://langchain-ai.github.io/langgraph/
- **[R9]** LangFuse documentation - https://langfuse.com/docs
- **[R10]** Databricks Lakebase announcement - https://www.databricks.com/blog/introducing-lakebase
- **[R11]** Databricks AI Gateway documentation - https://docs.databricks.com/en/ai-gateway/index.html
- **[R12]** Databricks MCP Connectors / OAuth documentation - https://docs.databricks.com/en/generative-ai/agent-framework/mcp.html

Additional references introduced in this document:

- **[R13]** LangFuse Python SDK - https://langfuse.com/docs/sdk/python
- **[R14]** LangFuse Tracing concepts (traces, spans, generations) - https://langfuse.com/docs/tracing
- **[R15]** LangFuse Datasets and Evaluation - https://langfuse.com/docs/datasets/overview
- **[R16]** LangFuse Scores API - https://langfuse.com/docs/scores/overview
- **[R17]** MLflow LangChain integration - https://mlflow.org/docs/latest/llms/langchain/index.html

---

# 1. Tool Relevance Assessment

This section evaluates each tool from the base architecture document against the requirements of the warehouse reorder agent. Relevance is assessed for two contexts: the current simulation setting and the projected real-world extension.

---

## 1.1 Unity Catalog - Relevant

**Verdict: Relevant in both simulation and production contexts.**

In the simulation, the agent is already interacting with a set of well-defined data tables (`ops_warehouse_state`, `hist_demand_actuals`, `ops_pending_orders`, etc.). These tables are natural candidates for registration in Unity Catalog, providing lineage, access control, and discoverability from the start. **[verified - [R3]]**

More directly, the agent's callable tools - the read operations it uses to query stock state, demand history, and cost accumulator - can be registered as Unity Catalog functions. This is the pattern UC is designed for in agentic contexts: tools the agent invokes are governed artifacts, not ad-hoc code. **[verified - [R3]]**

For the real-world extension, UC becomes even more central: live ERP/WMS data sources, supplier APIs, and reorder approval workflows would all be governed via UC, maintaining the same lineage and access model as the simulation. **[inferred - by extension of the UC tool-registration pattern to external data sources; no direct Databricks documentation was fetched for this specific claim]**

---

## 1.2 MLflow - Relevant (core)

**Verdict: Relevant and effectively mandatory for experiment tracking and agent versioning.**

Every simulation run produces a natural MLflow experiment: the agent version, the world configuration (items, suppliers, disruptions), the random seed, and the outcome metrics (total cost, stockout count, reorder count) are all loggable as MLflow run parameters and metrics. This gives a structured, queryable record of every agent iteration across every sim configuration. **[verified - [R4]]**

The `mlflow.evaluate()` API is applicable here as well, provided evaluation datasets are constructed from simulation output - specifically from `hist_reorder_decisions` and `hist_cost_by_tick`. This is discussed further in Section 4. **[verified - [R5]]**

MLflow Tracing, which captures the full span of an LLM invocation including intermediate tool calls, is directly useful for debugging per-tick agent reasoning within the simulation. **[verified - [R4]]**

For the real-world extension, the MLflow model registry becomes the deployment artefact store: the LLM agent, packaged as an MLflow model, is what gets promoted to a serving endpoint. **[verified - [R4], [R6]]**

---

## 1.3 Agent Bricks - Conditionally Relevant

**Verdict: Potentially useful as a scaffolding starting point, but likely not the primary construction layer for this application.**

Agent Bricks provides opinionated, high-level agent scaffolding that reduces boilerplate. **[inferred - from [R1], which could not be fetched for direct text verification]**

For this application, the agent contract is already defined and fixed: `BaseAgent.decide(context: AgentContext) -> list[ReorderDecision]`. Any scaffolding must conform to this interface. If Agent Bricks imposes its own execution model or tool-calling loop that conflicts with the simulation's tick-synchronous call pattern, it introduces friction rather than reducing it.

Two possible approaches:

- **Use Agent Bricks internally within the `LLMAgent` implementation**, treating it as a tool-composition helper behind the `decide` method. The simulation engine remains unaware of Agent Bricks.
- **Bypass Agent Bricks entirely** and build the agent as a direct LangGraph graph or plain LLM call, with the `decide` method as the entry point. This is lower-level but fully controllable.

The second approach is more conservative and more aligned with the existing architecture. The first approach becomes attractive if Agent Bricks offers pre-built patterns (context assembly, multi-tool orchestration, structured output parsing) that save meaningful development time. **[inferred - the specific feature set of Agent Bricks relevant here is drawn from training knowledge of [R1] and not directly verifiable]**

For the real-world extension, Agent Bricks becomes more relevant if the agent gains complexity (multi-step approval workflows, supplier negotiation sub-agents, escalation paths). A simple reorder decision agent does not require this level of scaffolding.

---

## 1.4 LangChain and LangGraph - Relevant (approach-dependent)

**Verdict: LangGraph is the more appropriate choice if the agent requires internal state or multi-step reasoning; LangChain alone is sufficient for simpler single-call patterns.**

The agent's `decide` method is called once per tick with a fully assembled `AgentContext`. Within that single call, the agent may:

1. Make a single LLM call and return decisions directly (simplest case).
2. Make an LLM call, invoke one or more read-only tools against the simulation tables, then make a final decision call (tool-augmented reasoning).
3. Run an internal multi-step loop - e.g. reason about each item type sequentially, consult a sub-agent for supplier selection, then consolidate - before returning decisions (graph-based orchestration).

LangChain is sufficient for case (1) and arguably for case (2). LangGraph is better suited to case (3), where the agent's internal reasoning is a stateful graph with conditional branching and possible retries. **[verified - [R8]]**

Databricks supports packaging LangGraph agents as MLflow models for deployment, which is directly applicable here. **[inferred - consistent with [R2] and general Databricks documentation; direct text not fetchable]**

For the real-world extension, case (3) becomes far more likely as the agent gains tools that interact with live ERP systems, require approval gates, or must handle supplier negotiation sub-workflows. LangGraph's stateful graph model is well-suited to this. **[inferred - by extension of LangGraph's documented stateful graph capabilities to this domain]**

---

## 1.5 LangFuse - Relevant (primary observability layer)

**Verdict: Highly relevant - the most directly applicable observability tool for per-tick LLM reasoning in this application. Treated as a primary tool, not an afterthought.**

LangFuse captures LLM call traces, token counts, latency, prompt versions, and structured scores at the granularity of individual LLM invocations. **[verified - [R9]]** In the simulation context, this maps naturally to per-tick traces: each call to `agent.decide()` is one trace, each internal LLM call or tool invocation within it is a span.

LangFuse is covered in full depth in Section 5.

---

## 1.6 Lakebase - Marginally Relevant in Simulation; More Relevant in Production

**Verdict: Not required for the simulation (which already has its own state tables); relevant in production for agent session memory and context persistence.**

The simulation already manages all state in its own `ops_*` and `hist_*` tables. There is no need for an additional agent-specific session store within the simulation - the `AgentContext` is assembled fresh each tick from these existing tables by the runner.

However, if the agent is designed with an explicit memory component - for example, maintaining a cross-tick summary of recent disruption patterns or supplier reliability signals - then Lakebase (or an equivalent managed Postgres store) becomes the natural home for that memory. **[inferred - Lakebase as a session/memory store is an inferred use case, not a documented one; see base reference document]**

For the real-world extension, where the agent runs as a persistent service rather than a tick-synchronous call within a simulation loop, agent session memory becomes a first-class concern. Lakebase or a similar operational database would store cross-invocation context, conversation history (if the agent is user-facing), and running cost/risk state. **[inferred - natural extension of the Lakebase Postgres-compatible interface to operational agent memory patterns]**

---

## 1.7 Model Serving Endpoints - Relevant (deployment target)

**Verdict: Relevant as the deployment target for the LLM agent when it moves beyond notebook execution.**

In the simulation's current form, the agent runs in-process - the `SimRunner` instantiates the agent and calls `decide()` directly. No serving endpoint is needed.

As soon as the agent is promoted to a standalone service - either for production-like simulation runs, for comparison testing across multiple agent versions simultaneously, or for real-world deployment - Model Serving Endpoints become the natural deployment target. The agent, packaged as an MLflow model, is deployed here and called via HTTP. **[verified - [R6]]**

Two deployment paths are possible:

- **Endpoint-per-agent-version**: Each candidate agent version gets its own endpoint; the simulation runner is pointed at a configurable endpoint URL.
- **Single endpoint with version routing via AI Gateway**: One endpoint handles all agent versions; the gateway routes to the right model based on a header or parameter. Relevant when running A/B evaluations across agent versions.

For the real-world extension, the serving endpoint is the production agent API, called by the ERP/WMS system when a reorder decision is required. **[inferred - by direct extension of the Model Serving Endpoint REST API pattern]**

---

## 1.8 Databricks Apps - Marginally Relevant

**Verdict: Not central to the agent layer, but useful as a lightweight evaluation or monitoring UI.**

The simulation already has a visualisation layer (`viz/dashboard.py`). Databricks Apps could host a more interactive version of this dashboard - allowing a human reviewer to inspect agent decisions, trace reasoning, and compare runs across agent versions without leaving the Databricks environment. **[inferred - consistent with Databricks Apps' described purpose as a lightweight UI host; not directly cited in [R1] or [R2]]**

For the real-world extension, Databricks Apps could serve as an operator dashboard: a human-in-the-loop interface where warehouse managers can review agent recommendations before they are actioned. This is more relevant if the production system includes a human approval step. **[inferred]**

---

## 1.9 AI Playground - Relevant (early-stage prototyping only)

**Verdict: Relevant during prompt engineering and model selection; not part of the ongoing development loop.**

Before writing the `LLMAgent` implementation, the AI Playground is useful for iterating on the system prompt, testing how different foundation models respond to the `AgentContext` payload, and exploring structured output formats for `ReorderDecision`. **[verified - [R7]]**

Once the prompt is stable enough to code against, the Playground recedes. The primary development loop thereafter is: edit prompt/agent code → run simulation → inspect LangFuse traces / MLflow metrics → repeat.

---

## 1.10 AI Gateway - Relevant (production path)

**Verdict: Not needed in the simulation's current in-process form; important on the path to production.**

In the simulation, there is no external LLM API call routing to control - the LLM is called directly by the agent. AI Gateway becomes relevant when:

- The agent is deployed as a Model Serving Endpoint and external callers (the simulation runner, or eventually an ERP system) hit it via HTTP.
- Multiple LLM backends are in use (e.g. Claude for reasoning, a smaller model for structured output parsing) and traffic needs routing.
- Token budgets, rate limits, or guardrails need enforcing per simulation run or per deployment environment. **[inferred - consistent with AI Gateway's documented capabilities in [R11]]**

For the real-world extension, AI Gateway is a standard production component: it sits in front of the agent endpoint, enforces cost controls, logs every inference call for compliance, and provides fallback routing if the primary model is unavailable. **[verified - [R11]]**

---

## 1.11 Managed OAuth MCP Connectors - Not Relevant (simulation); Potentially Relevant (production)

**Verdict: Not applicable in the simulation; relevant in production if the agent needs to interact with external supplier or ERP APIs.**

In the simulation, all tools the agent calls are internal - they read from Delta tables that are already inside the Databricks environment. No external service connectivity is required.

In the real-world extension, the agent may need to place actual purchase orders via a supplier API, query live inventory feeds from an ERP system, or look up pricing from an external catalogue. MCP Connectors provide a governed, OAuth-secured way to expose these external tools to the agent without embedding credentials in agent code. **[inferred - consistent with [R12]; the specific connector integration model for ERP systems is not documented in a source that was directly fetchable]**

The architectural value of MCP Connectors here is forward-compatibility: if the simulation's tool contract (UC Functions reading Delta tables) is designed to be structurally equivalent to the production tool contract (MCP Connectors calling external APIs), then swapping the underlying tool implementation when moving to production requires no agent code changes. This is a strong design argument for aligning the tool abstraction layer early. **[inferred - by application of the MCP protocol's tool-abstraction principle to this specific migration path]**

---

# 2. Component Shaping by Application Requirements

This section addresses how each relevant component from the base architecture is shaped - constrained, extended, or configured differently - by the specific requirements of the warehouse reorder agent.

---

## 2.1 Unity Catalog - Shaped by the Simulation's Tool Contract

The simulation exposes the agent's world through a typed `AgentContext` object assembled by the runner. When the agent is promoted to an LLM that calls tools, those tools correspond directly to the read operations already implicit in `AgentContext` assembly:

- `get_stock_state(sim_id, tick, item_id)` → reads `ops_warehouse_state`
- `get_pending_orders(sim_id, item_id)` → reads `ops_pending_orders`
- `get_demand_history(sim_id, item_id, window)` → reads `hist_demand_actuals`
- `get_cost_state(sim_id, item_id)` → reads `ops_cost_accumulator`
- `get_active_disruptions(sim_id, tick)` → reads `ops_active_disruptions`

These can each be registered as Unity Catalog SQL or Python functions, making them governed, versioned, and discoverable. **[verified - [R3]]**

This matters for the real-world extension: the same tool names and signatures remain; only the underlying data source changes (from simulated Delta tables to live ERP feeds). The agent code does not need to change. **[inferred - by application of the UC function abstraction principle]**

One approach to consider: pre-assembling the `AgentContext` as a single UC function (`get_agent_context(sim_id, tick)`) rather than exposing five separate tools. This reduces LLM tool-call overhead but gives the agent less control over what it queries. The choice depends on whether the agent is expected to reason selectively about which data to inspect or whether it always consumes the full context. Both approaches are valid; the multi-tool approach is more faithful to the agentic pattern. **[inferred]**

---

## 2.2 MLflow - Shaped by the Need for Reproducible, Comparable Runs

The simulation's reproducibility constraint (same seed, same config → same output) maps directly onto MLflow's experiment tracking model. Each simulation run should be one MLflow run, with the following logged:

**Parameters** (fixed inputs):
- `sim_id`, `random_seed`, `num_ticks`, `tick_unit`
- `agent_version` (the agent implementation being tested)
- `agent_model` (the foundation LLM used, e.g. `claude-sonnet-4-20250514`)
- `agent_prompt_version` (a hash or tag of the system prompt)
- Item configuration summary (number of items, disruption count)

**Metrics** (run outcomes):
- `total_cost`, `total_stockout_ticks`, `total_reorder_count`
- `mean_stock_on_hand`, `stockout_rate` (per item)
- `mean_lead_time_actual`, `total_transit_loss_units`
- `llm_call_count`, `total_tokens_used`, `mean_latency_ms_per_tick`

**Artefacts**:
- The agent's system prompt (as a text file)
- The full `SimWorld` config (as a JSON dump)
- LangFuse trace export for the run (if available)

This structure makes it straightforward to compare agent versions across identical sim configurations, or to compare the same agent version across different world configurations. **[verified - [R4]; the specific parameter/metric taxonomy is inferred from the simulation's data model]**

MLflow Tracing should be enabled alongside LangFuse (not instead of it). MLflow Tracing captures the Databricks-native span tree; LangFuse captures the richer LLM-specific observation data. They complement each other. **[inferred - by combination of [R4] and [R9]; no single source documents using both simultaneously in this configuration]**

---

## 2.3 Agent Bricks - Shaped by the Simplicity of the Decision Loop

For this application, the agent's decision loop has a fixed, known structure: receive context, reason, return a list of reorder decisions. This is simpler than most Agent Bricks target use cases, which involve open-ended multi-turn interactions or complex tool orchestration with human feedback loops. **[inferred - from the general characterisation of Agent Bricks in [R1]]**

If Agent Bricks is used at all, its most applicable contribution is **structured output enforcement** - ensuring the agent's response parses correctly into a `list[ReorderDecision]` - and **prompt management**, handling system prompt versioning. If these can be achieved more simply with a direct LangChain structured output parser and MLflow prompt logging, Agent Bricks adds little. **[inferred]**

A reasonable decision rule: start without Agent Bricks; introduce it if and when the agent's internal complexity grows to a point where its scaffolding becomes a net benefit rather than a constraint. **[inferred]**

---

## 2.4 LangChain / LangGraph - Shaped by Whether the Agent Needs Internal State

Two architecturally distinct agent designs are possible within the `BaseAgent` contract:

**Design A - Stateless single-call agent:**
The `decide` method makes one LLM call, passing the full `AgentContext` as context, and parses the response into `list[ReorderDecision]`. No internal tool calls; no multi-step reasoning. Fast, cheap, interpretable. Appropriate for a first implementation.

```
decide(context) →
    prompt = build_prompt(context)
    response = llm.invoke(prompt)
    decisions = parse_decisions(response)
    return decisions
```

**Design B - Tool-augmented agent (LangChain):**
The `decide` method invokes an LLM with tool definitions (the UC functions from Section 2.1). The LLM decides which tools to call, receives their outputs, and produces decisions. More flexible; the LLM can request only the data it needs rather than consuming the pre-assembled context wholesale.

```
decide(context) →
    agent = create_tool_calling_agent(llm, tools=[...], prompt=system_prompt)
    result = agent.invoke({"input": summarise(context)})
    decisions = parse_decisions(result)
    return decisions
```

**Design C - Multi-step graph agent (LangGraph):**
The `decide` method runs a LangGraph graph. Nodes represent reasoning steps (e.g. `assess_stock_risk`, `evaluate_disruption_impact`, `compute_reorder_quantities`, `validate_budget`), edges represent conditional transitions. The graph produces decisions after traversing a deterministic or conditional path. Slower; most interpretable when the reasoning steps are meaningful and inspectable individually.

```
decide(context) →
    graph = build_reorder_graph()
    state = {"context": context, "decisions": []}
    result = graph.invoke(state)
    return result["decisions"]
```

**[verified - LangGraph's stateful graph model supports this structure [R8]; the specific node taxonomy here is inferred from the simulation's decision requirements]**

All three designs satisfy the `BaseAgent` contract. The simulation runner is unaware of which design is used. Design A is the right starting point; the others are progressive elaborations justified by observing where Design A fails or produces poor reasoning.

For the real-world extension, Design C becomes more appropriate as the agent gains tools that interact with live systems and must handle asynchronous responses, partial failures, or approval gates. **[inferred]**

---

## 2.5 LangFuse - Shaped by the Tick-Level Observability Requirement

The simulation's tick structure creates a natural trace hierarchy: sim run → tick → item → LLM call. LangFuse's trace/span/generation model maps onto this cleanly. This is covered in full depth in Section 5.

The key shaping constraint here is that LangFuse must operate within the simulation's synchronous tick loop without introducing blocking I/O that disrupts tick timing. This is achievable by configuring the LangFuse SDK to flush asynchronously. **[verified - [R13]; the async flush configuration is documented in the LangFuse Python SDK]**

---

## 2.6 Lakebase - Shaped by Agent Memory Requirements

Three agent memory models are possible, each with different Lakebase implications:

**No persistent memory:** The agent uses only what is in the current `AgentContext`. Stateless across ticks. No Lakebase needed.

**In-process accumulating memory:** The agent maintains a Python object (e.g. a dict of recent decisions per item) that persists across `decide()` calls within a single simulation run. The runner holds the agent instance across ticks, so this is already possible. No external store needed.

**Cross-run persistent memory:** The agent retains information across simulation runs - for example, a learned prior about which disruption patterns tend to be followed by stockouts. This requires an external store. Lakebase is appropriate here. **[inferred - Lakebase as a persistent agent memory store is consistent with its Postgres-compatible interface and is an inferred use case]**

For the simulation, option 2 is sufficient and simpler. Option 3 is worth designing for (i.e. not designing against) so the real-world extension does not require architectural rework. **[inferred]**

---

## 2.7 Model Serving Endpoints - Shaped by the Agent Contract

When the LLM agent is deployed as a Model Serving Endpoint, the endpoint's request/response schema must correspond to the `AgentContext` → `list[ReorderDecision]` contract. This is a clean mapping: the endpoint receives a serialised `AgentContext` (JSON) and returns a serialised `list[ReorderDecision]` (JSON).

The simulation runner's agent invocation changes from:

```python
decisions = agent.decide(context)
```

to:

```python
response = requests.post(endpoint_url, json=context.model_dump())
decisions = [ReorderDecision(**d) for d in response.json()]
```

This is a drop-in swap at the runner level, provided the endpoint is designed to speak the same contract. **[inferred - by application of the Model Serving Endpoint REST API pattern to the existing BaseAgent contract; no direct documentation was fetched for this specific mapping]**

Two deployment configurations are relevant:

- **Development/evaluation**: Endpoint backed by the LLM agent, called by the simulation runner. Allows comparing multiple agent versions by changing the endpoint URL only.
- **Production**: Endpoint backed by the same agent, called by the ERP/WMS system. The simulation's test harness becomes the validation layer before each production deployment. **[inferred]**

---

## 2.8 AI Gateway - Shaped by Cost and Safety Constraints

In the simulation context, the primary AI Gateway concern is **token budget enforcement**. Each simulation run has a configurable `budget_limit`; there is an analogous concern at the LLM layer: a run should not consume more tokens than expected due to runaway prompts or unexpectedly verbose responses. AI Gateway can enforce per-run token limits and alert before exhaustion. **[inferred - consistent with AI Gateway's rate limiting and cost tracking capabilities in [R11]]**

A secondary concern is **guardrails**: for a reorder agent, guardrails would block the agent from placing orders that exceed `max_order_qty` or exhaust the `budget_limit` at the LLM output level, as a redundant safety check on top of the engine's own constraint enforcement. **[inferred]**

For the real-world extension, AI Gateway becomes more important: guardrails prevent the agent from generating reorder recommendations that are nonsensical (e.g. negative quantities, unknown item IDs), and rate limits prevent a runaway agent from flooding a supplier API with orders. **[inferred - consistent with [R11]]**

---

# 3. Testing and Deployment Approaches

---

## 3.1 Testing the Agent in Isolation

Before the agent is run inside the simulation loop, it should be testable in isolation against fixed `AgentContext` inputs. This is directly enabled by the existing `BaseAgent` contract.

**Approach A - Unit tests with hand-crafted contexts:**
Construct `AgentContext` objects representing known scenarios (stockout imminent, adequate stock, active disruption, budget near limit) and assert that the agent's decisions are reasonable. "Reasonable" here can mean: non-negative order quantities, quantities within `[min_order_qty, max_order_qty]`, no order placed when budget is exhausted. These are deterministic assertions that do not require an LLM. **[inferred - standard practice for testing LLM agents against typed input contracts]**

For the LLM agent specifically, a subset of these tests will be non-deterministic (the LLM may vary its decisions). The recommended approach is to set LLM temperature to 0 for testing and assert structural validity (correct types, valid ranges) rather than specific decision values. **[inferred]**

**Approach B - Golden-set evaluation:**
Maintain a set of (`AgentContext`, expected decision) pairs representing known-good decisions, generated from the `RuleBasedAgent` or from human review. Run the LLM agent against these and score agreement. This is the basis for MLflow evaluation datasets. Discussed further in Section 4. **[inferred - by application of the golden-set evaluation pattern to this agent type]**

**Approach C - Adversarial context injection:**
Construct `AgentContext` objects that are deliberately adversarial - extreme disruption magnitudes, near-zero budget, all items simultaneously below reorder point - and verify the agent does not crash, does not produce invalid decisions, and produces reasonable fallback behaviour. **[inferred]**

---

## 3.2 Integration Testing via the Simulation

The simulation itself is the primary integration test harness. A complete simulation run with the LLM agent is a full integration test: it exercises the agent contract, the tool calls, the state writes, and the event log in a controlled, reproducible environment.

**Configuration for integration testing:**
- Short runs (10-30 ticks) with known seeds for fast feedback.
- Known disruption schedules (pre-scheduled, not stochastic) for deterministic evaluation.
- Both `HoldAgent` and `ReorderAgent` stub runs as baselines (already produced in Stage 4).

**What to assert at integration test completion:**
- No negative stock values anywhere in `ops_warehouse_state`.
- `SIM_ENDED` event present in `event_log`.
- Total cost in `ops_cost_accumulator` matches sum of `hist_cost_by_tick`.
- All decisions in `hist_reorder_decisions` have order quantities within `[min_order_qty, max_order_qty]`.
- LangFuse traces present for every tick in which the agent was called.

**[inferred - by direct extension of the Stage 7 integration assertions from the devlog to the LLM agent context]**

---

## 3.3 Deployment Approaches

Three deployment configurations exist, representing a natural progression:

**Configuration 1 - In-process (current simulation model):**
The LLM agent is instantiated directly in the simulation runner, same as the stub agents. The LLM is called via an API client within the `decide()` method. No serving endpoint; no gateway. Appropriate for development and evaluation. This configuration is the starting point.

**Configuration 2 - Agent as a served endpoint, simulation as caller:**
The LLM agent is packaged as an MLflow model and deployed to a Model Serving Endpoint. The simulation runner is modified to call the endpoint via HTTP rather than invoking the agent directly. AI Gateway sits in front of the endpoint. This configuration enables: running multiple agent version evaluations in parallel, enforcing token budgets across runs, and beginning to treat the agent as a deployable artefact. **[inferred - by combination of [R4], [R6], [R11]]**

**Configuration 3 - Agent as a production service, ERP as caller:**
The same endpoint from Configuration 2 receives calls from a real ERP/WMS system. The simulation becomes the pre-production test harness: every agent version passes a simulation-based evaluation suite before being promoted to the production endpoint. **[inferred - this is the forward-compatible production path; not directly documented in any single reference]**

The transition from Configuration 1 to 2 is the most architecturally significant step. Configuration 2 to 3 is primarily an operational change (caller identity, authentication, SLA requirements), not an agent architecture change. **[inferred]**

---

# 4. Agent Evaluation

---

## 4.1 What Evaluation Means for This Agent

Evaluating a reorder agent is unusually tractable compared to many LLM agent evaluation problems, because the simulation provides a **ground truth signal**: total cost, stockout count, and reorder count are objective outcomes that can be computed precisely and compared across agent versions and baselines. This is a significant advantage. **[inferred - the simulation's closed-loop cost model makes objective outcome evaluation possible in a way that open-ended LLM tasks do not]**

Evaluation operates at two levels:

1. **Outcome-level evaluation**: Did the agent produce better simulation outcomes than the baseline?
2. **Decision-quality evaluation**: Were the agent's individual decisions reasonable given the context it received?

Both levels are necessary. An agent can produce good outcomes by luck (e.g. a passive hold-all strategy that happens to work well for the configured demand pattern) without making good decisions. Conversely, an agent can make locally reasonable decisions that compound into poor outcomes. **[inferred]**

---

## 4.2 Outcome-Level Evaluation (Simulation Metrics)

The primary outcome metrics are already computed by the simulation:

| Metric | Source | Interpretation |
|---|---|---|
| `total_cost` | `ops_cost_accumulator` | Primary objective; lower is better |
| `stockout_rate` | `hist_demand_actuals` | Fraction of ticks with unmet demand; lower is better |
| `total_reorder_count` | `hist_reorder_decisions` | Proxy for ordering frequency; contextual |
| `mean_stock_on_hand` | `ops_warehouse_state` | Proxy for overstock risk; contextual |
| `total_transit_loss_units` | `hist_supply_arrivals` | Disruption exposure; contextual |
| `total_tokens_used` | LangFuse / MLflow | Operational cost of the agent itself |

These metrics should be logged to MLflow for every evaluation run, enabling comparison across agent versions and world configurations. **[verified - [R4]]**

A composite score (e.g. a weighted combination of `total_cost` and `stockout_rate`) is useful for ranking agent versions, but the weighting is a business judgement that should be made explicit and configurable. **[inferred]**

---

## 4.3 Decision-Quality Evaluation (Per-Tick Reasoning)

For each tick where the agent made a decision, the `hist_reorder_decisions` table stores `agent_reasoning` (free-text or structured reasoning from the LLM). This reasoning can be evaluated in two ways:

**Approach A - LLM-as-judge:**
A separate LLM evaluator is given the `AgentContext` and the agent's reasoning, and asked to score whether the reasoning is coherent, correctly references the available data, and produces a defensible decision. MLflow's `mlflow.evaluate()` with a custom LLM judge metric supports this pattern. **[verified - [R5]; the LLM-as-judge pattern is documented in MLflow evaluation; LangFuse also supports this via its Scores API [R16]]**

**Approach B - Rule-based reasoning checks:**
Simpler assertions on the reasoning text: does it mention the correct item ID? Does it reference the current stock level? Does it cite the active disruption if one is present? These are weak signals but fast to compute and do not require an additional LLM call. **[inferred]**

**Approach C - Counterfactual evaluation:**
For ticks where the agent chose to hold, evaluate what would have happened if it had reordered (and vice versa). This requires running parallel simulation branches, which is computationally expensive but highly informative. Practical only for short evaluation windows. **[inferred]**

---

## 4.4 Robustness Evaluation (Disruption Scenarios)

The simulation's disruption framework makes it straightforward to construct evaluation scenarios of graduated difficulty:

- **Baseline scenario**: No disruptions, stable demand pattern. Tests basic reorder competence.
- **Single disruption scenario**: One demand spike or transit delay. Tests disruption awareness.
- **Compounding disruption scenario**: Simultaneous demand spike + transit delay on the same item. Tests multi-factor reasoning.
- **Budget-constrained scenario**: `budget_limit` set low enough to force prioritisation across item types. Tests resource allocation under constraint.
- **Stochastic scenario**: Disruptions are stochastic; the agent cannot predict them. Tests expected-value reasoning.

Each scenario is a separate MLflow experiment. The same agent version run across all five scenarios reveals where it is robust and where it degrades. **[inferred - by direct application of MLflow's multi-experiment comparison capability to this scenario taxonomy]**

---

## 4.5 Comparison Against Baselines

At minimum, three baselines should be maintained for comparison:

| Baseline | Implementation | Purpose |
|---|---|---|
| `HoldAgent` | Always returns `hold` decisions | Lower bound; worst realistic outcome |
| `ReorderAgent` | Always reorders to `max_order_qty` | Upper bound on stock; cost baseline |
| `RuleBasedAgent` | Reorders when `stock_on_hand < reorder_point` | Industry-standard heuristic |

The LLM agent is evaluated not against an absolute standard but against these baselines. Improvement over `RuleBasedAgent` is the most meaningful signal. **[inferred - by application of standard ML baseline comparison practice to this agent evaluation context]**

---

## 4.6 MLflow as Evaluation Infrastructure

MLflow's `mlflow.evaluate()` API, combined with a custom evaluation dataset, is the most direct way to structure formal evaluation:

1. **Build the evaluation dataset**: Sample `(AgentContext, decision, outcome)` tuples from historical simulation runs. Store as an MLflow dataset artefact.
2. **Define metrics**: Total cost delta vs. baseline, stockout rate, reasoning coherence score (from LLM judge).
3. **Run `mlflow.evaluate()`**: Produces a structured evaluation report that is stored in the MLflow run and comparable across agent versions. **[verified - [R5]]**

LangFuse complements this by providing the per-trace data that feeds into the evaluation dataset. Traces captured in LangFuse can be exported and used directly as evaluation inputs. **[inferred - by combination of LangFuse's dataset export capability [R15] and MLflow's evaluation dataset import]**

---

## 4.7 Production-Readiness Evaluation (Forward-Looking)

Before promoting an agent version to the production endpoint (Configuration 3 from Section 3.3), a production-readiness gate should be defined. Candidate criteria:

- Simulation outcome score (composite metric) exceeds `RuleBasedAgent` by at least X% across all five scenario types.
- No invalid decisions (negative quantities, budget violations) in any evaluation run.
- Mean latency per tick decision < Y ms (where Y is the ERP system's SLA for reorder response time).
- Total token cost per run within a defined budget.
- LangFuse reasoning scores above a defined threshold on the LLM-as-judge evaluation.

This gate is enforced as part of the deployment pipeline (Configuration 2 → 3). **[inferred - by application of standard ML model promotion criteria to this agent deployment context]**

---

# 5. LangFuse Integration - Deep Dive

---

## 5.1 What LangFuse Is and Is Not

LangFuse is an open-source LLM observability platform. It is **not** a Databricks product and has no Databricks-specific integration beyond standard API access. It captures traces at the LLM invocation level, storing the prompt, response, token counts, latency, and any custom scores or metadata attached by the application. It provides a UI for browsing traces, building evaluation datasets from trace history, and running evaluations with custom scoring functions or LLM judges. **[verified - [R9], [R13], [R14]]**

The key concepts:

- **Trace**: One logical unit of work - in this application, one call to `agent.decide()`.
- **Span**: A sub-step within a trace - an individual tool call or processing step.
- **Generation**: A specific LLM call within a trace, with prompt, response, and token metadata.
- **Score**: A numeric or categorical evaluation attached to a trace or generation - either computed automatically or provided by a human reviewer or LLM judge. **[verified - [R14], [R16]]**

---

## 5.2 Integration Points in the Simulation Architecture

LangFuse integrates at the agent boundary, not at the engine level. The engine knows nothing about LangFuse; it calls `agent.decide()` and receives decisions. LangFuse instrumentation lives entirely within the `LLMAgent` implementation.

The natural trace hierarchy for this application:

```
Trace: agent.decide(tick=T, sim_id=S)
│
├── Span: context_assembly          (building the prompt from AgentContext)
│
├── Span: tool_call/get_stock       (if Design B or C - UC function call)
├── Span: tool_call/get_disruptions (if Design B or C)
│
├── Generation: llm_call            (the LLM invocation - prompt + response)
│   ├── model: claude-sonnet-4-...
│   ├── prompt_tokens: N
│   ├── completion_tokens: M
│   └── latency_ms: L
│
└── Span: decision_parsing          (parsing LLM response into ReorderDecision objects)
    └── parsed_decisions: [{item_id, qty, order/hold}, ...]
```

Trace-level metadata (attached to every trace):

```python
{
    "sim_id": "sim_run_001",
    "tick": 14,
    "agent_version": "llm_agent_v2",
    "agent_model": "claude-sonnet-4-20250514",
    "prompt_version": "v3",
    "items_evaluated": ["ITEM_A", "ITEM_B", "ITEM_C"],
    "disruptions_active": ["DISR_001"],
    "budget_remaining": 4200.0
}
```

**[verified - the trace/span/generation model is as documented in [R14]; the specific metadata fields here are inferred from the simulation's data model]**

---

## 5.3 Approach A - Thin SDK Integration at the Agent Boundary

The simplest integration: wrap the entire `decide()` method in a single LangFuse trace, with the LLM call recorded as a generation. No span decomposition.

```python
from langfuse import Langfuse
from langfuse.decorators import observe, langfuse_context

langfuse = Langfuse()

class LLMAgent(BaseAgent):

    @observe(name="agent.decide")
    def decide(self, context: AgentContext) -> list[ReorderDecision]:
        langfuse_context.update_current_trace(
            metadata={"sim_id": context.sim_id, "tick": context.tick, ...}
        )
        prompt = self._build_prompt(context)
        response = self.llm.invoke(prompt)   # LangFuse auto-captures if LangChain instrumented
        decisions = self._parse(response)
        langfuse_context.update_current_observation(
            output={"decisions": [d.model_dump() for d in decisions]}
        )
        return decisions
```

**Pros**: Minimal instrumentation; easy to add. **Cons**: No visibility into sub-steps; if the decision is wrong, the trace shows only the final prompt and response with no indication of where reasoning broke down. **[verified - the `@observe` decorator pattern is documented in [R13]]**

---

## 5.4 Approach B - Span-per-Subtool Tracing

More granular: each sub-step in `decide()` is its own span. This is the recommended approach once the agent is past initial prototype stage, as it makes debugging significantly more effective.

```python
@observe(name="agent.decide")
def decide(self, context: AgentContext) -> list[ReorderDecision]:
    with langfuse.span(name="context_assembly") as span:
        prompt = self._build_prompt(context)
        span.end(output={"prompt_length": len(prompt)})

    # If using tool-calling (Design B):
    with langfuse.span(name="tool_call.get_disruptions") as span:
        disruptions = self._get_disruptions(context.sim_id, context.tick)
        span.end(output={"active_disruptions": disruptions})

    with langfuse.generation(name="llm_call", model="claude-sonnet-4-...") as gen:
        response = self.llm.invoke(prompt)
        gen.end(output=response.content, usage={"prompt_tokens": ..., "completion_tokens": ...})

    with langfuse.span(name="decision_parsing") as span:
        decisions = self._parse(response)
        span.end(output={"decisions": [d.model_dump() for d in decisions]})

    return decisions
```

**Pros**: Full visibility into where time is spent, which tools were called, and whether parsing succeeded. **Cons**: More instrumentation code. **[verified - the span/generation creation pattern is documented in [R13] and [R14]]**

---

## 5.5 Approach C - LangFuse as Evaluation Dataset Builder

LangFuse's dataset feature allows traces to be tagged and added to named datasets, which can then be used as evaluation inputs. This is directly applicable for building the golden-set evaluation described in Section 4.3.

Workflow:

1. Run the LLM agent across several simulation scenarios. Traces are captured automatically.
2. In the LangFuse UI (or via the API), browse traces and tag ticks where the agent made a particularly good or particularly poor decision.
3. Add tagged traces to a dataset named `reorder_evaluation_set`.
4. Run the evaluator (LLM judge or rule-based scorer) against the dataset via the LangFuse SDK.
5. Scores are attached to the dataset entries and browsable per trace and per agent version.

```python
from langfuse import Langfuse
langfuse = Langfuse()

# Create dataset
dataset = langfuse.create_dataset(name="reorder_evaluation_set")

# Add items to it (can be done from UI or programmatically)
dataset.create_item(
    input={"context": context.model_dump()},
    expected_output={"decisions": [{"item_id": "ITEM_A", "decision": "reorder", "qty": 50}]},
    metadata={"tick": 14, "scenario": "demand_spike"}
)

# Run evaluation
for item in dataset.items:
    with item.observe(run_name="llm_agent_v2") as trace_id:
        decisions = agent.decide(AgentContext(**item.input["context"]))
        langfuse.score(
            trace_id=trace_id,
            name="decision_validity",
            value=1.0 if all_decisions_valid(decisions) else 0.0
        )
```

**[verified - the dataset creation and item observation pattern is documented in [R15]; the scoring pattern is documented in [R16]]**

---

## 5.6 LangFuse in Development vs. Production

| Phase | LangFuse Role | Configuration |
|---|---|---|
| Development (Config 1) | Debug per-tick reasoning; identify prompt failures; build evaluation datasets | Self-hosted or cloud; all traces captured; sampling rate = 100% |
| Evaluation (Config 2) | Score agent versions; compare trace quality across seeds/scenarios | Cloud or self-hosted; all traces captured; scores computed automatically |
| Production (Config 3) | Monitor live decision quality; alert on reasoning degradation; audit trail | Cloud; sampling rate configurable (e.g. 20% for cost control); alerts on score thresholds |

**[inferred - the development/production configuration split is a standard observability practice applied to LangFuse; specific LangFuse sampling configuration is referenced in [R13] but the split itself is inferred from general observability best practice]**

---

## 5.7 LangFuse + MLflow: Complementary, Not Redundant

LangFuse and MLflow serve different observability concerns and should both be used:

| Concern | MLflow | LangFuse |
|---|---|---|
| Simulation run outcomes (total cost, stockout rate) | ✓ Logged as metrics | - |
| Agent version and prompt version tracking | ✓ Logged as parameters | ✓ Tracked as trace metadata |
| LLM call trace (prompt, response, tokens) | Partial (MLflow Tracing) | ✓ Primary tool |
| Per-tick decision quality scores | Via `mlflow.evaluate()` | ✓ Scores API |
| Evaluation dataset management | Via MLflow dataset artefacts | ✓ Native dataset feature |
| Comparison across agent versions | ✓ Experiment comparison | ✓ Run comparison |

**[verified - the MLflow capabilities are from [R4], [R5]; the LangFuse capabilities are from [R14], [R15], [R16]; the complementarity claim is inferred from the distinct scope of each tool]**

---

## 5.8 Forward Compatibility: LangFuse in the Real-World Extension

LangFuse's value grows in the real-world extension because the ground truth signal (simulation outcome cost) is no longer immediately available - there is no controlled environment to measure total cost over a fixed run. Observability of the agent's reasoning and its decision quality becomes the primary quality signal.

In the production setting:

- Every reorder decision the agent makes is captured as a LangFuse trace.
- Actual reorder outcomes (did the stock arrive on time? Was there a stockout?) can be written back as LangFuse scores after the fact, once the ERP system confirms delivery.
- This creates a delayed feedback loop: trace is captured at decision time; score is attached when the outcome is known. LangFuse's `score()` API supports this - scores can be attached to an existing trace by `trace_id` at any time after the trace was created. **[verified - [R16]]**
- Long-run monitoring: score distributions over time reveal whether agent decision quality is drifting, which can trigger retraining or prompt revision.

**[inferred - the delayed feedback scoring pattern is an application of LangFuse's score API to the warehouse domain; no direct documentation of this specific pattern was found]**

---

# 6. Limitations of Tick-by-Tick LLM Reasoning and Scalable Alternatives

---

## 6.1 What Tick-by-Tick LLM Reasoning Means in This Context

In the baseline architecture described in earlier sections, the LLM agent is called once per simulation tick - every time the runner reaches sub-step 4. For a simulation with N items and T ticks, this means up to T LLM calls per run. Each call receives the full `AgentContext`, reasons over all item states, and returns a list of decisions.

This is the simplest, most interpretable approach and the right starting point. However, it has meaningful limitations that become binding as the simulation or the real-world system scales. **[inferred - the following limitations follow from the inherent properties of LLM inference; they are not specific to any one product]**

---

## 6.2 Core Limitations

**Limitation 1 - Latency scales linearly with tick count:**
Each LLM call adds wall-clock latency. For a 1000-tick simulation run with a 1-2 second per-call latency, the simulation wall time is dominated by LLM inference. This is acceptable in development but makes large-scale evaluation (hundreds of simulation runs for hyperparameter search or scenario coverage) prohibitively slow. **[inferred - from the known latency characteristics of hosted LLM APIs]**

**Limitation 2 - Token cost scales linearly with tick count:**
The `AgentContext` payload grows as demand history accumulates (especially if `agent_history_window_ticks` is large). In a long run, each tick's context becomes larger, and the total token spend for a single run can be significant. At evaluation scale (many runs), this cost multiplies. **[inferred]**

**Limitation 3 - No cross-tick learning within a run:**
Each tick's LLM call is stateless with respect to the LLM's weights. The agent can be given memory by including prior decisions in the prompt, but the LLM does not update its parameters across ticks. If the disruption pattern changes mid-run, the agent can only adapt if the change is visible in the context window - it cannot generalise from early ticks in the way a trained model would. **[inferred]**

**Limitation 4 - Context window pressure at scale:**
For a simulation with many item types, the `AgentContext` grows proportionally. A run with 20 item types, each with 50 ticks of demand history, produces a large prompt. Most LLMs handle this, but at the cost of increased latency, higher token counts, and potential degradation in reasoning quality as the context grows. **[inferred - known limitation of long-context LLM reasoning; no single source directly documents degradation in this specific context]**

**Limitation 5 - Interpretability degrades with prompt complexity:**
As the prompt grows to accommodate more item types, more disruption types, and more history, it becomes harder to understand why the LLM made a particular decision. Debugging requires careful trace analysis, and attributing a poor decision to a specific part of the prompt is non-trivial. **[inferred]**

**Limitation 6 - Inconsistency across equivalent contexts:**
At non-zero temperature, the LLM may make different decisions for identical contexts across runs, which conflicts with the simulation's reproducibility requirement (FR-07). Temperature 0 mitigates but does not fully eliminate this. **[inferred - known property of LLM sampling; temperature 0 produces deterministic outputs for many models but this is model-dependent]**

---

## 6.3 Scalable Alternative Approaches

The following alternatives address the limitations above. They are not mutually exclusive; a production system is likely to combine elements of several.

---

### Alternative A - Sparse LLM Calling (Trigger-Based Reasoning)

**Addresses**: Limitations 1, 2.

Instead of calling the LLM every tick, call it only when a meaningful condition is met: stock falls below a threshold, a new disruption activates, an order fails to arrive. Between trigger events, a lightweight rule-based policy (equivalent to `RuleBasedAgent`) handles routine ticks.

The LLM's role shifts from tick-by-tick decision-maker to **exception handler**: it reasons only about unusual situations that the rule-based policy is not designed to handle well.

```
Each tick:
    if trigger_condition(context):        # disruption, near-stockout, etc.
        decisions = llm_agent.decide(context)
    else:
        decisions = rule_based_agent.decide(context)
```

This can reduce LLM call frequency by 80-95% in stable simulation conditions while preserving LLM reasoning exactly where it adds the most value. **[inferred - by application of standard event-driven agent design to this tick-based context]**

**Cross-reference**: The `reorder_point` parameter in the simulation spec is already a natural trigger signal. Trigger conditions can be defined as configurable parameters alongside `reorder_point`. Section 2.1 (UC Functions as tools) still applies; the tool contract does not change.

---

### Alternative B - Fine-Tuned Small Model

**Addresses**: Limitations 1, 2, 3, 6.

The LLM agent's `(AgentContext, decisions)` pairs from simulation runs constitute a training dataset. A smaller, fine-tuned model (e.g. a compact transformer trained on this dataset) can replicate the LLM's decision pattern at a fraction of the inference cost and latency, with full determinism.

Workflow:

1. Run the LLM agent across many simulation scenarios. Capture `(AgentContext, decisions)` pairs.
2. Fine-tune a smaller model on this dataset.
3. Deploy the fine-tuned model behind the same `BaseAgent` contract.
4. Use the LLM periodically (e.g. weekly in production, or when a novel scenario is detected) to generate new training data and update the fine-tuned model.

This is a **knowledge distillation** pattern: the LLM is the teacher; the fine-tuned model is the student. **[inferred - knowledge distillation from LLM to smaller model is a well-established pattern; the specific application here to the warehouse agent is inferred]**

**Cross-reference**: MLflow (Section 2.2) manages the fine-tuned model's lifecycle. LangFuse (Section 5) traces the LLM teacher runs that generate training data. Model Serving Endpoints (Section 2.7) deploy the fine-tuned student model. The evaluation framework (Section 4) applies to the fine-tuned model identically.

---

### Alternative C - LLM for Policy Generation, Not Decision Execution

**Addresses**: Limitations 1, 2, 3, 4.

Rather than calling the LLM per tick, call it infrequently (e.g. once per disruption event, or once per configurable N ticks) to **generate a reorder policy** - a set of rules or parameters - that the rule-based engine then executes for subsequent ticks.

```
Every N ticks (or on disruption events):
    policy = llm_agent.generate_policy(context_summary)
    # policy = {"ITEM_A": {"reorder_point": 80, "order_qty": 120}, ...}

Each tick:
    decisions = policy_executor.decide(context, policy)
```

The LLM's output is not a list of decisions but a parameterisation of the rule-based policy. This is interpretable (the policy is a readable data structure), cheap to execute per-tick (pure Python), and only requires LLM calls when the world changes significantly. **[inferred - this pattern is a form of policy search using LLMs as planners; it is discussed in the LLM planning literature but not in a directly citable Databricks or LangFuse reference]**

**Cross-reference**: This approach changes what `hist_reorder_decisions.agent_reasoning` stores: it stores the policy parameters rather than per-tick reasoning. LangFuse traces are generated at policy-generation events, not per tick. MLflow logs policy parameters as run artefacts.

---

### Alternative D - Reinforcement Learning from Simulation

**Addresses**: Limitations 1, 2, 3, 6.

The simulation is a natural RL environment: it has state (`AgentContext`), actions (`ReorderDecision`), and a reward signal (negative total cost per tick). A reinforcement learning agent - trained entirely within the simulation - eliminates LLM inference cost at decision time entirely.

The LLM's role in this approach is at **training time**, not inference time: it can be used to generate the initial policy or reward shaping signal, which the RL agent then refines through simulation. **[inferred - RLHF and RL-from-simulation patterns are established; the specific application to this simulation is inferred]**

This is the most scalable approach at inference time but the most complex to implement and the hardest to interpret. It is best considered as a longer-term evolution path rather than a near-term choice. **[inferred]**

**Cross-reference**: If this path is taken, MLflow becomes the primary experiment tracking tool (replacing LangFuse as the primary observability layer, since there are no LLM calls to trace at inference time). The simulation's reproducibility (FR-07) is critical for stable RL training. The `BaseAgent` contract remains valid; the RL agent is simply another `BaseAgent` subclass.

---

### Alternative E - Hybrid: LLM + Structured Optimiser

**Addresses**: Limitations 1, 2, 4.

The LLM handles qualitative reasoning (interpreting disruption signals, assessing supplier reliability, flagging unusual patterns), while a classical optimiser (e.g. an economic order quantity model, or a linear programme) handles the quantitative reorder calculation.

```
Each tick:
    qualitative_assessment = llm_agent.assess(context)   # called sparsely
    order_qty = optimiser.compute(context, qualitative_assessment)
```

This is a **tool-augmented planner** where the LLM is one of the tools rather than the sole decision-maker. The optimiser is fast and deterministic; the LLM is called only when qualitative judgement is needed. **[inferred - this is a standard decomposition of the planning problem; no single source documents this specific configuration]**

**Cross-reference**: The optimiser is a natural candidate for a Unity Catalog function (Section 2.1). LangFuse traces the LLM's qualitative assessment calls. MLflow logs the optimiser's parameters as experiment configuration. This approach is the most natural bridge to the real-world extension, where classical inventory optimisation methods have decades of validated practice.

---

## 6.4 Cross-Reference: Where These Alternatives Appear in Earlier Sections

| Alternative | Section 1 (Tool Relevance) | Section 2 (Component Shaping) | Section 3 (Testing/Deployment) | Section 4 (Evaluation) | Section 5 (LangFuse) |
|---|---|---|---|---|---|
| A - Sparse calling | 1.9 (AI Playground for trigger tuning) | 2.4 (Design A/B hybrid) | 3.1 (unit tests on trigger conditions) | 4.2 (LLM call count metric) | 5.3 (trace per trigger event) |
| B - Fine-tuned model | 1.2 (MLflow for fine-tune lifecycle) | 2.2 (MLflow artefact management) | 3.3 (Config 2 deployment) | 4.5 (baseline comparison) | 5.5 (LangFuse dataset as training data) |
| C - Policy generation | 1.3 (Agent Bricks for policy schema) | 2.3 (structured output for policy) | 3.1 (policy validity tests) | 4.3 (policy reasoning evaluation) | 5.4 (span per policy generation) |
| D - RL from simulation | 1.2 (MLflow for RL tracking) | 2.2 (MLflow as primary tool) | 3.2 (simulation as RL environment) | 4.2 (reward = negative cost) | 5.7 (LangFuse less central) |
| E - Hybrid optimiser | 1.1 (UC function for optimiser) | 2.1 (optimiser as UC function) | 3.1 (optimiser unit tests) | 4.2/4.3 (both levels) | 5.4 (LLM assessment as span) |

The tick-by-tick approach described throughout Sections 1-5 remains the correct starting point. These alternatives are not replacements for the initial implementation but evolutionary paths to be selected based on observed limitations during the development and evaluation process.