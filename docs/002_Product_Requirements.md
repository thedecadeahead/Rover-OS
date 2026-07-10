# 002 — Product Requirements

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft  
**Primary Vehicle Target:** 2014 Toyota Prius  

---

## 1. Purpose

This document defines the product requirements for ARGO Rover OS. It describes what the system must do, what it should do, and what it explicitly should not attempt to do during the early phases of the project.

The goal is feature parity with a premium aftermarket head unit, followed by capabilities that commercial head units do not normally provide.

---

## 2. Requirement Language

The following terms are used consistently:

- **Must:** required for the target release.
- **Should:** strongly desired but may be deferred if necessary.
- **May:** optional or future-facing.
- **Must not:** explicitly prohibited or out of scope.

---

## 3. Target Releases

### 3.1 v0.1 Alpha — Desktop Prototype

The system must run on a normal development computer with mock vehicle data.

Required:

- Core dashboard UI.
- Mock telemetry.
- Navigation screen placeholder.
- Media screen placeholder.
- Diagnostics screen placeholder.
- Settings persistence.
- Basic WebSocket event bus.
- Local backend service.

### 3.2 v0.2 Bench Prototype

The system must run on Raspberry Pi 5 with a touchscreen on a bench.

Required:

- Kiosk startup.
- Touchscreen support.
- Local database.
- Mock or real OBD-II integration.
- Camera test view.
- Audio output test.

### 3.3 v0.3 Vehicle Prototype

The system must operate in the Prius with non-invasive hardware integration.

Required:

- Safe power behavior.
- Backup camera view.
- Audio path to factory system or test amplifier.
- OBD-II telemetry.
- Manual shutdown recovery.

### 3.4 v1.0 Daily Driver

The system should be reliable enough to use instead of a normal head unit.

Required:

- Stable startup and shutdown.
- Reliable audio.
- Reliable camera view.
- Persistent settings.
- Useful diagnostics.
- Driving-safe UI mode.
- Documented installation.

---

## 4. User Personas

### 4.1 Primary User — Owner / Builder

The primary user is a technically capable owner who wants a custom vehicle computing platform, not just a stereo replacement.

Needs:

- Full control.
- Expandability.
- Debug access.
- Git-based development.
- ARGO integration.
- Ability to upgrade hardware later.

### 4.2 Driver

The driver may be the same person, but while driving they need simplicity and reliability.

Needs:

- Fast access to navigation.
- Reliable media controls.
- Backup camera.
- Clear status.
- Minimal distraction.
- No fragile developer workflows while driving.

### 4.3 Future Contributor

A future contributor may want to add support for another vehicle or feature.

Needs:

- Clear documentation.
- Modular architecture.
- Mock interfaces.
- Hardware abstraction.
- API documentation.

---

## 5. Core Functional Requirements

### 5.1 Dashboard

The system must provide a default dashboard screen.

The dashboard must show:

- Time.
- Vehicle connection status.
- Network status.
- Audio status.
- Shortcut to navigation.
- Shortcut to media.
- Shortcut to cameras.
- Shortcut to diagnostics.
- Shortcut to settings.

The dashboard should show:

- Fuel level or range when available.
- 12V battery voltage when available.
- ARGO Core status when available.
- Current trip summary.
- Weather when online.

### 5.2 Navigation

The system must provide a navigation entry point.

For early versions, navigation may be implemented as:

- Browser-based mapping.
- Android Auto projection.
- External phone navigation support.
- A placeholder route screen.

The system should eventually support:

- Fullscreen map view.
- Traffic when online.
- Cached maps where practical.
- Destination shortcuts.
- Work/home/favorites integration.

### 5.3 Media

The system must provide media controls.

Media requirements:

- Play/pause.
- Previous/next.
- Volume control.
- Current source display.
- Local audio playback support.

The system should support:

- Bluetooth audio input.
- Streaming services through browser or phone projection.
- Local media library.
- Steering wheel media controls.

### 5.4 Audio Output

The system must provide a path to vehicle speakers.

Initial acceptable paths:

- USB DAC to factory amplifier integration interface.
- USB DAC to aftermarket amplifier.
- HDMI audio to display during bench testing.

The production system should retain the factory JBL amplifier if practical.

### 5.5 Cameras

The system must support a backup camera view.

Backup camera requirements:

- Display rear camera manually.
- Support reverse-triggered display when vehicle signal is available.
- Avoid long delay when switching to reverse.

The system should support:

- Front camera.
- Cabin camera.
- Auxiliary camera.
- Dashcam recording.
- Clip saving.

### 5.6 Diagnostics

