# 003 — System Architecture

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft  
**Initial Runtime Target:** Raspberry Pi 5  

---

## 1. Purpose

This document defines the high-level system architecture for ARGO Rover OS. It explains how the frontend, backend services, hardware interfaces, vehicle adapters, and ARGO integrations should be organized.

The architecture must support a working Raspberry Pi implementation while avoiding long-term dependence on Raspberry Pi-specific assumptions.

---

## 2. Architectural Principles

### 2.1 Hardware Agnostic

The system must be able to move from Raspberry Pi 5 to another Linux-capable machine with minimal changes.

Hardware-specific concerns should be isolated in adapters, drivers, services, or configuration files.

### 2.2 Vehicle Agnostic

Prius-specific integration must not be baked into the core application.

The system should use a vehicle adapter layer.

Example adapters:

- Generic OBD-II adapter.
- Toyota Prius Gen 3 adapter.
- Future Honda Element adapter.
- Mock vehicle adapter.

### 2.3 Service Oriented

Major functions should be separate services or modules.

Examples:

- Vehicle service.
- Diagnostics service.
- Camera service.
- Audio service.
- Navigation service.
- Settings service.
- ARGO service.
- Update service.

### 2.4 Local First

Local functionality must not depend on ARGO Core, internet, cloud services, or remote APIs.

### 2.5 Mockable

Every hardware-facing service should have a mock mode.

Mock mode enables:

- Desktop development.
- Automated testing.
- UI development before hardware purchase.
- Claude-assisted implementation without physical devices.

---

## 3. High-Level Architecture

```text
+-------------------------------------------------------------+
|                         Rover UI                            |
|              React + TypeScript + Kiosk Browser             |
+-----------------------------+-------------------------------+
                              |
                              | HTTP / WebSocket
                              |
+-----------------------------v-------------------------------+
|                       Rover Backend                         |
|             Node.js Services + Local API Gateway             |
+-----+-----------+----------+----------+----------+----------+
      |           |          |          |          |
      |           |          |          |          |
      v           v          v          v          v
 Vehicle      Camera      Audio     Settings     ARGO
 Service      Service     Service    Service     Service
      |
      v
 Hardware Abstraction Layer
      |
      v
 Vehicle Adapter / OBD-II / CAN / Microcontroller
```

---

## 4. Runtime Layers

### 4.1 Operating System Layer

The host operating system should be Linux.

Initial target:

- Raspberry Pi OS or another stable Pi-compatible Linux distribution.

Future targets:

- Debian.
- Ubuntu Server.
- Fedora IoT.
- NixOS.
- Other Linux distributions suitable for embedded or appliance-like operation.

The OS layer is responsible for:

- Device drivers.
- Networking.
- Bluetooth.
- USB device access.
- Display output.
- Audio stack.
- Service supervision.
- Filesystem mounting.
- Time synchronization.

### 4.2 Application Runtime Layer

The application runtime should include:

- Node.js backend runtime.
- Browser-based frontend runtime.
- Local database.
- System service manager.

Recommended early implementation:

- Node.js with TypeScript.
- React with TypeScript.
- Vite for frontend development.
- SQLite for local persistence.
- WebSocket for live telemetry.

### 4.3 Presentation Layer

The presentation layer is the Rover UI.

It should run in Chromium kiosk mode for early versions.

The UI should not expose the user to browser controls.

Responsibilities:

- Dashboard.
- Navigation screen.
- Media screen.
- Cameras screen.
- Diagnostics screen.
- Maintenance screen.
- Home Assistant screen.
- Projects screen.
- Settings screen.

### 4.4 API Layer

The API layer is the boundary between frontend and backend.

It should expose:

- REST endpoints for request/response operations.
- WebSocket channels for live telemetry and events.

Examples:

- `GET /api/system/status`
- `GET /api/vehicle/status`
- `GET /api/settings`
- `POST /api/settings`
- `GET /api/cameras`
- `POST /api/media/play`
- `WS /api/events`

