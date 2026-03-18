# NEA MVP Schema — Configuration Domain

Schema Version: NEA MVP v1

**Related Documentation:**

- [MVP v1 Specification](../specification.md) - Overall requirements
- [Data Model Diagram](../architecture/data_model.md) - Visual schema overview
- [MVP v1 Roadmap](../roadmap.md) - Used throughout all milestones

---

# 1. Purpose

The Configuration Domain stores the runtime configuration that defines how NEA interprets data from the other domains.

This domain specifies:

- which corporation NEA operates against
- which region NEA uses for market data
- which locations represent blueprint storage, staging, production, and selling workflows
- economic assumptions used for profitability calculations
- default behavior parameters for recommendation output

Unlike the other domains, the Configuration Domain contains **user-defined operational settings**, not factual game data.

---

# 2. Scope

## Included

The Configuration Domain includes:

- active corporation selection
- active region selection
- mapping of workflow roles to corporation locations
- economic assumptions used by profitability calculations
- default recommendation output parameters

This domain defines **the operational context in which NEA interprets other domains**.

## Excluded

The Configuration Domain does **not** include:

- blueprint definitions
- corporation production state
- market pricing
- recommendation outputs
- API telemetry or ingestion metadata

These concerns belong to other domains.

---

# 3. Design Goals

The Configuration Domain schema must satisfy the following goals:

1. Provide a clear operational context for NEA execution.
2. Map workflow roles to corporation locations without storing location data itself.
3. Allow economic assumptions to be adjusted without altering recommendation logic.
4. Keep configuration simple for MVP implementation.
5. Avoid implicit assumptions about configuration state.

Configuration records are therefore identified explicitly by a primary key.

---

# 4. Canonical Entities

The Configuration Domain consists of a single table:

1. `nea_config`

Each row represents a complete configuration profile.

NEA v1 is expected to operate with **a single configuration row**, though the schema allows future expansion.

---

# 5. Table Definitions

## 5.1 `nea_config`

Defines the runtime configuration used by NEA when evaluating production opportunities.

### Primary Key

```
config_id
```

### Core Columns

| Column | Description |
|---|---|
| config_id | Unique configuration identifier |
| active_corporation_id | Corporation identifier used for corp-state queries |
| active_region_id | Region identifier used for market pricing |
| blueprint_source_location_id | Location containing blueprint instances used for production |
| staging_location_id | Location representing the staging pipeline for completed items |
| production_location_id | Location where industry jobs are installed |
| sell_location_id | Location used to check active sell orders |
| flat_market_cost_rate | Flat percentage used to approximate taxes and fees |
| default_top_n | Default number of recommendations returned |

### Notes

Locations referenced here correspond to factual location records stored in the **Corporation Domain**.

These locations may represent:

- stations
- structures
- hangars
- containers

Configuration assigns **workflow roles** to these locations.

---

# 6. Relationships

The Configuration Domain references identifiers from other domains.

```
nea_config.active_corporation_id → Corporation Domain

nea_config.active_region_id → Market Domain

nea_config.blueprint_source_location_id → corp_location.location_id
nea_config.staging_location_id → corp_location.location_id
nea_config.production_location_id → corp_location.location_id
nea_config.sell_location_id → corp_location.location_id
```

Configuration does **not own these records**.  
It only identifies which records are relevant for the active workflow.

---

# 7. Domain Rules

## Rule 1 — Explicit Configuration Identity

Each configuration record must have a unique `config_id`.

Although NEA v1 uses a single configuration row, explicit identifiers prevent ambiguous assumptions about table state.

## Rule 2 — Workflow Roles Reference Locations

Location fields reference factual locations stored in the Corporation Domain.

Configuration assigns **semantic roles** to those locations.

## Rule 3 — Production and Selling Locations Are Independent

Production and selling may occur in different locations.

The same location may be used for both roles if desired.

## Rule 4 — Single Location Per Role

NEA v1 supports exactly **one location per workflow role**.

Future versions may extend this to support multiple locations.

## Rule 5 — Configuration Contains Operational Assumptions

Values such as `flat_market_cost_rate` represent operational assumptions used during profitability calculations.

---

# 8. Expected Usage in NEA v1

The Configuration Domain supports the following operations.

## Usage Examples

### Determine Operating Context

Identify which corporation and region NEA should operate against.

### Identify Blueprint Sources

Determine which location contains blueprint instances used for manufacturing.

### Identify Production Pipeline

Identify the staging location representing the production-to-sale pipeline.

### Determine Market Exclusion Location

Identify the location used to check active sell orders.

### Apply Economic Assumptions

Apply the configured `flat_market_cost_rate` to profitability calculations.

---

# 9. Out-of-Scope Extensions

The following features are intentionally excluded from the MVP Configuration Domain:

- multiple configuration profiles
- multiple locations per workflow role
- item-level overrides
- recommendation rule customization

These may be introduced in future NEA versions.

---

# 10. Minimal MVP Table Summary

| Table | Purpose |
|---|---|
| nea_config | Runtime configuration defining NEA operational context and workflow mappings |

This table represents the **minimum Configuration Domain required to support NEA v1 operation**.
