# NEA MVP Schema — Reference Domain

Schema Version: NEA MVP v1

---

# 1. Purpose

The Reference Domain contains the static game data required for NEA v1 to determine:

- what items exist
- what blueprints exist
- what item each blueprint produces
- what materials are required to manufacture that item

This domain serves as the **canonical source of production definitions** used by the NEA recommendation engine.

The Reference Domain does **not** contain market data, corporation-specific blueprint ownership, active jobs, sell orders, or any other operational state.

---

# 2. Scope

## Included

The Reference Domain includes:

- item/type definitions relevant to manufacturing
- blueprint definitions
- blueprint-to-product relationships
- blueprint-to-material relationships

This information allows NEA to determine:

- which items can be manufactured
- what materials are required for production
- how items relate to blueprints

## Excluded

The Reference Domain does **not** include:

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

The Reference Domain schema must satisfy the following goals:

1. Identify all manufacturable items relevant to NEA.
2. Identify which blueprint produces each manufacturable item.
3. Identify the materials required to manufacture each item.
4. Provide clean join points for the Market Observation Domain and the Corporation Production State Domain.
5. Avoid embedding corporation-specific or market-specific logic.

This domain should remain **purely static game data**.

---

# 4. Identifier Convention

All item identifiers use the canonical `type_id` identifier.

Columns referencing items follow these naming patterns:

| Column Pattern | Meaning |
|---|---|
| `type_id` | Generic item reference |
| `product_type_id` | Produced item reference |
| `material_type_id` | Input material reference |

Blueprint identifiers follow the pattern:

| Column | Meaning |
|---|---|
| `blueprint_type_id` | Blueprint type identifier |

This convention is used consistently across all NEA MVP domains.

---

# 5. Canonical Entities

The Reference Domain for NEA v1 consists of four tables:

1. `item_type`
2. `blueprint`
3. `blueprint_product`
4. `blueprint_material`

---

# 6. Table Definitions

## 6.1 `item_type`

Defines the canonical registry of item types relevant to manufacturing.

This table represents both:

- items that can be produced
- items that are used as production materials

### Primary Key

```
type_id
```

### Core Columns

| Column | Description |
|---|---|
| type_id | Canonical EVE type ID |
| type_name | Human-readable item name |
| group_id | Optional grouping metadata |
| category_id | Optional category metadata |
| published | Optional flag from source data |

### Notes

Manufacturability is **derived**, not stored.

An item is considered manufacturable if it appears as an output in `blueprint_product`.

---

## 6.2 `blueprint`

Defines blueprint type definitions.

### Primary Key

```
blueprint_type_id
```

### Core Columns

| Column | Description |
|---|---|
| blueprint_type_id | Canonical blueprint type ID |
| blueprint_name | Human-readable blueprint name |
| published | Optional flag |

### Notes

This table represents the **blueprint definition only**.

Corporation-owned blueprint instances belong in the **Corporation Production State Domain**.

---

## 6.3 `blueprint_product`

Defines which item a blueprint produces when used for manufacturing.

### Primary Key

```
(blueprint_type_id, product_type_id)
```

### Core Columns

| Column | Description |
|---|---|
| blueprint_type_id | FK → blueprint.blueprint_type_id |
| product_type_id | FK → item_type.type_id |
| quantity | Units produced per run |

### Notes

This table only contains **manufacturing outputs**.

Other industry activities such as invention or reactions are intentionally excluded from the MVP schema.

---

## 6.4 `blueprint_material`

Defines the materials required to manufacture an item.

### Primary Key

```
(blueprint_type_id, material_type_id)
```

### Core Columns

| Column | Description |
|---|---|
| blueprint_type_id | FK → blueprint.blueprint_type_id |
| material_type_id | FK → item_type.type_id |
| quantity | Base quantity required before ME adjustment |

### Notes

Material quantities represent **base blueprint requirements**.

Material Efficiency adjustments are applied later using blueprint attributes stored in the **Corporation Production State Domain**.

---

# 7. Relationships

Core relationships within the Reference Domain:

```
blueprint_product.blueprint_type_id → blueprint.blueprint_type_id
blueprint_product.product_type_id → item_type.type_id

blueprint_material.blueprint_type_id → blueprint.blueprint_type_id
blueprint_material.material_type_id → item_type.type_id
```

Conceptually:

- a blueprint produces an item
- a blueprint requires materials
- both outputs and inputs are represented in `item_type`

---

# 8. Domain Rules

## Rule 1 — Static Data Only

This domain contains only static game definitions.

No corporation state, market state, or recommendation output may be stored here.

## Rule 2 — Blueprint Definitions vs Instances

Blueprint definitions stored here represent **blueprint types**.

Owned blueprint instances are stored in the **Corporation Production State Domain**.

## Rule 3 — Manufacturing Only

The Reference Domain contains **manufacturing-related data only**.

Other industry activities are excluded from the MVP schema.

## Rule 4 — Material Quantities Are Stored Pre-ME

Material quantities represent base blueprint values.

Material Efficiency adjustments are applied later using blueprint instance attributes.

## Rule 5 — Manufacturability Is Derived

An item is considered manufacturable if it appears in `blueprint_product`.

---

# 9. Expected Usage in NEA v1

The Reference Domain supports the following operations.

## Usage Examples

### Identify Manufacturable Items

By joining corporation blueprint instances with `blueprint_product`, NEA identifies items that can be produced.

### Determine Material Requirements

By joining `blueprint_product` with `blueprint_material`, NEA determines required inputs.

### Join Market Prices

By joining item IDs with the Market Observation Domain, NEA determines:

- output sale price
- input material cost

---

# 10. Out-of-Scope Extensions

The following potential extensions are intentionally excluded from the MVP Reference Domain:

- recursive production-chain modeling
- invention workflows
- reaction workflows
- blueprint time efficiency metadata
- structure modifiers
- multi-activity industry modeling

These may be introduced in future versions.

---

# 11. Minimal MVP Table Summary

| Table | Purpose |
|---|---|
| item_type | Canonical registry of manufacturable items and materials |
| blueprint | Blueprint type definitions |
| blueprint_product | Blueprint → output mapping |
| blueprint_material | Blueprint → material requirements |

This set of tables represents the **minimum Reference Domain required to support the NEA v1 recommendation engine**.
