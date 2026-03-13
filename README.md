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

This README is the primary project overview. Detailed MVP/schema specifications are
available in `docs/`, including:

- [NEA MVP v1 Specification](docs/nea_mvp_v1.md)
- `docs/schema/*.md`

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

Mature prototype and research platform developed for personal analytics.
Maintained in reference mode with stable core functionality.
