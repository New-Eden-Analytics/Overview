# ESI Endpoints for NEA MVP v1

**Reference:** <https://developers.eveonline.com/api-explorer>

**Authentication:** OAuth2 with required scopes (documented per endpoint below)

**Related Documentation:**

- [Corporation Domain Schema](../schema/corporation_domain.md) - Tables populated by these endpoints
- [Market Domain Schema](../schema/market_domain.md) - Market data tables
- [MVP v1 Roadmap](../roadmap.md) - When each endpoint is implemented

**Rate Limits:**

- Authenticated: 300 requests per minute per token
- Unauthenticated: 150 requests per minute per IP
- Error limit: 100 errors per minute (triggers temporary ban)

**Best Practices:**

- Respect cache headers (most endpoints cache for 5-30 minutes)
- Use If-None-Match with ETags to avoid unnecessary data transfer
- Handle 503 (service unavailable) with exponential backoff retry
- Handle 420 (error limit exceeded) by pausing requests

---

## Milestone 3: Corporation State Endpoints

### 1. Corporation Blueprints

**Endpoint:** `GET /v2/corporations/{corporation_id}/blueprints/`

**Purpose:** Fetch all blueprint instances owned by the corporation

**Required Scope:** `esi-corporations.read_blueprints.v1`

**Parameters:**

- `corporation_id` (path) - Your corporation ID
- `page` (query, optional) - Page number for pagination (default: 1)

**Response:** Array of blueprint objects

```json
[
  {
    "item_id": 1000000001,
    "type_id": 1234,
    "location_id": 60012345,
    "location_flag": "Hangar",
    "quantity": -2,
    "time_efficiency": 20,
    "material_efficiency": 10,
    "runs": -1
  }
]
```

**Notes:**

- `quantity = -1` means BPO (original)
- `quantity = -2` means BPC (copy)
- `runs = -1` for BPO (infinite runs)
- Paginated (1000 items per page)
- Cache: 3600 seconds (1 hour)

**Maps to:** `corp_blueprint_instance` table

---

### 2. Corporation Industry Jobs

**Endpoint:** `GET /v1/corporations/{corporation_id}/industry/jobs/`

**Purpose:** Fetch active and recent industry jobs

**Required Scope:** `esi-industry.read_corporation_jobs.v1`

**Parameters:**

- `corporation_id` (path) - Your corporation ID
- `include_completed` (query, optional) - Include completed jobs (default: false)

**Response:** Array of job objects

```json
[
  {
    "job_id": 12345,
    "installer_id": 98765,
    "facility_id": 60012345,
    "location_id": 60012345,
    "activity_id": 1,
    "blueprint_id": 1000000001,
    "blueprint_type_id": 1234,
    "blueprint_location_id": 60012345,
    "output_location_id": 60012345,
    "runs": 10,
    "cost": 100000,
    "licensed_runs": 100,
    "probability": 1.0,
    "product_type_id": 5678,
    "status": "active",
    "duration": 3600,
    "start_date": "2026-03-13T12:00:00Z",
    "end_date": "2026-03-13T13:00:00Z",
    "pause_date": null,
    "completed_date": null,
    "completed_character_id": null
  }
]
```

**Notes:**

- `activity_id = 1` is manufacturing
- `status` can be: "active", "paused", "ready", "delivered", "cancelled", "reverted"
- For MVP, filter on `status = "active"` and `activity_id = 1`
- Cache: 300 seconds (5 minutes)

**Maps to:** `corp_industry_job` table

---

### 3. Corporation Assets

**Endpoint:** `GET /v5/corporations/{corporation_id}/assets/`

**Purpose:** Fetch all corporation assets (for staging inventory tracking)

**Required Scope:** `esi-assets.read_corporation_assets.v1`

**Parameters:**

- `corporation_id` (path) - Your corporation ID
- `page` (query, optional) - Page number for pagination (default: 1)

**Response:** Array of asset objects

```json
[
  {
    "item_id": 1000000002,
    "type_id": 34,
    "location_id": 60012345,
    "location_type": "station",
    "location_flag": "CorpSAG1",
    "quantity": 1000,
    "is_singleton": false,
    "is_blueprint_copy": null
  }
]
```

**Notes:**

- Paginated (1000 items per page)
- Filter by `location_id` matching your staging location from `nea_config`
- `location_flag` indicates which hangar/container
- Cache: 3600 seconds (1 hour)

**Maps to:** `corp_staging_inventory` table (filtered by configured staging location)

---

### 4. Corporation Orders

**Endpoint:** `GET /v3/corporations/{corporation_id}/orders/`

**Purpose:** Fetch active market orders (to exclude items already for sale)

**Required Scope:** `esi-markets.read_corporation_orders.v1`

**Parameters:**

- `corporation_id` (path) - Your corporation ID
- `page` (query, optional) - Page number for pagination (default: 1)

**Response:** Array of order objects

```json
[
  {
    "order_id": 5678901,
    "type_id": 34,
    "region_id": 10000002,
    "location_id": 60003760,
    "range": "station",
    "is_buy_order": false,
    "price": 5.0,
    "volume_total": 1000000,
    "volume_remain": 500000,
    "issued": "2026-03-13T12:00:00Z",
    "state": "active",
    "min_volume": 1,
    "duration": 90,
    "escrow": 0.0,
    "wallet_division": 1
  }
]
```

**Notes:**

- Filter on `is_buy_order = false` (only sell orders matter for MVP)
- Filter on `state = "active"`
- Paginated (1000 items per page)
- Cache: 300 seconds (5 minutes)

