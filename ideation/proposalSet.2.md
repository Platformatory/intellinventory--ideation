<h1>Databricks Agentic AI - Hackathon Idea Catalogue 2</h1>

***Agentic AI for Data Platform Engineering***

> **Constraints**: 2-week prototype, data engineering background, agentic AI novices, minimal domain specificity, max horizontal impact, extensible by design.

---

**Contents**:

- [Framing Note](#framing-note)
- [Proposals at a Glance](#proposals-at-a-glance)
  - [How To Read These Proposals](#how-to-read-these-proposals)
  - [How They Fit Together](#how-they-fit-together)
- [Proposal 1 - Agentic Pipeline Health Narrator](#proposal-1---agentic-pipeline-health-narrator)
  - [*"Turn your pipeline logs into a structured, reasoned daily briefing"*](#turn-your-pipeline-logs-into-a-structured-reasoned-daily-briefing)
  - [Problem Statement](#problem-statement)
  - [Why This, Specifically](#why-this-specifically)
  - [Architecture Sketch](#architecture-sketch)
  - [Extensibility Path](#extensibility-path)
  - [Ethical-Practical Alignment](#ethical-practical-alignment)
  - [2-Week Feasibility Assessment](#2-week-feasibility-assessment)
- [Proposal 2 - Agentic Data Onboarding Assessor](#proposal-2---agentic-data-onboarding-assessor)
  - [*"First-look intelligence for every new data source"*](#first-look-intelligence-for-every-new-data-source)
  - [Problem Statement](#problem-statement-1)
  - [Why This, Specifically](#why-this-specifically-1)
  - [Architecture Sketch](#architecture-sketch-1)
  - [Extensibility Path](#extensibility-path-1)
  - [Ethical-Practical Alignment](#ethical-practical-alignment-1)
  - [2-Week Feasibility Assessment](#2-week-feasibility-assessment-1)
- [Proposal 3 - The Platform Archaeologist](#proposal-3---the-platform-archaeologist)
  - [*"An agent that reads your platform's operational history and tells you what kind of platform you actually have"*](#an-agent-that-reads-your-platforms-operational-history-and-tells-you-what-kind-of-platform-you-actually-have)
  - [The Quirk](#the-quirk)
  - [Why This, Specifically](#why-this-specifically-2)
  - [Architecture Sketch](#architecture-sketch-2)
  - [Extensibility Path](#extensibility-path-2)
  - [Ethical-Practical Alignment](#ethical-practical-alignment-2)
  - [2-Week Feasibility Assessment](#2-week-feasibility-assessment-2)
- [Proposal 4 - The Data Contract Diplomat](#proposal-4---the-data-contract-diplomat)
  - [*"An agent that negotiates data contracts from observed behaviour, not human effort"*](#an-agent-that-negotiates-data-contracts-from-observed-behaviour-not-human-effort)
  - [The Quirk](#the-quirk-1)
  - [Why This, Specifically](#why-this-specifically-3)
  - [Architecture Sketch](#architecture-sketch-3)
  - [Extensibility Path](#extensibility-path-3)
  - [Ethical-Practical Alignment](#ethical-practical-alignment-3)
  - [2-Week Feasibility Assessment](#2-week-feasibility-assessment-3)
- [Proposal 5 - The Platform Whisperer](#proposal-5---the-platform-whisperer)
  - [*"Ask your Databricks platform anything, in plain English, and get an answer it can actually justify"*](#ask-your-databricks-platform-anything-in-plain-english-and-get-an-answer-it-can-actually-justify)
  - [The Quirk](#the-quirk-2)
  - [Why This, Specifically](#why-this-specifically-4)
  - [Architecture Sketch](#architecture-sketch-4)
  - [Extensibility Path](#extensibility-path-4)
  - [Ethical-Practical Alignment](#ethical-practical-alignment-4)
  - [2-Week Feasibility Assessment](#2-week-feasibility-assessment-4)
- [Proposal 6 - The Platform Inquisitor](#proposal-6---the-platform-inquisitor)
  - [*"An agent that interviews your engineers to extract what they know - and turns it into platform memory before they walk out the door"*](#an-agent-that-interviews-your-engineers-to-extract-what-they-know---and-turns-it-into-platform-memory-before-they-walk-out-the-door)
  - [The Quirk](#the-quirk-3)
  - [What It Actually Does](#what-it-actually-does)
  - [Why This, Specifically](#why-this-specifically-5)
  - [Architecture Sketch](#architecture-sketch-5)
  - [Extensibility Path](#extensibility-path-5)
  - [Ethical-Practical Alignment](#ethical-practical-alignment-5)
  - [2-Week Feasibility Assessment](#2-week-feasibility-assessment-5)
- [Full Comparison](#full-comparison)
  - [Recommendation](#recommendation)
- [Shared Design Principles Across All Proposals](#shared-design-principles-across-all-proposals)

---

# Framing Note

Before the proposals: a constraint that shapes all of them.

Given the literature review's finding that *accountability infrastructure must precede autonomous action*, and given the autonomy tier framework established earlier, all proposals below are scoped to **Tier 2: Advise** - the agent surfaces a structured recommendation with reasoning, and a human decides. This is not a limitation for a prototype; it is the architecturally correct starting point. Autonomous action (Tier 3+) can only be responsibly added once the audit trail and oversight mechanisms are proven.

This also happens to be the right scope for novices: a single-agent, single-loop system with deterministic inputs and a structured output is comprehensible end-to-end. The complexity lives in the *data engineering*, which is the team's strength - not in the agent architecture.

---

# Proposals at a Glance

Six proposals in total. The first two are grounded and conventional in their approach - solid engineering problems with clear value. The latter four are deliberately unconventional: the same rigour, but approached from surprising angles. All six share the same constraints, the same ethical-practical framework, and the same Databricks-native substrate.

| # | Name | One Line | Quirk Factor | Demo Infra | Autonomy Tier |
|---|---|---|---|---|---|
| **1** | Pipeline Health Narrator | Turns pipeline logs into a daily reasoned briefing | Low | Medium (scheduled job) | Tier 2 |
| **2** | Data Onboarding Assessor | Profiles and assesses every new data source on arrival | Low | Low-Medium (event trigger) | Tier 2 |
| **3** | Platform Archaeologist | Reads your platform's full history and tells you what kind of platform you have | Medium | Medium (batch job) | Tier 1–2 |
| **4** | Data Contract Diplomat | Drafts data contracts from observed producer-consumer behaviour, not human effort | Medium-High | Low-Medium | Tier 2 |
| **5** | Platform Whisperer | Ask your platform anything in plain English; it answers only what it can evidence | High | Very Low (notebook + chat) | Tier 2 |
| **6** | Platform Inquisitor | Interviews your engineers to extract tacit knowledge before it walks out the door | Very High | Very Low (notebook + chat) | Tier 1–2 |

## How To Read These Proposals

**Proposals 1 and 2** are the sensible starting points. They are well-scoped, predictable in their value, and straightforward to justify internally or to a client. If the goal is a safe, demonstrable first build, start here.

**Proposals 3 and 4** are the strategic ones. They address problems that every platform has - historical opacity, unwritten contracts - that no existing tooling solves well. Higher novelty, slightly higher build complexity, but still within the 2-week constraint.

**Proposals 5 and 6** are the demo-first ones. Minimal infrastructure, immediate visceral impact, and a conversational interface that makes the value legible to any audience - technical or not. Proposal 5 answers questions; Proposal 6 asks them. Together they represent the human-platform interface layer that the other four proposals feed into over time.

## How They Fit Together

These are not six competing alternatives. They are six slices of the same larger system - the Agentic Platform Intelligence Layer identified as the core direction in the problem space exploration. Each proposal is a natural kernel that, if built with the shared design principles in mind, becomes a composable component of the whole:

```
Platform Inquisitor  ──→  [Institutional Knowledge Base]  ──→  Platform Whisperer
Platform Archaeologist ──→  [Historical Signal Layer]      ──→  Pipeline Health Narrator
Data Onboarding Assessor ──→  [Asset Registry + Quality]  ──→  Data Contract Diplomat
```

The recommended entry point remains **Proposal 2** for a first prototype - tightest scope, clearest output, easiest to evaluate. But the choice of entry point matters less than the shared infrastructure it is built on.

---

# Proposal 1 - Agentic Pipeline Health Narrator
## *"Turn your pipeline logs into a structured, reasoned daily briefing"*

## Problem Statement
Data pipelines generate rich operational signals - run durations, failure rates, record counts, schema changes, SLA breaches - but these signals are consumed *reactively*: an engineer looks when something breaks. There is no continuous, reasoned synthesis of what the platform is telling you about itself.

**Proposed system**: A single-agent loop that ingests pipeline metadata and operational logs from a Databricks environment on a scheduled cadence, reasons over them, and produces a structured, human-readable *health narrative* - a prioritised briefing of what changed, what degraded, what is anomalous, and what warrants attention. No autonomous action. Pure Tier 2: observe and advise.

## Why This, Specifically

**Connects to the problem space:** This is the kernel of the "Agentic Platform Observability and Intelligence Layer" identified as the core direction. It instantiates the *watch* and *reason* steps without yet requiring the *act* step.

**Connects to the literature:**
- *Trajectory-level explainability* (Vector Institute, 2025): the agent's output is itself an explanation of system behaviour over time - a narrative trace, not a point-in-time alert.
- *PROV-AGENT* (arXiv, 2025): the agent's reasoning over pipeline metadata is a form of provenance - what happened, when, with what effect. Every briefing is an auditable record.
- *Accountability gap*: because the agent only advises, the accountability question is clean. A human acts on the briefing; the agent's job is to make that human's decision better-informed.

**Connects to Databricks primitives** (from the uploaded document):
- **Episodic memory** → MLflow experiment/run history as the primary input
- **Semantic memory** → Vector Search over historical anomaly descriptions, for contextualised comparison
- **Orchestration** → A single Databricks Workflow (Jobs) on a schedule; no inter-agent complexity

**Data engineering leverage:** The hardest part is not the agent - it is curating the right signals from Delta tables, MLflow, and Unity Catalog. This is precisely where the team's existing skills apply. The agent layer is thin and learnable.

## Architecture Sketch

```
[Delta / MLflow / Unity Catalog]
         ↓
  [Signal Extraction Job]        ← pure data engineering
  (run stats, schema diffs,
   SLA deltas, record counts)
         ↓
  [Single Agent - LLM call]      ← thin agentic layer
  Prompt: "Given these signals,
  produce a prioritised health
  narrative with reasoning."
         ↓
  [Structured Output]            ← JSON + Markdown
  (priority issues, anomalies,
   trend observations, open
   questions for human review)
         ↓
  [Delivery + Logging]           ← MLflow Tracing logs
  (Slack / email / dashboard)     the full reasoning chain
```

## Extensibility Path

| Iteration | Addition | Autonomy Tier |
|---|---|---|
| **v1 (prototype)** | Scheduled narrative briefing | Tier 1–2: Observe + Advise |
| **v2** | Agent compares against baseline; flags regressions specifically | Tier 2 |
| **v3** | Agent proposes specific remediation actions (not executes) | Tier 2+ |
| **v4** | Agent executes low-risk remediations (job restarts) with notification | Tier 3: Act with notification |
| **v5** | Multi-agent: specialist agents per pipeline domain, supervisor narrates | Tier 3–4 |

## Ethical-Practical Alignment
- **Accountability**: every briefing is logged with its full reasoning chain via MLflow Tracing. Auditable from day one.
- **Autonomy boundary**: Tier 2 is hardcoded at the prototype stage. The agent cannot act.
- **Configurable objectives**: the briefing prompt can be parameterised - "prioritise cost signals" vs. "prioritise data quality signals" vs. "prioritise SLA signals" - making the optimisation objective explicit and client-adjustable.
- **Systemic risk**: none at this tier. The agent reads; it does not write.

## 2-Week Feasibility Assessment
| Task | Effort |
|---|---|
| Signal extraction from MLflow + Delta | Days 1–4 (core data engineering) |
| Prompt design + single LLM call via Mosaic AI | Days 3–5 |
| Structured output schema + logging | Days 5–7 |
| Scheduling via Databricks Workflows | Day 8 |
| MLflow Tracing integration for audit | Days 8–10 |
| Review, refinement, documentation | Days 11–14 |

**Verdict: Achievable.** The agent layer is a single LLM call with a well-designed prompt. The complexity is in the signal extraction - which is data engineering.

---

# Proposal 2 - Agentic Data Onboarding Assessor
## *"First-look intelligence for every new data source"*

## Problem Statement
When a new data source arrives on a platform, an engineer must manually profile it: inspect schemas, assess quality, flag anomalies, check for PII, estimate completeness. This is repetitive, time-consuming, and inconsistently done. The output - if it exists - is usually a one-off note, not a structured, queryable record.

**Proposed system**: A single-agent loop triggered on new dataset registration (or on-demand) that profiles an incoming dataset, reasons about its quality and fitness, and produces a structured *onboarding assessment report* - schema summary, quality flags, potential PII signals, anomaly observations, and a recommended readiness verdict. Again, Tier 2: the agent assesses and advises. A human approves.

## Why This, Specifically

**Connects to the problem space:** The "Intelligent Data Onboarding" capability identified in the problem space mapping. It directly addresses the toil that every data platform engineer recognises.

**Connects to the literature:**
- *Explainability by design*: the assessment report is the explanation. Every flag has a stated reason. This is trajectory-level reasoning applied to data, not agent behaviour.
- *Conflict between explainability and accountability* (arXiv): since the agent produces a report rather than a decision, the accountability for acting on it stays clearly with the human. The XAI-as-scapegoat failure mode is avoided by design.
- *Configurable objectives* (ACM FAccT 2025): what the agent prioritises in its assessment - quality? compliance? cost of storage? - is a parameter, not a hardcoded value.

**Connects to Databricks primitives:**
- **Semantic memory** → Vector Search over a library of known PII patterns, quality rule templates, and schema fingerprints from previous onboardings
- **Structured state** → Delta table as the persistent registry of all onboarding assessments (queryable history)
- **Orchestration** → triggered by Unity Catalog registration event or manual invocation; single-agent, no inter-agent complexity

**Data engineering leverage:** Dataset profiling (statistics, null rates, cardinality, type inference) is pure data engineering - Great Expectations, Databricks' built-in profiling, or custom SQL. The agent layer interprets these outputs, it does not compute them.

## Architecture Sketch

```
[New Dataset Registered]
  (Unity Catalog event / manual trigger)
         ↓
  [Profiling Job]                ← pure data engineering
  (schema, nulls, cardinality,
   value distributions, PII scan)
         ↓
  [Single Agent - LLM call]      ← thin agentic layer
  Context: profiling output
  + Vector Search over historical
    assessments for comparison
  Prompt: "Assess this dataset's
  readiness. Flag issues with
  reasoning. Recommend verdict."
         ↓
  [Structured Assessment Report] ← JSON output
  (quality score, flags + reasons,
   PII likelihood, readiness verdict,
   suggested next steps)
         ↓
  [Persisted to Delta]           ← queryable history
  + MLflow Tracing log           ← full reasoning audit trail
  + Human review interface
```

## Extensibility Path

| Iteration | Addition | Autonomy Tier |
|---|---|---|
| **v1 (prototype)** | On-demand assessment report | Tier 2: Advise |
| **v2** | Auto-triggered on Unity Catalog registration | Tier 2 |
| **v3** | Agent compares new dataset to similar historical ones; identifies schema drift | Tier 2 |
| **v4** | Agent auto-applies suggested quality rules (with notification) | Tier 3 |
| **v5** | Agent negotiates data contracts with upstream source agents | Tier 4 (multi-agent) |

## Ethical-Practical Alignment
- **Accountability**: assessment reports are immutable, Delta-persisted records with full reasoning. The human who acted on the report and their decision are logged alongside.
- **Autonomy boundary**: the agent produces a verdict recommendation; a human approves before the dataset is admitted to the platform.
- **Configurable objectives**: assessment priorities are parameterised per client - a regulated financial client weights PII and compliance flags highest; a research client weights completeness and schema richness.
- **Systemic risk**: minimal - the agent reads a dataset and writes a report. No downstream system is modified.

## 2-Week Feasibility Assessment
| Task | Effort |
|---|---|
| Dataset profiling pipeline | Days 1–4 |
| Vector Search setup with sample historical assessments | Days 3–6 |
| Prompt design + LLM call + structured output schema | Days 5–8 |
| Delta persistence + MLflow Tracing | Days 7–9 |
| Trigger mechanism (manual or Unity Catalog event) | Days 9–11 |
| Review, refinement, documentation | Days 11–14 |

**Verdict: Achievable**, and arguably slightly more self-contained than Proposal 1 since the trigger is event-based rather than requiring scheduled signal aggregation.

---

# Proposal 3 - The Platform Archaeologist
## *"An agent that reads your platform's operational history and tells you what kind of platform you actually have"*

## The Quirk

Most agentic AI proposals are about the future - predict, prevent, optimise. This one goes in the opposite direction. The agent's job is to **excavate the past**: to read the full operational history of a Databricks platform - MLflow run history, Delta table changelogs, job execution logs, Unity Catalog audit logs - and produce a **character portrait of the platform itself**.

Not a health report. Not a list of issues. A *characterisation*. Things like:

- "This platform has a reliability problem specifically on weekend runs - Saturday failures are 3.4× the weekday rate, with no corresponding incident tickets, suggesting they go unnoticed"
- "Dataset `customer_events` has been schema-patched 11 times in 6 months. It is the most structurally unstable asset on this platform."
- "Three pipelines account for 71% of all compute cost. None of them have SLA definitions."
- "This platform has never had a formally documented data contract. It has, however, had 47 informal schema renegotiations embedded in pipeline code."

The agent is doing **forensic platform anthropology**. It treats the platform's history as a text to be interpreted, not a set of metrics to be monitored.

## Why This, Specifically

**Connects to the problem space:** The most universal pain point in data platform engineering is inheriting a platform and spending weeks figuring out what it is. This agent does that work in an afternoon.

**Connects to the literature:**
- *Trajectory-level explainability* (Vector Institute, 2025): behaviour over time tells you things that point-in-time snapshots cannot. The Archaeologist applies this to the platform itself, not to an individual agent.
- *PROV-AGENT* (2025): provenance records are the primary analytical substrate. The agent's inputs are provenance; its output is a provenance interpretation.
- *Perspectival homogenisation* (ACM FAccT 2025): value decisions embedded in systems go unquestioned. The Archaeologist makes those decisions visible - "your platform has implicitly decided that weekend failures don't matter, because nobody has ever responded to one."

**Connects to Databricks primitives** (from the uploaded document):
- **Episodic memory** → MLflow run history as the primary analytical record
- **Semantic memory** → Vector Search over anomaly descriptions for contextualised comparison
- **Structured state** → Delta table changelogs via `DESCRIBE HISTORY` as the archaeological record
- **Orchestration** → single on-demand Databricks Job; no scheduling or inter-agent complexity at prototype stage

## Architecture Sketch

```
[MLflow run history]
[Delta DESCRIBE HISTORY]
[Unity Catalog audit logs]
[Databricks Jobs execution logs]
         ↓
  [Historical Signal Extractor]   ← pure data engineering
  (temporal aggregations, change
   frequency, cost attribution,
   failure pattern analysis)
         ↓
  [Single Agent - LLM call]       ← thin agentic layer
  Prompt: "You are reading the
  operational history of a data
  platform. Characterise it.
  What are its habits, its weak
  points, its implicit priorities?
  Cite specific evidence."
         ↓
  [Platform Character Portrait]   ← structured + narrative output
  (findings with evidence,
   implicit vs. stated priorities,
   stability profile per asset,
   institutional knowledge summary)
         ↓
  [Delta-persisted + MLflow Traced]
```

## Extensibility Path

| Iteration | Addition | Autonomy Tier |
|---|---|---|
| **v1 (prototype)** | On-demand character portrait | Tier 1–2: Observe + Advise |
| **v2** | Scheduled re-runs to track how the platform's character changes over time | Tier 2 |
| **v3** | Cross-client comparison - portrait across multiple platforms to identify patterns | Tier 2 |
| **v4** | Portrait feeds as background context into Platform Health Narrator | Tier 2 |
| **v5** | Portrait initialises memory layer for full multi-agent platform intelligence system | Tier 3–4 |

## Ethical-Practical Alignment
- **Accountability**: every portrait is Delta-persisted with full evidence citations. The agent makes no claims it cannot trace to a source record.
- **Autonomy boundary**: pure read. The agent produces a document; it touches nothing.
- **Configurable objectives**: the analytical lens is a parameter - cost lens, reliability lens, governance lens, or data quality lens. Same history, different portrait.
- **Systemic risk**: none. No writes to operational systems.

## 2-Week Feasibility Assessment
| Task | Effort |
|---|---|
| Historical signal extraction (MLflow, Delta history, Jobs logs) | Days 1–5 (core data engineering) |
| Temporal aggregation and pattern computation | Days 3–6 |
| Prompt design + LLM call + structured output schema | Days 5–8 |
| Delta persistence + MLflow Tracing | Days 8–10 |
| Review, refinement, documentation | Days 11–14 |

**Verdict: Achievable.** The signal extraction is the hardest part and is pure data engineering. The agent call is a single, richly-contextualised LLM invocation.

---

# Proposal 4 - The Data Contract Diplomat
## *"An agent that negotiates data contracts from observed behaviour, not human effort"*

## The Quirk

Data contracts - formal agreements between data producers and consumers about schema, quality, freshness, and semantics - are one of the most important and most consistently *ignored* practices in data platform engineering. Everyone agrees they're good. Almost nobody does them. The reason is not laziness; it's friction.

The quirky inversion: **what if the platform itself was a party to the negotiation?**

The agent reads both sides - the producer dataset's actual behaviour (schema, quality history, update cadence, null rates) and the consumer pipeline's actual expectations (what fields it accesses, how it breaks when things change, what SLAs it implies) - and drafts a contract that reflects **what the relationship already is**, not what someone wishes it would be. Then it flags the gaps: where the producer's actual behaviour diverges from what the consumer implicitly assumes.

It is, essentially, a **couples therapist for datasets and pipelines**.

## Why This, Specifically

**Connects to the problem space:** Every data platform accumulates implicit contracts - assumptions baked into transformation logic, hardcoded column names, silent expectations about null rates. These are contracts that were never written down. When they break, nobody knows why, because nobody knew they existed. The Diplomat makes them visible.

**Connects to the literature:**
- *Accountability gap*: unwritten contracts are an accountability vacuum. The Diplomat creates the agreement retroactively and prevents the next one.
- *Configurable objectives* (ACM FAccT 2025): what the contract prioritises - schema stability, freshness SLAs, null tolerance - is a negotiating parameter. Different consumers have different needs.
- *PROV-AGENT* (2025): the contract *is* a provenance document - what was agreed, when, between which parties, on what evidence.

**Connects to Databricks primitives:**
- **Semantic memory** → Vector Search over existing contracts for consistency checking
- **Structured state** → Delta table as the contract registry; Unity Catalog as the natural home for published contracts
- **Orchestration** → triggered by a new pipeline registration or on-demand; single-agent

**Data engineering leverage:** Schema comparison, null rate analysis, field access pattern extraction from pipeline code - all data engineering. The agent interprets and drafts; it does not compute.

## Architecture Sketch

```
[Producer dataset]              [Consumer pipeline]
(schema, quality history,       (field access patterns,
 update cadence, null rates,     transformation logic,
 Delta DESCRIBE HISTORY)         job failure history)
         ↓                               ↓
         └──────────[Signal Extractor]───┘
                    (both sides profiled
                     and compared)
                           ↓
              [Single Agent - LLM call]
              "Here is what the producer
               actually does. Here is what
               the consumer actually assumes.
               Draft the contract that reflects
               this relationship. Flag every
               point of divergence."
                           ↓
              [Draft Data Contract]        ← structured output
              (agreed schema, SLAs,        in Data Contract Spec
               quality thresholds,         open standard format
               divergence flags,
               suggested negotiation
               points for human review)
                           ↓
              [Human review + sign-off]   ← Tier 2 boundary
              [Persisted to Unity Catalog]
              [MLflow Traced]
```

## Extensibility Path

| Iteration | Addition | Autonomy Tier |
|---|---|---|
| **v1 (prototype)** | On-demand contract draft for one producer-consumer pair | Tier 2: Advise |
| **v2** | Batch contract generation across the full platform graph | Tier 2 |
| **v3** | Automated re-negotiation alerts when schemas drift post-contract | Tier 2 |
| **v4** | Agent auto-applies contract quality rules (with notification) | Tier 3 |
| **v5** | Platform contract graph - a live map of all agreements, health, and divergence history | Tier 3–4 |

## Ethical-Practical Alignment
- **Accountability**: contracts are immutable Unity Catalog records. Human sign-off is the terminal step before publication.
- **Autonomy boundary**: the agent drafts; a human approves before any contract takes effect.
- **Configurable objectives**: contract priority weighting is a parameter - compliance-first vs. freshness-first vs. schema-stability-first.
- **Systemic risk**: the agent reads operational data and writes a draft document. No live systems are modified until human approval.

## 2-Week Feasibility Assessment
| Task | Effort |
|---|---|
| Producer profiling (schema, Delta history, quality stats) | Days 1–3 |
| Consumer access pattern extraction | Days 2–4 |
| Divergence analysis between producer behaviour and consumer assumptions | Days 4–6 |
| Prompt design + LLM call + Data Contract Spec output schema | Days 5–8 |
| Unity Catalog persistence + MLflow Tracing | Days 8–10 |
| Human review interface + sign-off logging | Days 10–12 |
| Review, refinement, documentation | Days 12–14 |

**Verdict: Achievable**, with the note that the Data Contract Specification is an open standard - no need to invent an output schema.

---

# Proposal 5 - The Platform Whisperer
## *"Ask your Databricks platform anything, in plain English, and get an answer it can actually justify"*

## The Quirk

Every data platform has a de facto oracle - the one engineer everyone Slacks when they need to know something. *"Hey, where does this metric come from?" "Why did this job fail last Tuesday?" "Is this table safe to query in production?"* That person carries an enormous amount of platform knowledge in their head. They are also a single point of failure, a bottleneck, and eventually a flight risk.

The Whisperer replaces that Slack message. Not with a chatbot that hallucinates answers, but with an agent that **only answers what it can evidence** - and shows its working.

You ask: *"Why has the `orders` table been growing faster than usual this week?"*

It responds: *"Record volume increased 34% vs. the 4-week average, beginning Tuesday 14:23. The spike coincides with a schema change to the `promotions` table logged in Unity Catalog at 14:17 on the same day, which added a new event type. Three downstream pipelines ingest this event type without a filter - they are likely the source. Confidence: high. Evidence: [Delta changelog entry, MLflow run stats, pipeline DAG reference]."*

It doesn't guess. It cites. And if it cannot evidence an answer, it says so explicitly.

## Why This, Specifically

**Connects to the problem space:** Platform opacity is the universal pain point. The Whisperer dissolves it through conversation - no dashboard, no report, no prior knowledge required.

**Connects to the literature:**
- *Conflict between explainability and accountability* (arXiv): the failure mode is XAI producing the *appearance* of transparency without the substance. The Whisperer avoids this by tying every claim to a retrievable source.
- *Trajectory-level explainability* (Vector Institute, 2025): answers span time - "beginning Tuesday 14:23" - not just current state.
- *PROV-AGENT* (2025): every answer is a provenance trace. Every claim has a source, a timestamp, and a confidence level.

**Connects to Databricks primitives:**
- **Episodic memory** → MLflow run history as a queryable tool
- **Structured state** → Delta changelog queries via `DESCRIBE HISTORY`
- **Semantic memory** → Unity Catalog asset metadata
- **Orchestration** → ReAct loop: Reason → Act (query a tool) → Observe → Reason again

## Architecture Sketch

```
[User question - plain English]
         ↓
  [Single Agent - ReAct loop]
  Tools available:
  - query_mlflow_runs(filters)
  - query_delta_history(table, time_range)
  - query_unity_catalog(asset)
  - query_jobs_log(job_id, time_range)
         ↓
  [Agent retrieves evidence,        ← Reason → Act → Observe
   reasons over it, decides          → Reason again until
   if sufficient to answer]            confidence threshold met
         ↓
  [Structured response]
  - Direct answer (if evidenced)
  - Evidence citations with links
  - Confidence level: high / medium / insufficient
  - Open questions for human follow-up
  - Full reasoning chain (MLflow Traced)
```

## Extensibility Path

| Iteration | Addition | Autonomy Tier |
|---|---|---|
| **v1 (prototype)** | Evidence-grounded Q&A over one workspace | Tier 2 |
| **v2** | Multi-workspace support | Tier 2 |
| **v3** | Proactive alerts - "you haven't asked, but something changed" | Tier 2–3 |
| **v4** | Whisperer's retrieval layer becomes shared infrastructure for other agents | Tier 2 |
| **v5** | Natural language interface to trigger Proposals 1–4 | Tier 3 |

## Ethical-Practical Alignment
- **Accountability**: explicit confidence levels on every answer. "Insufficient evidence" is a first-class output, not a fallback. Full reasoning chain logged via MLflow Tracing.
- **Autonomy boundary**: the agent queries; it does not modify. Pure Tier 2.
- **Configurable objectives**: the agent can be parameterised to prioritise certain evidence sources - "trust Unity Catalog over MLflow when they conflict" is a configurable stance.
- **Systemic risk**: read-only tool calls. Zero write surface.

## 2-Week Feasibility Assessment
| Task | Effort |
|---|---|
| Query wrappers for MLflow, Delta history, Unity Catalog, Jobs logs | Days 1–5 (core data engineering) |
| ReAct prompt design with evidence-grounding constraint | Days 4–7 |
| Confidence scoring and "insufficient evidence" handling | Days 6–8 |
| Notebook chat interface (Gradio or ipywidgets) | Days 7–9 |
| MLflow Tracing for full reasoning chain | Days 8–10 |
| Review, refinement, documentation | Days 11–14 |

**Verdict: Achievable - and the easiest to demo.** A Databricks notebook with a text input cell is the entire interface. No deployment, no dashboard, no pipeline.

---

# Proposal 6 - The Platform Inquisitor
## *"An agent that interviews your engineers to extract what they know - and turns it into platform memory before they walk out the door"*

## The Quirk

Every data platform has two kinds of knowledge. The kind that lives in code, logs, and metadata - queryable, retrievable, structured. And the kind that lives in people's heads - why a particular design decision was made, what a field *actually* means versus what the documentation says, which pipeline is load-bearing despite looking abandoned, who to call when a specific thing breaks.

The second kind is worth more. It is also completely invisible to any tool, agent, or monitoring system. It evaporates when an engineer leaves.

The inversion: every other proposal has the agent *reading the platform*. This one has the agent **interviewing the humans who built it** - conducting a structured, adaptive conversation that extracts tacit knowledge, cross-references it against what the platform's data actually shows, challenges inconsistencies, and persists the result as structured, queryable institutional memory.

It is an agent whose entire job is to ask good questions.

## What It Actually Does

The engineer sits down with the Inquisitor. It opens with something like:

*"I've been looking at your platform. I have some questions. Let's start with `customer_events` - Delta history shows it's been schema-patched eleven times in six months. Nobody filed an incident for any of them. What's going on there?"*

The engineer explains. The agent listens, asks follow-up questions, probes inconsistencies, and cross-references answers against what it can verify in MLflow, Delta, and Unity Catalog. If the engineer says *"that pipeline runs every hour"* and the run history says it's been running every 90 minutes since March, the agent notices and asks about it. Gently. Curiously. Not as an interrogation - as a genuine attempt to understand.

At the end of the session, it produces:

- A **structured knowledge record** - decisions, rationale, tribal knowledge - persisted to Delta and queryable
- A **discrepancy report** - where stated knowledge and observed platform behaviour don't match
- An **open questions log** - things unresolved from either the conversation or the platform data
- A **confidence map** - which parts of the platform are well-understood by at least one human, and which are dark matter

## Why This, Specifically

**Connects to the problem space:** Institutional knowledge loss is universal. It affects every platform engagement, regardless of domain or industry. A tool that partially solves it is valuable to every client.

**Connects to the literature:**
- *Perspectival homogenisation* (ACM FAccT 2025): value decisions embedded in systems go unquestioned because nobody made them explicit. The Inquisitor's entire purpose is to surface those decisions.
- *PROV-AGENT* (2025): provenance is not just data lineage - it is *decision lineage*. Why was this schema chosen? These are provenance questions no log file can answer.
- *Conflict between explainability and accountability* (arXiv): accountability gaps emerge when the reasoning behind decisions is unavailable. The Inquisitor fills this gap by interviewing the people who made the decisions.
- *Trajectory-level explainability* (Vector Institute, 2025): understanding a platform's current state requires understanding the sequence of decisions that produced it. The Inquisitor reconstructs that trajectory from human memory, then cross-validates it against platform data.

**Connects to Databricks primitives:**
- **Episodic memory** → interview transcripts and extracted knowledge records, persisted to Delta
- **Semantic memory** → Vector Search over the knowledge base, so future agents (Whisperer, Archaeologist) can retrieve institutional knowledge alongside technical signals
- **Orchestration** → a single conversational loop; no multi-agent complexity at prototype stage

## Architecture Sketch

```
[Platform data - pre-loaded context]
(Delta DESCRIBE HISTORY summaries,
 MLflow anomaly flags, Unity Catalog
 asset inventory, job failure patterns)
         ↓
  [Inquisitor Agent - conversational loop]

  Opening: "I've looked at your platform.
  Here's what I found interesting. Let's talk."

  Per turn:
  - Listens to engineer's response
  - Cross-references against platform data
  - Identifies discrepancies or gaps
  - Generates next question
  (ReAct: each answer is an observation
   that shapes the next action)
         ↓
  [Structured knowledge extraction]     ← runs in background
  Per statement made by engineer:        during or after session
  - Classify: decision / rationale /
    tribal knowledge / warning / contact
  - Verify against platform data
  - Flag: confirmed / unverified /
    contradicted
         ↓
  [Session outputs]
  - Knowledge record (Delta-persisted)
  - Discrepancy report
  - Open questions log
  - Confidence map per platform asset
  - Full session trace (MLflow Tracing)
```

## Extensibility Path

| Iteration | Addition | Autonomy Tier |
|---|---|---|
| **v1 (prototype)** | Single-engineer session, one workspace | Tier 1–2 |
| **v2** | Multi-engineer sessions to surface knowledge conflicts | Tier 2 |
| **v3** | Scheduled "check-in" sessions to keep knowledge base current | Tier 2 |
| **v4** | Knowledge base feeds as background context into Whisperer and Archaeologist | Tier 2 |
| **v5** | Automated knowledge graph construction across the full team | Tier 3 |

## Ethical-Practical Alignment
- **Accountability**: every knowledge claim is classified as confirmed / unverified / contradicted. The engineer's identity is logged alongside their contribution - provenance for human knowledge.
- **Autonomy boundary**: the agent asks and records. It writes only to the knowledge base. It touches no operational system.
- **Configurable objectives**: the session focus is a parameter - governance lens, reliability lens, cost lens. The agent asks different questions depending on what the client wants to understand.
- **Systemic risk**: the only risk is the knowledge base itself being wrong. The discrepancy report and confidence map are the mitigation - they surface uncertainty rather than hiding it.

## 2-Week Feasibility Assessment
| Task | Effort |
|---|---|
| Platform context pre-loader (Delta history summaries, MLflow flags) | Days 1–4 (data engineering) |
| Conversational ReAct prompt with cross-referencing logic | Days 3–7 |
| Knowledge extraction + classification (post-session processing) | Days 6–9 |
| Delta persistence for knowledge records + confidence map | Days 8–10 |
| Notebook chat interface | Days 7–9 |
| MLflow Tracing for full session audit | Days 9–11 |
| Review, refinement, documentation | Days 12–14 |

**Verdict: Achievable.** The conversational loop is a multi-turn ReAct pattern - marginally more complex than Proposal 5, but the same fundamental architecture. The demo is a five-minute conversation that produces insights nobody expected.

---

# Full Comparison

| Dimension | P1: Health Narrator | P2: Onboarding Assessor | P3: Archaeologist | P4: Diplomat | P5: Whisperer | P6: Inquisitor |
|---|---|---|---|---|---|---|
| **Core primitive** | Scheduled reasoning over ops signals | Event-triggered dataset profiling | Historical pattern analysis | Producer-consumer contract drafting | Evidence-grounded Q&A | Human knowledge extraction |
| **Data eng. complexity** | Medium | Low-Medium | Medium-High | Medium | Low | Low-Medium |
| **Agent complexity** | Low | Low | Low | Low | Low-Medium (ReAct) | Low-Medium (ReAct) |
| **Demo infra** | Medium | Low-Medium | Medium | Low-Medium | Very Low | Very Low |
| **Demo impact** | High | High | High | Medium-High | Very High | Very High |
| **Extensibility ceiling** | Very High | High | High | High | Very High | Very High |
| **Horizontal applicability** | Universal | Universal | Universal | Universal | Universal | Universal |
| **Novelty** | Moderate | Moderate | High | High | High | Very High |
| **Autonomy tier** | Tier 2 | Tier 2 | Tier 1–2 | Tier 2 | Tier 2 | Tier 1–2 |

## Recommendation

**Start with Proposal 2, design for Proposal 5.**

Proposal 2 remains the tightest scope for a first prototype. But Proposal 5's query wrappers - the tool layer over MLflow, Delta, Unity Catalog - are the shared retrieval infrastructure that every other proposal eventually needs. Build Proposal 2 first, build Proposal 5's tool layer alongside it, and you have the foundation for the full system.

The sequencing:

> **Weeks 1–2**: Proposal 2 (Onboarding Assessor) with Proposal 5's retrieval wrappers as the data access layer
> **Month 2**: Proposal 5 (Whisperer) - conversational interface over the retrieval layer now built
> **Month 3**: Proposal 3 (Archaeologist) - historical analysis using the same retrieval layer
> **Month 4**: Proposal 6 (Inquisitor) - human knowledge layer, feeds into Archaeologist and Whisperer
> **Month 5+**: Proposal 4 (Diplomat) and Proposal 1 (Narrator) - built on the full stack

---

# Shared Design Principles Across All Proposals

These apply regardless of which proposal is chosen and reflect the ethical-practical framework established in the conversation:

1. **Audit trail from day one**: MLflow Tracing logs the full reasoning chain of every agent invocation. Non-negotiable, even in the prototype.

2. **Tier 2 hardcoded at prototype stage**: agents write to reports, knowledge bases, or Delta tables - not to pipelines, schemas, or operational platform configuration. This boundary is architectural, not just a policy.

3. **Structured output over free text**: every agent's output is a JSON schema with defined fields. Queryable, versionable, and comparable over time - and prevents the agent from hiding poor reasoning inside fluent prose.

4. **Configurable objective parameters in the prompt**: what the agent prioritises is passed as a parameter, not hardcoded. The optimisation objective is explicit, auditable, and client-adjustable from the first prototype.

5. **Human decision is the terminal step**: the workflow is not complete until a human has reviewed and acted (or consciously deferred). The agent makes that step faster and better-informed - it does not bypass it.

6. **"Insufficient evidence" is a first-class output**: no agent should confabulate. If evidence is absent or ambiguous, the agent surfaces that explicitly rather than filling the gap with plausible-sounding inference. This applies to all six proposals.