# New Eden Analytics

New Eden Analytics is an end-to-end platform for analyzing and forecasting EVE Online
market and production data using custom time-series and demand models.

The system ingests live and historical data via scheduled API pipelines, normalizes and
stores it in a relational database, applies domain-specific forecasting and activity
models, and serves results through a React-based web application backed by a
high-performance document store.

It was designed and implemented as a modular, multi-repository system and developed
as a complete, end-to-end engineering and modeling project.

---

## Overview

The primary goal of New Eden Analytics is to identify profitable production and trading
opportunities by analyzing:

- Historical and live market prices
- Trading volumes and sales velocity
- Material availability and costs
- End-to-end production chains

The platform forecasts material purchase prices, product sale prices, and demand trends,
and generates recommendations for optimal production volumes and market entry timing.

Results are presented through interactive dashboards, tabular reports, and
Sankey-style production chain visualizations.

---

## System Architecture
```
            EVE Online API
                    ↓
    Scheduled Ingestion Jobs (Python)
                    ↓
           MariaDB (Core Storage)
                    ↓
 Feature Engineering & Forecasting
 - Custom Exponential Smoothing
 - Sinusoidal Activity Modeling
                    ↓
           MongoDB (Serving Layer)
                    ↓
          React Web Application
```


---

## Modeling & Forecasting

The forecasting layer is implemented using custom domain-specific models rather than
off-the-shelf libraries.

Key components include:

- Custom exponential smoothing algorithms for short- and medium-term price and volume
  forecasting
- Sinusoidal regression models to capture cyclical player activity patterns
- Automated batch retraining and parameter persistence
- Integration with downstream profitability analysis

These models are optimized for noisy, irregularly sampled market data and evolving
player behavior patterns.

---

## Key Capabilities

- Automated ingestion of EVE Online market and production data
- Normalization and validation of heterogeneous data sources
- Relational and document-based storage layers
- Time-series and demand forecasting
- Profitability and margin analysis
- Production chain dependency modeling
- Interactive web-based exploration tools

---

## Repository Structure

This organization contains multiple modular components:

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

This project was developed and maintained as a solo, end-to-end engineering effort.

Responsibilities included:

- Overall system architecture and design
- API integration and data ingestion pipelines
- Database schema design (MariaDB)
- Feature engineering and modeling pipelines
- Development of custom forecasting models
- Implementation of dual-database serving architecture
- Frontend development (React)
- Deployment, scheduling, and maintenance

---

## Technology Stack

- **Languages:** Python, JavaScript
- **Databases:** MariaDB, MongoDB
- **Modeling:** Custom time-series and regression models (PyTorch-based components)
- **Frontend:** React
- **Infrastructure:** Local deployment
- **APIs:** [EVE Online API](https://developers.eveonline.com/api-explorer)

---

## Usage Model

The platform is deployed locally and operates through scheduled background jobs that
periodically refresh market and production data.

Users interact with the system primarily through the web interface, which provides
near-real-time access to forecasts, profitability analyses, and production planning tools.

---

## Status

New Eden Analytics is a mature prototype and research system.

Core functionality is stable, and the system has been used extensively for personal
analysis and experimentation. Public documentation and deployment automation are limited,
as the platform was primarily designed for individual use.

The project is maintained in a low-activity / reference mode.

---

## License

See individual repositories for license information.