The system must support OBD-II diagnostics.

Diagnostics must include:

- Connection status.
- Basic live data when available.
- Diagnostic trouble code reading where supported.
- Clear presentation of errors.

Diagnostics should include:

- Fuel economy.
- Coolant temperature.
- RPM where applicable.
- Vehicle speed.
- 12V battery voltage.
- Hybrid-specific metrics when safely available.

### 5.7 Settings

The system must provide persistent settings.

Settings must include:

- Display brightness preference.
- Theme preference.
- Network settings reference.
- Audio output preference.
- Debug mode toggle.
- Vehicle adapter selection.

Settings should include:

- Home Assistant endpoint.
- ARGO Core endpoint.
- Units.
- Startup behavior.
- Logging level.

### 5.8 Connectivity

The system must work without internet.

The system should support internet through:

- Driver phone hotspot.
- Wi-Fi network.
- Optional LTE modem or dedicated phone in future versions.

The system must clearly indicate online/offline state.

### 5.9 ARGO Integration

The system should support optional ARGO integration.

ARGO integration should include:

- ARGO Core online/offline status.
- Basic authenticated API connection.
- Event sync hooks.
- Trip log export.
- Home Assistant access.

ARGO integration must not be required for local vehicle functions.

---

## 6. Nonfunctional Requirements

### 6.1 Reliability

The system must tolerate:

- Network loss.
- OBD-II disconnection.
- Camera disconnection.
- ARGO Core unavailability.
- Unexpected reboot.

A failed subsystem must not crash the entire UI.

### 6.2 Startup

The system should boot into the dashboard as quickly as practical.

The system must not require manual login during normal operation.

### 6.3 Shutdown

The system must support graceful shutdown.

Shutdown should be triggered by:

- Ignition state when integrated.
- Manual software command.
- Power controller signal.

The system should reduce the risk of storage corruption.

### 6.4 Maintainability

The codebase must be organized so that:

- UI and hardware logic are separate.
- Mock hardware can be used during development.
- Vehicle-specific code is isolated.
- Services are documented.
- Configuration is not hard-coded.

### 6.5 Security

The system must not expose unauthenticated remote control interfaces by default.

The system should:

- Keep local APIs bound to localhost unless configured otherwise.
- Require authentication for remote access.
- Avoid storing sensitive secrets in source code.
- Support environment-based configuration.

### 6.6 Safety

The system must not control safety-critical vehicle systems.

The UI should reduce distraction while driving.

Driving mode should avoid:

- Text entry.
- Small buttons.
- Deep nested menus.
- Debug views.
- Excessive animation.

---

## 7. Hardware Requirements

### 7.1 Initial Compute

The initial compute target must be Raspberry Pi 5.

Recommended:

- Raspberry Pi 5, 8 GB.
- Active cooling.
- NVMe storage.
- Reliable 5V power supply.

### 7.2 Display

The system must support an HDMI capacitive touchscreen.

Recommended target:

- 13.3 inches.
- 1920x1080 or better.
- USB touch input.
- Sufficient brightness for vehicle use.

### 7.3 Storage

The system should avoid relying on microSD for heavy writes.

Recommended:

- NVMe SSD for OS and logs.
- Separate storage strategy for video recording if dashcam features are implemented.

### 7.4 Vehicle Interface

The system should support:

- OBD-II adapter.
- Optional CAN interface.
- Optional microcontroller-based vehicle I/O controller.
- Reverse signal detection.
- Ignition state detection.
- Steering wheel control interface.

---

## 8. Out of Scope for Early Versions

The following are out of scope for early development:

- Autonomous driving.
- Throttle control.
- Brake control.
- Steering control.
- Airbag system modification.
- ABS or traction control interaction.
- Permanent factory harness cutting.
- Safety-critical CAN writes.
- Remote control of vehicle movement.

---

## 9. Acceptance Criteria for v0.1

v0.1 is acceptable when:

- The app runs on a development computer.
- The dashboard loads with mock data.
- Navigation, media, cameras, diagnostics, and settings routes exist.
- A local backend service runs.
- The frontend receives live mock telemetry over WebSocket.
- Settings persist locally.
- The repository contains enough documentation for Claude or a developer to continue implementation.

---

## 10. Acceptance Criteria for v1.0

v1.0 is acceptable when:

- The system can be used daily in the Prius.
- Startup is reliable.
- Shutdown is safe.
- Audio works reliably.
- Backup camera works reliably.
- OBD-II diagnostics work reliably.
- The UI is safe and readable while driving.
- The system can operate offline.
- The installation is documented.
- The architecture supports future hardware migration.
