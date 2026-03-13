# NEA MVP Schema — Corporation Production State Domain

Schema Version: NEA MVP v1

---

# 1. Purpose

The Corporation Production State Domain contains the current corporation-specific operational data required for NEA v1 to determine:

- which blueprint instances are available for production
- the ME and TE values associated with those blueprint instances
- which items are currently in active production
- which items are already present in the staging pipeline
- which items are already listed for sale

This domain serves as the **canonical source of current corporation production state** used by the NEA recommendation engine.

The Corporation Production State Domain does **not** contain static production definitions, market pricing, recommendation outputs, or configuration that assigns semantic meaning to specific locations.

---

# 2. Scope

## Included

The Corporation Production State Domain includes:

- corporation-owned blueprint instances
- per-blueprint ME and TE values
- active industry jobs
- filtered staging inventory state
- active sell orders
- corporation locations used by those records

This information allows NEA to determine:

- whether a blueprint instance exists for a manufacturable item
- whether an item should be excluded from recommendation output

## Excluded

The Corporation Production State Domain does **not** include:

- blueprint definitions
- item definitions
- blueprint material requirements
- market prices
- market history
- recommendation results
- forecasting outputs
- configuration defining workflow locations
- buy orders
- historical snapshots of corporation state

These concerns belong to other domains.

---

# 3. Design Goals

The Corporation Production State Domain schema must satisfy the following goals:

1. Represent current corporation operational state relevant to production recommendations.
2. Support blueprint eligibility checks based on blueprint instance presence.
3. Support exclusion of items already in production, staging, or for sale.
4. Preserve factual corporation/location state while keeping workflow meaning in the Configuration Domain.
5. Remain focused on **current state only**.

This domain represents **factual corporation data**, not interpretation.

---

# 4. Location Model

All corporation locations stored in this domain are **anchored to a station or structure**.

Locations may represent:

- stations
- structures
- corporation hangars
- containers

Containers and hangars are used for storing:

- materials
- produced items
- blueprint instances

All locations are represented uniformly using `location_id`.

The **Configuration Domain assigns workflow meaning** to specific locations.

---

# 5. Canonical Entities

The Corporation Production State Domain consists of five tables:

1. `corp_location`
2. `corp_blueprint_instance`
3. `corp_industry_job`
4. `corp_staging_inventory`
5. `corp_active_sell_order`

---

# 6. Table Definitions

## 6.1 `corp_location`

Defines corporation-relevant locations.

### Primary Key

```
(corporation_id, location_id)
```

### Core Columns

| Column | Description |
|---|---|
| corporation_id | Corporation identifier |
| location_id | Location identifier |
| location_name | Optional readable name |
| location_type | Optional classification (station, hangar, container) |
| station_id | Parent station or structure |
| parent_location_id | Optional parent container |
| is_active | Optional active flag |

### Notes

This table represents **factual locations only**.

Workflow meaning is defined in the Configuration Domain.

---

## 6.2 `corp_blueprint_instance`

Represents corporation-owned blueprint instances.

### Primary Key

```
(corporation_id, blueprint_instance_id)
```

### Core Columns

| Column | Description |
|---|---|
| corporation_id | Corporation identifier |
| blueprint_instance_id | Unique blueprint instance identifier |
| blueprint_type_id | FK → Reference Domain `blueprint.blueprint_type_id` |
| location_id | FK → corp_location.location_id |
| material_efficiency | Blueprint ME value |
| time_efficiency | Blueprint TE value |
| is_copy | Indicates BPC vs BPO |
| runs_remaining | Remaining runs for BPC (ignored in MVP logic) |
| is_active | Indicates blueprint availability |

### Notes

NEA v1 only needs to determine **whether at least one blueprint instance exists**.

Blueprint counts are not relevant for recommendation ranking.

---

## 6.3 `corp_industry_job`

Represents active corporation industry jobs.

### Primary Key

```
(corporation_id, job_id)
```

### Core Columns

