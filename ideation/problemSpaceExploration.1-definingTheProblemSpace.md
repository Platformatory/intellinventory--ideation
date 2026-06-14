<h1>Agentic AI for Data Platform Engineering:<br><i>A Strategic Exploration</i></h1>

***Defining the Problem Space***

[Based on a conversation with Claude AI (Sonnet 4.6)](https://claude.ai/share/17ac0ac7-6fe3-4c2c-b38b-1954aec7f8b1)

---

**Contents**:

- [Overview](#overview)
- [1. Framing Agentic AI](#1-framing-agentic-ai)
  - [Definition](#definition)
  - [LLMs Are Not Definitional](#llms-are-not-definitional)
  - [Why "No Black-Box LLMs" Is a Principled Constraint](#why-no-black-box-llms-is-a-principled-constraint)
- [2. Databricks' Agentic AI Ecosystem](#2-databricks-agentic-ai-ecosystem)
  - [2.1 AI Functions](#21-ai-functions)
  - [2.2 AI/BI Genie](#22-aibi-genie)
  - [2.3 Agent Bricks *(Flagship - launched June 2025)*](#23-agent-bricks-flagship---launched-june-2025)
  - [2.4 Agent Framework](#24-agent-framework)
  - [2.5 Mosaic AI *(Umbrella Platform)*](#25-mosaic-ai-umbrella-platform)
  - [2.6 Lakewatch *(Newest - March 2026)*](#26-lakewatch-newest---march-2026)
  - [2.7 Notable Characteristic of the Stack](#27-notable-characteristic-of-the-stack)
- [3. Mapping the Problem Space](#3-mapping-the-problem-space)
  - [General Fit: Where Databricks Agents Shine](#general-fit-where-databricks-agents-shine)
- [4. The Organisational Context](#4-the-organisational-context)
- [5. The Arrived-At Direction: Agentic Platform Intelligence](#5-the-arrived-at-direction-agentic-platform-intelligence)
  - [The Core Insight](#the-core-insight)
  - [The Opportunity](#the-opportunity)
  - [Specific Capability Areas](#specific-capability-areas)
  - [Why This Is the Right Frame](#why-this-is-the-right-frame)
- [6. Open Questions / Next Steps](#6-open-questions--next-steps)

---

# Overview

THE CORE CONSTRAINT OF DESIRE:

***Maximum Impact + Minimum Specificity***

---

This document captures a strategic exploration into the application of agentic AI - specifically Databricks' agentic AI stack - to the domain of **data platform engineering itself**. The core thesis arrived at: rather than building domain-specific agents (for healthcare, finance, retail, etc.), the highest-impact, most horizontally applicable opportunity for a data platform engineering organisation is to build an **agentic platform observability and intelligence layer** - a system of agents that watches, reasons about, and acts on the platform's own operational signals.

This is compelling because:

- It is **domain-agnostic**: every client benefits regardless of what their data is *about*
- It is **Databricks-native**: Unity Catalog, MLflow, Mosaic AI, and Delta logs are all natural inputs
- It is **productisable**: built once, deployable as an intelligence layer on top of any platform engineered
- It plays to the organisation's **core competency**: data platform engineering, not a vertical domain

---

# 1. Framing Agentic AI

## Definition

Agentic AI refers to autonomous, adaptive systems that:

- Obtain information from their environment
- Create and execute workflows dynamically
- Perform actions - including communicating with other agents or using tools

The key distinction: **adaptive**, meaning not constrained by training data or a strictly predefined set of workflows.

## LLMs Are Not Definitional

A common misconception is that agentic AI requires LLMs at the core. It does not. LLMs are one *implementation* of the decision layer. Alternatives include:

- **Rule engines / expert systems** - explicit, auditable logic
- **Finite state machines / hierarchical state machines** - structured, predictable control flow
- **Planning algorithms** (STRIPS, PDDL-style) - the agent reasons about goals and actions
- **Reinforcement learning** - learned but inspectable policy, especially tabular or tree-based
- **Behaviour trees** - used heavily in robotics and game AI; composable, readable, debuggable

These can all be agentic - perceive, adapt, plan, act, use tools - without hiding decision logic inside a neural network.

## Why "No Black-Box LLMs" Is a Principled Constraint

- Decisions are opaque - you cannot audit *why* the agent chose an action
- Behaviour is stochastic and hard to guarantee
- Expensive to run in tight loops
- Extending or modifying behaviour is indirect (prompting, fine-tuning)

---

# 2. Databricks' Agentic AI Ecosystem

Databricks' stack follows a **"start simple, then specialise"** philosophy, with a progressive set of offerings:

## 2.1 AI Functions

- Optimised for automated, system-to-system workflows
- Not designed for human-in-the-loop interaction
- Ideal for high-volume, repetitive text jobs: transcript summarisation, ticket classification, entity extraction from financial reports

## 2.2 AI/BI Genie

- Conversational interface for business intelligence
- Lets non-technical users query structured data in plain English
- An evolution beyond traditional dashboards

## 2.3 Agent Bricks *(Flagship - launched June 2025)*

The centrepiece of Databricks' agentic offering. Automates the hardest parts of agent development:

1. Automatically generates task-specific evaluation suites and LLM judges
2. Creates synthetic data resembling the customer's own data to supplement learning
3. Searches across optimisation techniques to refine the agent
4. Presents the customer with quality/cost trade-off options to choose from

**Key integrations:**
- OpenAI models (via a $100M partnership, September 2025)
- Anthropic Claude 3.5 Sonnet (February 2025)
- Google Gemini (via Google Cloud alliance)

**Proven use cases:** AstraZeneca (400k clinical trial documents), Hawaiian Electric (40k legal documents), Flo Health, Mastercard, AT&T.

## 2.4 Agent Framework

- Fully custom, programmatic agent development
- Supports any model - classical ML through to GenAI
- Built-in evaluation with AI judges
- Unified deployment for agents, GenAI, and classical ML models

## 2.5 Mosaic AI *(Umbrella Platform)*

The substrate underlying all of the above:

- **Vector Search** - high-performance vector DB with real-time data syncing
- **Managed MLflow** - enterprise-grade ML lifecycle and observability
- **Lakehouse Monitoring** - anomaly detection, freshness tracking, quality signals
- **Unity Catalog** - end-to-end governance: access controls, rate limits, data lineage

## 2.6 Lakewatch *(Newest - March 2026)*

- Agentic security platform
- Deploys AI agents to automate detection, triage, and threat hunting
- Designed to defend against AI-powered attacks at "machine speeds"
- Currently in private preview

## 2.7 Notable Characteristic of the Stack

Databricks' entire agentic stack is **deeply LLM-centric**. The decision layer at every tier is a neural network. Their architectural writing defines agentic AI as systems that "leverage large language models." Their value-add is in the *scaffolding* - governance, evaluation, data grounding, deployment - but the reasoning core is always an LLM. This is not a gap Databricks is trying to fill.

---

# 3. Mapping the Problem Space

## General Fit: Where Databricks Agents Shine

The strongest use cases share a common shape:

> *Large volumes of messy enterprise data + a repetitive human decision loop + a need for auditability and governance*

**By data complexity:**

- Unstructured → structured pipelines (documents, PDFs, contracts)
- Multi-source reasoning across heterogeneous data
- Streaming and real-time decision making (fraud, anomaly detection, dynamic pricing)

**By organisational bottleneck:**

- Analytics democratisation for non-technical users
- Ops automation at scale (ticket triage, compliance checking, financial reconciliation)
- Decision augmentation - surfacing recommendations to humans

**By industry traction:**

- Financial services: fraud, risk, onboarding, regulatory reporting
- Healthcare/life sciences: clinical data extraction, trial monitoring
- Retail/CPG: demand forecasting, supplier data ingestion, personalisation
- Manufacturing: predictive maintenance, supply chain signal processing

---

# 4. The Organisational Context

The organisation in question is a **data platform engineering firm** - domain-agnostic, building and operating data platforms across a variety of client verticals.

The project objective: **strengthen platform engineering capabilities and offerings** using Databricks' agentic AI stack - not solve a domain-specific problem, but add intelligence to the platform engineering layer itself.

This reframes the question entirely: the "domain" the agents reason about is not healthcare or finance - it is **the data platform itself**.

---

# 5. The Arrived-At Direction: Agentic Platform Intelligence

## The Core Insight

Every data platform, regardless of domain, generates enormous amounts of signal about itself:

- Pipeline execution logs
- Schema change history
- Data quality metrics
- Cost and resource utilisation
- Query performance patterns
- Lineage and metadata

Almost none of this signal is being intelligently acted on. Engineers consume it reactively - when something breaks, when a stakeholder complains, when a cost spike shows up on a bill.

## The Opportunity

An **agentic platform observability and intelligence layer**: a system of agents that:

1. **Watches** the platform continuously - pipelines, data quality, costs, schema changes, usage patterns
2. **Reasons** about what it observes - root cause, blast radius, priority
3. **Acts or advises** - autonomously fixes what it can; surfaces what it cannot, with a diagnosis already done

## Specific Capability Areas

| Capability | Description |
|---|---|
| **Agentic Pipeline Management** | Monitors health, diagnoses failures, adapts to schema drift - reducing human on-call toil |
| **Intelligent Data Onboarding** | Automates profiling, schema inference, lineage mapping, and quality rule generation for new sources |
| **Self-Healing / Self-Optimising** | Observes performance and cost metrics; takes corrective action (repartitioning, query rewriting, cluster scaling) |
| **Agentic Data Quality** | Learns what "good" looks like per dataset; flags anomalies, traces to root cause, fixes or escalates with diagnosis |
| **Metadata & Governance Automation** | Auto-generates documentation, tags, lineage, and data contracts as data flows through - keeping the catalog current |

## Why This Is the Right Frame

- **Horizontal, not vertical** - applicable to every client engagement regardless of industry
- **Databricks-native** - Unity Catalog, MLflow, Mosaic AI, Delta logs are all natural inputs to this agent layer
- **Productisable** - built once, offered as an intelligence layer on top of any platform engineered
- **Plays to core competency** - the organisation is not pretending to be a domain expert; it is being an excellent platform engineering company that ships intelligence

---

# 6. Open Questions / Next Steps

- What is the **MVP slice** of the agentic platform intelligence layer that is demonstrable and defensible?
- Which of the five capability areas (pipeline management, onboarding, self-healing, data quality, metadata/governance) has the highest signal-to-noise ratio as a starting point?
- What does the **human-in-the-loop boundary** look like - which actions should the agent take autonomously vs. surface for approval?
- How does this layer get **packaged and offered** to clients - as a managed service, an accelerator, a framework?