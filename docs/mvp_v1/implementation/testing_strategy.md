# Testing Strategy - NEA MVP v1

This document defines how to validate that each milestone is complete and working correctly.

**Philosophy:** Manual validation against in-game reality. Automated tests are out of MVP scope.

**Related Documentation:**
- [MVP v1 Roadmap](../roadmap.md) - Milestone definitions being tested
- [Data Model Diagram](../architecture/data_model.md) - Schema to validate against
- [ESI Endpoints](esi_endpoints.md) - API endpoints and expected responses

---

## General Testing Principles

1. **Test against real game data** - EVE Online client is your source of truth
2. **Test incrementally** - Validate each milestone before moving to the next
3. **Document test cases** - Record what you tested and results
4. **Accept imperfection** - MVP doesn't need to be bug-free, just usable

---

## Milestone 0: Foundation

### Success Criteria
- Can connect to MariaDB from local machine
- Can create/drop tables via SQL client

### Testing Steps

1. **Verify k8s deployment:**
   ```bash
   kubectl get pods -n nea
   # Should show mariadb pod running
   ```

2. **Verify service is accessible:**
   ```bash
   kubectl get svc -n nea
   # Note the service IP/port
   ```

3. **Test database connection:**
   ```bash
   mysql -h <cluster-ip> -u nea -p
   # Should prompt for password and connect
   ```

4. **Test database operations:**
   ```sql
   CREATE DATABASE nea;
   USE nea;
   CREATE TABLE test (id INT);
   DROP TABLE test;
   # All commands should succeed
   ```

5. **Test persistence:**
   ```bash
   # Delete the pod
   kubectl delete pod <mariadb-pod-name> -n nea

   # Wait for it to restart
   kubectl get pods -n nea

   # Reconnect and verify database still exists
   mysql -h <cluster-ip> -u nea -p
   SHOW DATABASES;
   # Should still see 'nea' database
   ```

### Expected Results
- ✅ Pod runs without errors
- ✅ Can connect from local machine
- ✅ Can execute SQL commands
- ✅ Data persists after pod restart

### Troubleshooting
- If can't connect: Check k8s service, firewall rules
- If pod crashes: Check logs with `kubectl logs <pod-name> -n nea`
- If data doesn't persist: Verify PVC is bound

---

## Milestone 1: Single Item End-to-End

### Success Criteria
- Notebook runs without errors
- Displays one item with calculated profit
- Profit calculation matches hand-calculated expected value

### Testing Steps

1. **Manual calculation (reference):**
   - Pick a simple item (e.g., Tritanium bar from ore refining)
   - Look up material costs in-game
   - Look up sale price in-game
   - Calculate: `profit = sale_price - sum(material_costs)`
   - Record expected profit

2. **Verify schema:**
   ```sql
   USE nea;
   SHOW TABLES;
   # Should see: item_type, blueprint, blueprint_material, market_price_snapshot, nea_config

   SELECT * FROM item_type;
   SELECT * FROM blueprint;
   SELECT * FROM blueprint_material;
   SELECT * FROM market_price_snapshot;
   SELECT * FROM nea_config;
   # Should see your test data
   ```

3. **Run notebook:**
   - Open `manufacturing_recommendations.ipynb`
   - Run all cells
   - Verify no errors

4. **Validate output:**
   - Notebook should display:
     - Item name
     - Calculated profit
   - Compare calculated profit to your manual calculation
   - Should match within reasonable tolerance (±1% due to rounding)

5. **Test edge cases:**
   - Change material cost in database
   - Re-run notebook
   - Verify profit calculation updates correctly

### Expected Results
- ✅ Schema tables exist and contain data
- ✅ Notebook runs without exceptions
- ✅ Profit calculation is mathematically correct
- ✅ Output is readable and makes sense

### Troubleshooting
- If connection fails: Check database credentials
- If query fails: Verify foreign keys are correct
- If profit wrong: Check material cost calculation logic

---

## Milestone 2: Reference Domain Complete

### Success Criteria
- SDE import job runs successfully in k8s
- Reference Domain tables populated with hundreds/thousands of items
- Notebook displays list of all manufacturable items from real game data
- Material costs calculated correctly from blueprint_material

### Testing Steps

1. **Run SDE import:**
   ```bash
   kubectl apply -f k8s/sde-import-job.yaml
   kubectl logs -f job/sde-import -n nea
   # Watch for completion
   ```

