<h1>Agentic AI</h1>

---

**Contents**:

- [PRELIMINARY CONCEPT: AI Agent](#preliminary-concept-ai-agent)
- [PRELIMINARY CONCEPT: AI Orchestration](#preliminary-concept-ai-orchestration)
- [PRELIMINARY CONCEPT: AI Orchestration Platform](#preliminary-concept-ai-orchestration-platform)
- [MAIN CONCEPT: Agentic AI](#main-concept-agentic-ai)
  - [Single-Agent](#single-agent)
  - [Multi-Agent](#multi-agent)
- [AI Communication Standards](#ai-communication-standards)
- [Behavioural/Structural Hallmarks of Agentic AI](#behaviouralstructural-hallmarks-of-agentic-ai)
  - [Autonomous Operation](#autonomous-operation)
  - [Multi-Step Problem Solving](#multi-step-problem-solving)
  - [Adaptability](#adaptability)
  - [Goal-Driven Planning, Integration \& Autonomous Execution Loop](#goal-driven-planning-integration--autonomous-execution-loop)
- [Agentic AI vs. GenAI](#agentic-ai-vs-genai)

---

> **Key references**:
>
> - [*Explaining Agentic AI: The Good, the Bad & the Ugly* by ExplainingComputers, **youtube.com**](https://www.youtube.com/watch?v=Gvt4TwGpqOs)
> - [*What is agentic AI?*, **ibm.com/think/topics**](https://www.ibm.com/think/topics/agentic-ai)
> - [*What is Agentic AI?*, **databricks.com/blog**](https://www.databricks.com/blog/what-is-agentic-ai)

# PRELIMINARY CONCEPT: AI Agent
*Adaptive programs/systems designed to make decisions autonomously*. Adaptive => not constrained by training data or a well-defined set of use-case parameters. Autonomous decision-making => not dependent on human input/feedback to facilitate or validate its decision-making.

AI agents can:

- Act to obtain information
- Create and execute workflows
- Perform actions and tasks
- Interact with:
    - External systems
    - Applications/tools
    - Other AI agents

# PRELIMINARY CONCEPT: AI Orchestration
The adaptive coordination and management of multiple:

- AI agents
- Systems
- Data sources

**NOTE**:

- AI orchestration is adaptive, not rule-based
- Usually happens on an [AI orchestration platform](#preliminary-concept-ai-orchestration-platform)

# PRELIMINARY CONCEPT: AI Orchestration Platform
A software framework used to...

- create
- deploy
- manage
- monitor

... the safe operation of AI agents.

AI orchestration platforms:

- Automate AI workflows
- Facilitate external data access
- Facilitate human governance
- Enable agent-to-agent (A2A) communication

---

**NOTE**: *AI agent communication standards have been developed.*

See: [AI Communication Standards](#ai-communication-standards)

---

Some AI orchestration platforms:

- AutoGen (Microsoft) (open-source framework)
- watsonx Orchestrate (IBM)
- Bedrock AgentCore (Amazon)
- Vertex AI Agent Builder (Google)

# MAIN CONCEPT: Agentic AI
A system wherein:

- 1 or more AI agents work autonomously
- To accomplish 1 or more defined goals


The accomplishing of goals can look like:

- Performing a set of tasks within given constraints
- Performing adaptive actions as per given requirements

---

Hence, agentic AI involves:

- Definition of AI agents
- Definition of requirements/constraints/goals

## Single-Agent
1 AI agent undertakes the entire activity.

## Multi-Agent
Multiple AI agents perform sub-tasks in a coordinated way.

*Their activities are coordinated by AI orchestration*

**NOTE**: *AI orchestration usually happens in an AI orchestration platform.*

# AI Communication Standards
To allow AI agents to interact with...

- external systems
- external data sources
- other AI agents

... communication standards have been developed:

- Model Context Protocol (MCP) (Anthropic) <br> *To connect AI applications to external systems*
- A2A Protocol (A2A) (Google) <br> *Open-source standard for agent-to-agent communication*
- Agent Communication Protocol (ACP) (IBM) <br> *Now merged with Google's A2A*
- Other proprietary standards

**NOTE**: *A2A is now managed by the Linux foundation.*

# Behavioural/Structural Hallmarks of Agentic AI
## Autonomous Operation
The system decides when and how to act independently without constant human oversight. Even with a human in the loop, it is up to the system to identify what needs human feedback; this is an aspect of autonomous operation and decision-making.

## Multi-Step Problem Solving
The ability to take a high-level goal and autonomously work through multiple dependent steps - goal interpretation, planning, acting, checking results and adapting - until the goal is achieved or escalated.

## Adaptability
The ability to change its behavior during task execution based on new information, outcomes or changing conditions while pursuing a goal instead of rigidly following a fixed script. Agentic AI adapts using reasoning, heuristics, rules and short-term memory.

## Goal-Driven Planning, Integration & Autonomous Execution Loop
**NOTE**: *This draws from [Behavioural/Structural Hallmarks of Agentic AI](#behaviouralstructural-hallmarks-of-agentic-ai).*

- **Goal-driven planning**: AI agents can break complex goals into ordered subtasks and adjust plans as conditions change to enable complex workflows rather than single actions.
- **Integration with AI tools**: Using external tools, APIs, databases, code execution and services to move AI from analysis to execution.
- **Autonomous execution loops**: The core mechanism of goal achievement is a repeating control cycle of goal definition → Planning → Action → Observation → Adjust → Repeat

# Agentic AI vs. GenAI
\[TO BE CONTINUED\]