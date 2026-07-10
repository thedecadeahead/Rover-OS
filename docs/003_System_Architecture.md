# 003 — System Architecture

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft  
**Primary Vehicle Target:** 2014 Toyota Prius  
**Initial Compute Target:** Raspberry Pi 5  

---

## 1. Purpose

This document defines the initial system architecture for ARGO Rover OS. It explains how the frontend, backend services, hardware interfaces, vehicle adapters, and ARGO integrations should be separated so the project can start on a Raspberry Pi but remain portable to more powerful hardware later.

Rover OS should be designed as a modular vehicle computing platform, not a single monolithic carputer script.

---

## 2. Architectural Goals

The architecture must support the following goals:

- Run on Raspberry Pi 5 for the first vehicle prototype.
- Run on a normal laptop for development with mock hardware.
- Allow migration to a mini PC or other edge computer later.
- Keep vehicle-specific code isolated from the core application.
- Keep hardware drivers isolated behind stable service APIs.
- Allow the UI to continue running if one hardware service fails.
- Support offline-first operation.
- Support optional ARGO Core and Home Assistant integration.
- Provide a safe, reliable boot and shutdown path for vehicle use.

---

## 3. High-Level Architecture

Rover OS is composed of five major layers:

1. User Interface Layer
2. Application Service Layer
3. Hardware Abstraction Layer
4. Vehicle Adapter Layer
5. Platform / Operating System Layer

```text
+--------------------------------------------------+
|                 User Interface                   |
|        React / TypeScript / Kiosk Browser        |
+--------------------------------------------------+
|              Application Services                |
|   API, WebSocket, Settings, Media, Diagnostics   |
+--------------------------------------------------+
|             Hardware Abstraction Layer           |
|     OBD, Camera, Audio, GPS, Power, Input        |
+--------------------------------------------------+
|               Vehicle Adapter Layer              |
|       Toyota Prius Gen 3 / Generic OBD-II        |
+--------------------------------------------------+
|                Platform Layer                    |
|       Linux, systemd, Chromium, Storage          |
+--------------------------------------------------+
```

---

## 4. User Interface Layer

### 4.1 Technology

The UI should be built with:

- React
- TypeScript
- Vite
- CSS modules, Tailwind, or another maintainable styling system
- WebSocket client for live updates
- REST client for request/response operations

The UI should run in Chromium kiosk mode on the in-car system.

### 4.2 UI Responsibilities

The UI is responsible for:

- Dashboard rendering.
- Navigation screen.
- Media controls.
- Camera views.
- Diagnostics display.
- Maintenance display.
- Settings interface.
- ARGO status display.
- Home Assistant panel.
- Error and degraded-state messaging.

### 4.3 UI Non-Responsibilities

The UI must not directly:

- Read OBD-II devices.
- Access CAN hardware.
- Manage GPIO.
- Control shutdown.
- Talk directly to vehicle wiring.
- Store secrets in frontend code.

All hardware access must pass through backend services.

---

## 5. Application Service Layer

The application service layer is the main backend for Rover OS.

Recommended initial runtime:

- Node.js
- TypeScript
- Express or Fastify for HTTP APIs
- WebSocket server for live events
- SQLite for local persistence

### 5.1 Core Services

Initial services should include:

- `dashboard-service`
- `settings-service`
- `vehicle-service`
- `diagnostics-service`
- `camera-service`
- `media-service`
- `network-service`
- `argo-service`
- `logging-service`
- `power-service`

These may begin as modules inside a single backend process, but the code should be organized so they can be separated later.

### 5.2 API Gateway

The backend should expose one local API surface to the frontend.

Default bind behavior:

- Bind to `localhost` during early development.
- Avoid exposing remote APIs unless explicitly configured.

API types:

- REST for commands and configuration.
- WebSocket for live telemetry and status.

---

## 6. Hardware Abstraction Layer

The hardware abstraction layer hides implementation details of physical devices.

It should expose stable interfaces such as:

- `getVehicleSnapshot()`
- `subscribeToTelemetry()`
- `getCameraStream(cameraId)`
- `setAudioVolume(level)`
- `getPowerState()`
- `requestShutdown()`

The application should not care whether data comes from real OBD-II hardware, a mock service, or a future CAN adapter.

---

## 7. Vehicle Adapter Layer

The vehicle adapter layer translates generic Rover concepts into vehicle-specific behavior.

Initial adapters:

- `generic-obd2`
- `toyota-prius-gen3`
- `mock-vehicle`

### 7.1 Generic OBD-II Adapter

The generic adapter should support standard OBD-II PIDs where available.

Examples:

- Vehicle speed.
- RPM where available.
- Coolant temperature.
- Diagnostic trouble codes.
- Battery voltage if supported by the adapter.

### 7.2 Toyota Prius Gen 3 Adapter

The Prius adapter should eventually support Gen 3 Prius-specific readings where safely available.

Potential future metrics:

- Hybrid battery state of charge.
- Battery block voltages.
- Inverter temperatures.
- Engine run state.
- Fuel economy data.

All Prius-specific details must be documented before implementation.

### 7.3 Mock Vehicle Adapter

The mock adapter is required for development.

It should generate believable sample data for:

- Speed.
- Fuel level.
- Gear.
- Battery voltage.
- Temperatures.
- Alerts.
- Connection state.

---

## 8. Data Flow

### 8.1 Telemetry Flow

```text
Vehicle / Mock Source
        ↓
Vehicle Adapter
        ↓
Hardware Abstraction Layer
        ↓
Vehicle Service
        ↓
WebSocket Event Bus
        ↓
Frontend Dashboard
```

