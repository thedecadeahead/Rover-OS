# 026 — Logging and Observability

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft  

---

## 1. Purpose

This document defines how Rover OS records logs, reports service health, exposes diagnostics, and supports troubleshooting in development, bench, and vehicle modes.

Observability must help diagnose failures without overwhelming storage, leaking secrets, or distracting the driver.

---

## 2. Goals

Rover OS observability should:

- Make subsystem failures easy to identify.
- Preserve enough context to reproduce problems.
- Use structured logs.
- Rotate and retain logs safely.
- Separate developer detail from driver-facing alerts.
- Record service health transitions.
- Support export for remote analysis.
- Continue operating when ARGO Core is offline.

---

## 3. Logging Stack

Recommended initial stack:

- Structured JSON logs.
- `pino` or equivalent Node.js logger.
- `journald` through systemd in vehicle mode.
- Rotating application log files only where needed.
- SQLite for selected health and system events.

Console-friendly pretty logging may be enabled only in development mode.

---

## 4. Log Levels

Supported levels:

- `trace`: highly detailed development diagnostics.
- `debug`: developer-focused state and flow details.
- `info`: normal lifecycle events.
- `warn`: degraded but recoverable conditions.
- `error`: failed operation or unavailable subsystem.
- `fatal`: condition requiring process termination or restart.

Production default should be `info`.

---

## 5. Standard Log Fields

Every structured log should include, when applicable:

```json
{
  "timestamp": "2026-07-10T03:30:00.000Z",
  "level": "info",
  "service": "vehicle-service",
  "message": "Vehicle adapter connected",
  "eventType": "vehicle.connection_changed",
  "correlationId": "01J2ROVER...",
  "vehicleProfileId": "...",
  "adapterId": "toyota-prius-gen3"
}
```

Additional useful fields:

- Device path.
- Camera ID.
- Trip ID.
- API route.
- HTTP status.
- Error code.
- Duration milliseconds.
- Retry count.

---

## 6. Sensitive Data Rules

Logs must not contain:

- API tokens.
- Passwords.
- Full authorization headers.
- Private keys.
- Raw Home Assistant tokens.
- Full VIN unless explicitly required for a diagnostic export.
- Precise GPS history at normal info level.
- Unredacted phone identifiers.

Use redaction middleware for known secret fields.

---

## 7. Service Health

Every service must expose current health:

- `starting`
- `healthy`
- `degraded`
- `offline`
- `error`

A health transition should publish an event and may be persisted.

Health must include:

- Service name.
- Current state.
- Human-readable message.
- Last successful activity.
- Last error.
- Optional dependency status.

---

## 8. Health Endpoints

Recommended endpoints:

```http
GET /api/system/status
GET /api/system/health
GET /api/system/health/:service
```

The aggregate endpoint should distinguish:

- Core service failure.
- Optional integration failure.
- Hardware unavailable.
- Internet unavailable.

ARGO Core being offline must not mark the whole Rover system unhealthy.

---

## 9. Metrics

Initial metrics may be collected in memory and exposed locally.

Useful metrics:

- Process uptime.
- CPU temperature.
- CPU load.
- Memory usage.
- Disk usage.
- Database size.
- WebSocket client count.
- Event rate.
- Dropped event count.
- API request duration.
- Camera frame/stream health.
- OBD polling success rate.
- Service restart count.
- Startup duration.

A future Prometheus-compatible endpoint may be added, but is not required for v0.1.

---

## 10. Driver-Facing Alerts vs Logs

Logs are not driver notifications.

Driver-facing alerts should be created only for actionable or important states.

Examples:

- Rear camera unavailable while reversing.
- Storage nearly full.
- OBD disconnected.
- Low 12V voltage.
- Shutdown pending.

Repeated identical failures should be deduplicated.

---

## 11. Log Rotation and Retention

Recommended defaults:

- Journald size cap configured for vehicle storage.
- Application logs retained 14–30 days.
- Critical events retained longer in SQLite.
- Debug logs disabled by default in production.
- Old logs removed before media or database storage is endangered.

Rover OS should warn when free storage drops below configured thresholds.

---

## 12. Crash Reporting

On process crash, Rover OS should record:

- Service name.
- Exit code or signal.
- Last known health state.
- Recent relevant log lines.
- Restart count.
- Build version.
- Platform information.

Crash reports should remain local unless the user enables ARGO sync.

---

## 13. Diagnostic Bundles

Developer mode should allow export of a diagnostic bundle containing:

- System version.
- Hardware summary.
- Service health.
- Recent logs.
- Configuration with secrets removed.
- Database schema version.
- USB device list.
- Storage and temperature summary.

The bundle must exclude secrets and private trip/media data unless explicitly selected.

---

## 14. Development Event Inspector

The developer UI should support:

- Live event list.
- Filter by event type or source.
- Pause/resume.
- Payload inspection.
- Copy sanitized payload.
- Service health timeline.

This interface should be hidden in normal vehicle mode.

---

## 15. ARGO Integration

When enabled, Rover may send summarized observability data to ARGO Core.

Allowed examples:

- Service health summary.
- Update status.
- Storage warnings.
- Crash summary.

Raw logs and precise location data should require explicit opt-in.

---

## 16. Acceptance Criteria

Observability is acceptable for v0.1 when:

- All backend services use structured logs.
- Secrets are redacted.
- Service health is available through API and events.
- Logs rotate safely.
- Frontend shows meaningful degraded states.
- A local diagnostic export can be generated.
