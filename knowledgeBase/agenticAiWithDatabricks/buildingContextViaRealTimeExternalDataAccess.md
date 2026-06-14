**Agentic AI with Databricks**

<h1>Building Context via Real-Time External Data Access</h1>

---

**Contents**:

- [Key References](#key-references)
- [Problem Statement 1: Real-Time Intelligence](#problem-statement-1-real-time-intelligence)
- [Solution for Problem Statement 1: MCP Marketplace](#solution-for-problem-statement-1-mcp-marketplace)
- [Problem Statement 2: Need for Persistent Memor](#problem-statement-2-need-for-persistent-memor)
- [Solution for Problem Statement 2: Lakebase as State Engine](#solution-for-problem-statement-2-lakebase-as-state-engine)
- [Overall Workflow](#overall-workflow)

---

# Key References
- [*MCP Marketplace Brings Real-Time Intelligence to Agentic Applications*, **databricks.com/blog**](https://www.databricks.com/blog/mcp-marketplace-brings-real-time-intelligence-agentic-applications)

# Problem Statement 1: Real-Time Intelligence
Without this real-time intelligence, agents become knowledge-limited, constrained by historical data, unable to reason about the world as it is now. They can execute workflows, but they can't make informed decisions. Agents need a way to access live, trusted intelligence while they reason through complex problems.

# Solution for Problem Statement 1: MCP Marketplace
This is a central hub for discovering, connecting to and governing MCP servers (i.e. servers that communicate using the MCP protocol to receive requests for external data and serve it as appropriate) through which AI-ready data can be accessed, enabling agents to access external data at scale with minimal friction.

These servers are not just data feeds, but rahter, they are executable *intelligence sources* that agents can query, evaluate, and act upon, all within a governed, auditable environment. Every connection is authenticated through Unity Catalog, creating a single source of truth for access control and compliance.

By connecting your agentic apps to these MCP servers, you:

- **Bridge the knowledge gap**: <br> *Agentic apps access live data that was not in your training set or data warehouse*
- **Eliminating manual research**: <br> *Workflows that once required human intervention now run autonomously*
- **Staying governed**: <br> *Everything happens inside the Lakehouse, with full lineage and compliance tracking*

These are critical in building reliable and governable agentic apps.

# Problem Statement 2: Need for Persistent Memor
> **State engine** => A storage and data processing solution that allows persistence of states (i.e. persistent memory of ingested data, intermediate results, model context, etc.).

A complex workflow (e.g. a 3-day commercial loan review) involves multiple steps, human handoffs and decisions. Without a persistent state, agents restart from scratch each time they restart/reruns, context is lost and progress resets.

**NOTE**: *Addressing this problem statement matters for ensuring problem statement 1 is fully addressed long-term.*

# Solution for Problem Statement 2: Lakebase as State Engine
Lakebase is a serverless, stateful database purpose-built for agentic applications. It stores agent state, decisions and audit trails throughout multi-step workflows.

> **"Stateful database"** => It stores and remembers data permanently.

**Why this matters for real-time intelligence**:

*Lakebase can connect those insights and reasons across yime and across datasets. The agent does not re-request data; it builds on what it knows. Every decision is automatically logged for compliance.*

# Overall Workflow

1. Application arrives <br> -> *Lakebase stores status and metadata*
2. Agent awakens <br> -> *Retrieves context from Lakebase*
3. Agent queries intelligence <br> -> *Adds relevant external data into context*
4. Agent synthesizes <br> -> *Combines internal data with external intelligence*
5. Agent surfaces decision <br> -> *Proposes approval/denial with full reasoning via Genie*
6. Credit officer reviews <br> -> *One-click approval in your Databricks App*
7. Lakebase logs decision <br> -> *Complete audit trail (decision, data sources, timestamp, approver) for compliance*