### 4.5 Service Layer

The service layer contains backend modules for each major function.

Services should be independently testable.

Services should communicate through clear interfaces rather than importing hardware-specific logic directly.

### 4.6 Hardware Abstraction Layer

The hardware abstraction layer provides normalized interfaces for physical devices.

It should hide implementation details from the rest of the system.

Examples:

- A camera source may be USB capture, CSI camera, RTSP stream, or mock feed.
- Vehicle speed may come from OBD-II, CAN, GPS, or mock data.
- Steering wheel buttons may come from CAN, USB keyboard events, serial messages, or mock events.

### 4.7 Vehicle Adapter Layer

The vehicle adapter layer translates vehicle-specific data into Rover-standard data structures.

For the Prius, this may include:

- OBD-II PIDs.
- Toyota-specific hybrid data if available.
- Reverse state.
- Ignition state.
- Steering wheel control mapping.
- Illumination/dimmer state.

---

## 5. Core Services

### 5.1 System Service

Responsible for:

- System health.
- Uptime.
- CPU temperature.
- Storage status.
- Memory status.
- Service status.
- Shutdown and restart commands.

### 5.2 Vehicle Service

Responsible for:

- Vehicle state.
- OBD-II connection.
- Telemetry normalization.
- Trip state.
- Adapter selection.

Standard vehicle state should include:

```json
{
  "connected": true,
  "speedMph": 0,
  "gear": "park",
  "fuelPercent": null,
  "rangeMiles": null,
  "rpm": null,
  "coolantTempF": null,
  "battery12v": null,
  "odometerMiles": null
}
```

### 5.3 Diagnostics Service

Responsible for:

- Diagnostic trouble codes.
- Fault status.
- Readiness monitors where available.
- Diagnostic history.
- Human-readable explanations.

### 5.4 Camera Service

Responsible for:

- Camera enumeration.
- Camera preview streams.
- Reverse camera priority.
- Recording support in later versions.
- Camera health state.

### 5.5 Audio Service

Responsible for:

- Audio output state.
- Volume control.
- Source selection.
- Media commands.
- Bluetooth or local playback integration.

### 5.6 Settings Service

Responsible for:

- Persistent configuration.
- Theme.
- Units.
- Vehicle adapter selection.
- Network references.
- ARGO endpoint configuration.
- Debug settings.

### 5.7 ARGO Service

Responsible for:

- ARGO Core connectivity.
- Sync status.
- Optional remote commands.
- Home Assistant references.
- Event publishing.

ARGO must be optional.

### 5.8 Navigation Service

Responsible for:

- Navigation launch state.
- Destination shortcuts.
- Future map integration.
- Cached location data where appropriate.

Early versions may use external web mapping rather than implementing full navigation internally.

### 5.9 Update Service

Responsible for:

- Version display.
- Update availability.
- Safe update workflow.
- Rollback planning in future versions.

---

## 6. Data Flow

### 6.1 Telemetry Flow

```text
Vehicle Hardware
      |
      v
OBD-II / CAN / Vehicle Controller
      |
      v
Vehicle Adapter
      |
      v
Vehicle Service
      |
      +--> Local Database
      |
      +--> WebSocket Event Bus
                  |
                  v
               Rover UI
```

### 6.2 Camera Flow

```text
Camera Hardware
      |
      v
Capture Device / Stream Source
      |
      v
Camera Service
      |
      v
Rover UI Camera View
```

### 6.3 Settings Flow

```text
Rover UI
   |
   v
Settings API
   |
   v
Settings Service
   |
   v
Local Database / Config File
```

### 6.4 ARGO Sync Flow

```text
Rover Services
      |
      v
ARGO Service
      |
      v
Network Layer
      |
      v
ARGO Core / Vault / Home Assistant
```

---

## 7. Local Database

The system should use a local database for structured state.

Initial recommendation:

