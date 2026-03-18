# NEA MVP Schema — Operational Telemetry Domain

Schema Version: NEA MVP v1

**Related Documentation:**
- [MVP v1 Specification](../specification.md) - Overall requirements
- [Data Model Diagram](../architecture/data_model.md) - Visual schema overview
- [MVP v1 Roadmap](../roadmap.md) - Implementation timeline (M5)

---

# 1. Purpose

The Operational Telemetry Domain records the execution, status, and operational details of all data acquisition processes in NEA. These include API-based refresh operations and file-based data imports.

This domain provides the system with:

- Observability into data refresh operations
- Historical logs of refresh runs and request activity
- Operational status of each maintained dataset slice
- Debugging and auditing capability for failures and anomalies

The telemetry domain does **not** store the data itself. Instead, it records **how and when data acquisition processes operate**.

---

# 2. Conceptual Model

The telemetry system is built around a hierarchical model:

```text
source_instance
      │
      ├── source_sync_status (1:1 current state)
      │
      └── source_refresh_run (1:N execution history)
              │
              ├── api_request_log (1:N request logs)
              └── file_import_log (1:N file import logs)
```

Conceptually:

```text
Dataset Slice
    ↓
Refresh Runs
    ↓
Execution Operations
```

---

# 3. Core Concepts

## Source Instance

A **source instance** represents a specific dataset slice that NEA is responsible for keeping up to date.

Examples:

- Corporation assets for corporation X
- Market prices for region Y
- Static blueprint dataset imported from SDE

A source instance defines:

- the logical dataset
- the acquisition mechanism
- the scope (corporation or region)
- the refresh cadence

All telemetry records ultimately attach to a `source_instance`.

---

## Source Refresh Run

A refresh run represents **one attempt to refresh a source instance**.

Each run records:

- start time
- finish time
- execution status
- optional row-change metrics
- error information

Runs may produce multiple execution operations such as API calls or file imports.

---

## Request Logs

API-backed refreshes may involve multiple requests.

Each request log records:

- the request structure
- response metadata
- execution timing
- retry information
- error information

Request bodies are **not stored**, but request structure is preserved to allow request reconstruction.

---

## File Import Logs

File-backed sources, such as SDE imports, record file processing events.

Each file import log records:

- file location
- file hash
- processing timing
- optional row-change metrics

Multiple files may be processed within a single refresh run.

---

# 4. Security and Redaction

All telemetry logging must enforce strict secret redaction.

Any sensitive information appearing in request or response metadata must be replaced with:

```text
<REDACTED>
```

Redaction must occur **before data is persisted**.

Sensitive information includes, but is not limited to:

- authentication tokens
- session identifiers
- authorization headers
- cookies
- API keys

Request structure should remain intact while sensitive values are replaced.

---

# 5. Controlled Vocabulary

## Source Family

```text
eve_api
eve_sdk_import
other
```

| value | description |
|------|-------------|
| eve_api | data retrieved through EVE ESI API endpoints |
| eve_sdk_import | data imported from Static Data Export (SDE) or SDK-style dumps |
| other | any other data source mechanism |

---

## Run Status

```text
success
success_no_change
partial_failure
failure
skipped
other
```

| status | meaning |
|------|--------|
| success | run completed successfully and produced data changes |
| success_no_change | run completed successfully but resulted in no local data changes |
| partial_failure | run completed with some recoverable failures |
| failure | run failed to complete successfully |
| skipped | run intentionally skipped due to unchanged upstream data |
| other | unexpected or uncategorized run state |

---

## Endpoint Name

`endpoint_name` must be a controlled identifier representing the specific callable API interface.

Examples:

```text
esi_corporation_assets
esi_corporation_blueprints
esi_corporation_industry_jobs
esi_corporation_orders
esi_market_history
```

Endpoint names represent **transport interfaces**, not logical datasets.

---

# 6. Tables

## source_instance

Defines a refreshable dataset slice.

### Fields

```text
source_instance_id
source_name
source_family
corporation_id (nullable)
region_id (nullable)
is_enabled
refresh_interval_seconds (nullable)
source_description (nullable)
```

### Field Meaning

| field | description |
|------|-------------|
| source_instance_id | internal identifier for the dataset slice |
| source_name | logical dataset label |
| source_family | acquisition mechanism |
| corporation_id | corporation scope where applicable |
| region_id | region scope where applicable |
| is_enabled | enables or disables refresh operations |
| refresh_interval_seconds | scheduler cadence for refresh checks |
| source_description | optional descriptive text |

### Refresh Interval Semantics

| value | meaning |
|------|--------|
| integer | scheduled refresh interval |
| NULL | refresh cadence determined by process logic |

---

## source_sync_status

Maintains the **current operational state** of each source instance.

This table is strictly **1:1 with `source_instance`**.

### Fields

```text
source_instance_id
last_attempted_at
last_succeeded_at
next_refresh_due_at
last_refresh_run_id
consecutive_failure_count
last_error_message
```

### Semantics

- `success_no_change` counts as a successful run
- `last_error_message` is **not** cleared on success
- `next_refresh_due_at` must always be populated

---

## source_refresh_run

Represents one execution attempt to refresh a source instance.

### Fields

```text
refresh_run_id
source_instance_id
started_at
finished_at
status
rows_inserted (nullable)
rows_updated (nullable)
rows_deleted (nullable)
error_message (nullable)
notes (nullable)
```

### Row Change Metrics

Row-change metrics are **best-effort operational metrics**.

Rules:

- Record counts when the refresh process can determine them reliably
- Leave fields `NULL` if accurate counts are not available
- Do not estimate counts casually

### Notes Field

The `notes` field is a flexible container for additional run metadata that does not fit within the structured schema.

Content does not need to be human-readable.

---

## api_request_log

Records individual API request operations performed during a refresh run.

### Fields

```text
request_log_id
refresh_run_id
source_instance_id
endpoint_name
request_method
request_url_path
request_query
request_body
page_number (nullable)
request_started_at
request_finished_at
success
http_status_code
retry_count
duration_ms
response_record_count (nullable)
response_size_bytes (nullable)
response_headers
error_message (nullable)
```

### Storage Rules

- `request_query` and `request_body` should be stored as structured JSON where supported
- Fallback to text storage if JSON fields are unavailable
- Full response headers are stored
- Response body is **not** stored

---

## file_import_log

Records file import operations within a refresh run.

### Fields

```text
file_import_log_id
refresh_run_id
source_instance_id
file_path
file_hash
file_modified_at
process_started_at
process_finished_at
success
rows_read (nullable)
rows_inserted (nullable)
rows_updated (nullable)
rows_deleted (nullable)
error_message (nullable)
```

### Multi-File Support

A single refresh run may process multiple files.

Multiple `file_import_log` records may therefore reference the same `refresh_run_id`.

---

# 7. Design Principles

## Dataset-Oriented Sources

`source_name` identifies the logical dataset being refreshed and must **not** encode the acquisition mechanism.

Example:

```text
source_name = blueprints
source_family = eve_sdk_import
```

---

## Stable Source Identity

`source_instance` provides a stable root identity for each refreshable dataset slice.

All operational telemetry attaches to this identity.

---

## Flexible Logging

Operational telemetry allows the use of `other` categories and flexible fields such as `notes` to accommodate edge cases and unforeseen scenarios.

---

## Operational Transparency

The telemetry domain exists to ensure that every refresh process can be audited, debugged, and understood through recorded operational data.
