> **References**:
> 
> - https://medium.com/@AI-on-Databricks/building-an-investment-assistant-with-the-databricks-mosaic-ai-agent-framework-d2ff276a61d2
> - https://www.databricks.com/blog/agent-bricks-governed-enterprise-agent-platform

---

There are a number of components an agentic AI solution would need to be:

- Effective (especially in domain-specific contexts)
- Auditable
- Extensible

---

Tools:

- Unity Catalog
- MLflow
- Agent Bricks
- External frameworks (e.g. LangChain and LangGraph)
- LangFuse
- Lakebase
- Model serving endpoints
- Databricks apps

Question:

- What is Mosaic AI and why should I care?
- How does Mosaic AI fit into this?

---

Components:

1. Creating and logging agents
2. Agent tracing and debugging
3. Deploying agents
4. Creating AI agent tools
   1. Function calling for agents
   2. Unity Catalog functions?

---

Testing and deployment:

- Prototyping using AI playground (necessary/useful? If so, in what way?)
- Exporting and deploying AI agent
    - As Python package/files?
    - As model serving endpoint?
- Code-based MLflow logs

---

Evaluation:

- MLflow Evaluate API with Databricks Evaluation Agent
    - Registers the agent to Unity Catalog
    - Deploys the agent as a model serving endpoint

---

What you (the GenAI agent) should give me:

- A mapping between:
    - Tools
    - Components
    - Testing and deployment stages
    - Evaluation
- Operational summary of:
    - Each Agentic-AI-relevant Databricks component
    - How it helps in agentic solutions
- Broad architecture of an AI project involving:
    - Tools
    - Components
    - Testing and deployment stages
    - Evaluation