<h1>Interpreting Agentic Systems:<br><i>Beyond Model Explanations to System-Level Accountability</i></h1>

> **Paper reference**: [Interpreting Agentic Systems: Beyond Model Explanations to System-Level Accountability](https://arxiv.org/pdf/2601.17168) `|` [See Downloaded PDF](./interpreting-agentic-systems--beyond-model-explanations-to-system-level-accountability.pdf)

---

**Contents**:

- [Paper's Purpose](#papers-purpose)
- [AI Agent Orchestrates What?](#ai-agent-orchestrates-what)
- [Role of LLM](#role-of-llm)
- [AI Agents vs. Agentic System](#ai-agents-vs-agentic-system)
- [Shift from AI Agents to Agentic Systems](#shift-from-ai-agents-to-agentic-systems)
  - [The 3 Foundational Modules of AI Agents](#the-3-foundational-modules-of-ai-agents)
  - [How Agentic Systems Build upon This](#how-agentic-systems-build-upon-this)
  - [Modules that Together Represent the Backbone of Agentic Systems](#modules-that-together-represent-the-backbone-of-agentic-systems)
- [\[CONCEPTUAL CHECKPOINT\]: Key Points to Consider/Account for](#conceptual-checkpoint-key-points-to-consideraccount-for)
- [Early ML Paradigms to Agentic Systems](#early-ml-paradigms-to-agentic-systems)
  - [Why Learn this?](#why-learn-this)
  - [Foundations Provided by LLMs](#foundations-provided-by-llms)
  - [Shift 1 \& Steps toward Autonomy](#shift-1--steps-toward-autonomy)
  - [Shift 2 \& Steps toward Coordination](#shift-2--steps-toward-coordination)
  - [Shift 3 \& Steps toward Dynamic Systems](#shift-3--steps-toward-dynamic-systems)
  - [Shift 4 \& Steps toward Integrated, Tightly-Coupled Systems](#shift-4--steps-toward-integrated-tightly-coupled-systems)
- [Unique Interpretability Need \& Issues in Agentic Systems](#unique-interpretability-need--issues-in-agentic-systems)
- [Key Techniques Enabling Core Capabilities in Agentic Systems](#key-techniques-enabling-core-capabilities-in-agentic-systems)
- [Key Agentic System Framework Types to Reflect Design-Changes](#key-agentic-system-framework-types-to-reflect-design-changes)
- [\[CONCEPTUAL CHECKPOINT\]: Unique Risk Factors for Agentic Systems](#conceptual-checkpoint-unique-risk-factors-for-agentic-systems)

---

# Paper's Purpose
Examine interpretability as foundational to trust safety in agentic systems.

# AI Agent Orchestrates What?
- Perception
- Memory
- Tool utilisation
- Iterative action-feedback loop to interact with + affect the environment

# Role of LLM
Acts as the "brain" for:

- Understanding
- Planning
- Generating natural language

Hence, AI agent:

- Uses LLM for cognitive capability
- Provides the framework and mechanisms to *operationalise* this capability

-> Comprehension + intelligence in a dynamic environment.

# AI Agents vs. Agentic System
**AI Agent**:

Modular achitecture powered by LLM for task-specific automation.

---

**Agentic System**:

Complex, multi-agent systems characterised by:

- Emergent behaviours
- Coordiated autonomy

=> Can orchestrate workflows.

-> Shared objectives, not isolated tasks.

# Shift from AI Agents to Agentic Systems
## The 3 Foundational Modules of AI Agents
Traditional AI agents are built on 3 foundational modules:

1. Perception
2. Reasoning
3. Action

-> Enable limited autonomy within well-defined tasks.

## How Agentic Systems Build upon This
Agentic systems build upon this with advanced modules, including:

- Specialised agents (multi-agent collaboration)
- Advanced reasoning and planning (task decomposition)
- Persisted memory (shared context)
- Orchestration (system-wide coordination)

## Modules that Together Represent the Backbone of Agentic Systems
- Perception
- Reasoning
- Action
- Memory

... under a coordinated autonomy paradigm.

# \[CONCEPTUAL CHECKPOINT\]: Key Points to Consider/Account for
- Emergent behaviour of agents and agentic systems
- System-level actions (rather than individual agent actions)
- Shared context

# Early ML Paradigms to Agentic Systems
## Why Learn this?
Understanding how today's agentic systems evolved from early ML paradigms provides important context for their capabilities and limitations, highlighting how interpretability issues have developed into system-level challenges.

## Foundations Provided by LLMs
LLMs enabled open-ended behaviour across varied contexts, establishing the basis for interaction rather than one-off prediction.

-> Building on these capabilities, LLM-based agents introduced mechanisms for planning, memory and tool-use to pursue goals.

## Shift 1 & Steps toward Autonomy
SHIFT: Predictive -> Performative.

---

Early steps toward autonomy:

- Iterative reasoning
- Environment-aware decision-making

-> Autonomy integrated with context-aware adaptive reasoning.

## Shift 2 & Steps toward Coordination
SHIFT: Isolated intelligence to collective cognition.

---

Steps toward coordination:

- Specialised, communicating agents
- Meta-agents orchestrating task-decomposition and synthesis

## Shift 3 & Steps toward Dynamic Systems
SHIFT: Static model -> Dynamic system.

---

While large foundation models such as LLMs and multi-modal models provide the cognitive substrate for perception and reasoning, the progression toward agentic architectures has required specialized methods that support:

- Persistence
- Coordination
- Control

These techniques transform static model capabilities into dynamic system functions, ***enabling agents not only to reason, but also to remember, plan and act collaboratively across environments***.


## Shift 4 & Steps toward Integrated, Tightly-Coupled Systems
SHIFT: Localised decision-making -> Sequential, coordinated decision-making

---

In agentic architectures, perception, planning, reasoning, and tool execution form a compositional pipeline with strict dependencies. Evaluating system performance without perception is meaningless when downstream components require per-ceptual input, violating the assumption that each subset has a well-defined characteristic function value. Moreover, these components exhibit strong complementarity rather than additive contributions and the system performance is multiplicative across components rather than decomposable into independent marginal effects.

->

- ***Sequential, tightly-coupled*** system architectures
- Where components serve ***irreplaceable functional roles***

# Unique Interpretability Need & Issues in Agentic Systems
Unlike standalone models, agentic systems consist of multiple interacting components such as perception, reasoning, memory, planning, and action modules, often distributed across several agents. In a safety-critical scenario, each agent perceives and interprets its environment (e.g. medical readings, sensor logs, natural language descriptions), forms internal beliefs and strategies, exchanges messages with others, and acts based on both internal reasoning and external interactions. ***The opacity of any of these processes, within or between agents, can compromise the reliability and safety of the system as a whole.***

Drawing from [Shift 4 & Steps toward Integrated, Tightly-Coupled Systems](#shift-4--steps-toward-integrated-tightly-coupled-systems) is evident that current post-hoc explainability techniques, originally devised for single-model predictions, are not immediately suitable for agentic systems. They often provide fragmented, local insights (e.g. importance of one feature at one time step) that fail to capture the global, temporal, and interactive aspects of multi-agent systems.

# Key Techniques Enabling Core Capabilities in Agentic Systems
(Taken directly from the paper)

| Technique Area | Functionality | Implementation Examples |
| --- | --- | --- |
| Memory and Context Management | Supports persistence, continuity, and adaptive reasoning through episodic, semantic, and procedural memory layers that maintain both short- and long-term context. | Implemented in systems such as MIRIX and MemTool, which use vector databases to store and retrieve contextual information for long-horizon reasoning [Derouiche et al., 2025]. |
| Tool Integration and API Invocation | Extends agentic capabilities beyond pretraining by enabling agents to interact with and control external tools, APIs, and environments. | Frameworks like MATPO train “planner–executor” agent pairs to select and invoke tools reliably in dynamic, multi-agent settings [Yu et al., 2025]. |
| Agent Coordination and Role Assignment | Facilitates collaboration and dynamic role allocation among specialized agents, enhancing scalability and adaptability in complex tasks. | Systems such as AgentFlow employ orchestrators that monitor context, assign roles, and coordinate inter-agent communication in real time [Derouiche et al., 2025; Sapkota et al., 2025]. |
| Planning and Task Decomposition | Enables complex, multi-stage problem-solving through iterative task breakdown, reasoning validation, and replanning. | Implemented through graph-based orchestration frameworks like LangGraph, which formalize inter-agent dependencies and task refinement [Sapkota et al., 2025]. |
| Scalability and Efficiency Optimization | Balances autonomy and computational cost through hierarchical control mechanisms, selective activation, and token-efficient reasoning. | Recent architectures integrate multilayer scheduling and adaptive activation pipelines to optimize reasoning efficiency [Agrawal et al., 2025]. |

In high-performing deployments, these components operate jointly rather than in isolation. Such designs emphasize feedback loops between memory, planning, and execution layers, forming a cohesive reason-act-reflect cycle essential to adaptive autonomy.

# Key Agentic System Framework Types to Reflect Design-Changes
- Graph-based
- Workflow oriented
- Modular

\[NEEDS FURTHER CLARITY\]

# \[CONCEPTUAL CHECKPOINT\]: Unique Risk Factors for Agentic Systems
New categories of risk that exceed traditional AI failure modes:

- Stem from their ability to plan, act, and adapt:
  - Over long horizons
  - With limited human oversight
- The most prominent risks relate to:
  - Opacity
  - Planning fragility
  - Unchecked autonomy
  - Temporal uncertainty
  - Goal misalignment
  - Absence of accountability infrastructure

Taken together, these issues highlight a central vulnerability of the agentic era: ***as systems increasingly plan, reason, and act over extended horizons, opacity becomes a first-order failure mode rather than a peripheral concern***. The cumulative implication is that transparency and interpretability cannot remain optional, after-the-fact diagnostics; they must be treated as foundational design requirements embedded in the architecture and lifecycle of agentic systems.