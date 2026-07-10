# 009 — Database Schema

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft  
**Initial Database:** SQLite  

---

## 1. Purpose

This document defines the initial local data model for ARGO Rover OS. The database must support settings, vehicle profiles, trips, telemetry summaries, diagnostics, maintenance, service health, integrations, and media metadata without becoming tightly coupled to the 2014 Prius or Raspberry Pi.

The initial implementation should use SQLite because Rover OS is a single-node embedded system. The schema should remain portable enough to migrate to PostgreSQL or another database if Fleet or multi-node features are added later.

---

## 2. Database Principles

- Use SQLite for v0.x and v1.x single-vehicle deployments.
- Use migrations from the first implementation.
- Store timestamps in ISO 8601 UTC form.
- Use stable UUIDs or ULIDs for application-created records.
- Store measured values with explicit units in column names or metadata.
- Keep large binary files outside the database.
- Store file paths and metadata for dashcam clips, images, and exports.
- Avoid Prius-specific columns in generic tables.
- Allow raw source payloads where useful, but keep normalized columns for common queries.
- Enforce foreign keys.
- Use soft deletion only where recovery or audit history justifies it.

---

## 3. Migration Strategy

Recommended tooling:

- Drizzle ORM, Prisma, Kysely, or another TypeScript-friendly migration system.
- Numbered, immutable migration files.
- A `schema_migrations` table managed by the chosen tool.

Rules:

- Never edit a migration that has been released.
- Add a new migration for every schema change.
- Back up the database before destructive migrations.
- Keep migration execution automatic during controlled startup, but fail safely if migration fails.

---

## 4. Core Entity Overview

```text
vehicle_profiles
    ├── trips
    │    ├── trip_samples
    │    └── media_assets
    ├── diagnostic_sessions
    │    └── diagnostic_codes
    ├── maintenance_records
    └── telemetry_channels
         └── telemetry_samples

system_settings
service_health_events
integrations
system_events
```

---

## 5. `vehicle_profiles`

Stores one or more vehicle configurations. The first deployment will normally contain one active 2014 Prius profile.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID/ULID |
| `name` | TEXT | NOT NULL | Friendly name, e.g. Rover |
| `adapter_id` | TEXT | NOT NULL | e.g. `toyota-prius-gen3` |
| `make` | TEXT | NULL | Toyota |
| `model` | TEXT | NULL | Prius |
| `year` | INTEGER | NULL | 2014 |
| `trim` | TEXT | NULL | Exact trim when confirmed |
| `vin` | TEXT | NULL UNIQUE | Optional; sensitive data |
| `odometer_miles` | REAL | NULL | Last known odometer |
| `is_active` | INTEGER | NOT NULL DEFAULT 0 | Boolean |
| `metadata_json` | TEXT | NOT NULL DEFAULT '{}' | Adapter-specific metadata |
| `created_at` | TEXT | NOT NULL | UTC timestamp |
| `updated_at` | TEXT | NOT NULL | UTC timestamp |

Indexes:

- Unique active profile should be enforced in application logic or partial index.
- Index on `adapter_id`.

---

## 6. `system_settings`

Stores validated application settings.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `key` | TEXT | PK | Dot-delimited key |
| `value_json` | TEXT | NOT NULL | JSON serialized value |
| `value_type` | TEXT | NOT NULL | string, number, boolean, object, array |
| `scope` | TEXT | NOT NULL DEFAULT 'system' | system, vehicle, user, integration |
| `updated_at` | TEXT | NOT NULL | UTC timestamp |

Example keys:

- `display.theme`
- `display.brightnessMode`
- `audio.defaultVolume`
- `vehicle.activeProfileId`
- `argo.endpoint`
- `developer.enabled`

Secrets should not be stored here unless encrypted. Prefer environment files or a dedicated secrets mechanism.

---

## 7. `trips`

