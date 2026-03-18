# NEA MVP v1 Implementation Roadmap

**Strategy:** Vertical slice delivery - each milestone produces a working end-to-end system with expanding scope.

**Target Environment:** Kubernetes (k3s) from day one, MariaDB database, Jupyter notebook interface.

**Related Documentation:**

- [MVP v1 Specification](specification.md) - Requirements and scope
- [Component Repository Mapping](implementation/component_repository_mapping.md) - Which repos to work in
- [Testing Strategy](implementation/testing_strategy.md) - How to validate each milestone
- [Infrastructure Diagram](architecture/infrastructure.md) - Kubernetes architecture
- [Data Model Diagram](architecture/data_model.md) - Database schema overview

---

## Milestone 0: Foundation

**Goal:** Development environment ready, no NEA functionality yet.

**Deliverables:**

- MariaDB running in k3s
- Basic k8s manifests (namespace, deployment, service, pvc)
- Database connection verified from local development environment
- Empty NEA database created

**Components:**

- Infrastructure setup only
- No application code

**Success Criteria:**

- Can connect to MariaDB from local machine
- Can create/drop tables via SQL client

**Estimated Complexity:** Low (infrastructure setup)

---

## Milestone 1: Single Item End-to-End

**Goal:** Rank ONE manufacturable item with mock data to prove the full pipeline works.

**Deliverables:**

- Schema: Minimal tables (1 item type, 1 blueprint, 1 material, 1 market price, 1 config row)
- Data: Hardcoded/mock data for a single test item (e.g., "Tritanium" or similar)
- Analysis: Jupyter notebook that queries the database and calculates profit for that ONE item
- Output: Notebook displays item name and estimated profit

**Components Touched:**

- NEA-Schema: Create initial migration with minimal Reference + Market + Configuration tables
- NEA-Analysis: Create notebook with basic SELECT query and profit calculation
- No parsers yet - manual SQL INSERT for test data

**Success Criteria:**

- Notebook runs without errors
- Displays one item with calculated profit
- Profit calculation matches hand-calculated expected value

**Estimated Complexity:** Low (minimal schema, hardcoded data, simple query)

---

## Milestone 2: Reference Domain Complete

**Goal:** Populate the full Reference Domain from SDE data.

**Deliverables:**

- Schema: Complete Reference Domain tables (item_type, blueprint, blueprint_product, blueprint_material)
- SDE Parser: Script/service that reads SDE files and populates Reference Domain
- Kubernetes: Job or CronJob manifest for SDE import
- Analysis: Notebook updated to query against real SDE data
- Output: Notebook can now display ALL manufacturable items with material costs

**Components Touched:**

- NEA-Schema: Expand Reference Domain schema (add missing columns, indexes)
- NEA-SdeParser: Build Python script to parse SDE YAML/JSON → SQL inserts
- NEA-Analysis: Update notebook queries for multiple items

**Success Criteria:**

- SDE import job runs successfully in k8s
- Reference Domain tables populated with hundreds/thousands of items
- Notebook displays list of all manufacturable items from real game data
- Material costs calculated correctly from blueprint_material

**Estimated Complexity:** Medium (SDE parsing logic, larger dataset)

**SDE Files Needed:**

- typeIDs.yaml (item definitions)
- blueprints.yaml (blueprint definitions)
- Additional SDE files as needed for material relationships

---

## Milestone 3: Corporation State Integration

**Goal:** Add Corporation Domain and implement pipeline exclusion logic.

**Deliverables:**

- Schema: Corporation Domain tables (corp_location, corp_blueprint_instance, corp_industry_job, corp_staging_inventory, corp_active_sell_order)
- ESI Parser: Script/service that calls ESI API and populates Corporation Domain
- Kubernetes: Job or CronJob manifest for ESI refresh
- Configuration: Real corp_id, location_ids in nea_config table
- Analysis: Notebook updated with exclusion filters
- Output: Notebook displays ranked items EXCLUDING those in production/staging/for sale

**Components Touched:**

- NEA-Schema: Add Corporation Domain
- NEA-EsiParser: Build Python script for ESI endpoints (corporation/assets, blueprints, industry, orders)
- NEA-SsoAuth: May need basic OAuth token management (or manual token for MVP)
- NEA-Analysis: Add WHERE clauses for exclusion logic

**Success Criteria:**

- ESI parser authenticates and fetches corporation data
- Corporation Domain tables populated with real corp state
- Notebook correctly excludes items based on active jobs, staging inventory, and sell orders
- Manual verification that excluded items are actually in production/for sale in-game

**Estimated Complexity:** High (ESI authentication, API rate limits, multiple endpoints, complex exclusion logic)

