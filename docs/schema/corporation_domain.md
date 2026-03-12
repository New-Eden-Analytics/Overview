# NEA MVP Schema — Corporation Production State Domain

## 1. Purpose

The Corporation Production State domain contains the current corporation-specific operational data required for NEA v1 to determine:

- which blueprint instances are currently available for production
- the ME and TE values associated with those blueprint instances
- which items are currently in active production
- which items are already present in the staging pipeline
- which items are already listed for sale

This domain serves as the **canonical source of current corporation production state** used by the NEA recommendation engine.

The Corporation Production State domain does **not** contain static production definitions, market pricing, recommendation outputs, or configuration that assigns semantic meaning to specific containers or hangars.

---

# 2. Scope

## Included

The Corporation Production State domain includes:

- corporation-owned blueprint instances
- per-blueprint ME and TE values
- active industry jobs
- filtered staging inventory state
- active sell orders
- corporation location/container records relevant to these operational tables

This information allows NEA to determine:

- whether at least one usable blueprint instance exists for a manufacturable item
- whether an item is already in the production-to-sale pipeline
- whether an item should be excluded from recommendation output

## Excluded

The Corporation Production State domain does **not** include:

- static blueprint definitions
- item definitions
- blueprint material requirements
- market prices
- market history
- recommendation results
- forecasting outputs
- configuration declaring which container or hangar is used for blueprint storage, staging, or sales workflows
- buy orders
- historical snapshots of corporation production state

These concerns belong to other domains.

---

# 3. Design Goals

The Corporation Production State domain schema must satisfy the following goals:

1. Represent current corp operational state relevant to production recommendations.
2. Support existence-based blueprint eligibility checks.
3. Support exclusion of items already in production, staging, or on the market.
4. Preserve factual corporation/location/container state while keeping workflow meaning in configuration.
5. Remain current-state oriented, without introducing historical state management into MVP.
6. Provide clean joins into the Reference, Market Observation, and Configuration domains.

This domain should remain focused on **current corp facts**, not user interpretation or recommendation logic.

---

# 4. Canonical Entities

The Corporation Production State domain for NEA v1 consists of five core tables:

1. `corp_location`
2. `corp_blueprint_instance`
3. `corp_industry_job`
4. `corp_staging_inventory`
5. `corp_active_sell_order`

These tables define the current corp state required to determine which candidate items are actually eligible for recommendation.

---

# 5. Table Definitions

## 5.1 `corp_location`

Defines corporation-relevant locations, hangars, containers, or storage entities referenced by corporation production state tables.

This table stores the factual location/container identities used by blueprint inventory, staging inventory, and other corporation-state records.

### Purpose

- Provide stable identifiers for corporation storage and operational locations
- Avoid duplicating raw location metadata across operational tables
- Support configuration mappings that assign semantic meaning to these locations

### Primary Key

```
corporation_id
location_id
```

### Core Columns

| Column | Description |
|---|---|
| corporation_id | Corporation identifier |
| location_id | Canonical corporation location/container identifier |
| location_name | Optional human-readable name |
| location_type | Optional classification such as station hangar, container, office, etc. |
| parent_location_id | Optional parent location reference |
| station_id | Optional station identifier if applicable |
| is_active | Optional convenience flag indicating currently valid/seen location |

### Notes

This table stores factual corporation location/container records only.

The question of **which** location is used for blueprint storage, staging, or other workflow purposes is handled in the **Configuration domain**, not here.

---

## 5.2 `corp_blueprint_instance`

Defines corporation-owned blueprint instances currently available within the corporation state.

Each row represents a single owned blueprint instance.

### Purpose

- Represent actual corp-owned blueprint instances
- Support checks for whether at least one blueprint exists for a producible item
- Store blueprint-specific ME and TE values

### Primary Key

```
corporation_id
blueprint_instance_id
```

### Core Columns

