# 008 — API Design

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft  
**Backend Target:** Node.js + TypeScript  

---

## 1. Purpose

This document defines the initial API design for ARGO Rover OS. The API exists to separate the touchscreen frontend from backend services, hardware interfaces, vehicle adapters, and integrations.

The frontend should interact with Rover OS through stable APIs, not direct hardware access.

---

## 2. API Principles

The API should be:

- Local-first.
- Typed.
- Validated.
- Stable across hardware changes.
- Friendly to mock data.
- Safe by default.
- Clear about degraded states.

The API should not expose unsafe vehicle control features.

---

## 3. API Types

Rover OS should use two API styles:

1. REST API for commands, settings, snapshots, and historical data.
2. WebSocket event stream for live telemetry and status updates.

REST is for asking or commanding.

WebSocket is for subscribing.

---

## 4. Base URL

Default local API base:

```text
http://localhost:3746/api
```

Default WebSocket endpoint:

```text
ws://localhost:3746/events
```

Port `3746` is provisional and may change.

---

## 5. Common Response Shape

REST responses should use a consistent shape.

Success:

```json
{
  "ok": true,
  "data": {}
}
```

Failure:

```json
{
  "ok": false,
  "error": {
    "code": "SERVICE_UNAVAILABLE",
    "message": "Diagnostics service is offline",
    "details": {}
  }
}
```

---

## 6. Common Error Codes

Recommended initial error codes:

- `BAD_REQUEST`
- `NOT_FOUND`
- `VALIDATION_ERROR`
- `SERVICE_UNAVAILABLE`
- `HARDWARE_UNAVAILABLE`
- `UNAUTHORIZED`
- `FORBIDDEN`
- `CONFLICT`
- `INTERNAL_ERROR`

---

## 7. System API

### 7.1 Get System Status

```http
GET /api/system/status
```

Returns:

```json
{
  "ok": true,
  "data": {
    "version": "0.1.0",
    "mode": "mock",
    "uptimeSeconds": 1234,
    "services": [
      {
        "name": "vehicle-service",
        "state": "healthy",
        "message": "Mock telemetry active"
      }
    ]
  }
}
```

### 7.2 Request Shutdown

```http
POST /api/system/shutdown
```

Behavior:

- Requires parked/developer permission in future versions.
- Broadcasts shutdown pending event.
- Initiates graceful shutdown.

---

## 8. Settings API

### 8.1 Get Settings

```http
GET /api/settings
```

### 8.2 Update Settings

```http
PATCH /api/settings
```

Example body:

```json
{
  "display": {
    "theme": "dark",
    "brightnessMode": "auto"
  }
}
```

Settings updates should be validated and persisted.

---

## 9. Vehicle API

### 9.1 Get Vehicle Snapshot

```http
GET /api/vehicle/snapshot
```

Example response:

```json
{
  "ok": true,
  "data": {
    "adapter": "mock-vehicle",
    "connected": true,
    "speedMph": 0,
    "gear": "P",
    "fuelPercent": 72,
    "rangeMiles": 318,
    "battery12v": 12.7,
    "updatedAt": "2026-07-09T22:30:00-05:00"
  }
}
```

### 9.2 Get Vehicle Capabilities

```http
GET /api/vehicle/capabilities
```

Returns which vehicle data is available from the active adapter.

Example:

```json
{
  "ok": true,
  "data": {
    "speed": true,
    "gear": false,
    "fuelPercent": true,
    "hybridBattery": false,
    "reverseState": true,
    "steeringWheelControls": false
  }
}
```

---

## 10. Diagnostics API

### 10.1 Get Diagnostics Status

```http
GET /api/diagnostics/status
```

### 10.2 Get Live Data

```http
GET /api/diagnostics/live
```

### 10.3 Get Trouble Codes

```http
GET /api/diagnostics/codes
```

### 10.4 Refresh Trouble Codes

```http
POST /api/diagnostics/codes/refresh
```

Clearing codes should not be part of the first implementation unless separately documented and confirmed safe.

---

## 11. Camera API

### 11.1 List Cameras

```http
GET /api/cameras
```

Example response:

