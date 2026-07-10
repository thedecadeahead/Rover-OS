# 007 — Backend Architecture

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft  
**Initial Runtime:** Node.js + TypeScript  

---

## 1. Purpose

This document defines the backend architecture for ARGO Rover OS. The backend is responsible for hardware abstraction, local APIs, telemetry processing, settings persistence, service health, and integration with ARGO and other external systems.

The backend must allow the frontend to remain clean and hardware-agnostic.

---

## 2. Backend Goals

The backend should:

- Run on Raspberry Pi 5.
- Run on a development computer with mock hardware.
- Provide REST APIs for configuration and commands.
- Provide WebSocket events for live telemetry.
- Manage local settings and data storage.
- Abstract vehicle-specific hardware.
- Survive hardware disconnections gracefully.
- Support optional ARGO Core integration.
- Support future migration into separate services if needed.

---

## 3. Recommended Stack

Initial backend stack:

- Node.js.
- TypeScript.
- Fastify or Express.
- WebSocket server.
- SQLite.
- Zod or similar schema validation.
- pino or similar structured logging.
- systemd for process management in vehicle mode.

The first implementation may use a single backend process with modular internal services.

---

## 4. Backend Directory Structure

Recommended structure:

```text
backend/
├── src/
│   ├── index.ts
│   ├── config/
│   ├── api/
│   ├── services/
│   ├── adapters/
│   ├── hardware/
│   ├── database/
│   ├── events/
│   ├── logging/
│   └── types/
├── tests/
├── package.json
└── tsconfig.json
```

---

## 5. Core Backend Services

### 5.1 Settings Service

Responsibilities:

- Read settings.
- Update settings.
- Validate settings.
- Persist settings to SQLite.
- Broadcast setting changes.

Settings categories:

- Display.
- Audio.
- Vehicle.
- Network.
- ARGO.
- Cameras.
- Developer.

### 5.2 Vehicle Service

Responsibilities:

- Select active vehicle adapter.
- Normalize telemetry.
- Publish vehicle status events.
- Provide current vehicle snapshot.
- Track adapter health.

The vehicle service should not directly know how to talk to every hardware device. It should use adapters.

### 5.3 Diagnostics Service

Responsibilities:

- Manage OBD-II connection.
- Read standard live data.
- Read diagnostic trouble codes when supported.
- Store diagnostic history.
- Expose safe diagnostic APIs.

### 5.4 Camera Service

Responsibilities:

- Track configured cameras.
- Provide stream references.
- Report camera health.
- Support manual camera switching.
- Support reverse-triggered camera switching through vehicle events.

### 5.5 Media Service

Responsibilities:

- Track media state.
- Provide transport commands.
- Route volume commands.
- Integrate with steering wheel events.
- Support future local media and Bluetooth states.

### 5.6 Power Service

Responsibilities:

- Track ignition state.
- Track shutdown requests.
- Coordinate graceful shutdown.
- Publish power events.
- Interface with vehicle I/O controller where available.

### 5.7 Network Service

Responsibilities:

- Track online/offline state.
- Track Wi-Fi/hotspot connection where available.
- Provide network status to UI.
- Support future remote access configuration.

### 5.8 ARGO Service

Responsibilities:

- Track ARGO Core connection.
- Authenticate to ARGO services.
- Sync selected local data.
- Publish ARGO availability.
- Fail gracefully when ARGO is offline.

### 5.9 Logging Service

Responsibilities:

- Central structured logging.
- Log rotation configuration.
- Error collection.
- Service health records.
- Export for debugging.

---

## 6. Service Health Model

Every service should expose a health state.

Recommended states:

- `starting`
- `healthy`
- `degraded`
- `offline`
- `error`

Health object example:

```json
{
  "service": "diagnostics-service",
  "state": "degraded",
  "message": "OBD-II adapter disconnected",
  "updatedAt": "2026-07-09T22:30:00-05:00"
}
```

The dashboard should consume service health information.

---

## 7. Configuration

Configuration sources should be layered:

1. Defaults in code.
2. Local config file.
3. Environment variables.
4. User settings stored in database.

Secrets must not be committed to the repository.

Examples of secrets:

- ARGO API tokens.
- Home Assistant tokens.
- Remote tunnel credentials.

---

## 8. Local Persistence

SQLite should store:

- Settings.
- Vehicle profile.
- Service configuration.
- Trip summaries.
- Diagnostic history.
- Maintenance records.
- Event metadata.
- Camera clip metadata.

Large binary data such as video should be stored as files with database metadata.

---

## 9. Hardware Abstraction

Hardware access should be hidden behind interfaces.

Example interface categories:

- `VehicleAdapter`
- `ObdAdapter`
- `CameraAdapter`
- `AudioAdapter`
- `PowerAdapter`
- `InputAdapter`
- `GpsAdapter`

Each interface should have both real and mock implementations.

---

## 10. Mock Mode

Mock mode is required.

Mock mode should simulate:

- Vehicle telemetry.
- OBD connection.
- Camera availability.
- ARGO Core status.
- Network status.
- Power events.
- Alerts.

Mock mode enables development without the Prius or hardware bench.

---

## 11. Event Bus

The backend should publish WebSocket events for live data.

Event examples:

- `vehicle.telemetry`
- `vehicle.alert`
- `service.health`
- `camera.status`
- `media.status`
- `network.status`
- `power.status`
- `argo.status`
- `settings.updated`

Events should be typed and validated.

---

## 12. REST API

REST APIs should handle:

- Reading settings.
- Updating settings.
- Running commands.
- Fetching snapshots.
- Fetching historical records.
- Triggering maintenance actions.

REST should not be used for high-frequency telemetry; WebSocket events should handle live updates.

---

## 13. Error Handling

The backend must assume hardware fails.

Failure examples:

- USB device missing.
- Serial port unavailable.
- Camera disconnected.
- Database locked.
- ARGO Core offline.
- Network unavailable.

Rules:

- Log the error.
- Mark affected service degraded or offline.
- Keep other services running.
- Notify frontend through health events.
- Retry when appropriate.

---

## 14. Shutdown Handling

The backend must participate in graceful shutdown.

Shutdown procedure:

1. Receive shutdown request.
2. Broadcast `power.shutdown_pending`.
3. Stop high-write services.
4. Flush logs.
5. Close database.
6. Exit cleanly.
7. Allow systemd or power controller to finish shutdown.

---

## 15. Security

Initial security rules:

- Bind APIs to localhost by default.
- Require explicit configuration for LAN access.
- Do not expose control APIs publicly.
- Store secrets outside source control.
- Sanitize logs to avoid leaking tokens.
- Require authentication for future remote control.

---

## 16. Testing Requirements

Backend tests should cover:

- Settings validation.
- Mock vehicle telemetry.
- Service health transitions.
- WebSocket event payloads.
- REST API responses.
- Adapter error behavior.
- Database migrations.

Hardware-specific tests should be separated from unit tests.

---

## 17. Acceptance Criteria

The backend is acceptable for v0.1 when:

- It runs locally on a development computer.
- It starts without real hardware.
- It exposes settings APIs.
- It emits mock telemetry over WebSocket.
- It provides service health state.
- It persists settings to SQLite.
- It fails gracefully when mock services are disabled.

The backend is acceptable for v1.0 when:

- It runs reliably on the installed hardware.
- It handles shutdown cleanly.
- It survives hardware disconnects.
- It provides stable APIs for the frontend.
- It supports the active Prius vehicle adapter.
