<h1>Engine Architecture - V1</h1>

> **Module code**: `Eng`

> **Relevant context**: [`ideation/architecture/README.md`](./README.md); defines:
>
> - `RM` (reasoning model) + submodules/components
> - `HITL` (human-in-the-loop) + submodules/components

---

**Contents**:

- [Diagram](#diagram)
- [`Eng.Scheduler` (Scheduler)](#engscheduler-scheduler)
- [`Eng.Mon` (Monitoring)](#engmon-monitoring)
  - [`Eng.Mon._envIF`](#engmon_envif)
- [`Eng.AD` (Agent Decision)](#engad-agent-decision)
  - [`Eng.AD._FC` (Forecast Context)](#engad_fc-forecast-context)
  - [`Eng.AD.buildFC` (Build Context)](#engadbuildfc-build-context)
    - [`Eng.AD.buildFC._envIF`](#engadbuildfc_envif)
  - [`Eng.AD.adReq` (Agent Decision Request)](#engadadreq-agent-decision-request)
  - [`Eng.AD.Q` (Agent Decision Response Queue)](#engadq-agent-decision-response-queue)
  - [`Eng.AD.adResCheck` (Agent Decision Response Check)](#engadadrescheck-agent-decision-response-check)
- [`Eng._D-1` (Engine Decision Object - V1)](#eng_d-1-engine-decision-object---v1)
- [`Eng.HD` (Human Decision)](#enghd-human-decision)
  - [`Eng.HD.hdReq` (Human Decision Request)](#enghdhdreq-human-decision-request)
  - [`Eng.HD.Q` (Human Decision Response Queue)](#enghdq-human-decision-response-queue)
  - [`Eng.HD.hdResCheck` (Human Decision Response Check)](#enghdhdrescheck-human-decision-response-check)
- [`Eng._D-2` (Engine Decision Object - V2)](#eng_d-2-engine-decision-object---v2)
- [`Eng.Exec` (Executor)](#engexec-executor)
  - [`Eng.Exec._envIF`](#engexec_envif)

---

# Diagram
> **NOTE**: Details are given in the following sections.

![](../_resources/engine-architecture-v1.png)

> Editable XML file for the diagram: [`ideation/_resources/engine-architecture-v1.drawio`](../_resources/engine-architecture-v1.drawio)

# `Eng.Scheduler` (Scheduler)
Core loop of the engine that:

- Synchronises engine actions with discrete time steps
- Schedules engine actions (including dispatching of asynchronous threads)

# `Eng.Mon` (Monitoring)
Involves monitoring process(es) that are:

- Synchronous
- High frequency (ideally per engine tick)

These processes can be:

- Table read queries (especially for operational tables) <br> ... *pull approach*
- Received notifications/events <br> ... *push approach*

## `Eng.Mon._envIF`
Requirement for a 2-way interface with the environment for:

- Pull approach data collection
- Push approach data collection

# `Eng.AD` (Agent Decision)
Agent decision handler module, called by `Eng.Scheduler` every n ticks (configurable).

## `Eng.AD._FC` (Forecast Context)
Context needed for the forecast model (serves as its primary input).

## `Eng.AD.buildFC` (Build Context)
Builds `Eng.AD._FC`. May need to interact with `Env`.

> **NOTE**: Conceptually, `Eng.AD.buildFC` is asynchronous, i.e. it does not cause the monitoring loop to stop, although it does require `Eng.AD` to wait for it to respond. Practically, if there is no expected delay from `Eng.AD.buildFC` we can implement it as a synchronous part of the core engine loop managed by `Eng.Scheduler`, with `Eng.AD` checking for its response every tick or so. If the asynchronous approach is preferred (as it will may well be), we should ensure to add another component (e.g. `Eng.AD.buildFCResCheck`) that, if active is checks for `Eng.AD.buildFC`'s response every k ticks (ideally every tick), and that, upon receiving the response, deactivates until the next time `Eng.AD` is called by `Eng.Scheduler`.

### `Eng.AD.buildFC._envIF`
Requirement for a 2-way interface with the environment for building `Eng.AD._FC`.

## `Eng.AD.adReq` (Agent Decision Request)
- Triggered upon receiving `Eng.AD._FC`
- Sends a request to `RM` without waiting for response

> **NOTE**: If `Eng.AD.adResCheck` is not active, there are 2 approaches:
>
> 1. Do not activate `Eng.AD.adReq`
> 2. Activate `Eng.AD.adReq` but:
>    - Add requests to a request queue
>    - Ensure `RM` reads from this request queue
>
> Currently, we are likely to pursue approach 1 due to its ease.

## `Eng.AD.Q` (Agent Decision Response Queue)
Queue to hold responses from `RM`:

- Stores the newest decisions at its queue head
- Deletes any item consumed from its queue tail

> **NOTE**: Deletion upon consume is ideal as decisions need not be repeated.

## `Eng.AD.adResCheck` (Agent Decision Response Check)
- Activates upon `Eng.AD.adReq` trigger
- Attempts to obtain an item from the queue tail of `Eng.AD.Q`
- High-to-medium frequency (ideally per engine tick)
- Deactivates once the item with the required responses from `RM` is received <br> *Required responses are*...
  - `RM._AD` (agent decisions)
  - `RM._ADR` (agent decision reasoning (human-readable))

# `Eng._D-1` (Engine Decision Object - V1)
Consolidated `RM._AD` and `RM._ADR` after:

- Structural validation (ensuring the expected contracts are adhered to)
- Logical validation (ensuring values are within a valid range)

# `Eng.HD` (Human Decision)
## `Eng.HD.hdReq` (Human Decision Request)
- Asynchronous
- Triggered upon receiving decision response

> **NOTE**: If `Eng.HD.hdResCheck` is not active, there are 2 approaches:
>
> 1. Do not activate `Eng.HD.hdReq`
> 2. Activate `Eng.HD.hdReq` but:
>    - Add requests to a request queue
>    - Ensure `HITL` reads from this request queue
>
> Currently, we shall pursue approach 2 to ensure:
>
> - All agent decisions will be reviewed by the human, despite human delay
> - 1 or more agent decisions can be edited/cancelled by the human as a batch

## `Eng.HD.Q` (Human Decision Response Queue)
Queue to hold responses from `HITL`:

- Stores the newest decisions at its queue head
- Deletes any item consumed from its queue tail

> **NOTE**: Deletion upon consume is ideal as decisions need not be repeated.

## `Eng.HD.hdResCheck` (Human Decision Response Check)
- Synchronous
- Reads from the queue tail of `Eng.HD.Q`
- High-to-medium frequency (ideally per engine tick)
- Activates upon `Eng.HD.hdReq` trigger
- Deactivates once response from `HITL` (`HITL._D`) is received

# `Eng._D-2` (Engine Decision Object - V2)
`HITL._D` after passing structural and logical validation.

# `Eng.Exec` (Executor)
Executes the decisions in `Eng._D-2` via `Eng.Mon._envIF`.

> **NOTE**: Conceptually, `Eng.Exec` is asynchronous, i.e. it does not cause the monitoring loop to stop. Practically, if there is no expected delay from `Eng.Exec` we can implement it as a synchronous part of the core engine loop managed by `Eng.Scheduler`.

## `Eng.Exec._envIF`
Requirement for a 1-way interface with the environment for decision execution.