Stores trip-level summaries.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID/ULID |
| `vehicle_profile_id` | TEXT | FK NOT NULL | Vehicle used |
| `started_at` | TEXT | NOT NULL | UTC timestamp |
| `ended_at` | TEXT | NULL | UTC timestamp |
| `start_odometer_miles` | REAL | NULL | Odometer at start |
| `end_odometer_miles` | REAL | NULL | Odometer at end |
| `distance_miles` | REAL | NULL | Computed or measured |
| `duration_seconds` | INTEGER | NULL | Trip duration |
| `fuel_used_gallons` | REAL | NULL | Estimated or measured |
| `average_mpg` | REAL | NULL | Computed summary |
| `average_speed_mph` | REAL | NULL | Computed summary |
| `max_speed_mph` | REAL | NULL | Computed summary |
| `start_latitude` | REAL | NULL | Optional GPS |
| `start_longitude` | REAL | NULL | Optional GPS |
| `end_latitude` | REAL | NULL | Optional GPS |
| `end_longitude` | REAL | NULL | Optional GPS |
| `status` | TEXT | NOT NULL | active, completed, interrupted |
| `metadata_json` | TEXT | NOT NULL DEFAULT '{}' | Extra summary data |
| `created_at` | TEXT | NOT NULL | UTC timestamp |
| `updated_at` | TEXT | NOT NULL | UTC timestamp |

Indexes:

- `(vehicle_profile_id, started_at DESC)`
- `status`

---

## 8. `trip_samples`

Stores reduced-frequency trip samples suitable for route and summary analysis.

This table should not receive every high-frequency telemetry event. Use configurable sampling, such as every 1–5 seconds, and retain only useful fields.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INTEGER | PK AUTOINCREMENT | Internal ID |
| `trip_id` | TEXT | FK NOT NULL | Parent trip |
| `recorded_at` | TEXT | NOT NULL | UTC timestamp |
| `latitude` | REAL | NULL | GPS latitude |
| `longitude` | REAL | NULL | GPS longitude |
| `speed_mph` | REAL | NULL | Vehicle speed |
| `fuel_percent` | REAL | NULL | If available |
| `battery_12v` | REAL | NULL | 12V bus voltage |
| `hybrid_soc_percent` | REAL | NULL | If safely available |
| `coolant_temp_f` | REAL | NULL | If available |
| `metadata_json` | TEXT | NOT NULL DEFAULT '{}' | Other sampled values |

Indexes:

- `(trip_id, recorded_at)`

Retention should be configurable.

---

## 9. `telemetry_channels`

Defines generic telemetry channels exposed by adapters.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | Stable channel ID |
| `vehicle_profile_id` | TEXT | FK NOT NULL | Vehicle profile |
| `key` | TEXT | NOT NULL | e.g. `vehicle.speed` |
| `label` | TEXT | NOT NULL | Human label |
| `unit` | TEXT | NULL | mph, V, %, °F |
| `source` | TEXT | NOT NULL | obd2, can, gpio, mock |
| `data_type` | TEXT | NOT NULL | number, boolean, string |
| `is_enabled` | INTEGER | NOT NULL DEFAULT 1 | Boolean |
| `metadata_json` | TEXT | NOT NULL DEFAULT '{}' | PID/message details |

Unique index:

- `(vehicle_profile_id, key, source)`

---

## 10. `telemetry_samples`

Optional generalized time-series table for selected channels.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INTEGER | PK AUTOINCREMENT | Internal ID |
| `channel_id` | TEXT | FK NOT NULL | Telemetry channel |
| `trip_id` | TEXT | FK NULL | Associated trip |
| `recorded_at` | TEXT | NOT NULL | UTC timestamp |
| `number_value` | REAL | NULL | Numeric value |
| `text_value` | TEXT | NULL | String/boolean value |
| `quality` | TEXT | NOT NULL DEFAULT 'good' | good, stale, estimated, invalid |