**Maps to:** `corp_active_sell_order` table

---

## Milestone 4: Market Data Endpoints

### 5. Market History

**Endpoint:** `GET /v1/markets/{region_id}/history/`

**Purpose:** Fetch historical market data for pricing

**Required Scope:** None (public endpoint)

**Parameters:**

- `region_id` (path) - Region ID (e.g., 10000002 for The Forge/Jita)
- `type_id` (query) - Item type ID

**Response:** Array of history objects

```json
[
  {
    "date": "2026-03-12",
    "order_count": 150,
    "volume": 5000000,
    "highest": 5.5,
    "lowest": 4.8,
    "average": 5.0
  },
  {
    "date": "2026-03-13",
    "order_count": 148,
    "volume": 4800000,
    "highest": 5.6,
    "lowest": 4.9,
    "average": 5.1
  }
]
```

**Notes:**

- Returns up to 13 months of daily history
- For MVP, use average of last 7 days
- Must call once per type_id (can batch via multiple requests)
- Cache: 3600 seconds (1 hour)
- Consider filtering out low-volume items (e.g., volume < 10000)

**Maps to:** `market_price_snapshot` table

---

## Optional/Future Endpoints

These are mentioned in specs but not required for MVP:

### Corporation Locations (for resolving location names)

**Endpoint:** `POST /v2/corporations/{corporation_id}/assets/locations/`
**Use case:** Get human-readable names for location_ids

### Corporation Names (for resolving container names)

**Endpoint:** `POST /v2/corporations/{corporation_id}/assets/names/`
**Use case:** Get names for containers/hangars

### Universe Structures (for structure info)

**Endpoint:** `GET /v2/universe/structures/{structure_id}/`
**Use case:** Get structure names and details

---

## Authentication Setup

### Getting an OAuth Token

1. **Register application** at <https://developers.eveonline.com/applications>
   - Callback URL: <http://localhost> (for manual token flow)
   - Scopes needed:
     - `esi-corporations.read_blueprints.v1`
     - `esi-industry.read_corporation_jobs.v1`
     - `esi-assets.read_corporation_assets.v1`
     - `esi-markets.read_corporation_orders.v1`

2. **Get authorization code** (manual for MVP):
   - Visit: `https://login.eveonline.com/v2/oauth/authorize/?response_type=code&redirect_uri=http://localhost&client_id={client_id}&scope={scopes}`
   - User logs in and authorizes
   - Copy authorization code from redirect URL

3. **Exchange for access token**:

   ```bash
   curl -X POST https://login.eveonline.com/v2/oauth/token \
     -u "{client_id}:{client_secret}" \
     -d "grant_type=authorization_code&code={auth_code}"
   ```

4. **Use token in requests**:

   ```
   Authorization: Bearer {access_token}
   ```

5. **Token lifespan**: 20 minutes (must refresh)

**For MVP:** Manually get token, store in k8s Secret, refresh manually when expired.

**For production:** Implement refresh token flow (out of MVP scope).

---

## Error Handling

### Common Error Codes

- **200 OK** - Success
- **304 Not Modified** - Cache valid (when using If-None-Match)
- **400 Bad Request** - Invalid parameters
- **401 Unauthorized** - Missing/invalid token
- **403 Forbidden** - Token lacks required scope
- **404 Not Found** - Resource doesn't exist
- **420 Error Limited** - Too many errors, slow down
- **500 Internal Server Error** - ESI is having issues
- **502 Bad Gateway** - ESI is having issues
- **503 Service Unavailable** - ESI is having issues (retry)
- **504 Gateway Timeout** - ESI is having issues (retry)

### Retry Strategy

For 5xx errors:

1. Wait 1 second, retry
2. If fails, wait 2 seconds, retry
3. If fails, wait 4 seconds, retry
4. If fails 3 times, log error and continue (stale data is OK for MVP)

For 420:

1. Stop all requests for 60 seconds
2. Resume with reduced rate

---

## Rate Limiting Strategy

**Recommended approach:**

1. Track request count per minute
2. Stay under 200 requests/minute (buffer below 300 limit)
3. If approaching limit, pause and resume next minute
4. Log all API calls to `api_request_log` table

**For MVP:**

- Corporation endpoints: ~4 requests (blueprints, jobs, assets, orders) × pagination
- Market endpoint: N requests where N = number of unique items to price
- Expected: ~50-200 requests per refresh depending on data size

**Optimization:**

- Cache responses aggressively
- Don't refetch if ETags haven't changed
- Consider batching market history requests (fetch top 100 items only)

---

## Testing Strategy

### Manual Testing

1. Use ESI API explorer to test endpoints manually
2. Verify scope permissions are correct
3. Check response format matches documentation
4. Test pagination with your actual corporation data

### Integration Testing

1. Parse one blueprint response into database
2. Verify foreign keys to reference domain work
3. Check exclusion queries work with test data
4. Validate market price joins work correctly

---

## ESI Version Policy

- ESI uses versioned endpoints (v1, v2, v3, etc.)
- Versions are stable and don't change
- New versions are released when breaking changes occur
- **For MVP:** Use latest stable version documented above
- **Monitor:** ESI changelog at <https://github.com/esi/esi-issues> for deprecation notices

---

## Next Steps

1. Test authentication flow manually
2. Verify your corporation has the required roles (Director or appropriate roles)
3. Make test API calls using curl or Postman
4. Document your client_id, client_secret, and first token in k8s Secret
5. Proceed to M3 implementation with confidence
