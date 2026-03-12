# NEA MVP Schema — Reference Domain

## 1. Purpose

The Reference domain contains the static game data required for NEA v1 to determine:

- what items exist
- what blueprints exist
- what item each blueprint produces
- what materials are required to manufacture that item
- whether an item is eligible for single-step manufacturing analysis

This domain serves as the **canonical source of production definitions** used by the NEA recommendation engine.

The Reference domain does **not** contain market data, corporation-specific blueprint ownership, active jobs, sell orders, or any other operational state.

---

# 2. Scope

## Included

The Reference domain includes:

- item/type definitions relevant to manufacturing
- blueprint definitions
- blueprint-to-product relationships
- blueprint-to-material relationships

This information allows NEA to determine:

- which items can be manufactured
- what materials are required for production
- how items relate to blueprints

## Excluded

The Reference domain does **not** include:

- market prices
- market history
- blueprint ownership
- blueprint location
- blueprint material efficiency values
- blueprint time efficiency values
- corporation production jobs
- staging inventory
- sell orders
- recommendation outputs

These concerns belong to other domains.

---

# 3. Design Goals

The Reference domain schema must satisfy the following goals:

1. Identify all manufacturable items relevant to NEA.
2. Identify which blueprint produces each manufacturable item.
3. Identify the materials required to manufacture each item.
4. Provide clean join points for market pricing and corporation production state.
5. Avoid embedding corporation-specific or market-specific logic.

This domain should remain **purely static game data**.

---

# 4. Canonical Entities

The Reference domain for NEA v1 consists of four core tables:

1. `item_type`
2. `blueprint`
3. `blueprint_product`
4. `blueprint_material`

These tables define the manufacturing relationships required by the recommendation engine.

---

# 5. Table Definitions

## 5.1 `item_type`

Defines the canonical registry of item types relevant to manufacturing.

This table represents both:

- items that can be produced
- items that are used as production materials

### Purpose

- Provide stable identifiers for items
- Provide item names for output/reporting
- Provide join targets for blueprint outputs and materials

### Primary Key

```
type_id
```

### Core Columns

| Column | Description |
|------|-------------|
| type_id | Canonical EVE type ID |
| type_name | Human-readable item name |
| group_id | Optional grouping metadata |
| category_id | Optional category metadata |
| published | Optional flag from source data |

### Notes

Manufacturability is **not stored directly**.  
It is derived from the presence of a corresponding row in `blueprint_product`.

---

## 5.2 `blueprint`

Defines the canonical registry of blueprint definitions.

This table represents blueprint types independent of player ownership.

### Purpose

- Provide blueprint identity
- Serve as the parent entity for production relationships
- Support joins into blueprint products and materials

### Primary Key

```
blueprint_type_id
```

### Core Columns

| Column | Description |
|------|-------------|
| blueprint_type_id | Canonical blueprint type ID |
| blueprint_name | Human-readable blueprint name |
| published | Optional flag from source data |

### Notes

This table represents the **blueprint definition only**.

Corporation-owned blueprint instances belong in the **Corporation Production State domain**.

---

## 5.3 `blueprint_product`

Defines the manufacturing output produced by a blueprint.

Each row represents the item produced when the blueprint is used for manufacturing.

### Purpose

- Map blueprint definitions to produced items
- Identify candidate items for manufacturing analysis

### Primary Key

Composite key:

```
blueprint_type_id
product_type_id
```

### Core Columns

| Column | Description |
|------|-------------|
| blueprint_type_id | FK → `blueprint.blueprint_type_id` |
| product_type_id | FK → `item_type.type_id` |
| quantity | Number of output units produced per run |

### Notes

This table only contains **manufacturing outputs**.

Other industry activities (invention, reactions, copying, etc.) are intentionally excluded from the MVP Reference domain.

---

## 5.4 `blueprint_material`

Defines the material inputs required to manufacture an item using a blueprint.

### Purpose

- Represent the input materials required for production
- Support material-cost calculations

### Primary Key

Composite key:

```
blueprint_type_id
material_type_id
```

### Core Columns

| Column | Description |
|------|-------------|
| blueprint_type_id | FK → `blueprint.blueprint_type_id` |
| material_type_id | FK → `item_type.type_id` |
| quantity | Base material quantity required before ME adjustment |

### Notes

Material quantities stored here represent **canonical base values**.

Blueprint Material Efficiency adjustments are applied later using corporation-owned blueprint attributes.

---

# 6. Relationships

The core relationships within the Reference domain are:

```
blueprint_product.blueprint_type_id -> blueprint.blueprint_type_id
blueprint_product.product_type_id -> item_type.type_id

blueprint_material.blueprint_type_id -> blueprint.blueprint_type_id
blueprint_material.material_type_id -> item_type.type_id
```

Conceptually:

- a blueprint definition produces an item
- a blueprint definition requires one or more materials
- both outputs and materials are represented in `item_type`

---

# 7. Domain Rules

The following rules apply to the Reference domain.

### Rule 1 — Static Data Only

This domain contains only static game definitions.

No corporation state, user state, market state, or recommendation output may be stored here.

---

### Rule 2 — Blueprint Definition vs Blueprint Ownership

Blueprint definitions stored here represent **types of blueprints**, not specific blueprint instances.

Owned blueprints belong in the **Corporation Production State domain**.

---

### Rule 3 — Manufacturing Only

The Reference domain contains **manufacturing-related data only**.

Other industry activities are excluded from the MVP schema.

---

### Rule 4 — Material Quantities Are Stored Pre-ME

Material quantities represent base blueprint values.

Material Efficiency adjustments are applied later using corporation blueprint state.

---

### Rule 5 — Manufacturability Is Derived

An item is considered manufacturable if it appears as a valid output in `blueprint_product`.

Manufacturability is **not stored as a separate field**.

---

# 8. Expected Usage in NEA v1

The Reference domain supports the following core operations.

### Identify Candidate Items

By joining corporation-owned blueprint instances to `blueprint_product`, NEA can determine which items are eligible for production analysis.

---

### Determine Material Requirements

By joining `blueprint_product` and `blueprint_material`, NEA can determine the material list required to manufacture each candidate item.

---

### Join Market Prices

By joining item IDs from the Reference domain with market observation data, NEA can estimate:

- output item sale price
- material input cost

---

# 9. Out-of-Scope Extensions

The following potential extensions are intentionally excluded from the MVP Reference domain:

- recursive production-chain metadata
- invention and reaction activity modeling
- blueprint time efficiency metadata
- facility or structure production modifiers
- richer category or grouping hierarchies
- multi-activity industry modeling

These may be introduced in future versions if required.

---

# 10. Minimal MVP Table Summary

| Table | Purpose |
|---|---|
| `item_type` | Canonical registry of manufacturable outputs and material inputs |
| `blueprint` | Canonical blueprint definition registry |
| `blueprint_product` | Mapping from blueprint to produced item |
| `blueprint_material` | Mapping from blueprint to required materials |

This set of tables represents the **minimum Reference domain required to support the NEA v1 recommendation engine**.