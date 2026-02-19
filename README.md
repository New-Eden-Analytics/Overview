# New Eden Analytics

End-to-end EVE Online market analytics and forecasting platform with custom
time-series and demand models.

New Eden Analytics ingests live and historical market data, applies domain-specific
forecasting models, and serves profitability and production insights through a
web-based interface.

---

## Overview

The platform analyzes market prices, volumes, material costs, and production chains
to identify profitable manufacturing and trading opportunities.

It provides:

- Price and demand forecasts
- Production volume recommendations
- Full supply-chain profitability analysis
- Interactive visualizations and reports

---

## Architecture

```
                 EVE Online API
                       ↓
     Dynamic Ingestion & ETL Manager (Python)
                       ↓
             MariaDB (Relational Core)
                       ↓
       Containerized Forecasting Services
         - Custom Exponential Smoothing
         - Sinusoidal Activity Modeling
                       ↓
             MariaDB (Model Outputs)
                       ↓
          Serving ETL (MariaDB → MongoDB)
                       ↓
             MongoDB (Serving Layer)
                       ↓
             React Web Application

```

## Modeling

Forecasting is implemented using custom, domain-specific models:

- Exponential smoothing for price and volume prediction
- Sinusoidal regression for player activity cycles
- Automated retraining and parameter persistence

Models operate directly on relational data and publish results back to MariaDB.

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
