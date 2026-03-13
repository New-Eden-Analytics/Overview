# New Eden Analytics (NEA) - Minimum Viable Product (MVP) Specification

---

# 1. Purpose

The purpose of the NEA MVP is to deliver a lightweight analytical system that helps a solo industrialist identify the most profitable items to manufacture next.

The MVP focuses on a **single core question**:

> Given my blueprints and current market conditions, what items should I produce next?

The system produces a ranked list of candidate items based on estimated profitability while excluding items already in the production or sale pipeline.

The MVP prioritizes:

- operational simplicity
- reliable daily usage
- minimal user interaction
- fast result generation

The system should allow the user to generate a recommended production list in **minutes with a single request**.

---

# 2. Scope

## Included in MVP

The MVP includes the following capabilities:

- ingestion of static game data (blueprints, item types, manufacturing relationships)
- ingestion of corporation data needed to determine:
  - owned blueprints
  - active industry jobs
  - active sell orders
  - relevant inventory locations
- ingestion of market history data
- calculation of production cost based on blueprint material requirements and material efficiency
- estimation of expected sale price using recent market history
- filtering of items already in the production-to-sale pipeline
- ranking of candidate items by profitability
- generation of a ranked list of manufacturing opportunities

---

## Explicitly Excluded from MVP

The following features are intentionally excluded:

- multi-region analysis
- forecasting or predictive modeling
- automated production scheduling
- web UI or interactive dashboards
- advanced charting
- recommendation systems beyond profitability ranking
- multi-user or shared system features
- automated retraining or model pipelines
- personalized portfolio management
- structure-specific production modeling
- complex production chains or recursive manufacturing
- advanced fee/tax optimization
- API access for external consumers

The MVP is intended to support **a single user operating in a single region**.

---

# 3. Core System Behavior

## User Interaction Model

The system operates as a **pull-based analysis tool**.

The workflow:

1. The user initiates an analysis run.
2. NEA gathers the most recent available data.
3. The system computes candidate manufacturing opportunities.
4. A ranked list of items is returned.

The result is delivered as:

- a notebook output
- or a simple structured table

No UI is required for the MVP.

---

## Daily Usage Target

The system should support the following daily workflow:

1. User initiates analysis.
2. NEA generates ranked manufacturing opportunities.
3. User selects items to manufacture.
4. Production runs are configured in-game.

Total decision time should be **under 30 minutes**.

---

# 4. Unit of Analysis

The fundamental unit of analysis in NEA is the **item to be sold**.

Each candidate item must satisfy:

- the user possesses at least one blueprint capable of producing it
- the item is manufacturable in a **single production step**
- the item is not currently in the production-to-sale pipeline

Blueprints may be:

- Blueprint Originals (BPO)
- Blueprint Copies (BPC)

For MVP purposes:

- BPC run counts are ignored
- if at least one blueprint exists, the item is considered manufacturable

---

# 5. Pipeline Exclusion Rules

An item must be excluded from the candidate list if **any of the following conditions are true**:

1. An active industry job is producing the item.
2. The item exists in the production staging output location.
3. The user currently has an active sell order for the item.

If an item appears at **any stage of the production-to-sale pipeline**, it is excluded from recommendations.

These rules ensure the recommendation list contains only **new production opportunities**.

---

# 6. Profitability Model

The MVP profitability calculation is intentionally simple.

Estimated profit is calculated as:

```
Estimated Profit = Estimated Sale Price − Estimated Production Cost
```

Where:

## Price Components

### Estimated Sale Price

- derived from the **most recent average market price**
- pulled from the local market history snapshot

### Estimated Production Cost

Calculated using:

- blueprint material requirements
- blueprint material efficiency
- current material prices

Additional costs such as:

- industry job fees
- market taxes
- broker fees

are modeled using a **flat percentage estimate**.

These cost models can be refined in future versions.

---

# 7. Ranking Output

The system produces a **ranked list of candidate items**.

The ranking metric for MVP is:

- **estimated profit per unit**

Future versions may introduce:

- profit per production time
- demand-adjusted profitability
- forecast-adjusted profitability

The default output should return:

```
Top 5 candidate items
```

The user must be able to increase the value of **N** when desired.

---

# 8. Data Domains

The MVP system is built from five primary data domains.

---

## Reference Domain

Schema: [Reference Domain Schema](schema/reference_domain.md)

Stores static game data needed for production calculations.

Includes:

- item types
- blueprints
- blueprint material requirements
- blueprint product relationships

This domain represents **persistent static game data**.

---

## Corporation Production State Domain

Schema: [Corporation Production State Domain Schema](schema/corporation_domain.md)

Stores corporation state required to determine the current production pipeline.

Includes:

- corporation assets
- corporation blueprints
- corporation industry jobs
- corporation sell orders
- relevant inventory locations

This domain determines which items are **already in production or sale**.

---

## Market Domain

Schema: [Market Domain Schema](schema/market_domain.md)

Stores market data used to estimate prices.

Includes:

- market history snapshots
- average prices
- traded volumes

For MVP, only the **most recent resolved market data snapshot** is required.

---

## Configuration Domain

Schema: [Configuration Domain Schema](schema/configuration_domain.md)

Stores system configuration and operational parameters.

Includes:

- production station
- sales station
- inventory locations
- staging containers
- corporation identifiers

Configuration defines how NEA interprets corporation data.

---

## Operational Telemetry Domain

Schema: [Operational Telemetry Domain Specification](schema/operational_telemetry_domain.md)

Records the execution and operational state of all data ingestion processes.

Includes:

- source instance definitions
- refresh run history
- API request logs
- file import logs

This domain enables observability and debugging of all data acquisition activity.

---

# 9. Data Freshness Model

NEA maintains data freshness through a combination of:

- scheduled refresh operations
- API refresh cadence rules
- source-driven refresh processes

Data refresh operations are tracked using the Operational Telemetry Domain.

Each refresh operation records:

- execution timing
- status
- operational metadata
- request-level activity

This ensures that all data inputs used by the analysis pipeline are auditable and observable.

---

# 10. Success Criteria

The MVP is considered successful if it meets the following criteria:

1. The user can generate a ranked production list daily.
2. The list identifies items that are not currently in production or sale.
3. The system produces the result in minutes.
4. The output allows the user to make manufacturing decisions quickly.
5. The system operates reliably with minimal maintenance.

The ultimate success condition is:

> NEA consistently helps the user decide what to manufacture next with minimal effort.
