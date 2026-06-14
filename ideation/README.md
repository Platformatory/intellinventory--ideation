<h1>IDEATION</h1>

---

**Contents**:

- [Navigate](#navigate)
- [Overall Direction](#overall-direction)
  - [Use of Databricks](#use-of-databricks)
  - [Domain-Specific + Platform-Engineering Proposals](#domain-specific--platform-engineering-proposals)
  - [Considerations](#considerations)
  - [Consideration for Platform-Engineering Proposal](#consideration-for-platform-engineering-proposal)
- [PROPOSAL 1: \[Platform-Engineering\] Platform Prashanth (ABANDONED)](#proposal-1-platform-engineering-platform-prashanth-abandoned)
- [PROPOSAL 2: \[Domain-Specific\] Warehouse Reorder (APPROVED)](#proposal-2-domain-specific-warehouse-reorder-approved)

---

# Navigate
- [`problemSpaceExploration.1-definingTheProblemSpace.md`](./problemSpaceExploration.1-definingTheProblemSpace.md)
- [`problemSpaceExploration.2-ethicalAndPracticalConsiderations.md`](./problemSpaceExploration.2-ethicalAndPracticalConsiderations.md)
- [`problemSpaceExploration.3-literatureReview.md`](./problemSpaceExploration.3-literatureReview.md)
- [`proposal.dbxMonitoringReportingSummarising.md`](./proposal.dbxMonitoringReportingSummarising.md)
- [`proposal.fdhDbxIngestionAutomatedSetup.md`](./proposal.fdhDbxIngestionAutomatedSetup.md)
- [`proposalSet.1.md`](./proposalSet.1.md)
- [`proposalSet.2.md`](./proposalSet.2.md)
- [`proposal-1--platform-engineering--platform-prashanth.md`](./proposal-1--platform-engineering--platform-prashanth.md)
- [`proposal-2--domain-specific--warehouse-reorder.md`](./proposal-2--domain-specific--warehouse-reorder.md)

# Overall Direction
## Use of Databricks
The goal is to demonstrate capability in Databricks-based agentic AI systems.

## Domain-Specific + Platform-Engineering Proposals
We have 2 complementary proposals:

1. Domain-specific proposal
2. Platform-engineering proposal

Why?

- (1) = demonstrable use-case that serves as the context for (2)
- (2) shows the wider capabilities and potential of agentic AI <br> ... *for enhancing platform effectivenes for a variety of use-cases*

## Considerations
- Future-vision
- Databricks free-edition compatible
- Connection to/account for other external platforms (e.g. S3, Kafka, etc.) <br> **NOTE**: *This can be accounted for even by documentation or engineer feedback*
- End-to-end consideration for:
    - Data sources
    - Data movement
    - Platform evolution

## Consideration for Platform-Engineering Proposal
- Should work well with the chosen domain-specific proposal
- Should minimise LLM use complexity <br> *Since focus is on demonstrating agentic systems* <br> => Focus on problems that require/benefit from...
    - Continual automonous action
    - LLM-based capabilities (i.e. it should not be solvable by script-based automation)

# PROPOSAL 1: [Platform-Engineering] Platform Prashanth (ABANDONED)
\[2026-05-08\]

**See**: [`ideation/proposal-1--platform-engineering--platform-prashanth.md`](./proposal-1--platform-engineering--platform-prashanth.md)


# PROPOSAL 2: [Domain-Specific] Warehouse Reorder (APPROVED)
\[2026-05-08\]

**See**: [`proposal-2--domain-specific--warehouse-reorder.md`](./proposal-2--domain-specific--warehouse-reorder.md)