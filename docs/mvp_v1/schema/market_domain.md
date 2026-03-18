# NEA MVP Schema — Market Domain

Schema Version: NEA MVP v1

**Related Documentation:**
- [MVP v1 Specification](../specification.md) - Overall requirements
- [Data Model Diagram](../architecture/data_model.md) - Visual schema overview
- [ESI Endpoints](../implementation/esi_endpoints.md) - Market history API endpoint (M4)
- [MVP v1 Roadmap](../roadmap.md) - Implementation timeline (M4)

---

# 1. Purpose

The Market Domain contains the pricing data required for NEA v1 to estimate production profitability.

This domain stores the **latest pricing snapshot** used by the NEA recommendation engine.

It allows NEA to determine:

- estimated sale price of manufactured items
- estimated purchase cost of input materials
- the freshness of market pricing data used in calculations

The Market Domain does **not** store historical market time series data.  
Only the **latest pricing snapshot** is retained.

---

# 2. Scope

## Included

The Market Domain includes:

- resolved pricing data for items
- market volume information
- supporting statistical values (highest price, lowest price, order count)
- timestamps describing when the pricing snapshot was observed

This data supports the profitability calculations used by the recommendation engine.

## Excluded

The Market Domain does **not** include:

- historical price series
- forecasting models
- market order book snapshots
- station-level price modeling
- recommendation results

These concerns belong to other domains or future NEA versions.

---

# 3. Design Goals

The Market Domain schema must satisfy the following goals:

1. Provide a canonical latest pricing record for each item within a region.
2. Provide pricing data usable for both product sale price and material input cost.
3. Provide timestamps allowing the system to detect stale market data.
4. Keep the schema simple and lightweight for MVP implementation.
5. Decouple pricing records from the exact ingestion source format.

This domain represents **NEA’s canonical pricing snapshot**, not a mirror of external APIs.

---

# 4. Canonical Entities

The Market Domain consists of a single table:

1. `market_price_snapshot`

---

# 5. Table Definitions

## 5.1 `market_price_snapshot`

Defines the latest pricing snapshot used by NEA.

Each row represents the most recent known pricing information for an item within a region.

### Primary Key

```
(region_id, type_id)
```

### Core Columns

| Column | Description |
|---|---|
| region_id | Region identifier |
| type_id | FK → Reference Domain `item_type.type_id` |
| average_price | Average market price |
| highest_price | Highest observed price |
| lowest_price | Lowest observed price |
| volume | Market trading volume |
| order_count | Number of observed orders |
| observed_at | Timestamp when pricing data was last observed |

### Notes

This table stores the **current pricing snapshot** used by NEA.

When new market data is ingested, existing rows are updated rather than storing historical records.

---

# 6. Relationships

The Market Domain connects to the Reference Domain:

```
market_price_snapshot.type_id → Reference Domain item_type.type_id
```

This allows pricing data to be joined with:

- blueprint products
- blueprint material inputs

---

# 7. Domain Rules

## Rule 1 — Latest Snapshot Only

The domain stores only the **latest pricing snapshot**.

Historical market data is not retained in the MVP schema.

## Rule 2 — Region-Scoped Records

Pricing records are scoped by:

```
(region_id, type_id)
```

Even though NEA v1 uses a single region, region identifiers are retained for schema correctness and future expansion.

## Rule 3 — Unified Pricing Source

Product pricing and material pricing must originate from the same pricing snapshot.

## Rule 4 — Timestamp-Based Freshness

Pricing records must include an `observed_at` timestamp.

Although NEA v1 does not enforce freshness thresholds, timestamps allow stale data to be detected.

## Rule 5 — Station-Level Pricing Excluded

NEA v1 uses region-level pricing only.

Station-level price modeling is intentionally excluded.

---

# 8. Expected Usage in NEA v1

The Market Domain supports the following operations.

## Usage Examples

### Estimate Output Sale Price

By joining produced item IDs with `market_price_snapshot`, NEA determines the estimated sale price used in profitability calculations.

### Estimate Material Input Cost

By joining material IDs with `market_price_snapshot`, NEA determines the cost of required inputs.

### Validate Data Freshness

The `observed_at` timestamp allows NEA to determine whether pricing data may be outdated.

---

# 9. Out-of-Scope Extensions

The following features are intentionally excluded from the MVP Market Domain:

- historical price storage
- price forecasting
- liquidity-adjusted pricing
- station-level pricing
- order book modeling
- real-time market ingestion

These features may be introduced in future NEA versions.

---

# 10. Minimal MVP Table Summary

| Table | Purpose |
|---|---|
| market_price_snapshot | Canonical latest pricing snapshot used for profitability calculations |

This table represents the **minimum Market Domain required for NEA v1**.
