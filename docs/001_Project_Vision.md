# 001 — Project Vision

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Primary Vehicle Target:** 2014 Toyota Prius  
**Initial Compute Target:** Raspberry Pi 5  
**Parent System:** ARGO — Autonomous Resource Governance Operator

---

## 1. Vision Statement

ARGO Rover OS is a custom vehicle operating environment that transforms a 2014 Toyota Prius into a mobile ARGO node. It should replace the role of a commercial infotainment head unit while going beyond the capabilities of normal Android Auto or CarPlay receivers.

The system should feel like a purpose-built vehicle computer rather than a generic tablet mounted in a dashboard. It should provide navigation, media, vehicle diagnostics, camera views, telemetry, ARGO integration, Home Assistant access, and future automation features through a unified touchscreen interface.

Rover OS begins as a Prius project, but it should not be designed as Prius-only software. The Prius should be the first vehicle adapter for a broader modular platform.

---

## 2. Product Identity

### 2.1 Name

The product name is **ARGO Rover OS**.

Short names may include:

- Rover OS
- Rover
- ARGO Rover

### 2.2 Meaning

Rover is the mobile vehicle node in the ARGO ecosystem.

ARGO includes:

- **ARGO Core:** central compute and intelligence.
- **ARGO Vault:** storage and archival services.
- **ARGO Rover:** mobile vehicle system.
- **ARGO Home:** home automation and environmental control.
- **Future ARGO nodes:** purpose-built systems for specific contexts.

Rover OS should be the operating environment for the vehicle node.

---

## 3. Core Idea

A store-bought head unit is usually an appliance. Rover OS should be a platform.

A commercial head unit provides:

- A touchscreen.
- Audio playback.
- Phone projection.
- Backup camera display.
- Bluetooth.
- Some steering wheel control integration.
- Basic settings.

Rover OS should provide all of that, while also supporting:

- Custom dashboards.
- Vehicle telemetry.
- Trip history.
- Fuel efficiency tracking.
- Maintenance intelligence.
- Dashcam integration.
- Home Assistant access.
- ARGO Core communication.
- Local services.
- Plugin expansion.
- Future vehicle adapters.

---

## 4. Design Goals

### 4.1 Feature Parity

Rover OS should match or exceed the daily usability of a good aftermarket head unit.

It must support common driver expectations:

- Fast startup.
- Reliable touch controls.
- Media playback.
- Bluetooth or phone-based audio.
- Navigation access.
- Backup camera.
- Volume control.
- Settings.
- Stable power behavior.

### 4.2 Platform Flexibility

The system should not depend permanently on Raspberry Pi hardware.

The Raspberry Pi 5 is the initial development and deployment platform because it is accessible, affordable, documented, and flexible. Future versions may run on:

- Intel N-series mini PCs.
- AMD Ryzen mini PCs.
- ARM single-board computers.
- NVIDIA Jetson devices.
- Other ruggedized edge computers.

### 4.3 Vehicle Flexibility

The initial vehicle is a 2014 Toyota Prius. The software should avoid hard-coding Prius-specific behavior into the core application.

Vehicle-specific functionality should live in adapter modules.

Examples:

- `vehicle-toyota-prius-gen3`
- `vehicle-honda-element`
- `vehicle-generic-obd2`

### 4.4 Local-First Operation

Rover OS should remain useful without internet.

Offline capabilities should include:

- Dashboard.
- Media from local storage.
- Backup camera.
- Vehicle diagnostics.
- OBD-II telemetry.
- Trip logging.
- Maintenance records.
- Cached settings.

Internet should improve the experience, not define it.

### 4.5 ARGO Integration

When connected, Rover OS should integrate with ARGO Core and other ARGO services.

Potential integrations:

- Sync trip logs to ARGO Vault.
- Display ARGO Core availability.
- Trigger remote jobs.
- Show Home Assistant state.
- Sync maintenance data.
- Upload selected dashcam clips.
- Provide a mobile terminal into the broader ARGO ecosystem.

---

## 5. User Experience Vision

The system should boot directly into the Rover dashboard.

The driver should not see:

- A Linux desktop.
- A terminal.
- A browser address bar.
- A pile of generic apps.

The driver should see:

- A branded Rover interface.
- Large touch targets.
- Status at a glance.
- Context-aware controls.
- A clean parked mode and a simplified driving mode.

The interaction model should be closer to a custom embedded interface than a consumer tablet.

---

## 6. Visual Direction

Rover OS should feel like a clean engineering prototype: modern, functional, calm, and slightly futuristic.

Suggested themes:

- Dark cockpit mode.
- High-contrast driving mode.
- Minimal text while moving.
- Large typography.
- Clear icons.
- Status colors used sparingly.
- Optional ARGO branding.

The interface should avoid clutter. It should prioritize glanceable information over dense dashboards while driving.

---

## 7. Safety Vision

Rover OS must not interfere with safety-critical vehicle systems.

The system should not control:

- Steering.
- Braking.
- Throttle.
- Airbags.
- ABS.
- Traction control.
- Stability control.
- Safety restraints.

The system may read vehicle data through safe diagnostic channels. Control functions should be avoided unless explicitly researched, documented, and isolated from safety-critical systems.

The UI should reduce driver distraction.

Driving mode should:

- Reduce visual complexity.
- Limit deep menus.
- Use large buttons.
- Avoid text entry.
- Prioritize voice or simple controls.
- Keep camera and navigation functions accessible.

---

## 8. Long-Term Vision

Rover OS should become a reusable vehicle computing platform.

The first successful milestone is a working 2014 Prius installation.

The larger goal is a modular platform that could eventually be adapted to:

- Other Prius generations.
- Honda Element.
- Utility vehicles.
- Vans.
- RVs.
- Campers.
- Boats.
- Fleet vehicles.

A mature Rover OS should allow a new vehicle to be supported by adding a vehicle adapter and hardware integration guide rather than rewriting the whole system.

---

## 9. Success Definition

Rover OS is successful when it can be used daily instead of a commercial head unit.

Minimum success looks like:

- The car starts and Rover reliably boots.
- The display is readable and responsive.
- Audio works reliably.
- Backup camera works reliably.
- Navigation access is convenient.
- OBD-II telemetry is visible.
- Settings persist.
- The system shuts down safely.

Strong success looks like:

- Rover feels integrated into the car.
- ARGO services are useful from the dashboard.
- Diagnostics and maintenance tracking are better than a normal head unit.
- The architecture can move from Pi to mini PC with minimal changes.
- Another vehicle could be supported through a new adapter.

---

## 10. Development Principle

Build the software so it can run on a laptop before it ever runs in the car.

The development flow should be:

1. Mock vehicle data.
2. Desktop UI prototype.
3. Raspberry Pi bench test.
4. Hardware integration bench test.
5. Non-invasive vehicle install.
6. Daily-driver hardening.

This keeps the project from becoming blocked by hardware availability.
