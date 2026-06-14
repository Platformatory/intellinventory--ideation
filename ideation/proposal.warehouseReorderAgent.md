# Warehouse Inventory Reorder Agent
### A Databricks Agentic AI Prototype - Project Document

---

## Table of Contents

1. [Problem Statement](#1-problem-statement)
2. [Why This Is a Strong Agentic AI Use Case](#2-why-this-is-a-strong-agentic-ai-use-case)
3. [Architecture](#3-architecture)
4. [Implementation Plan - 2 Weeks](#4-implementation-plan--2-weeks)
5. [Deployment Plan on Databricks](#5-deployment-plan-on-databricks)
6. [Futurist Vision - Evolving into Supply Chain Disruption](#6-futurist-vision--evolving-into-supply-chain-disruption)

---

## 1. Problem Statement

### The Business Reality

Every warehouse - whether it's a retail store, an e-commerce fulfilment centre, or a manufacturing facility - faces the same fundamental challenge: **keeping the right amount of stock at the right time, without over-ordering or running out**.

Today, this is managed through a combination of manual reviews, static reorder rules ("order more when stock drops below X"), and periodic procurement meetings. This approach breaks down in four predictable ways:

| Problem | What Actually Happens |
|---|---|
| **Static thresholds don't adapt** | A threshold set in January doesn't account for a February promotion or a March demand spike |
| **Multi-variable decisions are too complex for rules** | A reorder decision involves stock level, sales velocity, supplier lead time, budget, and upcoming events - simultaneously |
| **Human attention is a bottleneck** | Procurement managers can't watch every SKU every hour; things slip through |
| **Reactive, not proactive** | Stockouts are noticed after they happen, not before |

### The Problem in One Sentence

> Inventory reorder decisions are continuous, complex, and context-dependent - but today they are managed by static rules and periodic human reviews that cannot keep up with the pace of real-world change.

### What Goes Wrong Without a Better System

- **Stockouts** - lost sales, customer frustration, brand damage
- **Overstocking** - capital tied up in slow-moving goods, storage costs, waste
- **Missed promotions** - a flash sale drains a SKU that was "fine" yesterday
- **Supplier surprises** - a delayed shipment creates an emergency that could have been anticipated

---

## 2. Why This Is a Strong Agentic AI Use Case

### The Three Criteria - Fulfilled

#### P1: Continuous Operations
Sales happen every minute. Inventory ticks down constantly. A SKU that was above threshold at 9am may be critical by 2pm. The reorder agent never "finishes" - it is always watching, always ready to act. This is fundamentally different from a nightly batch job or a weekly procurement review.

#### P2: Complex Decision Making
A naive rule says: "If stock < 50, order 200 units." An intelligent agent reasons:

- *How fast is this SKU selling right now - not last month?*
- *Is there a promotion starting in 3 days that will accelerate demand?*
- *Can the primary supplier deliver in time, or are they delayed?*
- *Is there a cheaper alternative supplier with shorter lead time?*
- *Does this order breach the monthly budget cap - and if so, who needs to approve it?*

This is genuine multi-variable, context-aware reasoning - not a decision tree.

#### P3: Dynamic Environment
The world around the warehouse changes constantly:
- Promotions are announced and cancelled
- Suppliers change their lead times
- Demand spikes due to seasonality, trends, or external events
- Budget periods reset
- New SKUs are added, old ones are discontinued

A static rule cannot adapt to these changes. An agent can.

### Why Databricks Specifically

The Databricks ecosystem is uniquely suited because all the relevant data already lives - or should live - in the lakehouse:

| Data Needed | Where It Lives |
|---|---|
| Current inventory levels | Delta table |
| Sales velocity (rolling) | DLT pipeline |
| Supplier catalogue & lead times | Unity Catalog |
| Upcoming promotions calendar | Delta table |
| Budget status | Delta table |
| Historical demand patterns | Delta table → MLflow model |
| Every agent decision & rationale | Delta table (audit log) |

The agent doesn't need to connect to five different systems - it reads and writes to one governed lakehouse, with Unity Catalog ensuring data quality and access control throughout.

---

## 3. Architecture

### High-Level System Overview

┌──────────────────────────────────────────────────────────────────┐
│                        DATA SOURCES                              │
│                                                                  │
│   POS / ERP Sales Feed    Supplier API    Promotions Calendar    │
│          │                     │                  │              │
│          └─────────────────────┴──────────────────┘             │
│                               │                                  │
│                    Auto Loader / LakeFlow Connect                │
└──────────────────────────────┬───────────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│                    DLT INGESTION PIPELINE                        │
│                                                                  │
│   raw_sales ──► cleaned_sales ──► inventory_snapshot            │
│                                                                  │
│   Enrichment at each stage:                                      │
│   • Sales velocity (rolling 7-day window)                        │
│   • Days-to-stockout forecast                                    │
│   • Supplier lead time (from Unity Catalog)                      │
│   • Upcoming promotions (next 14 days)                           │
│   • Budget remaining this month                                  │
└──────────────────────────────┬───────────────────────────────────┘
                               │ enriched SKU snapshot
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│                     MLFLOW DEMAND MODEL                          │
│                                                                  │
│   Input:  sales history, seasonality, promotions flag           │
│   Output: predicted demand for next 14 days per SKU             │
│   Served: via Databricks Model Serving endpoint                  │
└──────────────────────────────┬───────────────────────────────────┘
                               │ demand forecast per SKU
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│                     REORDER AGENT                                │
│              (Mosaic AI Agent Framework /                        │
│               OpenAI Agents SDK + ResponsesAgent)                │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  OBSERVE                                                 │    │
│  │  Read inventory_snapshot from Delta                     │    │
│  │  Identify all SKUs below reorder threshold              │    │
│  └─────────────────────┬───────────────────────────────────┘    │
│                        │                                         │
│  ┌─────────────────────▼───────────────────────────────────┐    │
│  │  REASON  (tools called here)                             │    │
│  │  • check_inventory()      → current stock + velocity    │    │
│  │  • forecast_demand()      → MLflow model endpoint       │    │
│  │  • get_supplier_options() → Unity Catalog lookup        │    │
│  │  • check_budget()         → Delta budget table          │    │
│  │  • check_promotions()     → upcoming events Delta table │    │
│  └─────────────────────┬───────────────────────────────────┘    │
│                        │                                         │
│  ┌─────────────────────▼───────────────────────────────────┐    │
│  │  DECIDE                                                  │    │
│  │  • Routine reorder      → auto-place PO                 │    │
│  │  • Supplier delayed     → switch to alternative         │    │
│  │  • Promotion detected   → inflate order quantity        │    │
│  │  • Budget breach        → escalate to procurement mgr   │    │
│  │  • Stockout < 24hrs     → emergency flag + escalate     │    │
│  └─────────────────────┬───────────────────────────────────┘    │
│                        │                                         │
│  ┌─────────────────────▼───────────────────────────────────┐    │
│  │  ACT                                                     │    │
│  │  • write_purchase_order()  → PO written to Delta        │    │
│  │  • notify_supplier()       → webhook / email trigger    │    │
│  │  • escalate_to_human()     → alert procurement manager  │    │
│  │  • log_decision()          → full rationale to Delta    │    │
│  └─────────────────────────────────────────────────────────┘    │
└──────────────────────────────┬───────────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│                  AUDIT & MONITORING LAYER                        │
│                                                                  │
│   Delta Table: agent_decisions                                   │
│   • SKU, timestamp, decision, rationale, outcome                 │
│                                                                  │
│   Lakeview Dashboard                                             │
│   • Stock health heatmap by SKU                                  │
│   • Purchase orders raised (last 7 days)                         │
│   • Supplier reliability score                                   │
│   • Budget utilisation                                           │
│   • Escalation queue for human review                            │
│   • Agent accuracy (did the PO arrive in time?)                  │
└──────────────────────────────────────────────────────────────────┘

The system flows through four layers:

**Data Sources** - POS/ERP Sales Feed, Supplier API, and Promotions Calendar are ingested via Auto Loader / LakeFlow Connect.

**DLT Ingestion Pipeline** - Raw sales are cleaned and enriched into an `inventory_snapshot`, adding: rolling 7-day sales velocity, days-to-stockout forecast, supplier lead time (from Unity Catalog), upcoming promotions (next 14 days), and budget remaining this month.

**MLflow Demand Model** - Takes sales history, seasonality, and promotions flags as input and outputs predicted demand per SKU for the next 14 days, served via a Databricks Model Serving endpoint.

**Reorder Agent** - The core reasoning loop, built on the Mosaic AI Agent Framework / OpenAI Agents SDK, operating in four stages:

- **OBSERVE** - Reads the inventory snapshot and identifies all SKUs below reorder threshold.
- **REASON** - Calls tools to gather context: current stock and velocity, MLflow demand forecast, supplier options from Unity Catalog, budget status, and upcoming promotions.
- **DECIDE** - Routes each SKU to the appropriate action:
  - Routine reorder → auto-place PO
  - Supplier delayed → switch to alternative
  - Promotion detected → inflate order quantity
  - Budget breach → escalate to procurement manager
  - Stockout < 24hrs → emergency flag + escalate
- **ACT** - Writes purchase orders to Delta, triggers supplier notifications, escalates to humans where needed, and logs every decision with full rationale.

**Audit & Monitoring Layer** - Every agent decision is written to a Delta table capturing SKU, timestamp, decision, rationale, and outcome. A Lakeview Dashboard provides a live view of stock health, POs raised, supplier reliability scores, budget utilisation, escalation queue, and agent accuracy.

### Key Delta Tables

| Table | Purpose |
|---|---|
| `raw_sales` | Raw sales events from POS/ERP |
| `inventory_snapshot` | Enriched current stock per SKU |
| `supplier_catalogue` | Supplier options, lead times, costs (governed in Unity Catalog) |
| `promotions_calendar` | Upcoming promotions and expected demand uplift |
| `budget_tracker` | Monthly budget remaining per category |
| `purchase_orders` | All POs raised - by agent or human |
| `agent_decisions` | Full audit log: every decision + rationale + outcome |

### Agent Tools

The agent operates through seven purpose-built tools:

- **check_inventory** - Queries current stock level, sales velocity, and days-to-stockout for a SKU
- **forecast_demand** - Calls the MLflow demand forecasting endpoint for predicted demand
- **get_supplier_options** - Retrieves available suppliers, lead times, and unit costs from Unity Catalog
- **check_budget** - Checks remaining budget for a product category this month
- **check_promotions** - Checks for upcoming promotions that will affect demand for a SKU
- **write_purchase_order** - Writes a confirmed purchase order to Delta and triggers supplier notification
- **escalate_to_human** - Flags a decision for human review with full context summary
- **log_decision** - Writes the agent's decision and full reasoning to the audit Delta table

---

## 4. Implementation Plan - 2 Weeks

### Overview

| Phase | Days | Focus |
|---|---|---|
| Phase 1: Data Foundation | Days 1–5 | Synthetic data, Delta tables, DLT pipeline |
| Phase 2: Agent Build | Days 6–9 | Tools, agent loop, LLM reasoning |
| Phase 3: Agent Build | Day 10 | Reorder agent Development |
| Phase 4: Demo & Polish | Day 13 | Dashboards and publish on Agent App |

---

### Phase 1: Data Foundation (Days 1–5)

#### Day 1–2: Synthetic Dataset

Generate a realistic synthetic dataset (no real data required). The following tables need to be created and loaded into Delta via Auto Loader:

- **products** - 200 SKUs across 5 categories, each with reorder threshold, reorder quantity, and unit cost
- **inventory** - Current stock levels per SKU
- **sales_events** - 90 days of sales history (simulated stream), capturing quantity sold, timestamp, and store
- **suppliers** - 2–3 suppliers per SKU with lead time, unit cost, and reliability score
- **promotions** - Upcoming promotional events with expected demand uplift percentage

#### Day 3–4: DLT Pipeline

Build the enrichment pipeline using Delta Live Tables. The pipeline processes `raw_sales` → `cleaned_sales` → `inventory_snapshot`, with data quality expectations (valid quantities and SKU IDs enforced) and enrichment at each stage to add rolling sales velocity, days-to-stockout, and promotion flags per SKU.

#### Day 5: Demand Forecasting Model

Train a demand forecasting model using sales history, seasonality, promotion flags, and product category as features. Register the model in MLflow and deploy it to a Databricks Model Serving endpoint for real-time inference by the agent.

---

### Phase 2: Agent Build (Days 6–9)

#### Day 6–7: Agent Tools and Loop

The agent is built using the Databricks Agent Framework with the OpenAI Agents SDK template. The agent is configured with a detailed system prompt that defines its reasoning approach:

- Check current inventory and sales velocity for each at-risk SKU
- Forecast demand for the next 14 days
- Check for upcoming promotions that may increase demand
- Evaluate supplier options by reliability, cost, and lead time
- Check budget availability
- Make a reorder decision with clear rationale

**Decision logic embedded in the agent:**
- Auto-place PO if routine reorder, primary supplier available, within budget, and lead time is sufficient
- Inflate order quantity by 30–50% if a promotion is coming
- Switch to alternative supplier if primary is delayed
- Escalate to human if budget is breached, stockout is < 24 hrs, or no reliable supplier is available
- Always log the decision with full rationale; always prefer action over inaction

#### Day 8: MLflow ResponsesAgent Wrapper

Wrap the agent loop in an MLflow `ResponsesAgent` so it can be logged, versioned, and registered as a model in the MLflow Model Registry. This enables reproducible deployments and evaluation tracking.

#### Day 9: Trigger Loop and End-to-End Test

Build a continuous trigger loop that scans all SKUs below threshold or within 7 days of stockout, ordered by urgency (days to stockout ascending), and invokes the agent for each. This loop is designed to be scheduled every 15 minutes via Databricks Workflows.

---

### Phase 3: Demo & Polish (Day 10)

#### Lakeview Dashboard

Create a Databricks Lakeview dashboard with tiles covering:

| Tile | What It Shows |
|---|---|
| SKUs at risk | All SKUs with days-to-stockout < 7, ordered by urgency |
| POs raised today | Count of purchase orders raised by the agent today |
| Escalation queue | All open escalations awaiting human review |
| Agent decision log | Last 50 agent decisions with reasoning |
| Budget utilisation | Remaining vs. total budget per category |

#### Demo Script (2-Minute Loop)

1. Show the inventory snapshot - "Here are 3 SKUs below threshold"
2. Trigger the agent - "The agent is now reasoning about each one"
3. Show the agent's reasoning trace in MLflow - "It checked supplier options, saw a promotion coming, inflated the order by 40%"
4. Show the purchase order written to Delta - "PO raised automatically"
5. Show an escalation - "This SKU breached the budget cap - the agent flagged it for human review rather than acting unilaterally"
6. Show the audit log - "Every decision, every reason, permanently recorded"

---

## 5. Deployment Plan on Databricks

*Based on the official Databricks Agent Framework documentation.*

### Step 1: Clone the Agent App Template

Use the official Databricks `app-templates` repository and select the **Agent - OpenAI Agents SDK** template. Alternatively, create via the workspace UI: **+ New → App → Agents → Agent - OpenAI Agents SDK**, and name the experiment `inventory-reorder-agent`.

### Step 2: Customise the Template

Replace the starter agent code with the `InventoryResponsesAgent`. The template ships with:
- A built-in chat UI (streaming, markdown, Databricks auth)
- MLflow AgentServer (async FastAPI, `/invocations` endpoint)
- MLflow tracing and evaluation hooks

### Step 3: Add Tools and Configure MCP

Register the agent's Python function tools using `@function_tool`. For Unity Catalog lookups, connect to Databricks MCP servers via `databricks.yml`, granting the app access to the demand model serving endpoint, the LLM endpoint (`databricks-claude-sonnet-4-5`), and the MLflow experiment.

### Step 4: Run Locally for Testing

Run the app locally and test via the built-in chat UI. Recommended test queries:
- *"Check inventory for SKU_001 and make a reorder decision"*
- *"Which SKUs are at risk of stockout in the next 72 hours?"*
- *"We have a promotion starting next week for Footwear - review all footwear SKUs"*

### Step 5: Configure Authentication

Use **App Authorization** (service principal) for the prototype - all agent actions run under a single shared identity. Grant the app's service principal access to the inventory Unity Catalog and a SQL warehouse via `databricks.yml`.

### Step 6: Evaluate the Agent

Before deploying to production, run MLflow agent evaluation (`uv run agent-evaluate`). This checks:
- **Response relevance** - Does the agent actually make a reorder decision?
- **Safety** - Does it escalate correctly rather than acting outside policy?
- **Tool call accuracy** - Does it query the right tables?

Review traces in the MLflow Tracking UI - every tool call and reasoning step is logged.

### Step 7: Deploy to Databricks Apps

Create the Databricks App, sync the source code to the workspace, and deploy via the Databricks CLI. The app runs as a persistent service with the built-in chat UI and `/invocations` endpoint.

### Step 8: Schedule Continuous Execution

The agent needs to run continuously, not just when queried via chat. Use **Databricks Workflows** to schedule the trigger loop every 15 minutes via a cron expression, pointing to the trigger loop notebook.

### Step 9: Query and Monitor the Deployed Agent

Monitor agent performance via:
- **MLflow Tracking** - full traces of every agent run, tool calls, and reasoning
- **Lakeview Dashboard** - business KPIs (stock health, POs raised, escalations)
- **Unity Catalog Lineage** - track which tables the agent read and wrote

---

## 6. Futurist Vision - Evolving into Supply Chain Disruption

### Where We Are Now (v1)

The current prototype assumes suppliers are **stable**. It knows their lead times, it trusts their availability, and it makes reorder decisions based on internal data (stock, sales, promotions, budget). The world it operates in is **inside the warehouse**.

### The Key Insight

The Warehouse Reorder Agent asks: *"Do I need to order more?"*

The Supply Chain Disruption Agent asks: *"Can I even get what I need - and what do I do if I can't?"*

The second question requires the agent to look **outside** the warehouse - at suppliers, logistics networks, geopolitical events, and market conditions. That is the evolution.

### The Evolution Roadmap

#### v2: Supplier Reliability Scoring (3 months out)
Add a live supplier reliability score to Unity Catalog, updated by a separate monitoring pipeline that watches:
- On-time delivery rates (from historical PO data already in Delta)
- Supplier self-reported delays (via API or email parsing)
- Public logistics delay indices

The reorder agent now doesn't just pick the cheapest supplier - it picks the most *reliable* one for the current situation.

New capability needed: `get_supplier_reliability_score(supplier_id, current_date)` backed by a `supplier_performance_history` Delta table.

#### v3: External Signal Monitoring (6 months out)
Add a second agent - the **Supply Chain Watcher** - that runs in parallel and monitors the outside world:

- Port delay feeds (e.g. Freightos API)
- Supplier news (LLM reads industry feeds)
- Weather events in supplier regions
- Exchange rate movements (cost impact)

This agent outputs supplier risk flags to Delta and disruption alerts that trigger the Reorder Agent to run immediately (rather than waiting for the 15-minute schedule). The two agents communicate via Delta: the Watcher writes a disruption event, the Reorder Agent reads it and acts.

#### v4: Multi-Supplier Negotiation (12 months out)
When a primary supplier is flagged as at-risk, the agent doesn't just switch to the next cheapest option. It **negotiates**:
- Queries multiple alternative suppliers simultaneously
- Compares lead time, cost, minimum order quantity, and reliability
- If LLM reasoning determines it's worth it, initiates a structured RFQ (Request for Quote) via supplier API
- Selects the best option and raises the PO - all autonomously

New capabilities needed: `request_supplier_quote`, `compare_quotes`, and `accept_quote`.

#### v5: Autonomous Budget Management (18 months out)
The agent is given a monthly procurement budget and full autonomy to manage it:
- Prioritises SKUs by margin contribution (high-margin SKUs get budget priority)
- Times orders strategically (buy before a price increase, defer non-urgent orders near month-end)
- Reports variance to procurement leadership - "I deferred 3 non-urgent reorders to stay within budget; stockout risk is low"

### Full v5 Architecture (Vision)

Three agents work in concert:

- **Supply Chain Watcher Agent** monitors the external world (port delays, supplier news, weather, exchange rates) and writes disruption events and risk flags to Delta.
- **Reorder Agent** handles internal warehouse decisions (Observe → Reason → Decide → Act), receives disruption signals from the Watcher, and passes supplier feedback back.
- **Negotiation Agent** handles multi-supplier RFQ and selection when the Reorder Agent triggers it.

All decisions flow through Delta Audit Log, Unity Catalog, a Lakeview Dashboard, and a Human Oversight Queue.

### What Never Changes

Regardless of how sophisticated the system becomes, three principles remain constant:

1. **Human-in-the-loop for irreversible decisions** - large orders, write-offs, supplier terminations always require human approval
2. **Every decision is logged with rationale** - the audit trail in Delta is sacred; regulators, auditors, and procurement managers can always see why the agent did what it did
3. **Delta Lake is the single source of truth** - all agent state, all supplier data, all decisions flow through the lakehouse; no shadow systems, no Excel spreadsheets

---

*Document prepared for Databricks Agentic AI Hackathon - May 2026*
*Architecture references: Databricks Agent Framework (docs.databricks.com/aws/en/generative-ai/agent-framework)*
