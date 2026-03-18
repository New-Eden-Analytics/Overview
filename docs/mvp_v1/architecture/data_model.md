# Data Model Diagram - NEA MVP v1

This document provides visual representations of the database schema and table relationships.

**Related Documentation:**

- [Reference Domain Schema](../schema/reference_domain.md) - Detailed table specifications
- [Corporation Domain Schema](../schema/corporation_domain.md) - Corp tables
- [Market Domain Schema](../schema/market_domain.md) - Market tables
- [Configuration Domain Schema](../schema/configuration_domain.md) - Config tables
- [Operational Telemetry Domain Schema](../schema/operational_telemetry_domain.md) - Telemetry tables
- [MVP v1 Specification](../specification.md) - Context for the data model

---

## Complete Schema Overview

```
                     NEA MVP v1 Database
                     ===================

  ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
  │  Reference   │      │ Corporation  │      │   Market     │
  │   Domain     │      │ Production   │      │ Observation  │
  │              │      │    State     │      │   Domain     │
  │ • item_type  │─────▶│              │      │              │
  │ • blueprint  │      │ • corp_loc   │      │ • market_    │
  │ • bp_product │      │ • corp_bp    │      │   price_     │
  │ • bp_        │      │ • corp_job   │      │   snapshot   │
  │   material   │      │ • corp_stg   │      │              │
  └──────────────┘      │ • corp_order │      └──────────────┘
                        └──────────────┘

  ┌──────────────┐                           ┌──────────────┐
  │ Operational  │                           │     Config   │
  │  Telemetry   │                           │    Domain    │
  │   Domain     │                           │              │
  │              │                           │ • nea_config │
  │ • source_    │                           └──────────────┘
  │   instance   │
  │ • source_    │
  │   sync_status│
  │ • source_    │
  │   refresh_run│
  │ • api_req_log│
  │ • file_imp_  │
  │   log        │
  └──────────────┘
```

---

## Reference Domain - Detailed

```
                    Reference Domain
                (Static Game Data from SDE)
                ============================

                  ┌────────────────┐
                  │   item_type    │
                  ├────────────────┤
                  │ PK type_id     │
                  │    type_name   │
                  │    description │
                  │    volume      │
                  │    published   │
                  └────────────────┘
                     ▲          ▲
                     │          │
       ┌─────────────┘          └────────────┐
       │                                     │
  ┌────────────────┐                  ┌────────────────┐
  │   blueprint    │                  │  blueprint_    │
  ├────────────────┤                  │   material     │
  │ PK blueprint_id│─────────────────▶├────────────────┤
  │    activity_id │                  │ FK blueprint_id│
  │    time        │                  │    activity_id │
  │    max_limit   │                  │ FK material_id │──┐
  └────────────────┘                  │    quantity    │  │
       │                              └────────────────┘  │
       │                                                  │
       ▼                                                  │
  ┌────────────────┐                                     │
  │  blueprint_    │                                     │
  │   product      │                                     │
  ├────────────────┤                                     │
  │ FK blueprint_id│                                     │
  │    activity_id │                                     │
  │ FK product_id  │─────────────────────────────────────┘
  │    quantity    │
  └────────────────┘

Key Relationships:
  • blueprint.blueprint_type_id → item_type.type_id
  • blueprint_product.blueprint_id → blueprint.blueprint_id
  • blueprint_product.product_type_id → item_type.type_id
  • blueprint_material.blueprint_id → blueprint.blueprint_id
  • blueprint_material.material_type_id → item_type.type_id
```

---

## Corporation Domain - Detailed