2. **Verify data volume:**
   ```sql
   SELECT COUNT(*) FROM item_type;
   # Should be ~10,000+

   SELECT COUNT(*) FROM blueprint;
   # Should be ~500+

   SELECT COUNT(*) FROM blueprint_material;
   # Should be ~5,000+

   SELECT COUNT(*) FROM blueprint_product;
   # Should be ~500+
   ```

3. **Spot-check specific items:**
   - Pick 5 random items from EVE client that you know
   - Verify they exist in database:
   ```sql
   SELECT * FROM item_type WHERE type_name LIKE '%Tritanium%';
   SELECT * FROM item_type WHERE type_name LIKE '%Drake%';
   -- etc.
   ```

4. **Verify blueprint-material relationships:**
   - Pick a known blueprint (e.g., Stabber cruiser)
   - Look up materials in EVE client
   - Query database:
   ```sql
   SELECT
     bp.blueprint_type_id,
     bp.activity_id,
     it.type_name as material,
     bm.quantity
   FROM blueprint bp
   JOIN blueprint_material bm ON bp.blueprint_type_id = bm.blueprint_type_id
   JOIN item_type it ON bm.material_type_id = it.type_id
   WHERE bp.blueprint_type_id = <stabber_blueprint_id>;
   ```
   - Compare to in-game: quantities should match

5. **Run updated notebook:**
   - Open `manufacturing_recommendations.ipynb`
   - Run all cells
   - Verify it displays multiple items (not just one)

6. **Validate sample calculations:**
   - Pick 3 random items from notebook output
   - For each, manually verify material costs:
     - Look up blueprint in-game
     - Check materials required
     - Calculate cost manually
     - Compare to notebook output
   - Should match within ±5% (due to ME variations)

### Expected Results
- ✅ SDE import completes without errors
- ✅ Database contains thousands of items
- ✅ Spot-checked items exist with correct data
- ✅ Blueprint material relationships are accurate
- ✅ Notebook displays comprehensive list
- ✅ Sample calculations match manual verification

### Troubleshooting
- If import fails: Check SDE file paths, format
- If counts low: Verify SDE files are complete
- If materials wrong: Check SDE blueprint parsing logic

---

## Milestone 3: Corporation State Integration

### Success Criteria
- ESI parser authenticates and fetches corporation data
- Corporation Domain tables populated with real corp state
- Notebook correctly excludes items based on active jobs, staging inventory, and sell orders
- Manual verification that excluded items are actually in production/for sale in-game

### Testing Steps

1. **Verify OAuth token:**
   ```bash
   kubectl get secrets -n nea
   kubectl describe secret esi-token -n nea
   # Should see token stored
   ```

2. **Run ESI parser:**
   ```bash
   kubectl apply -f k8s/esi-corp-refresh-cronjob.yaml
   # Trigger manual run:
   kubectl create job --from=cronjob/esi-corp-refresh esi-manual-test -n nea
   kubectl logs -f job/esi-manual-test -n nea
   ```

3. **Verify corporation data populated:**
   ```sql
   SELECT COUNT(*) FROM corp_blueprint_instance;
   # Should match your corp's blueprint count

   SELECT COUNT(*) FROM corp_industry_job WHERE is_active = 1;
   # Should match active jobs in-game

   SELECT COUNT(*) FROM corp_staging_inventory;
   # Should show items in configured staging location

   SELECT COUNT(*) FROM corp_active_sell_order WHERE is_active = 1;
   # Should match active sell orders in EVE
   ```

4. **Cross-reference with EVE client:**
   - Open EVE client
   - Go to Industry > Jobs
   - Note items currently being manufactured
   - Query database:
   ```sql
   SELECT product_type_id, type_name
   FROM corp_industry_job cij
   JOIN item_type it ON cij.product_type_id = it.type_id
   WHERE is_active = 1;
   ```
   - Verify matches in-game jobs

5. **Test exclusion logic:**
   - Before running notebook, note an item you're currently manufacturing
   - Run notebook
   - Verify that item does NOT appear in recommendations
   - Start a new manufacturing job in-game
   - Wait 30 minutes (for ESI refresh)
   - Re-run notebook
   - Verify new item is now excluded

6. **Test staging exclusion:**
   - Move items to staging hangar in-game
   - Wait for ESI refresh
   - Re-run notebook
   - Verify those items are excluded

7. **Test sell order exclusion:**
   - Create a sell order for an item
   - Wait for ESI refresh
   - Re-run notebook
   - Verify that item is excluded

### Expected Results
- ✅ ESI authentication succeeds
- ✅ Corporation data fetched and stored
- ✅ Data matches in-game state
- ✅ Active jobs correctly excluded
- ✅ Staged items correctly excluded
- ✅ Items with sell orders correctly excluded
- ✅ Exclusions update within 30 minutes of in-game changes

