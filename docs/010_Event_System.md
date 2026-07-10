# 010 — Event System

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft  
**Primary Transport:** WebSocket  

---

## 1. Purpose

This document defines the Rover OS event system. The event system distributes live telemetry, service health changes, vehicle state changes, camera status, media status, power events, integration status, and user-interface notifications.

The event system must keep the frontend responsive without coupling it directly to hardware drivers or polling every backend endpoint continuously.

---

## 2. Event System Goals

The event system should:

- Deliver live state to the touchscreen UI.
- Decouple hardware adapters from presentation logic.
- Support both real and mock hardware.
- Continue functioning when individual services are degraded.
- Provide typed, versioned event payloads.
- Allow new subscribers without modifying event producers.
- Avoid using the database as a real-time message bus.
- Make debugging and replay practical.

---

## 3. Initial Architecture

The v0.1 event system should run inside the Node.js backend process.

```text
Hardware / Mock Adapter
        ↓
Backend Service
        ↓
Internal Event Bus
        ├── State Store
        ├── Persistence Rules
        ├── Logging
        └── WebSocket Gateway
                 ↓
             Frontend
```

The first implementation does not require Kafka, MQTT, Redis, NATS, or another external broker.

A future multi-process or Fleet implementation may add an external broker behind the same event contracts.

---

## 4. Internal and External Events

### 4.1 Internal Events

Internal events are exchanged between backend services.

Examples:

- Raw adapter data.
- Device connection changes.
- Reverse trigger activation.
- Ignition state changes.
- Shutdown preparation stages.

Internal events may contain implementation detail that should not be exposed to the UI.

### 4.2 External Events

External events are normalized and safe for frontend consumption.

Examples:

- `vehicle.telemetry`
- `vehicle.state_changed`
- `camera.status`
- `service.health`
- `power.shutdown_pending`

The WebSocket gateway must only expose approved external event types.

---

## 5. Event Envelope

All external events must use a common envelope.

```ts
export interface RoverEvent<TPayload = unknown> {
  id: string;
  type: string;
  version: number;
  timestamp: string;
  source: string;
  correlationId?: string;
  payload: TPayload;
}
```

Example:

```json
{
  "id": "01J2ROVER8N4KM9D3G6Q5W1A2B",
  "type": "vehicle.telemetry",
  "version": 1,
  "timestamp": "2026-07-10T03:30:00.000Z",
  "source": "vehicle-service",
  "payload": {
    "speedMph": 42,
    "gear": "D",
    "fuelPercent": 68,
    "battery12v": 13.9
  }
}
```

---

## 6. Required Envelope Fields

### `id`

Unique event identifier, preferably UUID or ULID.

### `type`

Dot-delimited event name.

### `version`

Integer payload contract version for the event type.

### `timestamp`

UTC ISO 8601 time when the event was created.

### `source`

Service or adapter that published the normalized event.

### `correlationId`

Optional identifier linking events created by the same request or workflow.

### `payload`

Validated event-specific content.

---

## 7. Naming Convention

Event names should use lowercase dot notation.

Pattern:

```text
<domain>.<event>
```

Examples:

- `vehicle.telemetry`
- `vehicle.reverse_changed`
- `service.health`
- `camera.status`
- `media.state`
- `network.status`
- `power.shutdown_pending`
- `settings.updated`

Avoid names tied to implementation classes or hardware models.

Bad:

- `elm327DataReceived`
- `pi_gpio_17_changed`

Good:

- `diagnostics.telemetry`
- `vehicle.reverse_changed`

---

## 8. Core Event Catalog

### 8.1 Vehicle Events

- `vehicle.telemetry`
- `vehicle.connection_changed`
- `vehicle.state_changed`
- `vehicle.reverse_changed`
- `vehicle.ignition_changed`
- `vehicle.trip_started`
- `vehicle.trip_updated`
- `vehicle.trip_completed`
- `vehicle.alert`

### 8.2 Diagnostics Events

- `diagnostics.connection_changed`
- `diagnostics.telemetry`
- `diagnostics.scan_started`
- `diagnostics.scan_completed`
- `diagnostics.codes_changed`
- `diagnostics.error`

### 8.3 Camera Events

- `camera.status`
- `camera.active_changed`
- `camera.recording_started`
- `camera.recording_stopped`
- `camera.storage_warning`
- `camera.error`

### 8.4 Media Events

- `media.state`
- `media.source_changed`
- `media.volume_changed`
- `media.error`

### 8.5 Power Events

- `power.status`
- `power.ignition_changed`
- `power.low_voltage`
- `power.shutdown_requested`
- `power.shutdown_pending`
- `power.shutdown_cancelled`

### 8.6 Service Events

- `service.health`
- `service.started`
- `service.stopped`
- `service.error`

### 8.7 Network and Integration Events

- `network.status`
- `argo.status`
- `argo.sync_started`
- `argo.sync_completed`
- `argo.sync_failed`
- `home_assistant.status`

### 8.8 Settings and System Events

- `settings.updated`
- `system.notification`
- `system.storage_warning`
- `system.update_available`
- `system.update_progress`

---

## 9. Telemetry Event Design

