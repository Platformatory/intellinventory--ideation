**Warehouse Reorder Simulation**

<h1>Simulation - Specification</h1>

---

**Contents**:

- [0. Overview](#0-overview)
- [1. Constraints](#1-constraints)
  - [Fixed-World Constraints](#fixed-world-constraints)
  - [Structural Constraints](#structural-constraints)
  - [Temporal Constraints](#temporal-constraints)
- [2. Requirements](#2-requirements)
  - [Functional Requirements](#functional-requirements)
  - [Non-Functional Requirements](#non-functional-requirements)
- [3. Parameters](#3-parameters)
  - [3.1 Simulation-Level Parameters](#31-simulation-level-parameters)
  - [3.2 Item-Type Parameters](#32-item-type-parameters)
  - [3.3 Supplier Parameters](#33-supplier-parameters)
  - [3.4 Consumer Parameters](#34-consumer-parameters)
  - [3.5 Demand and Supply Pattern Parameters](#35-demand-and-supply-pattern-parameters)
  - [3.6 Lead Time Parameters](#36-lead-time-parameters)
  - [3.7 Cost Model Parameters](#37-cost-model-parameters)
  - [3.8 Disruption Parameters](#38-disruption-parameters)
- [4. Environment Data Tables](#4-environment-data-tables)
  - [`env_item_types`](#env_item_types)
  - [`env_suppliers`](#env_suppliers)
  - [`env_supplier_item_map`](#env_supplier_item_map)
  - [`env_consumers`](#env_consumers)
  - [`env_consumer_item_map`](#env_consumer_item_map)
  - [`env_patterns`](#env_patterns)
  - [`env_disruption_schedule`](#env_disruption_schedule)
  - [`env_sim_config`](#env_sim_config)
- [5. Operational Data Tables](#5-operational-data-tables)
  - [`ops_warehouse_state`](#ops_warehouse_state)
  - [`ops_pending_orders`](#ops_pending_orders)
  - [`ops_cost_accumulator`](#ops_cost_accumulator)
  - [`ops_active_disruptions`](#ops_active_disruptions)
- [6. Historical Data Tables](#6-historical-data-tables)
  - [`hist_demand_actuals`](#hist_demand_actuals)
  - [`hist_supply_arrivals`](#hist_supply_arrivals)
  - [`hist_reorder_decisions`](#hist_reorder_decisions)
  - [`hist_cost_by_tick`](#hist_cost_by_tick)
- [7. Event Logs](#7-event-logs)
  - [`event_log`](#event_log)

---

# 0. Overview

The goal:

***Implement an agentic AI solution for a warehouse reorder use-case.***

---

Essentially, the use-case is as follows:

- Warehouse stocks items of types A, B, C... etc.
- Suppliers exist for each item type
- Consumers exist for each item type

***First, we need to simulate the use-case.***

To keep it simple, we have the following assumptions:

- The set of suppliers and consumers are fixed
- One supplier per item type
- One consumer per item type

But, naturally, consumer demand patterns and supplier supply patterns can vary.

However, to ensure the simulation setup is extensible:

- I want to be able to vary supply and demand patterns as per:
   - Statistical distributions
   - Custom patterns
- I want to be able to make 1 supplier the source of 1 or more item types
- I want to be able to make 1 consumer the demander for 1 or more item types

# 1. Constraints

## Fixed-World Constraints
- The set of item types is fixed for the duration of a simulation run
- The set of suppliers is fixed <br> => *No supplier can enter or exit during a run*
- The set of consumers is fixed <br> => *No consumer can enter or exit during a run*
- Each item type has exactly:
    - 1 designated supplier
    - 1 designated consumer
- Supplier failures are excluded as a disruption type <br> => *All suppliers remain operational throughout the run*

## Structural Constraints
- The warehouse is a single-echelon node<br> => *No distribution centre hierarchy*
- Stock levels cannot go below zero <br> => *Unmet demand is recorded as a stockout, not a backorder (**no negative inventory**)*
- A reorder can be placed:
    - Only if warehouse has enough budget remaining
    - Unless the budget constraint is disabled.
- Reorder quantities must be non-negative integers <br> => *Partial units are not supported*
- Lead time:
    - Is expressed in simulation ticks (e.g. days)
    - Must be a positive integer of at least 1
    - Must be decided per item per supplier per tick

> **NOTE**: *Lead time is the total latency between the initiation and completion of a process, such as the time from placing an order to receiving it, or from starting a project to finishing it.*

## Temporal Constraints
- The simulation has a number of modes:
    - Running for a defined number of discrete ticks
    - Running forever non-cyclically
    - Running forever cyclically
- All events are resolved in a fixed sequence within each tick:
    - (1) Pending supply arrives
    - (2) Demand is drawn
    - (3) Stock is updated
    - (4) The agent evaluates and optionally places reorders
- Disruption events are either of the following:
    - Pre-scheduled
    - Stochastically injected before the tick resolves

# 2. Requirements

## Functional Requirements
- **FR-01**: The simulation must:
    - Advance through discrete time ticks
    - Persist state at each tick
- **FR-02**: Demand must be drawn:
    - Per item type per tick
    - From a configurable pattern; either one of:
        - Statistical distribution
        - Custom schedule
- **FR-03**: Supply per reorder must arrive:
    - After the configured lead time
    - For that item type, supplier and tick
- **FR-04**: The agent must receive a complete view of: <br> **NOTE**: *This must be received before making a reorder decision*
    - Current stock
    - Pending orders (with expected arrival ticks)
    - Recent demand history
    - Active disruptions
- **FR-05**: The cost model must:
    - Accumulate (each tick):
        - Holding costs
        - Stockout costs
        - Order costs
    - Expose a running total to:
        - The agent
        - The observability layer
- **FR-06**: Disruption events:
    - Must be injectable as either:
        - Pre-defined schedules
        - Stochastically generated
    - Must affect:
        - Demand magnitude
        - Lead time
        - Transit loss
- **FR-07**: The simulation must be fully reproducible given:
    - The same random seed
    - The same world configuration
- **FR-08** All state transitions must be written to an event log **NOTE**: *This event log must be append-only*

## Non-Functional Requirements
- **NFR-01**: The configuration schema must allow:
    - Any item type to be mapped to:
        - Any supplier
        - Any consumer entity
    - Without code changes
- **NFR-02**: Pattern definitions (distributions and custom schedules) must be:
    - Swappable per item type
    - Without altering the engine
- **NFR-03**: The simulation clock tick size (e.g. daily, hourly) must be:
    - A configuration parameter
    - Not a hardcoded assumption
- **NFR-04**: The cost model parameters must be:
    - Independently configurable
    - Per item type
- **NFR-05**: The system must support:
    - At least 2 disruption types
    - Simultaneous activation of these on the same item type

# 3. Parameters

## 3.1 Simulation-Level Parameters

| Parameter | Type | Description |
|---|---|---|
| `sim_id` | string | Unique identifier for the simulation run |
| `random_seed` | integer | Seed for all stochastic processes; ensures reproducibility |
| `num_ticks` | integer | Total number of ticks to simulate; null = infinite |
| `tick_unit` | enum | Granularity of one tick: `hour`, `day`, `week` |
| `budget_limit` | float | Total spend cap across all reorders for the run; null = unlimited |
| `start_timestamp` | datetime | Wall-clock anchor for tick 0 |

## 3.2 Item-Type Parameters

| Parameter | Type | Description | Relates to |
|---|---|---|---|
| `item_id` | string | Unique item type identifier | All layers |
| `initial_stock` | integer | Stock on hand at tick 0 | Warehouse state |
| `reorder_point` | integer | Stock level that triggers agent evaluation | Agent decision |
| `min_order_qty` | integer | Minimum units per reorder (MOQ) | Lead time, cost model |
| `max_order_qty` | integer | Maximum units per single reorder | Agent constraint |
| `unit_value` | float | Value of one unit, used to compute holding cost | Cost model |

## 3.3 Supplier Parameters

| Parameter | Type | Description | Relates to |
|---|---|---|---|
| `supplier_id` | string | Unique supplier identifier | Mapping tables |
| `item_ids_supplied` | list[string] | Item types this supplier covers | Supplier-item mapping |
| `base_lead_time_ticks` | integer | Normal ticks between reorder placement and arrival | Lead time |
| `lead_time_variability` | float | Std dev of lead time in ticks (0 = deterministic) | Lead time, disruptions |

## 3.4 Consumer Parameters

| Parameter | Type | Description | Relates to |
|---|---|---|---|
| `consumer_id` | string | Unique consumer identifier | Mapping tables |
| `item_ids_demanded` | list[string] | Item types this consumer demands | Consumer-item mapping |

## 3.5 Demand and Supply Pattern Parameters

Each pattern entry applies to one...

`(item_id, role)` pair

... where role is `demand` or `supply`.

| Parameter | Type | Description |
|---|---|---|
| `pattern_type` | enum | `statistical` or `custom` |
| `distribution` | enum | `poisson`, `normal`, `uniform`, `negative_binomial`, `zero_inflated_poisson`; null if custom |
| `dist_params` | json | Distribution-specific parameters (e.g. `{"mu": 50}` for Poisson) |
| `custom_schedule` | list[float] | Ordered list of values, one per tick; cycled if shorter than `num_ticks`; null if statistical |
| `seasonal_multiplier_schedule` | list[float] | Optional per-tick multiplier applied on top of the base pattern (e.g. for weekly seasonality) |
| `noise_std` | float | Optional Gaussian noise added after pattern evaluation |

## 3.6 Lead Time Parameters

> **NOTE**: *Lead time is the delay (in ticks) between when a reorder is placed and when the goods arrive at the warehouse.*

| Parameter | Type | Description |
|---|---|---|
| `base_lead_time_ticks` | integer | Set on supplier; see [3.3 Supplier Parameters](#33-supplier-parameters) |
| `lead_time_variability` | float | Set on supplier; see [3.3 Supplier Parameters](#33-supplier-parameters) |
| `disruption_lead_time_multiplier` | float | Applied during active transit delay disruption (e.g. 2.0 = double lead time) |

At reorder placement, the engine:

- Samples an actual lead time as: <br> `actual_lead_time = round(Normal(base_lead_time_ticks, lead_time_variability))`
- Then applies any active disruption multiplier <br> **NOTE**: *This multiplier is clamped to a minimum of 1*

## 3.7 Cost Model Parameters

| Parameter | Type | Per Item? | Description |
|---|---|---|---|
| `holding_cost_per_unit_per_tick` | float | Yes | Cost of holding one unit in stock for one tick |
| `stockout_cost_per_unit_per_tick` | float | Yes | Penalty cost per unit of unmet demand per tick |
| `order_fixed_cost` | float | Yes | Fixed cost incurred each time a reorder is placed, regardless of quantity |
| `order_variable_cost_per_unit` | float | Yes | Variable cost per unit ordered (price paid to supplier) |
| `transit_loss_cost_per_unit` | float | Yes | Cost per unit lost in transit during a transit loss disruption |

**Cost accumulation per tick:**
- Holding cost = `current_stock × holding_cost_per_unit_per_tick`
- Stockout cost = `unmet_demand × stockout_cost_per_unit_per_tick`
- Order cost (at placement) = `order_fixed_cost + (order_qty × order_variable_cost_per_unit)`
- Transit loss cost (at arrival) = `units_lost × transit_loss_cost_per_unit`

## 3.8 Disruption Parameters

Each disruption record applies to:

- A specific item type
- Over a specific tick range

| Parameter | Type | Description |
|---|---|---|
| `disruption_id` | string | Unique identifier |
| `item_id` | string | Affected item type |
| `disruption_type` | enum | `demand_spike`, `demand_suppression`, `transit_delay`, `transit_loss` |
| `start_tick` | integer | First tick of the disruption |
| `end_tick` | integer | Last tick of the disruption (inclusive) |
| `magnitude` | float | Type-specific: <ul><li>multiplier for demand disruptions</li><li>multiplier for lead time in transit delay</li><li>fraction of in-transit units lost for transit loss</li></ul> |
| `is_stochastic` | boolean | If true, the disruption is injected probabilistically each tick using `trigger_probability` |
| `trigger_probability` | float | Per-tick probability of a stochastic disruption activating (null if not stochastic) |

**Disruption type effects**:

`demand_spike`:

Multiplies drawn demand by `magnitude` (e.g. 3.0 = triple demand).

`demand_suppression`:

Multiplies drawn demand by `magnitude` (e.g. 0.2 = 80% reduction).

`transit_delay`:

Multiplies computed lead time by `magnitude`:

  - For any reorder placed
  - During the disruption window

`transit_loss`

On arrival:

- A fraction `magnitude` of the incoming order quantity is lost
- Only the remainder enters stock

# 4. Environment Data Tables

- These tables define the static world configuration
- They are written once before the simulation runs
- They do not change during a run

## `env_item_types`
Defines all item types in the simulation.

| Column | Type | Description |
|---|---|---|
| `item_id` | string PK | Unique item identifier |
| `item_name` | string | Human-readable label |
| `unit_value` | float | Value per unit (for cost model) |
| `initial_stock` | integer | Stock at tick 0 |
| `reorder_point` | integer | Stock threshold that prompts agent evaluation |
| `min_order_qty` | integer | Minimum reorder quantity (MOQ) |
| `max_order_qty` | integer | Maximum reorder quantity per order |
| `holding_cost_per_unit_per_tick` | float | Holding cost rate |
| `stockout_cost_per_unit_per_tick` | float | Stockout penalty rate |
| `order_fixed_cost` | float | Fixed cost per reorder event |
| `order_variable_cost_per_unit` | float | Variable cost per unit ordered |
| `transit_loss_cost_per_unit` | float | Cost per unit lost in transit |

## `env_suppliers`
Defines all supplier entities.

| Column | Type | Description |
|---|---|---|
| `supplier_id` | string PK | Unique supplier identifier |
| `supplier_name` | string | Human-readable label |
| `base_lead_time_ticks` | integer | Baseline lead time for orders placed with this supplier |
| `lead_time_variability` | float | Standard deviation of lead time noise |

## `env_supplier_item_map`
Many-to-many mapping between suppliers and item types.

**NOTE**: *Now constrained to 1 supplier per item type per simulation run.*

| Column | Type | Description |
|---|---|---|
| `sim_id` | string FK | Simulation run identifier |
| `supplier_id` | string FK | Supplier |
| `item_id` | string FK | Item type served by this supplier |

## `env_consumers`
Defines all consumer entities.

| Column | Type | Description |
|---|---|---|
| `consumer_id` | string PK | Unique consumer identifier |
| `consumer_name` | string | Human-readable label |

## `env_consumer_item_map`
Many-to-many mapping between consumers and item types.

| Column | Type | Description |
|---|---|---|
| `sim_id` | string FK | Simulation run identifier |
| `consumer_id` | string FK | Consumer |
| `item_id` | string FK | Item type demanded by this consumer |

## `env_patterns`
Demand and supply pattern configurations.

| Column | Type | Description |
|---|---|---|
| `pattern_id` | string PK | Unique pattern identifier |
| `sim_id` | string FK | Simulation run |
| `item_id` | string FK | Item type this pattern applies to |
| `role` | enum | `demand` or `supply` |
| `pattern_type` | enum | `statistical` or `custom` |
| `distribution` | string | Distribution name; null if custom |
| `dist_params` | json | Distribution parameters; null if custom |
| `custom_schedule` | array[float] | Per-tick values; null if statistical |
| `seasonal_multiplier_schedule` | array[float] | Optional seasonal overlay |
| `noise_std` | float | Optional Gaussian noise; 0 if none |

## `env_disruption_schedule`
Pre-defined disruption events for a simulation run.

| Column | Type | Description |
|---|---|---|
| `disruption_id` | string PK | Unique disruption identifier |
| `sim_id` | string FK | Simulation run |
| `item_id` | string FK | Affected item type |
| `disruption_type` | enum | `demand_spike`, `demand_suppression`, `transit_delay`, `transit_loss` |
| `start_tick` | integer | First tick of effect |
| `end_tick` | integer | Last tick of effect (inclusive) |
| `magnitude` | float | Effect multiplier or fraction |
| `is_stochastic` | boolean | Whether activation is probabilistic per tick |
| `trigger_probability` | float | Per-tick activation probability; null if not stochastic |

## `env_sim_config`
Top-level simulation run configuration.

| Column | Type | Description |
|---|---|---|
| `sim_id` | string PK | Simulation run identifier |
| `random_seed` | integer | Global RNG seed |
| `num_ticks` | integer | Total ticks to simulate |
| `tick_unit` | enum | `hour`, `day`, `week` |
| `budget_limit` | float | Total budget cap; null = unlimited |
| `start_timestamp` | datetime | Wall-clock anchor for tick 0 |
| `created_at` | datetime | When this config was registered |

# 5. Operational Data Tables

These tables hold live state during the simulation run. They are read and written by the simulation engine each tick.

## `ops_warehouse_state`
Current stock level per item type, updated at the end of each tick.

| Column | Type | Description |
|---|---|---|
| `sim_id` | string FK | Simulation run |
| `tick` | integer | Current tick |
| `item_id` | string FK | Item type |
| `stock_on_hand` | integer | Units currently in warehouse |
| `stock_in_transit` | integer | Units on order, not yet arrived |
| `expected_arrivals_next_tick` | integer | Units due to arrive next tick |
| `updated_at` | datetime | Timestamp of last write |

## `ops_pending_orders`
Reorders that have been placed but not yet received.

| Column | Type | Description |
|---|---|---|
| `order_id` | string PK | Unique order identifier |
| `sim_id` | string FK | Simulation run |
| `item_id` | string FK | Item type ordered |
| `supplier_id` | string FK | Supplier fulfilling the order |
| `order_tick` | integer | Tick at which the order was placed |
| `expected_arrival_tick` | integer | Tick at which goods are expected to arrive |
| `order_qty` | integer | Units ordered |
| `status` | enum | `pending`, `arrived`, `partially_lost` |
| `disruptions_active_at_order` | array[string] | Disruption IDs active when the order was placed |

## `ops_cost_accumulator`
Running cost totals per item type, updated each tick.

| Column | Type | Description |
|---|---|---|
| `sim_id` | string FK | Simulation run |
| `tick` | integer | Current tick |
| `item_id` | string FK | Item type |
| `cumulative_holding_cost` | float | Total holding cost accrued so far |
| `cumulative_stockout_cost` | float | Total stockout penalty accrued so far |
| `cumulative_order_cost` | float | Total order cost (fixed + variable) accrued so far |
| `cumulative_transit_loss_cost` | float | Total transit loss cost accrued so far |
| `cumulative_total_cost` | float | Sum of all cost components |
| `remaining_budget` | float | Budget remaining for the run; null if unlimited |

## `ops_active_disruptions`
Disruptions currently in effect at the current tick.

| Column | Type | Description |
|---|---|---|
| `sim_id` | string FK | Simulation run |
| `tick` | integer | Current tick |
| `disruption_id` | string FK | Disruption record |
| `item_id` | string FK | Affected item type |
| `disruption_type` | enum | Type of disruption |
| `effective_magnitude` | float | Actual magnitude applied this tick (may differ from scheduled if stochastic) |
| `is_active_this_tick` | boolean | Whether stochastic disruption triggered this tick |

# 6. Historical Data Tables

- These are append-only summaries used for agent context and observability
- They accumulate across ticks

## `hist_demand_actuals`
Realised demand drawn each tick, before stockout clipping.

| Column | Type | Description |
|---|---|---|
| `sim_id` | string FK | Simulation run |
| `tick` | integer | Tick |
| `item_id` | string FK | Item type |
| `consumer_id` | string FK | Consumer that generated the demand |
| `raw_demand` | float | Demand sampled from pattern before disruption |
| `disrupted_demand` | float | Demand after disruption multipliers applied |
| `fulfilled_demand` | integer | Units actually issued from stock |
| `unmet_demand` | integer | Units of demand not fulfilled (stockout volume) |
| `pattern_id` | string FK | Pattern used to generate this demand |

## `hist_supply_arrivals`
Record of every order arrival.

| Column | Type | Description |
|---|---|---|
| `sim_id` | string FK | Simulation run |
| `tick` | integer | Tick of arrival |
| `order_id` | string FK | Order that arrived |
| `item_id` | string FK | Item type |
| `supplier_id` | string FK | Supplier |
| `ordered_qty` | integer | Original order quantity |
| `arrived_qty` | integer | Units that actually arrived (after transit loss) |
| `lost_qty` | integer | Units lost in transit (0 if no disruption) |
| `actual_lead_time_ticks` | integer | Ticks between order placement and this arrival |

## `hist_reorder_decisions`
Every reorder decision made by the agent, including decisions not to reorder.

| Column | Type | Description |
|---|---|---|
| `sim_id` | string FK | Simulation run |
| `tick` | integer | Tick of decision |
| `item_id` | string FK | Item type evaluated |
| `stock_on_hand_at_decision` | integer | Stock level seen by agent |
| `stock_in_transit_at_decision` | integer | In-transit units seen by agent |
| `decision` | enum | `reorder` or `hold` |
| `order_qty` | integer | Units ordered; 0 if hold |
| `order_id` | string FK | Created order ID; null if hold |
| `agent_reasoning` | string | Free-text or structured reasoning from agent (for LLM agents) |
| `agent_version` | string | Version/identifier of the agent policy used |

## `hist_cost_by_tick`
Snapshot of cost components accrued in each individual tick (not cumulative).

| Column | Type | Description |
|---|---|---|
| `sim_id` | string FK | Simulation run |
| `tick` | integer | Tick |
| `item_id` | string FK | Item type |
| `holding_cost` | float | Holding cost this tick |
| `stockout_cost` | float | Stockout cost this tick |
| `order_cost` | float | Order cost this tick (0 if no order placed) |
| `transit_loss_cost` | float | Transit loss cost this tick |
| `total_cost` | float | Sum of above |

# 7. Event Logs

The event log is an append-only, immutable record of every state-changing event in the simulation. It is the source of truth for replay and audit.

## `event_log`
Single unified event stream; all event types write here.

| Column | Type | Description |
|---|---|---|
| `event_id` | string PK | Unique event identifier (UUID) |
| `sim_id` | string FK | Simulation run |
| `tick` | integer | Tick at which the event occurred |
| `event_type` | enum | See event types below |
| `item_id` | string FK | Item type involved; null for sim-level events |
| `entity_id` | string | ID of the primary entity (supplier, consumer, order, disruption); null if not applicable |
| `payload` | json | Event-specific data (quantities, costs, magnitudes, reasoning) |
| `logged_at` | datetime | Wall-clock time the event was written |

**Event types**:

| Event Type | Fired When | Key Payload Fields |
|---|---|---|
| `SIM_STARTED` | Tick 0 initialisation | `config_snapshot` |
| `SIM_ENDED` | Final tick completes | `total_cost`, `total_stockout_ticks`, `total_reorders` |
| `DEMAND_DRAWN` | Demand sampled each tick per item | `raw_demand`, `disrupted_demand`, `fulfilled`, `unmet` |
| `SUPPLY_ARRIVED` | Pending order arrives | `order_id`, `ordered_qty`, `arrived_qty`, `lost_qty` |
| `REORDER_PLACED` | Agent places a reorder | `order_id`, `order_qty`, `expected_arrival_tick`, `order_cost` |
| `REORDER_HELD` | Agent evaluates and decides not to reorder | `stock_on_hand`, `stock_in_transit`, `reasoning` |
| `DISRUPTION_ACTIVATED` | Disruption begins or stochastic disruption triggers | `disruption_id`, `disruption_type`, `effective_magnitude` |
| `DISRUPTION_DEACTIVATED` | Disruption window ends | `disruption_id` |
| `STOCKOUT_OCCURRED` | Unmet demand > 0 in a tick | `unmet_demand`, `stockout_cost` |
| `BUDGET_WARNING` | Remaining budget falls below 10% of limit | `remaining_budget`, `budget_limit` |
| `BUDGET_EXHAUSTED` | Budget reaches zero | `tick`, `remaining_budget` |
| `COST_ACCRUED` | End-of-tick cost accumulation | `holding_cost`, `stockout_cost`, `order_cost`, `transit_loss_cost`, `tick_total` |
| `TRANSIT_LOSS_APPLIED` | Units lost from an in-transit order | `order_id`, `lost_qty`, `arrived_qty`, `disruption_id` |
| `LEAD_TIME_EXTENDED` | Transit delay disruption increases lead time of a placed order | `order_id`, `original_lead_time`, `extended_lead_time`, `disruption_id` |