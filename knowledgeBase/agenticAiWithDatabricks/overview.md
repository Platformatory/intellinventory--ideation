**Agentic AI with Databricks**

<h1>Overview</h1>

---

**Contents**:

- [Key References](#key-references)
- [Purpose of this Document](#purpose-of-this-document)
- [Motivation](#motivation)
- [Key Ideas](#key-ideas)
- [AI Agent Development Lifecyle](#ai-agent-development-lifecyle)

---

# Key References
- [*AI Agents with Databricks in 5 Minutes* by Databricks, **youtube.com**](https://www.youtube.com/watch?v=RqRlhGO2an0)

# Purpose of this Document
Provide:

- An overview of how agentic AI apps can be developed in Databricks
- Give high-level details about the various stages of development
- Provide additional information about key steps <br> E.g.: *Governance, model-serving, etc.*

# Motivation
*General answer (LLM) vs. Domain-specific answer (LLM + tools + data)*

In other terms:

*Generalised intelligence vs. Data intelligence*

We want to enable data intelligence.

# Key Ideas
- LLM + tools -> Agent
- LLM + tools + data -> Agent with data intelligence

# AI Agent Development Lifecyle

```
                |   +-->Prepare data
                |   |   |
                |   |   v
                |   |   Build agents
Govern agents---+   |   |
                |   |   v
                |   |   Evaluate agents
                |   |   |
                |   |   v
                |   +---Deploy agents
```

- Prepare data:
    - Data ingestion
    - ML features
    - Vector index (for semantic search)
- Build agents:
    - Model tuning
    - Tool catalog
    - Function calling
- Evaluate agents:
    - LLM judges
    - Tracing (in MLflow) <br> **Reference**: [*MLflow Tracing - GenAI observability*, **docs.databricks.com/aws/en/mlflow3**](https://docs.databricks.com/aws/en/mlflow3/genai/tracing/)
        - Records the full execution flow of an AI agent, including:
            - Inputs
            - Outputs
            - Intermediate tool calls
            - Retrieved data.
        - Provides end-to-end visibility for multi-step LLM apps to:
            - Enable debugging
            - Facilitate performance optimisation
            - Enable quality evaluation
    - Peer labeling (or "labeling session") <br> **Reference**: [](https://docs.databricks.com/aws/en/mlflow3/genai/human-feedback/concepts/labeling-sessions)
        - A structured method for domain experts to... <br> - review <br> - rate <br>- annotate <br> .... model responses to improve performance
        - <details><summary>Click for more info</summary>A labeling session is a special type of MLflow run that contains a specific set of traces that you want domain experts to review using the <b>MLflow Review App</b>. The goal of a labeling session is to collect human-generated assessments (labels) on existing MLflow Traces. You can capture either Feedback or Expectation data, which can then be used to improve your GenAI app through systematic evaluation. For more information on collecting assessments during app development, see Label during development. Labeling sessions appear in the Evaluations tab of the MLflow UI. Because labeling sessions are logged as MLflow runs, you can also access the traces and associated assessments using the MLflow API <code>mlflow.search_runs()</code>.</details>
- Deploy agents
    - Agent/model-serving; i.e.:
        - Exposing functionality via API endpoints
        - Creating an MLflow Review app for human review/feedback
    - Lineage <br> **Reference**: [*What is Data Lineage?*, **databricks.com/blog**](https://www.databricks.com/blog/what-is-data-lineage)
    - MLOps/LLMOps (deploy agents with infrastructure)

---

**Additional points of evaluating agents**:

Evaluation primarily consists of evaluation datasets, which subject matter experts should keep updating as required. We can use MLflow to perform the evaluation across a number of parameters (e.g. correctness, context sufficiency, groundedness, safety). Furthermore, we can track the evolution of our agent's performance over multiple evaluation runs.

---

**Additional points of deploying agents**:

Using MLflow, we can register our agent to Unity Catalog for governance and lineage. We can deploy our agent behind one of the mmodel-serving endpoints provided by Mosaic AI (learn more: [*Deploy models using Mosaic AI Model Serving*, **docs.databricks.com/aws/en/machine-learning**](https://docs.databricks.com/aws/en/machine-learning/model-serving/)). These model-serving endpoints allow you to deploy your agent behind an API with:

- Built-in traffic-splitting (for load balancing)
- Inference logging (for tracing)
- Metrics tracking (for monitoring)
- etc.

The endpoint also automatically creates a **MLflow Review** application that gives acess to subject matter experts, so they can try their own questions and manually test the agent. Feedback from this testing can be captured and added to your evaluation dataset(s).

Unity AI Gateway (see: [*Unity AI Gateway*, **docs.databricks.com**](https://docs.databricks.com/aws/en/ai-gateway/)) (which is the central Databricks AI governance layer for agents, LLM endpoints, MCP servers and coding agents) allows you to analyse usage, configure permissions, enforce guardrails, set rate limiting, apply safety and content filters and manage storage/compute capacity across providers.

> **TERMINOLOGY NOTE**: **MCP = Model Context Protocol**, a protocol to connect AI applications to external systems.