| Column | Description |
|---|---|
| corporation_id | Corporation identifier |
| blueprint_instance_id | Unique blueprint instance identifier |
| blueprint_type_id | FK → Reference domain `blueprint.blueprint_type_id` |
| location_id | FK → `corp_location.location_id` within corporation |
| material_efficiency | Blueprint-specific ME value |
| time_efficiency | Blueprint-specific TE value |
| quantity | Optional quantity field if source provides stack semantics |
| is_copy | Optional flag indicating BPC vs BPO |
| runs_remaining | Optional remaining run count; ignored by v1 logic |
| is_active | Optional convenience flag for current availability |

### Notes

NEA v1 uses this table primarily to answer:

> Does at least one usable blueprint instance exist?

Blueprint instance counts are not directly important to recommendation ranking in v1, but instance-level storage is still canonical and future-safe.

BPC run counts may be stored if available, but they are ignored by MVP recommendation logic.

---

## 5.3 `corp_industry_job`

Defines active corporation industry jobs relevant to production-state exclusion.

Each row represents a current industry job record.

### Purpose

- Identify items currently being produced
- Exclude items already in active production from recommendation output
- Preserve job-level visibility for debugging and future extension

### Primary Key

```
corporation_id
job_id
```

### Core Columns

| Column | Description |
|---|---|
| corporation_id | Corporation identifier |
| job_id | Unique industry job identifier |
| blueprint_type_id | Optional FK → Reference domain `blueprint.blueprint_type_id` |
| product_type_id | FK → Reference domain `item_type.type_id` |
| location_id | Optional FK → `corp_location.location_id` |
| status | Job status |
| start_date | Optional job start timestamp |
| end_date | Optional job end timestamp |
| installer_id | Optional character identifier |
| runs | Optional run count |
| is_active | Convenience flag indicating non-completed state |

### Notes

NEA v1 filters on **product_type_id**, since product identity is what matters for recommendation exclusion.

This table should only retain jobs relevant to current-state evaluation. Historical jobs are out of scope.

If implementation cost proves unexpectedly high, this table may later be simplified to a more minimal active-product representation, but full job rows are the preferred MVP design.

---

## 5.4 `corp_staging_inventory`

Defines filtered corporation staging inventory for the configured production-to-sale staging workflow.

Each row represents the presence of an item currently sitting in the relevant staging pipeline.

### Purpose

- Identify items already staged and therefore already in the production-to-sale workflow
- Support binary recommendation exclusion logic

### Primary Key

```
corporation_id
location_id
type_id
```

### Core Columns

| Column | Description |
|---|---|
| corporation_id | Corporation identifier |
| location_id | FK → `corp_location.location_id` |
| type_id | FK → Reference domain `item_type.type_id` |
| quantity | Quantity currently present |
| last_seen_at | Optional timestamp of most recent state refresh |

### Notes

This table intentionally stores **filtered staging state only**, not full corporation asset inventory.

NEA v1 only needs to answer:

> Is this item already sitting in staging?

Quantity is stored because it is cheap to retain, but v1 logic uses presence/absence only.

---

## 5.5 `corp_active_sell_order`

Defines active corporation sell orders relevant to recommendation exclusion.

Each row represents an active sell order for an item.

### Purpose

- Identify items already on the market
- Exclude items already listed for sale from recommendation output

### Primary Key

```
corporation_id
order_id
```

### Core Columns

| Column | Description |
|---|---|
| corporation_id | Corporation identifier |
| order_id | Unique market order identifier |
| type_id | FK → Reference domain `item_type.type_id` |
| location_id | Optional FK → `corp_location.location_id` if relevant |
| station_id | Station identifier |
| volume_total | Optional original order quantity |
| volume_remaining | Optional remaining quantity |
| price | Optional listed price |
| issued_at | Optional order issue timestamp |
| is_active | Convenience flag indicating active sell order |

### Notes

Only **active sell orders** are stored in this domain.

Buy orders are explicitly excluded from MVP scope.

NEA v1 uses this table as a binary exclusion source:

> If any active sell order exists for the item at the relevant station, exclude the item.

---

# 6. Relationships

The core relationships within the Corporation Production State domain are:

```
corp_blueprint_instance.location_id -> corp_location.location_id
corp_industry_job.location_id -> corp_location.location_id
corp_staging_inventory.location_id -> corp_location.location_id
corp_active_sell_order.location_id -> corp_location.location_id
```