Only channels explicitly configured for history should be written here. Live dashboard telemetry should remain event-driven.

Indexes:

- `(channel_id, recorded_at DESC)`
- `(trip_id, recorded_at)`

---

## 11. `diagnostic_sessions`

Represents an OBD or adapter diagnostic scan.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID/ULID |
| `vehicle_profile_id` | TEXT | FK NOT NULL | Vehicle |
| `started_at` | TEXT | NOT NULL | UTC timestamp |
| `completed_at` | TEXT | NULL | UTC timestamp |
| `adapter_type` | TEXT | NOT NULL | OBD/CAN adapter |
| `status` | TEXT | NOT NULL | running, completed, failed |
| `error_message` | TEXT | NULL | Failure detail |
| `raw_result_json` | TEXT | NULL | Optional source response |

---

## 12. `diagnostic_codes`

Stores codes discovered during a diagnostic session.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID/ULID |
| `diagnostic_session_id` | TEXT | FK NOT NULL | Parent scan |
| `code` | TEXT | NOT NULL | e.g. P0420 |
| `system` | TEXT | NULL | powertrain, chassis, body, network |
| `description` | TEXT | NULL | Human-readable explanation |
| `status` | TEXT | NOT NULL | active, pending, historic |
| `severity` | TEXT | NULL | info, advisory, warning, critical |
| `first_seen_at` | TEXT | NOT NULL | UTC timestamp |
| `last_seen_at` | TEXT | NOT NULL | UTC timestamp |
| `metadata_json` | TEXT | NOT NULL DEFAULT '{}' | Manufacturer-specific data |

Index:

- `(code, last_seen_at DESC)`

---

## 13. `maintenance_records`

Stores completed or scheduled maintenance.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID/ULID |
| `vehicle_profile_id` | TEXT | FK NOT NULL | Vehicle |
| `type` | TEXT | NOT NULL | oil, tires, coolant, EGR, etc. |
| `title` | TEXT | NOT NULL | Display name |
| `description` | TEXT | NULL | Notes |
| `status` | TEXT | NOT NULL | scheduled, due, completed, dismissed |
| `performed_at` | TEXT | NULL | UTC timestamp |
| `performed_odometer_miles` | REAL | NULL | Mileage at service |
| `due_at` | TEXT | NULL | Date-based due point |
| `due_odometer_miles` | REAL | NULL | Mileage-based due point |
| `cost_usd` | REAL | NULL | Optional cost |
| `provider` | TEXT | NULL | Shop/person |
| `metadata_json` | TEXT | NOT NULL DEFAULT '{}' | Parts, receipt path, etc. |
| `created_at` | TEXT | NOT NULL | UTC timestamp |
| `updated_at` | TEXT | NOT NULL | UTC timestamp |

Indexes:

- `(vehicle_profile_id, status, due_at)`
- `(vehicle_profile_id, due_odometer_miles)`

---

## 14. `service_health_events`

Stores significant service state changes rather than every heartbeat.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INTEGER | PK AUTOINCREMENT | Internal ID |
| `service_name` | TEXT | NOT NULL | e.g. camera-service |
| `previous_state` | TEXT | NULL | Prior state |
| `new_state` | TEXT | NOT NULL | starting, healthy, degraded, offline, error |
| `message` | TEXT | NULL | Human-readable detail |
| `metadata_json` | TEXT | NOT NULL DEFAULT '{}' | Device/error details |
| `recorded_at` | TEXT | NOT NULL | UTC timestamp |

Index:

- `(service_name, recorded_at DESC)`

---

## 15. `integrations`

