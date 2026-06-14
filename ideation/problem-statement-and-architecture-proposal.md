<h1>Problem Statement & Architecture</h1>

---

**Content**:

- [Definitions](#definitions)
  - [Reliability](#reliability)
  - [Sufficiency](#sufficiency)
- [Context](#context)
- [What Good Is Agentic AI w.r.t. Inventory Management?](#what-good-is-agentic-ai-wrt-inventory-management)
- [3 Proposed Architectural Components](#3-proposed-architectural-components)
  - [ArchComp 1: Human-in-the-Loop (HITL)](#archcomp-1-human-in-the-loop-hitl)
  - [ArchComp 2: Decision-Reasoning](#archcomp-2-decision-reasoning)
  - [ArchComp 3: Long-Term Observability](#archcomp-3-long-term-observability)
- [Additional Resources](#additional-resources)

---

# Definitions
## Reliability
The quality of being able to consistently meet expectations/standards.

## Sufficiency
The quality of being able to meet expectations/standards.

=> Reliability means being able to achieve sufficiency consistently.

# Context
**Required function**: inventory management, involving:

- Restocking
- Fulfilling demand
- Optimising for inventory cost

**Potential end-users**:

- Warehouse managers
- Logistics data analysts

# What Good Is Agentic AI w.r.t. Inventory Management?
- Higher compute capacity than humans <br> => Rapid decision-making potential
- Larger working memory <br> => Can consider a richer evolving context
- Adaptive parameterisation of high-dimensional models <br> => Adaptability to complex, nonlinear, fast-evolving patterns

**Hence, the most suitable use-case**:

Diverse, fast-moving goods. Here, especially with perishable items, a wide range of potentially in-demand items (which implies a variety of suppliers and supplier patterns to consider), elastic potentially rapidly evolving demand (with trends, competitors, promotional campaigns, etc.), fast-pace (rapid warehouse-to-consumer expectations), we benefit from fast-paced, adaptive and context-considerate decision-making.

That being said, aligning with the feedback ["Need for Tighter Control in Financial Decision", `ideation/README.md`](./README.md#need-for-tighter-control-in-financial-decision), it is key to note that financial decision-making involves high risk (from a business and social perspective), which means there is high desire for control over the decision-making.

---

Hence, the architecture is based on 2 key requirements:

1. Adaptability to complex, nonlinear, fast-evolving patterns <br> => *Scalability, efficiency, reliability*
2. Enforceable control over decision-making <br> => *Interpretability, reliable outcome-control mechanisms, auditability*

# 3 Proposed Architectural Components
## ArchComp 1: Human-in-the-Loop (HITL)
Human should be able to:

- Validate the agent's decisions
- Control the impact the agent has over the environment
- Take ownership of the decisions

---

But to facilitate this in a fast-based, complex environment, we need to be able to provide human-readable reasoning that is as reliable as possible and sufficiently consolidated. To address this point, we have ["ArchComp 2: Decision-Reasoning"](#archcomp-2-decision-reasoning).

## ArchComp 2: Decision-Reasoning
This component relies on the following:

- Forecasting model (FM) to make prediction-based decisions
- Data to provide context for the decision-making, e.g.:
  - FM inputs
  - FM outputs
  - Understanding how inputs are mapped to outputs in the FM
- Generative AI model (e.g. LLM) to:
  - Process the decision-making context
  - Generate human-readable reasoning for the decisions
  - (Optionally) Offer human-readable responses for decision-related queries, e.g.:
      - Past decisions made
      - Changes from past decisions
      - Causes of the changes

A key fact to consider here is that a model such as an LLM is a statistical model and a black-box; in other words, its outputs are driven by high-dimensional statistical pattern-matching, not logical inference, and the mapping between model inputs and model outputs is too complex to be interpretable by a human. Hence, the main lever we have in improving the reliability of the LLM's reasoning on the decisions made is to ensure the context provided for this reasoning is consistently, sufficiently and concisely:

- Rich in its information
- Accurate in its details

> **NOTE**: Conciseness => Unnecessary details do not complicate the reasoning. This is relevant for an LLM, since LLM's are pattern-matching tools, not logical inference tools, and as such, may not be able to reliably distinguish between relevant and irrelevant information.

***Hence, defining and enforcing the structure of the reasoning context is crucial.***

As mentioned above, the following data would be useful for the reasoning context:

- FM inputs <br> ... *inputs shape the output*
- FM outputs <br> ... *the actual prediction-driven decisions made*
- Understanding how inputs are mapped to outputs in the FM <br> ... *grasping this is key to grasping why a set of decisions were made*

---

Now, to be able to check and validate the performance of the overall system, including the decision-reasoning component but also the HITL component, we need to provide long-term observability, i.e. an unambiguous, sufficiently comprehensive record of what happened within the various parts of the system that made it act the way it did. To address this point, we have ["ArchComp 3: Long-Term Observability"](#archcomp-3-long-term-observability).

## ArchComp 3: Long-Term Observability
The way we define sufficiency for this component is as follows:

> We must be able to replicate the decisions made, in order to:
>
> - Study its context/conditions
> - Validate the performance of the various models

We must have sufficient information about:

- The inputs/outputs of each model (FM, LLM)
- Preservation of decisions before and after human edits <br> ... *to ensure we can still validate the system's performance apart from the human*

# Additional Resources
- [`ideation/_resources/architecture-proposal-draft--2026-06-12.jpeg`](./_resources/architecture-proposal-draft--2026-06-12.jpeg)