### Troubleshooting
- If auth fails: Verify token, scopes, character roles
- If data missing: Check ESI endpoint responses, pagination
- If exclusions wrong: Verify location_id configuration, check query logic

---

## Milestone 4: Market Pricing Integration

### Success Criteria
- Market prices fetched and stored for relevant region
- Profit calculations use real market averages
- Ranked list makes sense compared to in-game market conditions
- Manual verification of top-ranked items shows they are actually profitable

### Testing Steps

1. **Verify market data populated:**
   ```sql
   SELECT COUNT(*) FROM market_price_snapshot;
   # Should have prices for most manufactureable items

   SELECT * FROM market_price_snapshot ORDER BY last_updated DESC LIMIT 10;
   # Should show recent timestamps
   ```

2. **Spot-check market prices:**
   - Pick 5 random items from market_price_snapshot
   - Look up prices in EVE market (The Forge region)
   - Compare:
   ```sql
   SELECT type_id, type_name, average_price
   FROM market_price_snapshot mps
   JOIN item_type it ON mps.type_id = it.type_id
   WHERE type_name IN ('Tritanium', 'Mexallon', 'Isogen', 'Nocxium', 'Zydrine');
   ```
   - Database prices should be within 10% of in-game prices

3. **Run notebook with market data:**
   - Open `manufacturing_recommendations.ipynb`
   - Run all cells
   - Should now show:
     - Estimated sale price (from market)
     - Material cost
     - Estimated profit
     - Profit margin percentage

4. **Validate top recommendations:**
   - Note top 5 items from notebook
   - For each item:
     - Look up blueprint in-game
     - Check material costs
     - Check current market sell price
     - Manually calculate profit
     - Compare to notebook calculation
   - Should match within ±10%

5. **Sanity check rankings:**
   - Top items should have positive profit
   - Items should be ranked by profit (highest first)
   - No items with negative profit at top of list
   - Profit margins should be reasonable (not 1000000%)

6. **Test with market fluctuations:**
   - Note current top item
   - Wait 24 hours (for market refresh)
   - Re-run notebook
   - If market moved, rankings should change

### Expected Results
- ✅ Market data fetched and stored
- ✅ Prices match in-game market (within tolerance)
- ✅ Profit calculations are correct
- ✅ Rankings are in correct order
- ✅ Top recommendations are actually profitable in-game
- ✅ Rankings update with market changes

### Troubleshooting
- If prices wrong: Check region_id, verify averaging logic
- If profits negative: Check material cost calculation
- If rankings nonsensical: Verify sort order in query

---

## Milestone 5: Operational Telemetry

### Success Criteria
- All refresh operations logged with status and timing
- Can query when each domain was last updated
- Failed refreshes are visible and debuggable
- Secrets properly redacted in logs

### Testing Steps

1. **Verify telemetry tables populated:**
   ```sql
   SELECT * FROM source_instance;
   # Should show: SDE, ESI-Corp, ESI-Market

   SELECT * FROM source_sync_status;
   # Should show last_success_at timestamps

   SELECT COUNT(*) FROM source_refresh_run;
   # Should have multiple entries

   SELECT COUNT(*) FROM api_request_log;
   # Should have entries for ESI calls

   SELECT COUNT(*) FROM file_import_log;
   # Should have SDE import entries
   ```

2. **Check data freshness:**
   ```sql
   SELECT
     source_name,
     last_success_at,
     TIMESTAMPDIFF(MINUTE, last_success_at, NOW()) as minutes_ago
   FROM source_sync_status;
   # Should show recent timestamps
   ```

3. **Test notebook freshness warnings:**
   - Run notebook
   - Should display "Data Freshness" section at top
   - Should show last update time for each domain
   - If data is stale (>24 hours), should show warning

4. **Test failure logging:**
   - Temporarily break ESI token (invalid token)
   - Trigger manual ESI refresh
   - Check logs:
   ```sql
   SELECT * FROM source_refresh_run WHERE status = 'failed' ORDER BY start_time DESC LIMIT 1;
   # Should show error details
   ```
   - Verify error message is helpful
   - Verify no secrets leaked in error message

5. **Test API request logging:**
   ```sql
   SELECT
     endpoint,
     http_status,
     response_time_ms,
     created_at
   FROM api_request_log
   ORDER BY created_at DESC
   LIMIT 10;
   # Should show recent ESI calls
   ```
   - Verify status codes are reasonable (200, 304, etc.)
   - Verify response times are reasonable (<1000ms typically)

