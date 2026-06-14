<h1>Problem Statement & Architecture</h1>

---

**Content**:

- [Definitions](#definitions)
  - [Reliability](#reliability)
  - [Sufficiency](#sufficiency)
- [Context](#context)
- [What Good Is Agentic AI w.r.t. Inventory Management?](#what-good-is-agentic-ai-wrt-inventory-management)
- [3 Core Architectural Components](#3-core-architectural-components)
  - [ArchComp 1: Human-in-the-Loop](#archcomp-1-human-in-the-loop)
  - [ArchComp 2: Decision-Reasoning](#archcomp-2-decision-reasoning)

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

# 3 Core Architectural Components
## ArchComp 1: Human-in-the-Loop
Human should be able to:

- Validate the agent's decisions
- Control the impact the agent has over the environment
- Take ownership of the decisions

---

But to facilitate this in a fast-based, complex environment, we need to be able to provide human-readable reasoning that is as reliable as possible and sufficiently consolidated. To address this point, we have "ArchComp 2: Decision-Reasoning".

## ArchComp 2: Decision-Reasoning
This component relies on the following 