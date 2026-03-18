# New Eden Analytics

End-to-end EVE Online industrial analytics platform.

Historically, NEA included forecasting-heavy analytics and model-driven workflows.
The current version is focused on a gradual rebuild using explicit MVP milestones
and versioned specifications, with v1 centered on reliable profitability ranking.

---

## Overview

The platform analyzes market prices, material costs, blueprint capability, and
current corporation pipeline state to identify profitable manufacturing opportunities.

It provides:

- Ranked candidate items by estimated per-unit profitability
- Exclusion of items already in production, staging, or active sale
- Lightweight, fast pull-based analysis for daily decision support
- Operational telemetry for ingestion observability and debugging

Current delivery approach:

- milestone-driven MVP development
- explicit versioned specifications (for example, MVP v1)
- incremental scope expansion after reliability and usability validation

---

## Architecture

```
           EVE Online API + SDE Sources
                       ↓
         Ingestion and Refresh Pipelines
                       ↓
            Relational Domain Storage
  (Reference, Corporation, Market, Configuration)
                       ↓
              Analysis Query/Compute
                       ↓
          Ranked Manufacturing Candidates
                       ↓
         Run and Request Telemetry Logs

```

## Analysis Model

The MVP uses a deliberately simple profitability model:

- Estimated Profit = Estimated Sale Price - Estimated Production Cost
- Sale price from current market snapshot data
- Production cost from blueprint material requirements, ME, and material pricing
- Flat configured cost rate for taxes/fees approximation

Forecasting is intentionally deferred from MVP scope.

---

## Documentation

This README is the primary project overview. Detailed specifications and implementation guides are available in `docs/`:

**Core Specifications:**

- [NEA MVP v1 Specification](docs/mvp_v1/specification.md) - Requirements and scope
- [MVP v1 Roadmap](docs/mvp_v1/roadmap.md) - Implementation milestones (M0-M6)

**Implementation Guides:**

- [Component Repository Mapping](docs/mvp_v1/implementation/component_repository_mapping.md) - GitHub repo structure
- [Testing Strategy](docs/mvp_v1/implementation/testing_strategy.md) - Manual validation approach
- [ESI Endpoints](docs/mvp_v1/implementation/esi_endpoints.md) - EVE API integration details

**Architecture & Design:**

- [Data Model Diagram](docs/mvp_v1/architecture/data_model.md) - Database schema visualization
- [Infrastructure Diagram](docs/mvp_v1/architecture/infrastructure.md) - Kubernetes architecture

**Schema Specifications:**

- [Reference Domain](docs/mvp_v1/schema/reference_domain.md) - Static game data (SDE)
- [Corporation Domain](docs/mvp_v1/schema/corporation_domain.md) - Corp state from ESI
- [Market Domain](docs/mvp_v1/schema/market_domain.md) - Market pricing data
- [Configuration Domain](docs/mvp_v1/schema/configuration_domain.md) - Runtime configuration
- [Operational Telemetry Domain](docs/mvp_v1/schema/operational_telemetry_domain.md) - Observability

---

## Repository Structure

This organization contains multiple modular components:

Note: Most repos use a `main` and `dev` branch strategy, with ongoing work
typically integrated in `dev`.

### Core Components

- [**NEA-EsiParser**](https://github.com/New-Eden-Analytics/NEA-EsiParser)  
  API ingestion and normalization layer

- [**NEA-SdeParser**](https://github.com/New-Eden-Analytics/NEA-SdeParser)  
  Static data export parsing utilities

- [**NEA-Schema**](https://github.com/New-Eden-Analytics/NEA-Schema)  
  Canonical relational database schema

- [**NEA-WebApp**](https://github.com/New-Eden-Analytics/NEA-WebApp)  
  React-based user interface

- [**NEA-SsoAuth**](https://github.com/New-Eden-Analytics/NEA-SsoAuth)  
  User token authentication platform (OAuth2)

- [**NEA-Analysis**](https://github.com/New-Eden-Analytics/NEA-Analysis)  
  Analytics toolkit

### Supporting Components

- [**NEA-Mapper**](https://github.com/New-Eden-Analytics/NEA-Mapper)  
  Visualization and mapping utilities

- [**NEA-Mining-Manager**](https://github.com/New-Eden-Analytics/NEA-Mining-Manager)  
  Production and resource management tools

- [**StateSmoother**](https://github.com/Calvinxc1/StateSmoother)  
  Time-series smoothing and forecasting engine

---

## My Role

This project was developed and operated as a solo, end-to-end engineering effort.

Responsibilities included:

- System and infrastructure architecture
- Kubernetes (k3s) cluster on NixOS
- Secure networking, VPN, and TLS
- API ingestion and ETL pipelines
- Database design (MariaDB, MongoDB)
- Custom forecasting model development
- Frontend development (React)
- Deployment and long-term maintenance

AI tooling may assist with test authoring, documentation drafting/editing, GitHub Actions/workflow authoring and maintenance, and development guidance for planning/decision support.

---

## Technology Stack

- Python, JavaScript
- MariaDB, MongoDB
- PyTorch-based modeling
- React
- Kubernetes (k3s), Nginx
- NixOS
- EVE Online ESI API

---

## Status

Active refactor and rebuild in progress using milestone-driven MVP development with explicit versioned specifications.