Stores non-secret integration configuration.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | e.g. `argo-core`, `home-assistant` |
| `type` | TEXT | NOT NULL | Integration type |
| `name` | TEXT | NOT NULL | Friendly name |
| `endpoint` | TEXT | NULL | Base endpoint |
| `enabled` | INTEGER | NOT NULL DEFAULT 0 | Boolean |
| `auth_reference` | TEXT | NULL | Reference to secure secret storage |
| `config_json` | TEXT | NOT NULL DEFAULT '{}' | Non-secret config |
| `last_success_at` | TEXT | NULL | Last successful connection |
| `last_error_at` | TEXT | NULL | Last failure |
| `last_error_message` | TEXT | NULL | Failure detail |
| `updated_at` | TEXT | NOT NULL | UTC timestamp |

---

## 16. `media_assets`

Stores metadata for dashcam clips, snapshots, exports, and other generated files.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | TEXT | PK | UUID/ULID |
| `trip_id` | TEXT | FK NULL | Associated trip |
| `type` | TEXT | NOT NULL | dashcam_clip, snapshot, export |
| `camera_id` | TEXT | NULL | rear, front, cabin |
| `file_path` | TEXT | NOT NULL | Local path |
| `mime_type` | TEXT | NULL | Media type |
| `size_bytes` | INTEGER | NULL | File size |
| `started_at` | TEXT | NULL | Clip start |
| `ended_at` | TEXT | NULL | Clip end |
| `is_protected` | INTEGER | NOT NULL DEFAULT 0 | Exempt from normal deletion |
| `upload_state` | TEXT | NOT NULL DEFAULT 'local' | local, queued, uploading, synced, failed |
| `metadata_json` | TEXT | NOT NULL DEFAULT '{}' | GPS/event details |
| `created_at` | TEXT | NOT NULL | UTC timestamp |

Indexes:

- `(trip_id, started_at)`
- `(upload_state, created_at)`
- `(is_protected, created_at)`

---

## 17. `system_events`

Stores important application events for audit and troubleshooting.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | INTEGER | PK AUTOINCREMENT | Internal ID |
| `event_type` | TEXT | NOT NULL | e.g. `power.shutdown_requested` |
| `source` | TEXT | NOT NULL | Service/source |
| `severity` | TEXT | NOT NULL | debug, info, warning, error, critical |
| `message` | TEXT | NULL | Human-readable detail |
| `payload_json` | TEXT | NOT NULL DEFAULT '{}' | Structured event data |
| `recorded_at` | TEXT | NOT NULL | UTC timestamp |

Indexes:

- `(event_type, recorded_at DESC)`
- `(severity, recorded_at DESC)`

---

## 18. Data Retention

Recommended defaults:

- System events: 30–90 days.
- Service health events: 90 days.
- Detailed telemetry samples: configurable, 7–30 days.
- Trip summaries: indefinite unless deleted by user.
- Trip route samples: indefinite or compressed after a retention period.
- Diagnostic history: indefinite.
- Maintenance records: indefinite.
- Unprotected dashcam clips: storage-based rolling retention.
- Protected media: retained until manual deletion or successful archival policy.

Retention jobs should never block the driving UI.

---

## 19. Privacy and Sensitive Data

Potentially sensitive fields include:

- VIN.
- GPS coordinates.
- Trip history.
- Home/work destinations.
- Integration endpoints.
- Dashcam footage.

Requirements:

- Do not expose the database over the network.
- Restrict file permissions.
- Avoid logging secrets.
- Allow trip and location history to be disabled.
- Allow export and deletion of user data.

---

## 20. Backup and Recovery

The database should support:

- Periodic SQLite online backups.
- Backup before migrations.
- Optional encrypted sync to ARGO Vault.
- Recovery from the most recent valid backup.

Do not copy a live SQLite database file without using a safe backup method.

---

## 21. Acceptance Criteria

The database design is acceptable for v0.1 when:

- Migrations create the required tables.
- Settings persist across restarts.
- A vehicle profile can be created and selected.
- Trips can be started, sampled, and completed.
- Diagnostic sessions and codes can be stored.
- Service health transitions can be recorded.
- Mock mode can populate the schema without vehicle hardware.