- SQLite.

Data categories:

- Settings.
- Trip logs.
- Telemetry snapshots.
- Diagnostic events.
- Maintenance records.
- Camera metadata.
- Sync queue.

The database should be designed so Rover can operate offline and sync later.

---

## 8. Event Bus

The backend should provide a WebSocket event bus for live updates.

Example event types:

- `system.status.updated`
- `vehicle.telemetry.updated`
- `vehicle.connection.changed`
- `camera.reverse.enabled`
- `camera.reverse.disabled`
- `diagnostics.codes.updated`
- `media.state.updated`
- `argo.connection.changed`
- `settings.updated`

Events should use predictable JSON envelopes.

Example:

```json
{
  "type": "vehicle.telemetry.updated",
  "timestamp": "2026-07-09T12:00:00Z",
  "payload": {
    "speedMph": 42,
    "battery12v": 13.8
  }
}
```

---

## 9. Vehicle I/O Controller Concept

Rover OS should consider a small microcontroller as the boundary between the vehicle and main computer.

Possible controller hardware:

- RP2040.
- ESP32.
- STM32.
- Arduino-compatible automotive board.

Responsibilities may include:

- Ignition state detection.
- Reverse signal detection.
- Steering wheel button interpretation.
- Safe shutdown signal.
- Wake signal.
- Simple CAN message forwarding.

The Raspberry Pi should not be responsible for every low-level timing-sensitive vehicle signal.

---

## 10. Process Model

Early versions may run as a small number of processes:

```text
rover-backend
rover-frontend-kiosk
rover-watchdog
```

Future versions may split services into separate processes if needed.

The architecture should not require microservices at the beginning. It should require clean module boundaries.

---

## 11. Failure Handling

The system must handle subsystem failure gracefully.

Examples:

- If OBD-II disconnects, show telemetry unavailable.
- If ARGO Core is offline, local features continue.
- If a camera fails, other cameras remain available.
- If the database is temporarily locked, the UI should not crash.
- If internet is unavailable, cached/local views remain functional.

The UI should communicate degraded state clearly without overwhelming the driver.

---

## 12. Security Architecture

Default security posture:

- Local APIs bind to localhost.
- Remote APIs disabled unless explicitly configured.
- Secrets stored outside source code.
- Debug routes disabled in driving mode.
- Logs should avoid sensitive tokens.

Future remote access should require authentication and should be designed deliberately.

---

## 13. Development Architecture

The project should support three execution modes.

### 13.1 Desktop Development Mode

- Runs on laptop or desktop.
- Uses mock vehicle data.
- Uses mock camera feeds.
- Uses local browser.

### 13.2 Bench Hardware Mode

- Runs on Raspberry Pi.
- Uses real touchscreen.
- May use real OBD-II adapter.
- May use test cameras.
- Does not require vehicle installation.

### 13.3 Vehicle Mode

- Runs in the Prius.
- Uses production configuration.
- Uses vehicle power behavior.
- Uses real camera and OBD-II sources.

---

## 14. Recommended Initial Tech Stack

Frontend:

- React.
- TypeScript.
- Vite.
- CSS modules or Tailwind.
- WebSocket client.

Backend:

- Node.js.
- TypeScript.
- Express or Fastify.
- SQLite.
- WebSocket server.
- Zod or similar schema validation.

System:

- Linux.
- systemd.
- Chromium kiosk mode.
- Git-based deployment.

Testing:

- Vitest for frontend/backend units.
- Playwright for UI smoke tests where practical.
- Mock hardware adapters.

---

## 15. Architecture Success Criteria

The architecture is successful when:

- The UI can run with mock data.
- Hardware-specific code is isolated.
- Prius-specific code is isolated.
- The backend can be tested without the car.
- The same UI can run on laptop, Pi, or mini PC.
- The system can lose internet and continue working.
- The system can lose ARGO Core and continue working.
- The system can be extended without rewriting the foundation.