Cross-domain relationships include:

```
corp_blueprint_instance.blueprint_type_id -> Reference.blueprint.blueprint_type_id
corp_industry_job.blueprint_type_id -> Reference.blueprint.blueprint_type_id
corp_industry_job.product_type_id -> Reference.item_type.type_id
corp_staging_inventory.type_id -> Reference.item_type.type_id
corp_active_sell_order.type_id -> Reference.item_type.type_id
```

Conceptually:

- corp blueprint instances connect corp ownership to static blueprint definitions
- corp jobs connect current production state to product outputs
- staging inventory and sell orders connect corp operational state to item exclusion logic
- corp locations provide factual storage/location structure for all corporation-state tables

---

# 7. Domain Rules

The following rules apply to the Corporation Production State domain.

### Rule 1 — Current State Only

This domain stores only current corporation operational state.

Historical snapshots, old jobs, prior sell orders, and prior staging states are out of scope for MVP.

---

### Rule 2 — Corporation-Scoped Rows

All operational tables must include `corporation_id`.

Even though NEA v1 is single-corp in practice, corporation scoping is part of the canonical schema and prevents future redesign.

---

### Rule 3 — Blueprint Instances, Not Blueprint Definitions

This domain stores owned blueprint instances, not static blueprint definitions.

Static blueprint metadata belongs in the Reference domain.

---

### Rule 4 — Factual Locations, Semantic Meanings Elsewhere

This domain stores factual corporation location/container/hangar records.

The meaning of those locations — for example, “blueprint storage” or “staging” — is defined in the Configuration domain, not here.

---

### Rule 5 — Product Identity Drives Exclusion

Recommendation exclusion is based on product/item identity.

For this reason, `corp_industry_job` must store `product_type_id` explicitly.

---

### Rule 6 — Binary Exclusion Logic

For MVP recommendation purposes:

- if an active production job exists for an item, exclude it
- if the item exists in staging inventory, exclude it
- if an active sell order exists for the item, exclude it

Quantities do not alter the binary exclusion rule in v1.

---

### Rule 7 — Buy Orders Excluded

Buy orders are not tracked in this domain for MVP.

They are irrelevant to current recommendation exclusion logic.

---

# 8. Expected Usage in NEA v1

The Corporation Production State domain supports the following core operations.

### Determine Blueprint Eligibility

By joining `corp_blueprint_instance` to the Reference domain, NEA can determine whether at least one blueprint instance exists for a manufacturable item.

---

### Exclude Items Already In Production

By checking `corp_industry_job.product_type_id`, NEA can exclude items currently being produced.

---

### Exclude Items Already In Staging

By checking `corp_staging_inventory.type_id`, NEA can exclude items already present in the production-to-sale pipeline.

---

### Exclude Items Already Listed For Sale

By checking `corp_active_sell_order.type_id`, NEA can exclude items already listed on the market.

---

### Apply Workflow Meaning Through Configuration

By combining corporation-state facts with Configuration domain selections, NEA can determine which blueprint locations, staging locations, and sale stations are relevant to the active workflow.

---

# 9. Out-of-Scope Extensions

The following possible extensions are intentionally excluded from the MVP Corporation Production State domain:

- historical corp-state snapshots
- buy-order tracking
- multi-stage staging workflow modeling
- production capacity or slot modeling
- richer character-level responsibility tracking
- logistics or hauling state
- market-order analytics beyond binary exclusion
- automatic suppression rules based on quantities or thresholds

These may be introduced in future versions if required.

---

# 10. Minimal MVP Table Summary

| Table | Purpose |
|---|---|
| `corp_location` | Canonical registry of corporation locations/containers/hangars referenced by corp-state tables |
| `corp_blueprint_instance` | Corporation-owned blueprint instances and their ME/TE values |
| `corp_industry_job` | Current active industry jobs used for production exclusion |
| `corp_staging_inventory` | Filtered staging inventory used for binary pipeline exclusion |
| `corp_active_sell_order` | Active sell orders used for binary market exclusion |

This set of tables represents the **minimum Corporation Production State domain required to support the NEA v1 recommendation engine**.