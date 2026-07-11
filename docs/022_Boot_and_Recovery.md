# 022 — Boot and Recovery

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft  

## 1. Purpose

This document defines startup, kiosk launch, service recovery, degraded operation, maintenance access, and recovery after crashes or power loss.

## 2. Boot Priorities

Rover OS should provide useful local functions before optional online services.

Priority order:

1. Operating system and storage.
2. Backend API and event bus.
3. Touchscreen UI.
4. Power and vehicle-state services.
5. Camera and audio services.
6. OBD-II and optional integrations.
7. ARGO, weather, and other online features.

Internet availability must not block the dashboard.

## 3. Target Boot Sequence

```text
Power applied
  → Linux kernel and filesystem
  → system services
  → Rover backend
  → health endpoint available
  → kiosk session
  → Chromium opens local UI
  → bootstrap snapshot
  → dashboard
  → optional services finish connecting
```

## 4. systemd Services

Recommended units:

- `rover-backend.service`
- `rover-kiosk.service`
- `rover-power.service` or backend-integrated power module
- `rover-watchdog.service`
- Optional camera/media helper units

Dependencies should be explicit. Kiosk startup should wait for the local backend health endpoint, with a bounded timeout and visible fallback screen.

## 5. Boot Screen

The boot screen should show:

- Rover branding.
- Current phase.
- A concise degraded/error message if startup stalls.
- A maintenance-access gesture or key sequence.

It should not expose raw Linux console output during normal operation.

## 6. Service Restart Policy

Services should use bounded restart policies.

Recommended behavior:

- Restart on unexpected failure.
- Apply increasing delay after repeated failures.
- Stop rapid restart loops.
- Record restart count and last error.
- Keep unrelated services running.

## 7. Frontend Recovery

Chromium should reopen after a crash.

The frontend should:

- Reconnect to WebSocket.
- Reload snapshots.
- Restore the last safe route where appropriate.
- Default to dashboard after uncertain recovery.
- Allow reverse-camera override during degraded states if the camera path remains operational.

## 8. Backend Recovery

On backend restart:

- Open and validate the database.
- Run pending safe migrations.
- Recover interrupted trip state.
- Mark stale services offline.
- Reconnect adapters independently.
- Publish a fresh system snapshot.

An active trip found after an unexpected restart should be marked `interrupted` or resumed only through documented rules.

## 9. Unexpected Power Loss

Mitigations:

- Use NVMe/SSD rather than high-write microSD.
- Keep write frequency controlled.
- Use SQLite WAL and safe transaction settings.
- Flush important state during shutdown.
- Run filesystem/database checks after suspicious shutdown.
- Maintain recent database backups.

## 10. Degraded Modes

Rover OS should distinguish:

- Backend unavailable.
- Database unavailable.
- Vehicle adapter unavailable.
- Camera unavailable.
- Audio unavailable.
- Network unavailable.
- ARGO unavailable.

A noncritical failure must not create a blank screen.

## 11. Safe Mode

Safe mode should start a minimum configuration:

- Local backend.
- Dashboard/status UI.
- Settings and logs.
- Mock or disabled vehicle adapter.
- No third-party plugins.
- No remote integrations.
- No automatic update.

Safe mode may activate after repeated failed boots or through a maintenance command.

## 12. Maintenance Access

Maintenance access should support:

- Local keyboard shortcut or touch gesture.
- SSH only when explicitly enabled.
- Service status viewer.
- Log export.
- Network configuration.
- Reboot and shutdown.
- Safe-mode selection.

Maintenance access must not be casually exposed while driving.

## 13. Recovery Media

The project should eventually provide:

- Reproducible installation script or image.
- Documented NVMe replacement process.
- Database restore process.
- Configuration backup/export.
- Known-good release rollback.

## 14. Health Checks

Backend health endpoints:

- `/health/live` — process is alive.
- `/health/ready` — essential services are ready.
- `/api/system/status` — detailed application status.

The kiosk should rely on readiness, not merely an open TCP port.

## 15. Watchdog

A watchdog may monitor:

- Backend health.
- Kiosk process.
- Event loop responsiveness.
- Storage health.
- Vehicle-controller heartbeat.

Automatic reboot should be a last resort and rate-limited.

## 16. Rollback

Application updates should retain at least one previous known-good version.

Rollback should preserve:

- User data.
- Compatible configuration.
- Database backup from before migration.

## 17. Acceptance Criteria

The boot/recovery design is acceptable when:

- The system boots without network access.
- Backend and kiosk restart independently.
- Mock hardware failures produce degraded UI, not a crash.
- Unexpected backend restart restores usable state.
- Repeated failure enters safe mode or stops reboot looping.
- A documented recovery path exists for storage or application failure.
