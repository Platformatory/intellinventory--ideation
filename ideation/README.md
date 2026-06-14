<h1>IDEATION</h1>

---

**Contents**:

- [Feedback on Phase 1 to Work on](#feedback-on-phase-1-to-work-on)
  - [Terminology Revision](#terminology-revision)
  - [Need for Tighter Control in Financial Decision](#need-for-tighter-control-in-financial-decision)
  - [Questioning the Use of LLM](#questioning-the-use-of-llm)
  - [Clarify Specific Use-Cases](#clarify-specific-use-cases)
  - [Potential Direction: Control-Tower Decisions](#potential-direction-control-tower-decisions)
- [Addressing Feedback on Phase 1](#addressing-feedback-on-phase-1)
  - [POINT: Terminology Revision](#point-terminology-revision)
  - [POINT: Need for Tighter Control in Financial Decisions](#point-need-for-tighter-control-in-financial-decisions)
  - [POINT: Questioning the Use of LLM](#point-questioning-the-use-of-llm)
  - [POINT: Clarify Specific Use-Cases](#point-clarify-specific-use-cases)
  - [POINT: Potential Direction: Control-Tower Decisions](#point-potential-direction-control-tower-decisions)
- [Read Further](#read-further)

---

# Feedback on Phase 1 to Work on
> **`phase-1` branch for reference**: [`phase-1`](https://github.com/Platformatory/intellinventory--ideation/tree/phase-1)

## Terminology Revision
Terminology must be revised:

- Disruption
- Reasoning model

Align it with common technological use-cases.

## Need for Tighter Control in Financial Decision
Financial operations

=> Higher risk

=> More desire for control

*Hence, fully autonomous agent may be less favoured here.
*

## Questioning the Use of LLM
- Use of LLM for our use-case may not not ideal
- May be okay at a smaller scale, but is not scalable
- Time-series models may be better for forecasting purposes

## Clarify Specific Use-Cases
What are the actual business requirements we are trying to solve?

- Fast-moving goods?
- E-commerce setup?
- Industrial warehouses with bulk orders?
- etc.

The business requirements will inform the technical requirements.

## Potential Direction: Control-Tower Decisions
Potentially more valuable use-case:

*Automating warehouse control-tower decisions.*

# Addressing Feedback on Phase 1
## POINT: Terminology Revision

| Old Term | New Term(s) | Remarks/References |
| --- | --- | --- |
| Disruption | Supply Chain Disruption (SCD) | [*What is Supply Chain Disruption?*, **www.hyperbots.com/glossary**](https://www.hyperbots.com/glossary/supply-chain-disruption); this term covers: demand/supply shocks, transportation/logistical disruptions, natural disaster-related disruption, etc. |
| Reasoning Model | Reasoning Model | This is the correct term, but for clarity, it is better to specify its subcategories: predictive model, LLM. The "AI agent" is the overall system that consists of reasoning model(s) and tool-calling capabilities. |

## POINT: Need for Tighter Control in Financial Decisions
**Context**:

For ensuring tight governance of decision-making format, constraint definition and constraint fulfillment, we have established a separation between the AI layer and the programmatic layer (see: [`__docs__/reasoningIntegrationSpecs-1.md` in `Platformatory/intellinventory--warehouse-sim`, **github.com**](https://github.com/Platformatory/intellinventory--warehouse-sim/blob/main/__docs__/reasoningIntegrationSpecs-1.md)). While the exact architecture needs refinement, the overarching idea of the separation - between the programmatic layer handling the actual interactions with the environment (data tables, reordering APIs, etc.) and the AI layer handling reasoning - our chosen approach.

**Suggested direction**:

Insert a human interaction (HI) layer between the programmatic layer and the AI layer. To allow for flexibility in applicability, the HI layer should be optional (hence, we should be able to toggle between fully-autonomous mode and human-in-the-loop mode). The HI layer should allow:

- Reviewing decisions and reasoning
- Making edits with optional comments

For reliable auditing, ensure edited decisions are marked as edited.

## POINT: Questioning the Use of LLM
Skepticism about the use of LLM for what is essentially forecast-based decision-making is valid skepticism indeed, since forecasting is not what LLMs were designed to do. Moreover, even if an LLM were capable of achieving similar performance as forecasting models, we should consider that LLM-calls would likely be more expensive due to the larger number of parameters that allow for a larger variety of capabilities.

To this end, it is worth looking into forecasting models. However, LLM is not rendered obsolete with the use of forecasting models, since an LLM can provide capabilities necessary for decision-making, especially human-in-the-loop and/or auditable decision-making, such as: drafting explanations, summarising analyses, answering planner questions, etc. Furthermore, with retrieval-augmented generation (RAG), the LLM can use the organisation's historical data - assumptions, prior plans and supply/demand records - to give organisation-specific insights.

Hence, the most promising potential direction may be:

- Using structured forecasting models for:
    - Prediction/forecasting
    - Preditive decision-making
- Using LLM for:
    - Human-readable reasoning
    - Auditable decision-making
    - Providing data-driven insights beyond mathematical modelling

Promising references:

- [*AI in demand forecasting: Use cases, benefits, architecture, solution and implementation*, **www.leewayhertz.com**](https://www.leewayhertz.com/ai-in-demand-forecasting/) <br> ... offers clarity about the layers of the solution, including
    - Modelling layer (structured models for prediction)
    - Generative AI layer (for LLM-driven capabilities)
- [*AI Agents in Supply Chain Management: Automating Inventory and Demand Forecasting*, **onereach.ai/blog**](https://onereach.ai/blog/ai-agents-in-supply-chain-inventory-and-forecasting-automation/)

## POINT: Clarify Specific Use-Cases
**See**: ["What Good Is Agentic AI w.r.t. Inventory Management?", `ideation/problem-statement-and-architecture-proposal.md`](./problem-statement-and-architecture-proposal.md#what-good-is-agentic-ai-wrt-inventory-management)

## POINT: Potential Direction: Control-Tower Decisions
Out of scope.

# Read Further
- [`problem-statement-and-architecture-proposal.md`](./problem-statement-and-architecture-proposal.md)
- [`architecture`](./architecture)