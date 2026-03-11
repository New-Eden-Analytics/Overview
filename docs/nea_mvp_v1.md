# New Eden Analytics (NEA) — MVP Specification (v1)

## 1. Purpose

New Eden Analytics (NEA) v1 is a **single-user production recommendation engine** designed to assist an industrialist in identifying the most profitable items to manufacture next.

The system analyzes the user’s available blueprints, current market pricing, and production state to produce a **ranked list of candidate items** that are profitable to manufacture and are **not already in the production-to-sale pipeline**.

The primary goal of NEA v1 is to **reduce the time required to identify viable production opportunities**, enabling daily decision-making with minimal manual analysis.

### Success Criteria

- The system can be run on demand.
- It produces a ranked list of candidate items within seconds.
- The output is reliable enough to guide daily production decisions.

---

# 2. Scope

## Included in MVP

NEA v1 provides:

- Identification of **manufacturable items based on owned blueprints**
- Profitability calculation using **current market prices**
- Filtering of items already in production or already listed for sale
- Ranking of candidate items by **estimated unit profit**
- On-demand execution through a **script or notebook**

Output format is a **ranked table of recommended items**.

## Explicitly Excluded From MVP

The following features are intentionally deferred to later versions:

- Price forecasting
- Demand/liquidity modeling
- Recursive production-chain optimization
- Detailed tax/fee modeling
- Web UI
- Stored recommendation history
- Scheduling or automation services
- API interfaces
- Multi-user support
- Multi-region analysis
- Detailed explanation layers
- Advanced charting or reporting

---

# 3. Target User

NEA v1 is designed for:

**A single industrialist operating within one corporation and one production location.**

The user owns blueprints and manufactures items for sale on the regional market.

The system is intended to assist the user in answering:

> “What should I manufacture next that I am not already producing or selling?”

---

# 4. System Overview

NEA v1 operates as an **on-demand analysis engine**.

When executed, the system:

1. Determines all items that can be produced from blueprints currently available to the user.
2. Removes items already in the production-to-sale pipeline.
3. Estimates profitability using current market pricing.
4. Returns a ranked list of candidate items.

The system is **pull-based**: analysis runs only when requested by the user.

---

# 5. Core Recommendation Logic

An item is considered **eligible** for recommendation if all of the following conditions are met.

## Blueprint Availability

- A blueprint capable of producing the item exists in the specified corporation hangar/container.
- Blueprint may be either a **BPO or BPC**.
- Blueprint run counts are ignored in v1.

## Production Model

- The item must be producible via a **single manufacturing step**.
- Intermediate manufacturing chains are not considered.

## Production State Exclusion

Items are excluded if:

- A **non-completed production job** exists producing the item.
- The item already exists in the **staging output container**.
- A **sell order for the item exists** at the specified station.

These exclusions are **binary**.

If any of these conditions are true, the item is removed from consideration.

---

# 6. Profit Estimation Model

NEA v1 estimates profitability using the following formula.

## Estimated Unit Profit

```
estimated_profit =
    estimated_sale_price
    - adjusted_material_cost
    - flat_fee_percentage
```

## Estimated Sale Price

Estimated sale price is defined as:

```
most recent average market price
```

derived from regional market history data.

## Material Cost

Material cost is calculated using:

- Blueprint material requirements
- Blueprint-specific **Material Efficiency (ME)**
- Most recent average price for each input material

## Fees and Overhead

Transaction and operational costs are approximated as:

```
flat percentage of sale price
```

This simplification is intentional for MVP.

Precise modeling of taxes, broker fees, and structure bonuses is deferred to future versions.

---

# 7. Ranking Metric

Items are ranked by:

```
estimated unit profit
```

Tie-breaking rules are intentionally ignored in MVP.

If two items have identical values, ordering between them is not significant.

---

# 8. Output

The system returns a **ranked table of candidate items**.

### Default Output Size

```
Top 5 items
```

The user may optionally increase this number.

### Output Fields

| Field | Description |
|------|-------------|
| item_name | Name of the producible item |
| estimated_sale_price | Current estimated market sale value |
| estimated_material_cost | Estimated cost of materials |
| estimated_unit_profit | Expected profit per produced unit |
| profit_margin_percent (optional) | Profit margin relative to sale price |

No explanation or breakdown layers are included in MVP.

---

