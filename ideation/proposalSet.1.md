<h1>Databricks Agentic AI - Hackathon Idea Catalogue 1</h1>

> A structured evaluation of 7 agentic AI use cases against a defined set of problem-space and implementation-level criteria, with architecture sketches and implementation approaches for each.

---

**Contents**:

- [Evaluation Criteria](#evaluation-criteria)
  - [Problem-Space Requirements](#problem-space-requirements)
  - [Implementation-Level Requirements](#implementation-level-requirements)
- [Summary Comparison](#summary-comparison)
- [Part 1 - Original Suggestions](#part-1---original-suggestions)
  - [1. Real-Time Fraud Detection \& Response Agent](#1-real-time-fraud-detection--response-agent)
  - [Criteria Fulfilment](#criteria-fulfilment)
  - [Architecture](#architecture)
  - [Implementation Approach](#implementation-approach)
  - [2. Supply Chain Disruption \& Reorder Agent](#2-supply-chain-disruption--reorder-agent)
  - [Criteria Fulfilment](#criteria-fulfilment-1)
  - [Architecture](#architecture-1)
  - [Implementation Approach](#implementation-approach-1)
  - [3. Patient Deterioration Early Warning Agent](#3-patient-deterioration-early-warning-agent)
  - [Criteria Fulfilment](#criteria-fulfilment-2)
  - [Architecture](#architecture-2)
  - [Implementation Approach](#implementation-approach-2)
- [Part 2 - Predecessor Suggestions Re-Evaluated](#part-2---predecessor-suggestions-re-evaluated)
  - [4. Real-Time E-Commerce Customer Support \& Order Recovery Agent](#4-real-time-e-commerce-customer-support--order-recovery-agent)
  - [Criteria Fulfilment](#criteria-fulfilment-3)
  - [Architecture](#architecture-3)
  - [Implementation Approach](#implementation-approach-3)
  - [5. Warehouse / Inventory Reorder Agent ⭐ Top Pick](#5-warehouse--inventory-reorder-agent--top-pick)
  - [Criteria Fulfilment](#criteria-fulfilment-4)
  - [Architecture](#architecture-4)
  - [Implementation Approach](#implementation-approach-4)
  - [6. Network / Infrastructure Anomaly Response Agent](#6-network--infrastructure-anomaly-response-agent)
  - [Criteria Fulfilment](#criteria-fulfilment-5)
  - [Architecture](#architecture-5)
  - [Implementation Approach](#implementation-approach-5)
  - [7. Food Delivery / Logistics Dispatch Agent](#7-food-delivery--logistics-dispatch-agent)
  - [Criteria Fulfilment](#criteria-fulfilment-6)
  - [Architecture](#architecture-6)
  - [Implementation Approach](#implementation-approach-6)
- [Appendix - Recommended Databricks Stack (All Candidates)](#appendix---recommended-databricks-stack-all-candidates)
- [Appendix - Universal Agent Loop](#appendix---universal-agent-loop)

---

# Evaluation Criteria

## Problem-Space Requirements
| # | Criterion | Description |
|---|---|---|
| P1 | **Continuous Operations** | The system must operate without pause - events arrive constantly and decisions must be made in real or near-real time |
| P2 | **Complex Decision Making** | Decisions must involve multiple variables, context-awareness, and reasoning - not simple rules |
| P3 | **Dynamic Environment** | The world the agent operates in changes frequently - policies, data patterns, external conditions |

## Implementation-Level Requirements
| # | Criterion | Description |
|---|---|---|
| I1 | **Novice-Friendly** | A team new to agentic AI can build a working prototype within 2 weeks |
| I2 | **Platform Showcase** | The implementation meaningfully demonstrates Databricks ecosystem capabilities |
| I3 | **Prototype → Infinite Depth** | v1 is simple to build; subsequent iterations add genuine complexity and real-world value |

---

# Summary Comparison

| # | Use Case | P1 | P2 | P3 | I1 | I2 | I3 | Demo Impact |
|---|---|---|---|---|---|---|---|---|
| 1 | Fraud Detection & Response | ✅ | ✅ | ✅ | ✅ | ⭐⭐⭐ | ✅ | 💰 High |
| 2 | Supply Chain Disruption & Reorder | ✅ | ✅ | ✅ | ✅ | ⭐⭐⭐ | ✅ | 📦 Practical |
| 3 | Patient Deterioration Early Warning | ✅ | ✅ | ✅ | ✅ | ⭐⭐⭐ | ✅ | ❤️ Powerful |
| 4 | E-Commerce Customer Support & Order Recovery | ✅ | ✅ | ✅ | ✅ | ⭐⭐⭐ | ✅ | 💬 Relatable |
| 5 | Warehouse / Inventory Reorder | ✅ | ✅ | ✅ | ✅ | ⭐⭐⭐ | ✅ | 📦 Clean |
| 6 | Network / Infrastructure Anomaly Response | ✅ | ✅ | ✅ | ⭐⭐ | ⭐⭐⭐ | ✅ | 🔧 Technical |
| 7 | Food Delivery / Logistics Dispatch | ✅ | ✅ | ✅ | ⭐⭐ | ⭐⭐⭐ | ✅ | 🗺️ Visual |

> **Top picks:** #5 Inventory Reorder (cleanest loop, most platform-native), #4 E-Commerce Support (best for NLP-leaning teams), #3 Patient Deterioration (strongest demo impact).

---

# Part 1 - Original Suggestions

*The following three candidates were generated directly against the stated criteria.*

---

## 1. Real-Time Fraud Detection & Response Agent

## Criteria Fulfilment

| Criterion | Fulfilment | Rationale |
|---|---|---|
| P1 Continuous Ops | ✅ | Transactions flow 24/7 - the agent never has a quiet moment |
| P2 Complex Decisions | ✅ | Multi-factor scoring: amount, device, location, velocity, merchant category |
| P3 Dynamic Environment | ✅ | Fraud patterns evolve constantly; what was normal last month is today's attack vector |
| I1 Novice-Friendly | ✅ | Synthetic transaction data is trivial; core loop is watch → score → decide → act |
| I2 Platform Showcase | ⭐⭐⭐ | Structured Streaming, DLT, MLflow, Model Serving, Delta, Unity Catalog all used naturally |
| I3 Infinite Depth | ✅ | v1: threshold rules → v2: ML scoring → v3: graph fraud networks → v4: real-time retraining |

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    TRANSACTION STREAM                       │
│              (Kafka / Auto Loader / Zerobus)                │
└──────────────────────────┬──────────────────────────────────┘
                           │ real-time events
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   DLT INGESTION PIPELINE                    │
│   raw_transactions → cleaned_transactions → enriched_txns   │
│   (attach customer profile, device history, merchant info)  │
└──────────────────────────┬──────────────────────────────────┘
                           │ enriched transaction
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                  FRAUD SCORING AGENT                        │
│                                                             │
│  OBSERVE: read enriched transaction from Delta              │
│     │                                                       │
│  REASON: call MLflow model → get fraud probability          │
│          + LLM reasons over edge cases                      │
│     │                                                       │
│  DECIDE:  score < 0.3  → ✅ auto-approve                    │
│           score 0.3-0.7 → ⚠️  soft block + challenge        │
│           score > 0.7  → 🚫 block + alert                   │
│     │                                                       │
│  ACT:     write decision to Delta                           │
│           trigger notification (SMS / email / flag in UI)   │
│           log rationale for audit                           │
└──────────────────────────┬──────────────────────────────────┘
                           │ decisions + rationale
                           ▼
┌─────────────────────────────────────────────────────────────┐
│              AUDIT & MONITORING LAYER                       │
│   Delta table: every decision, score, rationale stored      │
│   Lakeview dashboard: false positive rate, block rate,      │
│                        model drift, agent performance       │
│   Human review queue: edge cases escalated here             │
└─────────────────────────────────────────────────────────────┘
```

## Implementation Approach

**Week 1**
- Day 1–2: Generate synthetic transaction dataset (Faker library), load into Delta via Auto Loader
- Day 3–4: Build DLT pipeline - raw → cleaned → enriched (join with synthetic customer/merchant tables)
- Day 5: Train a simple fraud classifier in MLflow (XGBoost on amount, merchant category, hour, device age)

**Week 2**
- Day 6–7: Build the agent loop - Mosaic AI Agent Framework, wire up 3 tools: `score_transaction()`, `lookup_customer_history()`, `block_transaction()`
- Day 8–9: Add LLM reasoning layer for edge cases (0.3–0.7 score band), write decisions back to Delta
- Day 10: Lakeview dashboard + demo script

**Future Iterations:** graph fraud networks, real-time model retraining, cross-customer collusion detection, regulatory reporting automation.

---

## 2. Supply Chain Disruption & Reorder Agent

## Criteria Fulfilment

| Criterion | Fulfilment | Rationale |
|---|---|---|
| P1 Continuous Ops | ✅ | Inventory levels, supplier lead times, and demand forecasts change every hour |
| P2 Complex Decisions | ✅ | Reorder decisions involve quantity, supplier choice, timing, budget, and demand forecast simultaneously |
| P3 Dynamic Environment | ✅ | Port strikes, demand spikes, supplier failures - the environment changes without warning |
| I1 Novice-Friendly | ✅ | Synthetic inventory data is trivial; the decision loop is clean and explainable |
| I2 Platform Showcase | ⭐⭐⭐ | Auto Loader, DLT, MLflow demand forecasting, Delta, Unity Catalog for supplier catalogue |
| I3 Infinite Depth | ✅ | v1: threshold reorder → v2: ML forecast → v3: multi-supplier negotiation → v4: autonomous budget management |

## Architecture

```
┌──────────────────────────────────────────────────────────────┐
│              DATA SOURCES (via LakeFlow Connect)             │
│   ERP system │ Supplier APIs │ Logistics feeds │ Demand data │
└──────────────────────────┬───────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────┐
│                   DLT PIPELINE                               │
│  raw_inventory → current_stock → enriched_stock              │
│  (attach demand forecast, supplier lead time, reorder rules) │
└──────────────────────────┬───────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────┐
│              SUPPLY CHAIN AGENT                              │
│                                                              │
│  OBSERVE: scan all SKUs below reorder threshold              │
│     │                                                        │
│  REASON: forecast days-to-stockout                           │
│          check primary supplier lead time                    │
│          check alternative suppliers                         │
│          calculate optimal order quantity                    │
│     │                                                        │
│  DECIDE: routine reorder    → auto-place order               │
│          supplier delayed   → switch to alternative          │
│          budget breach      → escalate to procurement mgr    │
│          stockout imminent  → emergency alert                │
│     │                                                        │
│  ACT:    write PO to Delta → trigger downstream workflow     │
│          log decision rationale                              │
└──────────────────────────┬───────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────┐
│           MONITORING DASHBOARD (Lakeview)                    │
│  Stock health │ Agent decisions │ Supplier reliability score │
│  Cost savings from auto-reorder │ Human escalation queue     │
└──────────────────────────────────────────────────────────────┘
```

## Implementation Approach

**Week 1**
- Day 1–2: Synthetic product catalogue + inventory + supplier dataset in Delta
- Day 3–4: DLT pipeline enriching inventory with demand forecast and supplier lead time
- Day 5: Simple demand forecasting model in MLflow (moving average is fine for prototype)

**Week 2**
- Day 6–7: Agent loop with tools: `check_stock_levels()`, `get_supplier_status()`, `place_order()`, `escalate_to_human()`
- Day 8–9: LLM reasoning for multi-supplier trade-off decisions, log rationale to Delta
- Day 10: Dashboard + demo

**Future Iterations:** seasonal ML models, multi-supplier negotiation, autonomous budget management, carbon footprint-aware ordering.

---

## 3. Patient Deterioration Early Warning Agent

## Criteria Fulfilment

| Criterion | Fulfilment | Rationale |
|---|---|---|
| P1 Continuous Ops | ✅ | Patient vitals stream every few minutes - a deteriorating patient doesn't wait for a batch job |
| P2 Complex Decisions | ✅ | Single signals mean nothing; the agent synthesises heart rate + BP + temperature + medication history across time |
| P3 Dynamic Environment | ✅ | A patient stable at 6am can be critical by 9am; risk assessment must keep updating |
| I1 Novice-Friendly | ✅ | Synthetic vitals are easy to generate; NEWS2 is a published clinical formula - no training data needed |
| I2 Platform Showcase | ⭐⭐⭐ | Structured Streaming, DLT rolling windows, MLflow, Unity Catalog for governed patient data, Lakeview ward dashboard |
| I3 Infinite Depth | ✅ | v1: NEWS2 alerts → v2: early sepsis detection → v3: medication interaction checking → v4: discharge prediction |

## Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                 VITALS STREAM                                │
│       (IoT sensors / Auto Loader / simulated feed)           │
└──────────────────────────┬───────────────────────────────────┘
                           │ every ~5 minutes per patient
                           ▼
┌──────────────────────────────────────────────────────────────┐
│                  DLT INGESTION PIPELINE                      │
│  raw_vitals → cleaned_vitals → patient_snapshot              │
│  (rolling window: last 6 readings, trend direction,          │
│   attach medication log, diagnosis, ward info)               │
└──────────────────────────┬───────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────┐
│              EARLY WARNING AGENT                             │
│                                                              │
│  OBSERVE: latest patient_snapshot from Delta                 │
│     │                                                        │
│  REASON: compute NEWS2 deterioration score                   │
│          detect trend (is BP falling over last 3 readings?)  │
│          check: any missed medications?                      │
│          check: how long since last nurse review?            │
│     │                                                        │
│  DECIDE: score 0-4  → ✅ routine monitoring                  │
│          score 5-6  → ⚠️  flag to nurse, increase frequency  │
│          score 7+   → 🚨 urgent alert, suggest intervention  │
│     │                                                        │
│  ACT:    write alert to Delta                                │
│          generate human-readable summary for nurse           │
│          log every decision with full rationale              │
└──────────────────────────┬───────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────┐
│              WARD DASHBOARD (Lakeview)                       │
│  All patients ranked by deterioration risk                   │
│  Alert history │ Response times │ Agent accuracy over time   │
│  Human override log (nurse disagreed → feeds back to model)  │
└──────────────────────────────────────────────────────────────┘
```

## Implementation Approach

**Week 1**
- Day 1–2: Generate synthetic patient vitals time series (Python + NumPy), load into Delta
- Day 3–4: DLT pipeline - raw vitals → rolling window snapshot per patient
- Day 5: Implement NEWS2 scoring as an MLflow model (published clinical formula - no training required)

**Week 2**
- Day 6–7: Agent loop with tools: `get_patient_snapshot()`, `compute_risk_score()`, `send_alert()`, `log_decision()`
- Day 8–9: LLM generates natural language summaries for nurses, all decisions logged to Delta
- Day 10: Lakeview ward dashboard + demo

**Future Iterations:** early sepsis detection, medication interaction checking, bed allocation optimisation, discharge prediction, post-discharge follow-up agent.

---

# Part 2 - Predecessor Suggestions Re-Evaluated

*The following four candidates were proposed previously and re-evaluated against the same criteria.*

---

## 4. Real-Time E-Commerce Customer Support & Order Recovery Agent

## Criteria Fulfilment

| Criterion | Fulfilment | Rationale |
|---|---|---|
| P1 Continuous Ops | ✅ | Customer complaints arrive 24/7 - the ticket queue never empties |
| P2 Complex Decisions | ✅ | Same complaint gets different treatment based on LTV, complaint history, order value, current promotions, and inventory |
| P3 Dynamic Environment | ✅ | Active promotions, warehouse outages, and supplier changes all shift what resolutions are valid |
| I1 Novice-Friendly | ✅ | Synthetic order + ticket data is trivial; core loop is readable and explainable; Gradio UI in hours |
| I2 Platform Showcase | ⭐⭐⭐ | DLT for ticket ingestion, Delta for order history, MLflow for LTV model, Unity Catalog for policy rules, Model Serving for LLM responses |
| I3 Infinite Depth | ✅ | v1: rule-based resolution → v2: LTV-aware decisions → v3: multi-agent triage + resolver → v4: outcome learning |

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                  TICKET STREAM                              │
│         (web form / email / chat / Auto Loader)             │
└──────────────────────────┬──────────────────────────────────┘
                           │ new support tickets
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                DLT INGESTION PIPELINE                       │
│  raw_tickets → cleaned_tickets → enriched_tickets           │
│  (attach order history, customer LTV, past complaints,      │
│   current promotions, inventory status)                     │
└──────────────────────────┬──────────────────────────────────┘
                           │ enriched ticket
                           ▼
┌─────────────────────────────────────────────────────────────┐
│               TRIAGE AGENT                                  │
│  classify ticket type (delivery / payment / returns)        │
│  score urgency (sentiment + LTV + SLA timer)                │
│  route to → RESOLVER AGENT                                  │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│               RESOLVER AGENT                                │
│                                                             │
│  OBSERVE: enriched ticket + customer profile from Delta     │
│     │                                                       │
│  REASON: what happened? (late / lost / wrong item)          │
│          who is this customer? (LTV score from MLflow)      │
│          what can I offer? (check policy in Unity Catalog)  │
│          what's in stock? (check inventory Delta table)     │
│     │                                                       │
│  DECIDE: auto-resolve   → refund / reship / voucher         │
│          needs info     → send clarification request        │
│          out of policy  → escalate to human agent           │
│     │                                                       │
│  ACT:    generate personalised response (LLM)               │
│          write resolution to Delta                          │
│          update ticket status                               │
│          log full rationale for audit                       │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│           MONITORING DASHBOARD (Lakeview)                   │
│  Resolution rate │ Escalation rate │ Avg handle time        │
│  Customer satisfaction score │ Policy breach flags          │
│  Human override queue                                       │
└─────────────────────────────────────────────────────────────┘
```

## Implementation Approach

**Week 1**
- Day 1–2: Synthetic orders + customers + tickets dataset (Faker), load via Auto Loader into Delta
- Day 3–4: DLT pipeline - raw tickets → enriched with order history + customer LTV
- Day 5: Train simple LTV scoring model in MLflow (RFM - recency, frequency, monetary value)

**Week 2**
- Day 6–7: Agent loop with tools: `get_customer_profile()`, `get_order_status()`, `check_policy()`, `issue_resolution()`, `escalate_to_human()`
- Day 8–9: LLM generates personalised resolution messages, decisions written back to Delta
- Day 10: Gradio UI for live demo + Lakeview dashboard

**Future Iterations:** sentiment-based prioritisation, multi-agent triage/resolver split, A/B testing resolution strategies, outcome tracking (did the customer churn post-resolution?).

---

## 5. Warehouse / Inventory Reorder Agent ⭐ Top Pick

## Criteria Fulfilment

| Criterion | Fulfilment | Rationale |
|---|---|---|
| P1 Continuous Ops | ✅ | Sales tick down inventory every minute; the agent always has something to watch |
| P2 Complex Decisions | ✅ | Reorder involves stock level, sales velocity, upcoming promotions, supplier lead time, alternative supplier cost, and monthly budget cap - simultaneously |
| P3 Dynamic Environment | ✅ | A flash sale or supplier delay can make yesterday's "fine" SKU today's emergency |
| I1 Novice-Friendly | ✅ | Kaggle retail datasets are abundant; the decision loop is the cleanest of all 7 candidates |
| I2 Platform Showcase | ⭐⭐⭐ | Auto Loader → DLT → Delta → MLflow → Agent → Unity Catalog - hits the full stack naturally |
| I3 Infinite Depth | ✅ | v1: threshold rules → v2: ML demand forecast → v3: multi-supplier negotiation → v4: autonomous budget management |

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│               DATA SOURCES                                  │
│   POS sales stream │ Supplier API │ Promotions calendar     │
│              (via Auto Loader / LakeFlow Connect)           │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│               DLT PIPELINE                                  │
│  raw_sales → current_inventory → enriched_inventory         │
│  (attach: sales velocity, days-to-stockout forecast,        │
│   supplier lead time, upcoming promotions, budget status)   │
└──────────────────────────┬──────────────────────────────────┘
                           │ enriched SKU snapshot
                           ▼
┌─────────────────────────────────────────────────────────────┐
│               REORDER AGENT                                 │
│                                                             │
│  OBSERVE: all SKUs below reorder threshold from Delta       │
│     │                                                       │
│  REASON: days-to-stockout (MLflow demand forecast)          │
│          is a promotion coming? (inflate the order qty)     │
│          primary supplier available? (check Unity Catalog)  │
│          does order breach monthly budget cap?              │
│          is there a cheaper alternative supplier?           │
│     │                                                       │
│  DECIDE: routine reorder   → auto-place PO                  │
│          supplier delayed  → switch to alternative          │
│          budget breach     → escalate to procurement        │
│          stockout < 24hrs  → emergency flag + escalate      │
│     │                                                       │
│  ACT:    write PO to Delta                                  │
│          notify supplier (webhook / email)                  │
│          log full decision rationale                        │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│           PROCUREMENT DASHBOARD (Lakeview)                  │
│  Stock health by SKU │ POs raised │ Supplier reliability    │
│  Budget utilisation │ Stockout risk heatmap                 │
│  Human escalation queue │ Agent decision audit log          │
└─────────────────────────────────────────────────────────────┘
```

## Implementation Approach

**Week 1**
- Day 1–2: Load Kaggle retail dataset (e.g. Instacart or UK Retail) into Delta via Auto Loader
- Day 3–4: DLT pipeline - sales → current inventory → enriched with velocity + days-to-stockout
- Day 5: Demand forecasting model in MLflow (start with moving average, upgrade to Prophet if time allows)

**Week 2**
- Day 6–7: Agent loop with tools: `check_inventory()`, `forecast_demand()`, `get_supplier_options()`, `raise_purchase_order()`, `escalate_to_human()`
- Day 8–9: LLM reasoning layer for multi-supplier trade-off decisions, log rationale to Delta
- Day 10: Lakeview dashboard + demo script

**Future Iterations:** seasonal ML models, multi-supplier negotiation, autonomous budget management, carbon footprint-aware ordering.

---

## 6. Network / Infrastructure Anomaly Response Agent

## Criteria Fulfilment

| Criterion | Fulfilment | Rationale |
|---|---|---|
| P1 Continuous Ops | ✅ | Infrastructure metrics stream every few seconds - a 3am spike matters as much as a 3pm one |
| P2 Complex Decisions | ✅ | CPU spike alone means nothing; correlated with rising error rate + recent deployment = rollback candidate |
| P3 Dynamic Environment | ✅ | Traffic patterns, deployments, and new services mean "normal" shifts constantly |
| I1 Novice-Friendly | ⭐⭐ | Convincing demo data requires either real infra or a careful simulator - more setup friction than others |
| I2 Platform Showcase | ⭐⭐⭐ | Auto Loader for log ingestion, DLT for metric enrichment, MLflow anomaly model, Delta for audit, Unity Catalog for service topology |
| I3 Infinite Depth | ✅ | v1: threshold alerts → v2: ML anomaly detection → v3: dependency graph reasoning → v4: predictive alerting |

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│             METRICS & LOG STREAM                            │
│   (Prometheus / CloudWatch / synthetic generator)           │
│    CPU │ Memory │ Latency │ Error rate │ Request volume     │
└──────────────────────────┬──────────────────────────────────┘
                           │ via Auto Loader
                           ▼
┌─────────────────────────────────────────────────────────────┐
│               DLT PIPELINE                                  │
│  raw_metrics → cleaned_metrics → service_snapshot           │
│  (rolling window: last 15 mins, trend, baseline deviation,  │
│   attach: recent deployments, dependency graph)             │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│               ANOMALY DETECTION AGENT                       │
│                                                             │
│  OBSERVE: service_snapshot from Delta                       │
│     │                                                       │
│  REASON: anomaly score from MLflow model                    │
│          correlate across dependent services                │
│          check recent deployment log                        │
│          is this a known pattern? (check historical Delta)  │
│     │                                                       │
│  DECIDE: low severity  → log + monitor                      │
│          medium        → auto-remediate (restart/reroute)   │
│          high          → escalate + generate incident report│
│     │                                                       │
│  ACT:    trigger remediation webhook                        │
│          write structured incident report to Delta          │
│          page on-call engineer with full context            │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│           OPS DASHBOARD (Lakeview)                          │
│  Service health map │ Alert history │ MTTR trend            │
│  Auto-remediation success rate │ Escalation queue           │
└─────────────────────────────────────────────────────────────┘
```

## Implementation Approach

**Week 1**
- Day 1–2: Build a Python metrics simulator - inject anomalies on a schedule (e.g. CPU spike every 2 hours), stream into Delta via Auto Loader
- Day 3–4: DLT pipeline - raw metrics → rolling window snapshot per service
- Day 5: Anomaly detection model in MLflow (Isolation Forest - no labelled data needed)

**Week 2**
- Day 6–7: Agent loop with tools: `get_service_snapshot()`, `score_anomaly()`, `check_recent_deployments()`, `trigger_remediation()`, `generate_incident_report()`
- Day 8–9: LLM generates human-readable incident reports ("Payment service latency up 340% - correlates with auth-service deployment at 14:32 - recommend rollback"), logged to Delta
- Day 10: Dashboard + demo

**Future Iterations:** dependency graph reasoning, predictive alerting before anomaly hits, cross-service root cause analysis, post-incident learning loop.

---

## 7. Food Delivery / Logistics Dispatch Agent

## Criteria Fulfilment

| Criterion | Fulfilment | Rationale |
|---|---|---|
| P1 Continuous Ops | ✅ | Orders pour in constantly; drivers go online/offline; the agent makes decisions every few seconds |
| P2 Complex Decisions | ✅ | Driver assignment weighs proximity, current load, restaurant prep time, traffic, SLA timer, and multi-order routing simultaneously |
| P3 Dynamic Environment | ✅ | Driver cancellations, restaurant closures, rain-driven demand surges - the environment is never static |
| I1 Novice-Friendly | ⭐⭐ | Routing logic adds friction; fake coordinates and travel times work, but it's one more moving part than the others |
| I2 Platform Showcase | ⭐⭐⭐ | Structured Streaming for order/location feeds, DLT for state management, MLflow for prep time prediction, Delta for dispatch log, Lakeview for live map |
| I3 Infinite Depth | ✅ | v1: nearest-driver assignment → v2: multi-order batching → v3: predictive pre-positioning → v4: autonomous demand surge management |

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│              LIVE DATA STREAMS                              │
│   Incoming orders │ Driver locations │ Restaurant status    │
│   Traffic feed │ SLA timers                                 │
└──────────────────────────┬──────────────────────────────────┘
                           │ via Structured Streaming
                           ▼
┌─────────────────────────────────────────────────────────────┐
│               DLT STATE PIPELINE                            │
│  raw_orders → active_orders → enriched_orders               │
│  (attach: nearest drivers, restaurant prep estimate,        │
│   SLA deadline, traffic delay estimate)                     │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│               DISPATCH AGENT                                │
│                                                             │
│  OBSERVE: new order + current driver availability           │
│     │                                                       │
│  REASON: score each available driver                        │
│          (distance + load + route efficiency)               │
│          estimate delivery time vs SLA deadline             │
│          check restaurant prep time (MLflow model)          │
│     │                                                       │
│  DECIDE: assign best driver   → confirm dispatch            │
│          no driver available  → widen search radius         │
│          SLA breach imminent  → escalate + notify customer  │
│          driver cancels       → reassign immediately        │
│     │                                                       │
│  ACT:    write dispatch event to Delta                      │
│          notify driver + customer                           │
│          log full assignment rationale                      │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│           DISPATCH DASHBOARD (Lakeview)                     │
│  Live order map │ SLA breach rate │ Driver utilisation      │
│  Avg delivery time │ Re-assignment rate │ Customer alerts   │
└─────────────────────────────────────────────────────────────┘
```

## Implementation Approach

**Week 1**
- Day 1–2: Simulate orders + driver positions (random lat/long within a city bounding box), stream into Delta
- Day 3–4: DLT pipeline - orders enriched with nearest drivers, prep time estimate, SLA deadline
- Day 5: Restaurant prep time prediction model in MLflow (simple regression on cuisine type + order size)

**Week 2**
- Day 6–7: Agent loop with tools: `get_available_drivers()`, `score_driver_assignment()`, `assign_order()`, `handle_cancellation()`, `flag_sla_breach()`
- Day 8–9: LLM generates customer-facing notifications and internal dispatch notes, all logged to Delta
- Day 10: Lakeview live map dashboard + demo

**Future Iterations:** multi-order batching per driver, predictive driver pre-positioning, demand surge detection, restaurant reliability scoring.

---

# Appendix - Recommended Databricks Stack (All Candidates)

| Layer | Component | Purpose |
|---|---|---|
| Ingestion | Auto Loader / LakeFlow Connect | Stream files and events into Delta |
| Pipeline | Delta Live Tables (DLT) | Clean, enrich, and model data declaratively |
| Storage | Delta Lake + Unity Catalog | Single governed source of truth for all data |
| ML | MLflow + Model Serving | Train, register, and serve scoring models |
| Agent | Mosaic AI Agent Framework | Orchestrate observe → reason → act loop |
| LLM | Claude / DBRX via Model Serving | Natural language reasoning and generation |
| Memory | Delta tables | Persistent agent state and decision audit log |
| UI | Gradio / Databricks Apps | Demo interface |
| Monitoring | Lakeview Dashboards | Agent performance and business KPI visibility |

---

# Appendix - Universal Agent Loop

All 7 candidates share the same core pattern. Build this first, then layer on domain logic.

```
┌─────────────┐
│   TRIGGER   │  New event arrives (transaction, ticket, vital, order...)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   OBSERVE   │  Read enriched context from Delta
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   REASON    │  Score with MLflow model + LLM reasoning over edge cases
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   DECIDE    │  Auto-act / soft intervention / escalate to human
└──────┬──────┘
       │
       ▼
┌─────────────┐
│     ACT     │  Write decision + rationale back to Delta, trigger action
└──────┬──────┘
       │
       ▼
┌─────────────┐
│     LOG     │  Full audit trail in Delta - every decision, every reason
└─────────────┘
```

---

*Document compiled from hackathon ideation session - Databricks Agentic AI, May 2026.*