| Column | Description |
|---|---|
| corporation_id | Corporation identifier |
| job_id | Industry job identifier |
| blueprint_type_id | FK → Reference Domain `blueprint.blueprint_type_id` |
| product_type_id | FK → Reference Domain `item_type.type_id` |
| location_id | FK → corp_location.location_id |
| status | Job status |
| start_date | Job start time |
| end_date | Job completion time |
| runs | Run count |
| is_active | Indicates job not completed |

### Notes

Recommendation exclusion is based on `product_type_id`.

---

## 6.4 `corp_staging_inventory`

Represents filtered staging inventory.

### Primary Key

```
(corporation_id, location_id, type_id)
```

### Core Columns

| Column | Description |
|---|---|
| corporation_id | Corporation identifier |
| location_id | FK → corp_location.location_id |
| type_id | FK → Reference Domain `item_type.type_id` |
| quantity | Quantity present |
| last_seen_at | Timestamp of last refresh |

### Notes

This table stores **only staging inventory relevant to the workflow**.

NEA uses **binary exclusion logic**.

---

## 6.5 `corp_active_sell_order`

Represents active corporation sell orders.

### Primary Key

```
(corporation_id, order_id)
```

### Core Columns

| Column | Description |
|---|---|
| corporation_id | Corporation identifier |
| order_id | Order identifier |
| type_id | FK → Reference Domain `item_type.type_id` |
| location_id | FK → corp_location.location_id |
| volume_total | Original order quantity |
| volume_remaining | Remaining quantity |
| price | Order price |
| issued_at | Order creation time |
| is_active | Indicates active sell order |

### Notes

Only **active sell orders** are stored.

Buy orders are excluded from MVP scope.

---

# 7. Relationships

Key relationships within the domain:

```
corp_blueprint_instance.location_id → corp_location.location_id
corp_industry_job.location_id → corp_location.location_id
corp_staging_inventory.location_id → corp_location.location_id
corp_active_sell_order.location_id → corp_location.location_id
```

Cross-domain relationships:

```
corp_blueprint_instance.blueprint_type_id → Reference Domain blueprint.blueprint_type_id
corp_industry_job.product_type_id → Reference Domain item_type.type_id
corp_staging_inventory.type_id → Reference Domain item_type.type_id
corp_active_sell_order.type_id → Reference Domain item_type.type_id
```

---

# 8. Domain Rules

## Rule 1 — Current State Only

Only current corporation operational state is stored.

Historical records are out of scope for MVP.

## Rule 2 — Corporation Scoped Records

All rows include `corporation_id`.

## Rule 3 — Blueprint Instances

Blueprint definitions are stored in the Reference Domain.

This domain stores only **owned blueprint instances**.

## Rule 4 — Factual Locations Only

Location records represent factual storage locations.

Workflow meaning is defined by the Configuration Domain.

## Rule 5 — Binary Exclusion Logic

Binary exclusion logic means:

An item is excluded if it appears in any of the following states:

- active production job
- staging inventory
- active sell order

Quantities do not alter exclusion behavior.

## Rule 6 — Buy Orders Excluded

Buy orders are not tracked in the MVP schema.

---

# 9. Expected Usage in NEA v1

The Corporation Production State Domain supports the following operations.

## Usage Examples

### Blueprint Eligibility

Determine whether a blueprint instance exists for a producible item.

### Production Exclusion

Exclude items currently in production.

### Pipeline Exclusion

Exclude items already staged.

### Market Exclusion

Exclude items already listed for sale.

---

# 10. Out-of-Scope Extensions

The following features are excluded from the MVP schema:

- historical state tracking
- buy order tracking
- logistics tracking
- production slot modeling
- staging quantity thresholds
- market analytics beyond exclusion logic

---

# 11. Minimal MVP Table Summary

| Table | Purpose |
|---|---|
| corp_location | Corporation locations and containers |
| corp_blueprint_instance | Owned blueprint instances |
| corp_industry_job | Active industry jobs |
| corp_staging_inventory | Filtered staging inventory |
| corp_active_sell_order | Active sell orders |

These tables represent the **minimum Corporation Production State Domain required to support NEA v1 recommendations**.