### 8.2 Command Flow

```text
Frontend Button / UI Action
        ↓
REST API
        ↓
Application Service
        ↓
Hardware Abstraction Layer
        ↓
Hardware / Adapter / Mock Handler
```

### 8.3 Settings Flow

```text
Frontend Settings Screen
        ↓
Settings API
        ↓
Settings Service
        ↓
SQLite Database
        ↓
Service Reload / Event Broadcast
```

---

## 9. Local Database

The initial local database should be SQLite.

SQLite is appropriate because Rover OS is a single-node embedded system during early development.

Data categories:

- System settings.
- Vehicle configuration.
- Trip summaries.
- Maintenance records.
- Diagnostic history.
- Event logs.
- Integration configuration.

The database should not store large dashcam video files directly. Video should be stored as files with metadata in the database.

---

## 10. Event Bus

Rover OS should use a WebSocket-based live event bus for the frontend.

Event categories:

- `vehicle.telemetry`
- `vehicle.alert`
- `camera.status`
- `network.status`
- `argo.status`
- `media.status`
- `power.status`
- `system.error`

Events should include:

- Event type.
- Timestamp.
- Source service.
- Payload.

Example:

```json
{
  "type": "vehicle.telemetry",
  "timestamp": "2026-07-09T22:30:00-05:00",
  "source": "vehicle-service",
  "payload": {
    "speedMph": 42,
    "fuelPercent": 68,
    "batteryVoltage": 13.9,
    "gear": "D"
  }
}
```

---

## 11. Platform Layer

The initial platform should be Linux.

Recommended early target:

- Raspberry Pi OS or Debian-based Linux.
- systemd services.
- Chromium kiosk mode.
- Node.js runtime.
- Local SQLite database.

### 11.1 Boot Behavior

The platform should boot into Rover OS automatically.

Target sequence:

```text
Power applied
    ↓
Linux boot
    ↓
systemd starts backend
    ↓
systemd starts kiosk session
    ↓
Chromium opens Rover UI
    ↓
Dashboard displays latest available state
```

### 11.2 Shutdown Behavior

The platform must support graceful shutdown.

Potential shutdown triggers:

- Ignition state from vehicle I/O controller.
- Manual UI shutdown command.
- Low voltage warning.
- Maintenance/debug command.

---

## 12. Vehicle I/O Controller

A dedicated microcontroller is recommended for vehicle-facing electrical signals.

Potential platforms:

- RP2040
- ESP32
- STM32

The microcontroller may handle:

- Ignition state.
- Reverse signal.
- Illumination signal.
- Button inputs.
- Safe shutdown signal.
- Watchdog behavior.

The main computer should communicate with the controller over USB serial or another simple protocol.

This prevents the Linux host from being directly responsible for all real-time electrical behavior.

---

## 13. Camera Architecture

Camera support should be modular.

Initial camera types may include:

- USB camera.
- HDMI camera through USB capture.
- Analog backup camera through USB capture.
- IP camera stream.

The camera service should expose camera availability and stream URLs to the frontend.

The frontend should not directly discover hardware.

---

## 14. Audio Architecture

Audio should be treated as a subsystem.

Initial audio path options:

- USB DAC to factory amplifier integration.
- USB DAC to aftermarket amplifier.
- HDMI audio for bench testing.

The media service should not assume one specific output path.

The production Prius install should attempt to retain the factory JBL amplifier if practical.

---

## 15. ARGO Integration Architecture

ARGO integration must be optional.

The `argo-service` should handle:

- ARGO Core endpoint configuration.
- Health checks.
- Authentication.
- Event sync.
- Trip log upload.
- Remote task hooks.

If ARGO Core is unavailable, Rover OS must continue normal local operation.

---

## 16. Development Modes

Rover OS should support at least three modes.

### 16.1 Desktop Development Mode

- Runs on laptop or desktop.
- Uses mock hardware.
- Hot reload enabled.
- Developer logging enabled.

### 16.2 Bench Mode

- Runs on Raspberry Pi.
- Uses real display and selected hardware.
- Vehicle data may be mocked or from OBD-II.

### 16.3 Vehicle Mode

- Runs in the car.
- Uses real power behavior.
- Uses real hardware where installed.
- Kiosk mode enabled.
- Debug interfaces restricted.

---

## 17. Failure Handling

Rover OS must assume hardware can fail or disconnect.

Examples:

- OBD-II adapter missing.
- Camera unavailable.
- Network offline.
- ARGO Core offline.
- Audio device missing.
- Vehicle adapter crash.

The UI should show degraded status instead of crashing.

Services should report health through a common status interface.

---

## 18. Security Boundaries

Rover OS should default to local-only access.

Rules:

- Do not expose control APIs on the network by default.
- Do not store secrets in frontend code.
- Keep integration tokens in backend configuration.
- Require explicit configuration for remote access.
- Treat the car network as untrusted.

---

## 19. Initial Implementation Recommendation

The first implementation should be a monorepo with separate packages or directories:

```text
frontend/      React touchscreen UI
backend/       Node.js services and APIs
shared/        Shared TypeScript types
firmware/      Future vehicle I/O controller firmware
docs/          Architecture and project documentation
hardware/      Hardware notes and wiring docs
scripts/       Development and deployment scripts
```

This keeps early development simple while still supporting future separation.

---

## 20. Architectural Rule

No core Rover OS feature should require the physical Prius to be present during development.

Every vehicle-facing interface must have a mock implementation.
