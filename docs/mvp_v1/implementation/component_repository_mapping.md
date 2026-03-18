# Component Repository Mapping - NEA MVP v1

This document maps each milestone's deliverables to the appropriate GitHub repositories.

**Related Documentation:**
- [MVP v1 Roadmap](../roadmap.md) - Milestone definitions and timeline
- [Testing Strategy](testing_strategy.md) - How to validate deliverables
- [Infrastructure Diagram](../architecture/infrastructure.md) - Deployment architecture

---

## Repository Overview

| Repository | Primary Purpose | Languages/Tools |
|---|---|---|
| [Overview](https://github.com/New-Eden-Analytics/Overview) | Documentation, specifications, project management | Markdown |
| [NEA-Schema](https://github.com/New-Eden-Analytics/NEA-Schema) | Database schema definitions and migrations | SQL, Alembic/Flyway |
| [NEA-SdeParser](https://github.com/New-Eden-Analytics/NEA-SdeParser) | Static Data Export parsing and ingestion | Python |
| [NEA-EsiParser](https://github.com/New-Eden-Analytics/NEA-EsiParser) | ESI API client and data ingestion | Python |
| [NEA-SsoAuth](https://github.com/New-Eden-Analytics/NEA-SsoAuth) | OAuth token management | Python |
| [NEA-Analysis](https://github.com/New-Eden-Analytics/NEA-Analysis) | Analytics queries and Jupyter notebooks | Python, Jupyter, SQL |

---

## Milestone 0: Foundation

**Infrastructure only - no code repositories involved**

**Deliverables:**
- k8s manifests (MariaDB deployment, service, PVC)
- Database connection validation

**Location:**
- Create new directory: `infrastructure/` or `k8s/` in Overview repo
- Or create separate repo: `NEA-Infrastructure` (optional)

**Files:**
```
infrastructure/
├── mariadb/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── pvc.yaml
│   └── secret.yaml (template, not committed)
└── README.md
```

---

## Milestone 1: Single Item End-to-End

### NEA-Schema

**Deliverables:**
- Initial schema with minimal tables
- Setup for migration tool (if using)

**Files:**
```
NEA-Schema/
├── migrations/
│   └── 001_initial_schema.sql
├── README.md
└── requirements.txt (if using Alembic)
```

**Tables created:**
- `item_type`
- `blueprint`
- `blueprint_material`
- `market_price_snapshot`
- `nea_config`

---

### NEA-Analysis

**Deliverables:**
- First Jupyter notebook
- Database connection utilities
- Basic profitability query

**Files:**
```
NEA-Analysis/
├── notebooks/
│   └── manufacturing_recommendations.ipynb
├── src/
│   └── db_connection.py
├── requirements.txt
└── README.md
```

---

## Milestone 2: Reference Domain Complete

### NEA-Schema

**Deliverables:**
- Complete Reference Domain tables
- Migration to add indexes and missing columns

**Files:**
```
NEA-Schema/
├── migrations/
│   ├── 001_initial_schema.sql
│   └── 002_reference_domain_complete.sql
```

**Tables added/modified:**
- `item_type` (expanded)
- `blueprint` (expanded)
- `blueprint_product` (new)
- `blueprint_material` (expanded)

---

### NEA-SdeParser

**Deliverables:**
- SDE file parser
- Database insertion logic
- Kubernetes Job manifest

**Files:**
```
NEA-SdeParser/
├── src/
│   ├── main.py
│   ├── parsers/
│   │   ├── type_parser.py
│   │   └── blueprint_parser.py
│   ├── db/
│   │   └── inserter.py
│   └── config.py
├── k8s/
│   └── sde-import-job.yaml
├── Dockerfile
├── requirements.txt
└── README.md
```

---

### NEA-Analysis

**Deliverables:**
- Updated notebook with multi-item queries
- Material cost calculations

**Files:**
```
NEA-Analysis/
├── notebooks/
│   └── manufacturing_recommendations.ipynb (updated)
├── src/
│   ├── db_connection.py
│   └── queries.py (new - reusable query functions)
└── requirements.txt (updated)
```

---

## Milestone 3: Corporation State Integration

### NEA-Schema

**Deliverables:**
- Corporation Domain tables
- Migration for corp tables

**Files:**
```
NEA-Schema/
├── migrations/
│   ├── 001_initial_schema.sql
│   ├── 002_reference_domain_complete.sql
│   └── 003_corporation_domain.sql
```

**Tables added:**
- `corp_location`
- `corp_blueprint_instance`
- `corp_industry_job`
- `corp_staging_inventory`
- `corp_active_sell_order`

---

### NEA-EsiParser

**Deliverables:**
- ESI API client
- OAuth token handling
- Corporation endpoint parsers
- Kubernetes CronJob manifest

**Files:**
```
NEA-EsiParser/
├── src/
│   ├── main.py
│   ├── api/
│   │   ├── client.py
│   │   └── auth.py
│   ├── parsers/
│   │   ├── blueprints_parser.py
│   │   ├── industry_parser.py
│   │   ├── assets_parser.py
│   │   └── orders_parser.py
│   ├── db/
│   │   └── inserter.py
│   └── config.py
├── k8s/
│   ├── esi-corp-refresh-cronjob.yaml
│   └── esi-secret-template.yaml
├── Dockerfile
├── requirements.txt
└── README.md
```

---

### NEA-SsoAuth (Optional for MVP)

**Deliverables:**
- Token management utilities (if not embedded in NEA-EsiParser)

**Files:**
```
NEA-SsoAuth/
├── src/
│   ├── token_manager.py
│   └── oauth_flow.py
├── requirements.txt
└── README.md
```

**Note:** For MVP, OAuth token handling can live in NEA-EsiParser directly. NEA-SsoAuth can be used if you want to separate concerns.

---

### NEA-Analysis

**Deliverables:**
- Updated notebook with exclusion logic
- Exclusion queries

**Files:**
```
NEA-Analysis/
├── notebooks/
│   └── manufacturing_recommendations.ipynb (updated)
├── src/
│   ├── db_connection.py
│   ├── queries.py (updated with exclusions)
│   └── exclusions.py (new)
└── requirements.txt
```

---

## Milestone 4: Market Pricing Integration

### NEA-Schema

**Deliverables:**
- Market Domain table
- Migration for market table

**Files:**
```
NEA-Schema/
├── migrations/
│   ├── 001_initial_schema.sql
│   ├── 002_reference_domain_complete.sql
│   ├── 003_corporation_domain.sql
│   └── 004_market_domain.sql
```

**Tables added:**
- `market_price_snapshot`

---

### NEA-EsiParser

**Deliverables:**
- Market history endpoint parser
- Price calculation logic
- Updated CronJob for market refresh

**Files:**
```
NEA-EsiParser/
├── src/
│   ├── main.py
│   ├── api/
│   │   └── client.py
│   ├── parsers/
│   │   ├── (existing corp parsers)
│   │   └── market_history_parser.py (new)
│   ├── db/
│   │   └── inserter.py (updated)
│   └── config.py
├── k8s/
│   ├── esi-corp-refresh-cronjob.yaml
│   └── esi-market-refresh-cronjob.yaml (new)
└── README.md (updated)
```

---

### NEA-Analysis

**Deliverables:**
- Updated notebook with real profit calculations
- Ranking logic

**Files:**
```
NEA-Analysis/
├── notebooks/
│   └── manufacturing_recommendations.ipynb (updated)
├── src/
│   ├── db_connection.py
│   ├── queries.py (updated with market joins)
│   ├── exclusions.py
│   └── profitability.py (new)
└── requirements.txt
```

---

## Milestone 5: Operational Telemetry

### NEA-Schema

**Deliverables:**
- Operational Telemetry Domain tables
- Migration for telemetry tables

**Files:**
```
NEA-Schema/
├── migrations/
│   ├── 001_initial_schema.sql
│   ├── 002_reference_domain_complete.sql
│   ├── 003_corporation_domain.sql
│   ├── 004_market_domain.sql
│   └── 005_operational_telemetry_domain.sql
```

**Tables added:**
- `source_instance`
- `source_sync_status`
- `source_refresh_run`
- `api_request_log`
- `file_import_log`

---

### NEA-SdeParser

**Deliverables:**
- Telemetry logging integration

**Files:**
```
NEA-SdeParser/
├── src/
│   ├── main.py (updated with logging)
│   ├── parsers/
│   │   └── (existing parsers)
│   ├── db/
│   │   └── inserter.py
│   └── telemetry/
│       └── logger.py (new)
└── README.md (updated)
```

---

### NEA-EsiParser

**Deliverables:**
- Telemetry logging integration
- API request logging

**Files:**
```
NEA-EsiParser/
├── src/
│   ├── main.py (updated)
│   ├── api/
│   │   └── client.py (updated with request logging)
│   ├── parsers/
│   │   └── (existing parsers)
│   ├── db/
│   │   └── inserter.py
│   └── telemetry/
│       └── logger.py (new)
└── README.md (updated)
```

---

### NEA-Analysis

**Deliverables:**
- Data freshness checks in notebook
- Telemetry query utilities

**Files:**
```
NEA-Analysis/
├── notebooks/
│   └── manufacturing_recommendations.ipynb (updated with freshness section)
├── src/
│   ├── db_connection.py
│   ├── queries.py
│   ├── exclusions.py
│   ├── profitability.py
│   └── telemetry.py (new)
└── requirements.txt
```

---

## Milestone 6: Production Hardening

**All repositories touched for polish and reliability**

### Infrastructure/Overview

**Deliverables:**
- Updated k8s manifests with proper schedules
- ConfigMaps and Secrets templates
- Production-ready deployment instructions

**Files:**
```
infrastructure/
├── mariadb/
│   └── (existing files)
├── cronjobs/
│   ├── sde-refresh-cronjob.yaml
│   ├── esi-corp-refresh-cronjob.yaml
│   └── esi-market-refresh-cronjob.yaml
├── config/
│   ├── configmap-template.yaml
│   └── secret-template.yaml
└── README.md (comprehensive setup guide)
```

---

### All Parser Repositories

**Deliverables:**
- Error handling and retry logic
- Better logging and error messages
- Updated READMEs

**Files in each parser repo:**
```
src/
├── (existing files updated with error handling)
└── utils/
    └── retry.py (new)
README.md (updated with MVP v1 context)
```

---

### NEA-Analysis

**Deliverables:**
- Polished notebook
- Export functionality
- Usage documentation

**Files:**
```
NEA-Analysis/
├── notebooks/
│   └── manufacturing_recommendations.ipynb (polished)
├── src/
│   ├── (existing files)
│   └── export.py (new - CSV export)
└── README.md (comprehensive usage guide)
```

---

## Summary: Repository Activity by Milestone

| Repository | M0 | M1 | M2 | M3 | M4 | M5 | M6 |
|---|---|---|---|---|---|---|---|
| Overview | ✓ | - | - | - | - | - | ✓ |
| NEA-Schema | - | ✓ | ✓ | ✓ | ✓ | ✓ | - |
| NEA-SdeParser | - | - | ✓ | - | - | ✓ | ✓ |
| NEA-EsiParser | - | - | - | ✓ | ✓ | ✓ | ✓ |
| NEA-SsoAuth | - | - | - | ✓ | - | - | - |
| NEA-Analysis | - | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

---

## Development Workflow

### Starting a New Milestone

1. Create feature branch in relevant repo(s): `git checkout -b milestone-X-description`
2. Work on milestone deliverables
3. Test locally
4. Commit incrementally to `dev` branch
5. Deploy to k8s for integration testing
6. When milestone complete, create PR from `dev` to `main`

### Cross-Repository Changes

When a milestone touches multiple repos:
1. Work on repos in dependency order (Schema → Parsers → Analysis)
2. Test integration after each repo update
3. Can commit to `dev` in each repo independently
4. When full milestone works, promote all repos to `main` together

---

## Docker Image Organization

**Registry:** Docker Hub (or GitHub Container Registry)

**Naming Convention:**
- `neweden/nea-sde-parser:latest`
- `neweden/nea-sde-parser:v1.0.0`
- `neweden/nea-esi-parser:latest`
- `neweden/nea-esi-parser:v1.0.0`

**Tagging Strategy:**
- `latest` - current dev branch
- `v1.0.0` - tagged release when MVP complete
- `dev` - development builds (optional)

---

## Next Steps

1. Clone all relevant repositories locally
2. Checkout `dev` branch in each (or create it if doesn't exist)
3. Begin M0 in Overview repo (infrastructure)
4. Proceed through milestones, updating repos as mapped above