```json
{
  "ok": true,
  "data": [
    {
      "id": "rear",
      "name": "Rear Camera",
      "state": "available",
      "streamUrl": "/api/cameras/rear/stream"
    }
  ]
}
```

### 11.2 Get Camera Status

```http
GET /api/cameras/:cameraId/status
```

### 11.3 Camera Stream

```http
GET /api/cameras/:cameraId/stream
```

The stream format may be MJPEG, WebRTC, HLS, or another implementation-specific format.

---

## 12. Media API

### 12.1 Get Media State

```http
GET /api/media/state
```

### 12.2 Media Commands

```http
POST /api/media/play
POST /api/media/pause
POST /api/media/next
POST /api/media/previous
POST /api/media/volume
```

Example volume body:

```json
{
  "level": 42
}
```

---

## 13. Network API

### 13.1 Get Network Status

```http
GET /api/network/status
```

Returns:

- Online/offline.
- Active interface if known.
- Signal details if known.
- Current network name if safe to show.

---

## 14. ARGO API

### 14.1 Get ARGO Status

```http
GET /api/argo/status
```

Returns:

- Core availability.
- Last sync.
- Pending sync count.
- Auth state.

### 14.2 Trigger Sync

```http
POST /api/argo/sync
```

ARGO sync must be optional and must not block local vehicle operation.

---

## 15. Maintenance API

### 15.1 Get Maintenance Items

```http
GET /api/maintenance
```

### 15.2 Add Maintenance Record

```http
POST /api/maintenance
```

### 15.3 Update Maintenance Item

```http
PATCH /api/maintenance/:itemId
```

Maintenance records may include:

- Oil change.
- Tire rotation.
- 12V battery replacement.
- EGR cleaning.
- Spark plugs.
- Coolant service.
- Transaxle fluid.

---

## 16. Trip API

### 16.1 Get Current Trip

```http
GET /api/trips/current
```

### 16.2 List Trips

```http
GET /api/trips
```

### 16.3 Get Trip Details

```http
GET /api/trips/:tripId
```

Trip data should be locally stored and optionally synced to ARGO Vault later.

---

## 17. WebSocket Event Envelope

All WebSocket events should use a common envelope.

```json
{
  "type": "vehicle.telemetry",
  "timestamp": "2026-07-09T22:30:00-05:00",
  "source": "vehicle-service",
  "payload": {}
}
```

Required fields:

- `type`
- `timestamp`
- `source`
- `payload`

---

## 18. Initial WebSocket Events

### 18.1 Vehicle Telemetry

```json
{
  "type": "vehicle.telemetry",
  "timestamp": "2026-07-09T22:30:00-05:00",
  "source": "vehicle-service",
  "payload": {
    "speedMph": 36,
    "gear": "D",
    "fuelPercent": 68,
    "battery12v": 13.9
  }
}
```

### 18.2 Service Health

```json
{
  "type": "service.health",
  "timestamp": "2026-07-09T22:30:00-05:00",
  "source": "system",
  "payload": {
    "service": "camera-service",
    "state": "degraded",
    "message": "Rear camera unavailable"
  }
}
```

### 18.3 Power Status

```json
{
  "type": "power.status",
  "timestamp": "2026-07-09T22:30:00-05:00",
  "source": "power-service",
  "payload": {
    "ignition": "off",
    "shutdownPending": true,
    "secondsRemaining": 30
  }
}
```

---

## 19. API Security

Default security posture:

- Bind to localhost.
- Do not expose remote control APIs by default.
- Store secrets server-side only.
- Require explicit configuration before LAN access.
- Require authentication for future remote access.

---

## 20. Versioning

API should be versioned once it stabilizes.

Initial development may use:

```text
/api/...
```

Future stable versions may use:

```text
/api/v1/...
```

Breaking changes should be documented.

---

## 21. Acceptance Criteria

The API design is acceptable for v0.1 when:

- The frontend can load dashboard data.
- Mock vehicle telemetry streams over WebSocket.
- Settings can be read and updated.
- Service health is visible.
- Camera and diagnostics APIs exist, even if mocked.
- API payloads are typed and validated.