6. **Test debugging workflow:**
   - Simulate: "Why is my market data stale?"
   - Query:
   ```sql
   SELECT * FROM source_refresh_run
   WHERE source_name = 'ESI-Market'
   ORDER BY start_time DESC LIMIT 5;
   ```
   - Should be able to see when last refresh ran, if it succeeded, how long it took

### Expected Results
- ✅ All refresh operations logged
- ✅ Last success timestamps visible
- ✅ Failed refreshes logged with error details
- ✅ Secrets not exposed in logs
- ✅ Notebook displays data freshness
- ✅ Can debug refresh failures from telemetry data

### Troubleshooting
- If telemetry missing: Check logger integration in parsers
- If secrets exposed: Review logging code, add redaction

---

## Milestone 6: Production Hardening

### Success Criteria
- System runs for 7 consecutive days without manual intervention
- Daily refresh jobs complete successfully
- Notebook produces valid recommendations each day
- Can recover from transient ESI API failures

### Testing Steps

1. **7-Day Reliability Test:**
   - Day 0: Deploy everything, verify working
   - Day 1-7: Check once per day:
     ```sql
     SELECT * FROM source_sync_status;
     # All sources should show recent updates

     SELECT COUNT(*) FROM source_refresh_run WHERE created_at > NOW() - INTERVAL 1 DAY;
     # Should show refresh runs
     ```
   - Record any failures

2. **Test CronJob schedules:**
   ```bash
   kubectl get cronjobs -n nea
   # Verify schedules are correct

   kubectl get jobs -n nea
   # Should see successful job runs
   ```

3. **Test error recovery:**
   - Simulate transient ESI failure (rate limit)
   - Check telemetry:
   ```sql
   SELECT * FROM source_refresh_run WHERE status = 'failed' ORDER BY start_time DESC LIMIT 1;
   ```
   - Wait for next scheduled run
   - Verify it retries and succeeds

4. **Test configuration changes:**
   - Update ConfigMap (change region_id)
   - Restart relevant pods
   - Verify new configuration takes effect

5. **Test notebook daily workflow:**
   - Each day for 7 days:
     - Open notebook
     - Run all cells
     - Verify recommendations are produced
     - Verify data is fresh (<24 hours)
     - Pick top item, start manufacturing
   - On following days:
     - Verify manufactured items are excluded

6. **Test documentation:**
   - Have someone else (or future you) follow README
   - Can they set up and run the system from scratch?
   - Document any confusing steps

### Expected Results
- ✅ System runs 7 days without crashes
- ✅ Refresh jobs complete successfully (>95% success rate)
- ✅ Notebook produces valid daily recommendations
- ✅ Transient failures don't stop the system
- ✅ Configuration is easy to change
- ✅ Documentation is sufficient for setup

### Troubleshooting
- If reliability low: Add more error handling, increase retry attempts
- If jobs fail: Check k8s resources, logs
- If documentation unclear: Update with learnings

---

## Acceptance Testing - Full MVP

### Final Validation (After M6)

**Scenario:** "I want to find the most profitable item to manufacture today"

**Steps:**
1. Open notebook
2. Run all cells
3. Read top 5 recommendations

**Validation:**
- ✅ Notebook runs without errors
- ✅ Top items have positive profit
- ✅ Items are not already in production
- ✅ Items are not in staging
- ✅ Items are not for sale
- ✅ Prices reflect current market
- ✅ Data is fresh (<24 hours)
- ✅ Can understand the output

**Time to result:** <5 minutes

---

## Testing Anti-Patterns (Things NOT to do)

❌ **Don't write unit tests for MVP** - Not worth the time investment yet

❌ **Don't automate testing** - Manual validation is fine for MVP

❌ **Don't test every item** - Spot-checking is sufficient

❌ **Don't expect perfection** - 95% accuracy is good enough

❌ **Don't test without in-game validation** - Database could be wrong and tests pass

---

## When to Stop Testing and Ship

**MVP is ready when:**
- All 6 milestones pass their success criteria
- You personally use the notebook for 7 consecutive days
- You trust the recommendations enough to act on them
- You can debug issues when they occur

**MVP is NOT ready when:**
- Frequently produces nonsensical recommendations
- Data is often stale (>50% of the time)
- Exclusion logic doesn't work
- Can't figure out what went wrong when it breaks

---

## Post-MVP Testing

Once MVP is validated, consider:
- Automated integration tests
- Performance testing (query speed)
- Load testing (concurrent notebook users)
- Edge case testing (rare items, market crashes)

But for MVP: **Manual validation is sufficient.**
