<h1>Reasoning Integration - 2</h1>

> **Context**: [`planning/reasoningIntegration-1.md`](./reasoningIntegration-1.md)
>
> This document explores "Suggestion 2" (as proposed in the above-linked document) in greater architectural and practical detail, with the aim being to reach an actionable plan for implementation that enables agentic system development that is both achievable and extensible to production-level environments, as per the considerations made in the above-linked document.

---

# Overview
## Overall Solution Structure

```
----+<--sc--+-------
LLM |       | engine
----+--sr-->+-------
```

- sc = Structured context
- sr = Structured response

Engine must handle:

- Structurally invalid responses
- Logically invalid responses
- LLM call failures/timeouts

Each case requires a well-defined fallback (retry, rule-based policy, halt with alert, etc.). Defining this is IMPORTANT for the implementation.

## Clarifications
As flagged in ["Tentative Approach", `reasoningIntegation-1.md`](./reasoningIntegration-1.md#tentative-approach), the following statement is too vague to act upon: "Start with suggestion 2, while keeping an eye on suggestion 1." Hence, specifically, we must consider:

1. Make decision trigger condition a configurable parameter from the start
2. Log LLM call frequency and token cost per simulation run in MLflow
3. Define a threshold at which suggestion 1's meta-loop becomes worth implementing <br> E.g.: token cost exceeds X per run, call frequency exceeds Y/hour in PROD, etc.

## KEY POINT 1: Working toward PROD while in Simulation
Instead of the simulation's tick engine enacting the Delta table writes based on decisions, we could define Unity Catalog (UC) functions to do the same, thereby moving the implementation closer to PROD level.

## KEY POINT 2: Design Approach
The focus should be on implementing a simulation-based agent, leaving PROD options open where possible and specifying PROD-related decisions/considerations where necessary.

# Raising Decision-Making Points
## 1. Parameterisation Scope
- What can be parameterised for the simulation
- What need not be done for the simulation but must be considered for PROD

## 2. Trigger Mode
In simulation, we must consider what to implement to enable this trigger mode in PROD.

> Choice of trigger mode determines executor design.

## 3. Resilience Measures
To be defined.

## 4. Tool Abstraction Layer
- Unity Catalog function definitions (tools)
- Depends on environment decomposition <br> I.e.: What are the different environment components to interact with

## 5. Making Trigger Condition a Governable Artefact
Preferably through Unity Catalog?

## 6. Define push targets and pull consumers
For downstream use-cases, e.g. dashboard, alerting, tracing, etc.

## 7. Agent Reasoning Context Enrichment
The agent as such is stateless, but we can consider adding a shared state using Lakebase to enrich the reasoning context in a way that can be shared by multiple agents (allowing for horizontal scaling).

## 8. Incorporate Feedback Latency in Design
E.g.: Make sure the reasoning context records account for lag between decision made and the decision's effects.

## 9. Inter-Process Communication between Monitoring & Executor
To ensure decoupling between monitoring and decision-execution processes.

# Addressing Decision-Making Points
## 1