# 9. Data Domains

NEA v1 defines four canonical MariaDB data domains.

## 9.1 Reference Domain

Static game data required to determine production relationships.

Includes:

- item definitions
- blueprint definitions
- blueprint output items
- blueprint material requirements
- industry activity metadata

This data changes infrequently.

---

## 9.2 Market Observation Domain

Market data used to estimate pricing.

Includes:

- regional market history
- most recent average price values

The system maintains a **rolling local snapshot** of market history.

Only recent data is retained.

---

## 9.3 Corporation Production State Domain

Corporation-specific operational state.

Includes:

- blueprint inventory
- blueprint material efficiency
- active production jobs
- staging output inventory
- active market sell orders

This domain is used to determine which items are **already in the production pipeline**.

---

## 9.4 Configuration Domain

System configuration parameters.

Includes:

- selected corporation
- selected station
- selected hangar/container
- selected market region
- flat fee percentage
- default recommendation list size

This domain contains only configuration data.

---

# 10. Execution Workflow

When NEA is executed:

1. Load system configuration.
2. Retrieve blueprint-backed producible items.
3. Filter to single-step manufacturable items.
4. Remove items currently in production.
5. Remove items present in staging inventory.
6. Remove items with active sell orders.
7. Retrieve required blueprint materials.
8. Retrieve most recent average market prices for outputs and inputs.
9. Compute estimated unit profit.
10. Rank items by profitability.
11. Return top N results.

---

# 11. Performance Expectations

NEA v1 is designed to run interactively.

Expected runtime target:

```
< 10 seconds
```

Execution occurs locally via script or notebook.

No persistent recommendation storage is required.

---

# 12. Success Criteria

NEA v1 is considered successful if:

- The user can run the analysis daily.
- The output identifies plausible production opportunities.
- The system reduces manual opportunity analysis time to **minutes per day**.

---

# 13. Future Extensions (Out of Scope)

Potential future improvements include:

- price forecasting models
- demand/liquidity adjustments
- recursive production-chain analysis
- accurate tax and fee modeling
- web UI
- persistent recommendation history
- automated retraining and scheduling
- multi-region analysis
- multi-user support

# 14. Assumptions

The following assumptions intentionally simplify the NEA v1 system. These constraints define the operational boundaries of the MVP and prevent scope expansion during development.

### Pricing Model

- Market pricing is derived from the **most recent average price** available in regional market history.
- No smoothing, forecasting, or trend analysis is applied.
- Input materials and output products use the **same pricing rule**.

### Demand and Liquidity

- Market liquidity is **not modeled**.
- Items with low trading volume are not penalized or filtered.
- Profitability calculations assume items can be sold at the estimated price.

### Production Modeling

- Only **single-step manufacturing** processes are supported.
- Intermediate production chains are not evaluated.
- All required materials are assumed to be **purchased from the market**.

### Blueprint Handling

- Blueprints may be either **BPOs or BPCs**.
- Blueprint copy run counts are ignored.
- Blueprint **Material Efficiency (ME)** is applied when calculating material requirements.

### Operational Costs

- Transaction costs, broker fees, taxes, and other overhead are approximated using a **flat percentage of sale price**.
- Exact fee modeling is intentionally deferred to later versions.

### Output Scope

- Recommendations are ranked only by **estimated unit profit**.
- Tie-breaking logic between equal-profit items is not implemented.

---

# 15. Failure Conditions

Certain conditions may prevent an item from being evaluated by the recommendation engine.

Items will be excluded from consideration if any of the following conditions occur:

- Market history data is missing for the **output item**.
- Market history data is missing for **any required input material**.
- Blueprint material definitions are incomplete.
- Blueprint Material Efficiency (ME) is not known.
- The item cannot be verified as a **single-step manufacturing output**.

In these cases, the item is skipped and the system continues evaluating other candidates.

Failure to evaluate individual items does **not cause the entire recommendation process to fail**.

---

# 16. Data Freshness Rules

NEA v1 relies on locally stored market history data.

To ensure reliable output, the following data freshness expectations apply:

- Market history data should be **no older than 24 hours**.
- If market data is missing or outdated for specific items, those items are skipped.
- The recommendation engine must tolerate partial market datasets.

The system is expected to operate even when some market data is unavailable.

Future versions may introduce more sophisticated data refresh policies or background synchronization.