```
                  Corporation Domain
              (Current Corp State from ESI)
              ==============================

              ┌─────────────────┐
              │  corp_location  │
              ├─────────────────┤
              │ PK corp_id      │
              │ PK location_id  │
              │    name         │
              │    type         │
              │    station_id   │
              └─────────────────┘
                       ▲
                       │ Referenced by all corp tables
        ┌──────────────┼──────────────┬──────────────┐
        │              │              │              │
  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐
  │ corp_bp │   │ corp_   │   │ corp_   │   │ corp_   │
  │ instance│   │ industry│   │ staging │   │ active_ │
  ├─────────┤   │ _job    │   │ _invntry│   │ sell_ord│
  │ PK corp │   ├─────────┤   ├─────────┤   ├─────────┤
  │ PK bp_id│   │ PK corp │   │ PK corp │   │ PK corp │
  │ FK bp_  │   │ PK job  │   │ PK loc  │   │ PK order│
  │    type │   │ FK bp_ty│   │ PK type │   │ FK type │
  │ FK loc  │   │ FK prod │   │    qty  │   │ FK loc  │
  │    ME   │   │ FK loc  │   └─────────┘   │    vol  │
  │    TE   │   │    stat │        │        │    price│
  └─────────┘   └─────────┘        │        └─────────┘
       │             │              │            │
       └─────────────┴──────────────┴────────────┘
                     │
                     ▼
            ┌─────────────┐
            │  Reference  │
            │   Domain    │
            │ (item_type, │
            │  blueprint) │
            └─────────────┘

Key Relationships:
  • All corp tables scoped by corporation_id
  • corp_blueprint_instance.blueprint_type_id → blueprint.blueprint_id
  • corp_industry_job.product_type_id → item_type.type_id
  • corp_staging_inventory.type_id → item_type.type_id
  • corp_active_sell_order.type_id → item_type.type_id
  • All location_id fields → corp_location.location_id
```

---

## Market Domain

```
                  Market Domain
        (Latest Market Pricing from ESI)
        =================================

        ┌───────────────────────────┐
        │  market_price_snapshot    │
        ├───────────────────────────┤
        │ PK region_id              │
        │ PK type_id                │───────▶ Reference Domain
        │    average_price          │         (item_type)
        │    volume                 │
        │    order_count            │
        │    last_updated           │
        └───────────────────────────┘

Simple domain with one table storing latest pricing data
for items in specific regions.
```

---

## Configuration Domain

```
                Configuration Domain
            (Runtime NEA Configuration)
            ============================

        ┌──────────────────────────┐
        │      nea_config          │
        ├──────────────────────────┤
        │ PK config_id             │
        │ FK active_corp_id        │─────▶ Corp Domain
        │ FK active_region_id      │─────▶ Market Domain
        │ FK bp_source_loc_id      │─────▶ corp_location
        │ FK staging_loc_id        │─────▶ corp_location
        │ FK production_loc_id     │─────▶ corp_location
        │ FK sell_loc_id           │─────▶ corp_location
        │    flat_market_cost_rate │
        │    default_top_n         │
        └──────────────────────────┘

Single configuration table storing operational parameters.
MVP expects only one row (single configuration).
```

---

## Operational Telemetry Domain

```
            Operational Telemetry Domain
        (Data Refresh and Observability)
        =================================

        ┌──────────────────┐
        │ source_instance  │
        ├──────────────────┤
        │ PK source_name   │
        │    source_type   │
        │    description   │
        └──────────────────┘
                 ▲
                 │ Referenced by all telemetry tables
     ┌───────────┼───────────┬───────────┐
     │           │           │           │
┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
│ source_ │ │ source_ │ │ api_    │ │ file_   │
│ sync_   │ │ refresh │ │ request │ │ import_ │
│ status  │ │ _run    │ │ _log    │ │ log     │
├─────────┤ ├─────────┤ ├─────────┤ ├─────────┤
│ PK src  │ │ PK runid│ │ PK reqid│ │ PK impid│
│    last_│ │ FK src  │ │ FK src  │ │ FK src  │
│   success│ │    start│ │    endpt│ │    file │
│    last_│ │    end  │ │    stat │ │    size │
│   error │ │    stat │ │    time │ │    rows │
└─────────┘ └─────────┘ └─────────┘ └─────────┘

Key Relationships:
  • All telemetry tables reference source_instance.source_name
  • source_sync_status: Latest refresh status per source
  • source_refresh_run: History of all refresh attempts
  • api_request_log: ESI API call details
  • file_import_log: SDE file import details
```

---

## Cross-Domain Analysis Query Flow