High-frequency telemetry should use one normalized aggregate event rather than many tiny events.

Example payload:

```ts
export interface VehicleTelemetryPayload {
  vehicleProfileId: string;
  connected: boolean;
  speedMph?: number;
  gear?: string;
  fuelPercent?: number;
  rangeMiles?: number;
  battery12v?: number;
  hybridSocPercent?: number;
  coolantTempF?: number;
  engineRpm?: number;
  odometerMiles?: number;
  latitude?: number;
  longitude?: number;
  quality: Record<string, "good" | "stale" | "estimated" | "invalid">;
}
```

Rules:

- Omit unavailable values rather than inventing zeroes.
- Include data-quality metadata for uncertain readings.
- Frontend must tolerate partial payloads.
- Publish only as frequently as the UI needs.

Recommended initial dashboard frequency: 2–5 updates per second.

Historical storage may use a lower sample rate.

---

## 10. State Events vs. Notifications

A state event reports current system state.

Example:

- `camera.status`

A notification requests user attention.

Example:

- `system.notification`

A disconnected camera may produce both:

1. `camera.status` with state `offline`.
2. `system.notification` if the rear camera is required and reverse is selected.

The frontend should not infer all notifications from raw state events.

---

## 11. Service Health Events

Every backend service should publish health transitions.

```ts
export interface ServiceHealthPayload {
  service: string;
  state: "starting" | "healthy" | "degraded" | "offline" | "error";
  message?: string;
  details?: Record<string, unknown>;
}
```

Publish only when state or meaningful detail changes. Do not flood the event bus with identical heartbeats.

---

## 12. Subscription Model

The first frontend may subscribe to all approved events over one WebSocket connection.

Future protocol messages may support filters:

```json
{
  "action": "subscribe",
  "types": [
    "vehicle.telemetry",
    "service.health",
    "media.state"
  ]
}
```

The server should always permit a system-critical event channel for shutdown, critical alerts, and reconnect state.

---

## 13. Initial Snapshot and Reconnection

WebSocket events alone are insufficient because a newly connected UI has no prior state.

On startup or reconnection, the frontend should:

1. Connect to WebSocket.
2. Request current snapshots through REST.
3. Apply snapshot state.
4. Process subsequent events.

The backend may later provide a single bootstrap endpoint:

```http
GET /api/bootstrap
```

The UI must reconnect automatically after WebSocket loss.

Recommended reconnect strategy:

- Immediate first retry.
- Exponential backoff.
- Maximum delay around 30 seconds.
- Visible degraded connection state.

---

## 14. Ordering and Idempotency

Events from one source should preserve publication order where practical.

Consumers must not assume perfect ordering across multiple services.

For state events:

- Use timestamps.
- Ignore clearly older state than the currently applied state.
- Make handlers idempotent.

Command workflows may use `correlationId` to connect request, progress, and completion events.

---

## 15. Persistence Rules

Most live events should not be written directly to the database.

Persist:

- Service health transitions.
- Critical alerts.
- Trip summaries.
- Configured telemetry samples.
- Diagnostic scan results.
- Shutdown and startup events.
- Media recording metadata.

Do not persist by default:

- Every volume update.
- Every UI navigation event.
- Every high-frequency telemetry frame.
- Repeated identical healthy heartbeats.

---

## 16. Backpressure and Slow Consumers

The UI may freeze, reconnect, or process events slowly.

The backend should:

- Avoid unbounded per-client queues.
- Drop superseded telemetry state when necessary.
- Preserve critical alerts and shutdown events.
- Disconnect clients that cannot keep up.
- Log abnormal client behavior without crashing services.

Telemetry is state-like and may be coalesced. Critical events are not disposable.

---

## 17. Validation

Every external event payload must be validated before publication.

Recommended approach:

- Define TypeScript interfaces.
- Define matching Zod schemas.
- Validate adapter data before normalization.
- Reject or quarantine invalid payloads.
- Publish a service error event when appropriate.

---

## 18. Mock and Replay Support

The event system should support development replay.

Capabilities should include:

- Generated mock telemetry.
- Scripted reverse and ignition changes.
- Scripted service failures.
- Replay from newline-delimited JSON event files.
- Adjustable playback speed.

Replay makes UI development and regression testing possible without the vehicle.

---

## 19. Debugging

Developer mode should provide an event inspector showing:

- Event type.
- Timestamp.
- Source.
- Payload.
- Validation status.
- Correlation ID.

The inspector should be disabled or hidden in normal vehicle mode.

Structured event logs should redact secrets and sensitive integration data.

---

## 20. Future Broker Compatibility

The internal event API should make it possible to replace the in-process bus later.

Potential future transports:

- MQTT for ARGO node messaging.
- NATS for service messaging.
- Redis Streams.
- WebSocket relay through ARGO Core.

Core event contracts should remain transport-independent.

---

## 21. Acceptance Criteria

The event system is acceptable for v0.1 when:

- Mock telemetry reaches the frontend over WebSocket.
- Events use the common envelope.
- Payloads are typed and validated.
- Service health transitions are delivered.
- The frontend reconnects automatically.
- Current state can be restored through REST snapshots.
- One failing event handler does not crash the backend.