**ESI Endpoints Needed:**

- /corporations/{corporation_id}/blueprints/
- /corporations/{corporation_id}/industry/jobs/
- /corporations/{corporation_id}/assets/
- /corporations/{corporation_id}/orders/

---

## Milestone 4: Market Pricing Integration

**Goal:** Add Market Domain and implement real profitability calculations.

**Deliverables:**

- Schema: Market Domain table (market_price_snapshot)
- ESI Parser: Extend to fetch market history data
- Analysis: Notebook updated with real sale price and profit margin calculations
- Output: Notebook displays ranked items by REAL estimated profitability

**Components Touched:**

- NEA-Schema: Add Market Domain
- NEA-EsiParser: Add market history endpoint parsing
- NEA-Analysis: Replace mock prices with market_price_snapshot queries

**Success Criteria:**

- Market prices fetched and stored for relevant region
- Profit calculations use real market averages
- Ranked list makes sense compared to in-game market conditions
- Manual verification of top-ranked items shows they are actually profitable

**Estimated Complexity:** Medium (new ESI endpoint, regional data)

**ESI Endpoints Needed:**

- /markets/{region_id}/history/

---

## Milestone 5: Operational Telemetry

**Goal:** Add observability and refresh tracking for production readiness.

**Deliverables:**

- Schema: Operational Telemetry Domain tables (source_instance, source_sync_status, source_refresh_run, api_request_log, file_import_log)
- Parser Updates: SDE and ESI parsers now log all refresh operations
- Monitoring: Can query telemetry to see last refresh times, error rates
- Analysis: Notebook displays data freshness warnings if stale

**Components Touched:**

- NEA-Schema: Add Operational Telemetry Domain
- NEA-SdeParser: Add telemetry logging
- NEA-EsiParser: Add telemetry logging
- NEA-Analysis: Add data freshness checks

**Success Criteria:**

- All refresh operations logged with status and timing
- Can query when each domain was last updated
- Failed refreshes are visible and debuggable
- Secrets properly redacted in logs

**Estimated Complexity:** Medium (cross-cutting concern, affects all parsers)

---

## Milestone 6: Production Hardening

**Goal:** Make the system reliable and maintainable for daily use.

**Deliverables:**

- Scheduling: CronJobs configured for automatic SDE and ESI refresh
- Error Handling: Retry logic, failure notifications
- Configuration Management: Environment variables, k8s ConfigMaps/Secrets
- Documentation: README updates for all repos explaining MVP v1 approach
- Notebook Improvements: Better formatting, error messages, usage instructions

**Components Touched:**

- All repos: Update READMEs
- Kubernetes manifests: Add CronJob schedules, resource limits, health checks
- Parsers: Add retry logic, better error messages

**Success Criteria:**

- System runs for 7 consecutive days without manual intervention
- Daily refresh jobs complete successfully
- Notebook produces valid recommendations each day
- Can recover from transient ESI API failures

**Estimated Complexity:** Medium (operational concerns, edge cases)

---

## Post-MVP: Future Enhancements (Out of Scope)

These are intentionally deferred from MVP v1:

- Web UI (NEA-WebApp)
- Multi-region analysis
- Price forecasting
- Advanced charting
- Recursive production chains
- Structure/tax optimization
- Automated production scheduling

---

## Development Principles

1. **Each milestone is deployable** - Can push to k8s and use it
2. **Each milestone is testable** - Can verify against in-game data
3. **Schema changes are migrations** - Never drop/recreate, always ALTER
4. **Commit early, commit often** - Incremental progress on dev branch
5. **Notebook is the interface** - No UI until MVP validated
6. **Manual configuration is OK** - Can hardcode corp_id, region_id for MVP
7. **Kubernetes-native** - No local-only scripts, everything runs in cluster

---

## Estimated Timeline

**Conservative estimate (solo development):**

- M0: Foundation - 1-2 days
- M1: Single Item - 2-3 days
- M2: Reference Domain - 1 week
- M3: Corporation State - 2 weeks (ESI complexity)
- M4: Market Pricing - 1 week
- M5: Operational Telemetry - 1 week
- M6: Production Hardening - 1 week

**Total: 7-9 weeks** for a fully operational MVP v1 system.

Actual timeline will vary based on:

- Kubernetes familiarity
- EVE API/SDE familiarity
- Time availability
- Debugging unexpected issues

---

## Next Steps

1. Review this roadmap with stakeholders (you!)
2. Set up Milestone 0 infrastructure
3. Create GitHub Project or issue tracking for milestones
4. Begin M0: Get MariaDB running in k3s