```
Query: "What should I manufacture next?"
========================================

Step 1: Get Configuration
    nea_config → active_corporation_id, active_region_id, locations

Step 2: Find Manufacturable Items (Reference Domain)
    item_type ←→ blueprint ←→ blueprint_product
                     ↓
                 blueprint_material

Step 3: Calculate Material Costs (Reference + Market)
    blueprint_material.quantity × market_price_snapshot.avg_price
    = total_material_cost

Step 4: Get Sale Price (Market)
    market_price_snapshot.average_price = sale_price

Step 5: Calculate Profit
    profit = sale_price - material_cost

Step 6: Apply Exclusions (Corporation State)
    EXCLUDE items in:
    - corp_industry_job (active manufacturing)
    - corp_staging_inventory (already produced)
    - corp_active_sell_order (already for sale)

Step 7: Filter by Blueprint Availability
    INCLUDE ONLY items with blueprints in:
    - corp_blueprint_instance

Step 8: Rank and Return
    ORDER BY profit DESC
    LIMIT default_top_n
```

---

## Primary Keys Summary

| Table | Primary Key |
|-------|-------------|
| **Reference Domain** |
| item_type | type_id |
| blueprint | blueprint_type_id, activity_id |
| blueprint_product | blueprint_type_id, activity_id |
| blueprint_material | blueprint_type_id, activity_id, material_type_id |
| **Corporation Domain** |
| corp_location | corporation_id, location_id |
| corp_blueprint_instance | corporation_id, blueprint_instance_id |
| corp_industry_job | corporation_id, job_id |
| corp_staging_inventory | corporation_id, location_id, type_id |
| corp_active_sell_order | corporation_id, order_id |
| **Market Domain** |
| market_price_snapshot | region_id, type_id |
| **Configuration Domain** |
| nea_config | config_id |
| **Operational Telemetry Domain** |
| source_instance | source_name |
| source_sync_status | source_name |
| source_refresh_run | run_id |
| api_request_log | request_id |
| file_import_log | import_id |

---

## Recommended Indexes

```sql
-- Reference Domain
CREATE INDEX idx_item_published ON item_type(published);
CREATE INDEX idx_blueprint_type ON blueprint(blueprint_type_id);
CREATE INDEX idx_bp_material_type ON blueprint_material(material_type_id);

-- Corporation Domain
CREATE INDEX idx_corp_bp_active ON corp_blueprint_instance(is_active);
CREATE INDEX idx_corp_job_active ON corp_industry_job(is_active, product_type_id);
CREATE INDEX idx_corp_staging_loc ON corp_staging_inventory(location_id);
CREATE INDEX idx_corp_order_active ON corp_active_sell_order(is_active, type_id);

-- Market Domain
CREATE INDEX idx_market_region ON market_price_snapshot(region_id, type_id);
CREATE INDEX idx_market_updated ON market_price_snapshot(last_updated);

-- Operational Telemetry
CREATE INDEX idx_refresh_time ON source_refresh_run(source_name, start_time DESC);
CREATE INDEX idx_api_created ON api_request_log(created_at DESC);
```

---

## Data Volume Estimates

| Table | Expected Rows (MVP) | Estimated Size |
|-------|---------------------|----------------|
| item_type | ~10,000 | ~5 MB |
| blueprint | ~500 | ~100 KB |
| blueprint_product | ~500 | ~50 KB |
| blueprint_material | ~5,000 | ~500 KB |
| corp_location | ~50 | ~10 KB |
| corp_blueprint_instance | ~100 | ~20 KB |
| corp_industry_job | ~10 | ~5 KB |
| corp_staging_inventory | ~100 | ~10 KB |
| corp_active_sell_order | ~20 | ~5 KB |
| market_price_snapshot | ~500 | ~50 KB |
| nea_config | 1 | <1 KB |
| source_instance | 3 | <1 KB |
| source_sync_status | 3 | <1 KB |
| source_refresh_run | ~1,000 | ~500 KB |
| api_request_log | ~10,000 | ~5 MB |
| file_import_log | ~10 | ~5 KB |
| **Total** | **~27,697** | **~12 MB** |

---

## Next Steps

When implementing schema:

1. Start with Reference Domain (M1/M2)
2. Add Corporation Domain (M3)
3. Add Market Domain (M4)
4. Add Operational Telemetry Domain (M5)
5. Configuration Domain throughout (needed from M1)

Use these diagrams as reference when writing DDL and queries.
