# 005 — Vehicle Integration

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft  
**Primary Vehicle Target:** 2014 Toyota Prius / Toyota Prius Gen 3  

---

## 1. Purpose

This document defines the vehicle integration strategy for installing ARGO Rover OS in a 2014 Toyota Prius. It focuses on safe, reversible, non-invasive integration with the vehicle’s existing systems.

The goal is to replace the practical role of a store-bought head unit while preserving factory safety systems and avoiding unnecessary modification of factory wiring.

---

## 2. Vehicle Scope

Initial target:

- 2014 Toyota Prius.
- Toyota Prius Gen 3 platform.
- Possible JBL amplified audio system.
- Factory steering wheel controls.
- Factory or aftermarket backup camera potential.

This document should be updated once the exact vehicle trim, factory radio, JBL status, camera status, and harness configuration are confirmed.

---

## 3. Integration Philosophy

Rover OS should integrate with the Prius using the least destructive path practical.

Preferred approach:

1. Use vehicle-specific adapter harnesses.
2. Use established aftermarket interfaces where available.
3. Use OBD-II for diagnostics first.
4. Use a microcontroller for simple vehicle signals where needed.
5. Avoid cutting factory wiring.
6. Avoid modifying safety-critical systems.

---

## 4. Safety Boundaries

Rover OS must not modify or interfere with:

- Airbags.
- Steering wheel airbag cover.
- Seatbelts.
- ABS.
- Traction control.
- Stability control.
- Brake control.
- Throttle control.
- Hybrid high-voltage wiring.
- High-voltage battery service components.

Rover OS may read low-risk vehicle information through OBD-II or documented read-only interfaces.

Do not interact with orange high-voltage Prius wiring. High-voltage hybrid work requires proper training, tools, and procedures.

---

## 5. Factory Radio Replacement Strategy

Rover OS is not a normal double-DIN radio, but it must still interface with the same vehicle systems a head unit normally uses.

A commercial head unit usually connects to:

- Constant 12V.
- Accessory / ignition.
- Ground.
- Speaker outputs or amplifier interface.
- Reverse signal.
- Parking brake signal.
- Illumination / dimmer.
- Steering wheel controls.
- Backup camera.
- Antenna.

Rover OS should handle these through adapter harnesses and dedicated interfaces rather than direct cutting.

---

## 6. Audio Integration

### 6.1 JBL System Consideration

The Prius may have the JBL amplified audio system. If so, audio integration is more complex than connecting speaker wires directly.

The preferred path is to retain the factory JBL amplifier using a compatible Toyota/JBL integration interface.

Potential interface categories:

- Toyota radio replacement harness.
- JBL amplifier retention interface.
- Steering wheel control retention interface.
- Line-level audio adapter.

Exact part selection belongs in `hardware/bill_of_materials.md` after vehicle trim confirmation.

### 6.2 Rover Audio Path

Recommended path:

```text
Rover Compute Device
        ↓
USB Audio DAC
        ↓
Line-Level Audio Interface
        ↓
Toyota/JBL Amplifier Retention Interface
        ↓
Factory Amplifier and Speakers
```

Alternative path:

```text
Rover Compute Device
        ↓
USB Audio DAC
        ↓
Aftermarket DSP / Amplifier
        ↓
Speakers
```

The factory-retention path is preferred for v1 because it reduces rewiring and preserves existing speakers.

---

## 7. Steering Wheel Controls

Steering wheel controls are a core usability feature.

Desired controls:

- Volume up.
- Volume down.
- Next track.
- Previous track.
- Mode/source.
- Voice / phone button if available.

Implementation options:

1. Commercial steering wheel control interface that outputs USB, serial, or programmable signals.
2. Vehicle I/O controller reading resistive ladder or CAN messages, depending on Prius wiring.
3. Aftermarket adapter designed for Toyota radio replacement.

Rover OS should consume steering wheel input as generic events, not Toyota-specific raw signals.

Example events:

- `input.volume_up`
- `input.volume_down`
- `input.media_next`
- `input.media_previous`
- `input.voice_toggle`

---

## 8. Backup Camera Integration

The backup camera is a must-have feature for parity with a modern head unit.

### 8.1 Required Behavior

Rover OS must support:

- Manual rear camera view.
- Automatic rear camera view when reverse is detected, once reverse signal integration exists.
- Fast switching into camera mode.
- Clear degraded-state warning if the camera is unavailable.

### 8.2 Camera Sources

Possible rear camera sources:

- Existing factory camera if present.
- Aftermarket analog camera.
- USB camera.
- HDMI camera through USB capture.

### 8.3 Reverse Trigger

Reverse detection may come from:

- Head unit harness reverse signal.
- Vehicle I/O controller.
- CAN/OBD-derived gear state if reliable.

A physical reverse signal is preferred for fast and dependable switching.

---

## 9. Ignition / Accessory State

Rover OS needs to know whether the car is on, off, or in accessory mode.

Potential signals:

- Accessory wire from radio harness.
- Ignition state from fuse panel.
- Vehicle I/O controller input.
- Power supply ignition sense input.

The ignition state should be used to trigger:

- Startup.
- Graceful shutdown.
- Low-power mode.
- Delayed accessory timeout.

---

## 10. Illumination / Dimmer

If available, the vehicle illumination or dimmer signal should be used to switch Rover OS between day and night display modes.

Desired behavior:

- Headlights on → night theme or reduced brightness.
- Headlights off → day theme.
- Manual override in settings.

---

## 11. OBD-II Integration

OBD-II is the first and safest diagnostics interface.

Rover OS should support:

- USB OBD-II adapter preferred.
- Adapter connection status.
- Standard OBD-II live data.
- Diagnostic trouble code reading.
- Trip data logging.

Initial standard data may include:

- Vehicle speed.
- Coolant temperature.
- Engine RPM where applicable.
- Fuel system status where available.
- Diagnostic trouble codes.

Prius-specific hybrid data should be researched before implementation.

---

## 12. CAN Bus Integration

CAN bus integration may eventually unlock better vehicle data, but it increases complexity and risk.

Rules:

- Read-only by default.
- No safety-critical messages.
- No control messages without extensive documentation.
- Keep all CAN code behind the vehicle adapter.
- Document every message used.

CAN is not required for v0.1.

---

## 13. Parking Brake Signal

Some commercial head units use the parking brake signal to lock out video or certain settings while driving.

Rover OS should implement its own safe driving mode rather than relying only on parking brake state.

Parking brake signal may still be useful for:

- Allowing full settings UI only when parked.
- Enabling maintenance screens.
- Restricting keyboard entry.

---

## 14. Physical Mounting

The large touchscreen will require careful dash integration.

Mounting requirements:

- Does not obstruct driver visibility.
- Does not block hazard switch or critical controls.
- Does not interfere with airbags.
- Does not create sharp edges.
- Can be removed for service.
- Supports cable strain relief.
- Handles vibration.

A custom bezel or bracket is expected.

---

## 15. Wiring Strategy

Preferred wiring strategy:

- Use Toyota-specific adapter harnesses.
- Use fused power taps where needed.
- Label every wire.
- Document every connector.
- Avoid permanent changes until bench-tested.
- Keep all added wiring secured and protected.

All final wiring should be documented in future `hardware/wiring.md`.

---

## 16. Power and Shutdown Integration

The Prius is well-suited to powering electronics because the hybrid system can maintain the 12V system when READY, but Rover OS must still protect itself.

Power integration should support:

- Stable regulated power to compute.
- Stable regulated power to display.
- Safe shutdown on vehicle off.
- Low-voltage protection.
- Manual maintenance override.

The system should not be left running indefinitely from the 12V battery when the car is off unless a dedicated power management strategy exists.

---

## 17. Factory Functions to Preserve

The integration should preserve:

- Factory HVAC controls.
- Hazard lights.
- Airbag systems.
- Steering wheel safety systems.
- Factory vehicle displays where applicable.
- Factory warning behavior.
- Required driver controls.

If any factory function is affected by radio removal, document it before installation.

---

## 18. Installation Phases

### Phase 1 — Bench Simulation

- Display and Pi on bench.
- Mock vehicle data.
- Test camera input.
- Test audio output.

### Phase 2 — Non-Invasive Vehicle Test

- Power from temporary fused accessory source.
- OBD-II data only.
- Temporary display mount.
- No factory audio integration yet.

### Phase 3 — Audio and Camera Integration

- Add factory amplifier interface.
- Add backup camera input.
- Add reverse trigger.

### Phase 4 — Controls and Polish

- Add steering wheel controls.
- Add illumination trigger.
- Add custom bezel.
- Secure final wiring.

### Phase 5 — Daily Driver Hardening

- Validate boot reliability.
- Validate shutdown reliability.
- Validate heat performance.
- Validate vibration and cable retention.

---

## 19. Vehicle Adapter Responsibilities

The Toyota Prius Gen 3 vehicle adapter should eventually expose:

- Vehicle speed.
- Gear state if available.
- Fuel/range information if available.
- 12V battery voltage.
- Hybrid battery data if safely available.
- Reverse state.
- Ignition state.
- Lighting state.
- Steering wheel button events.

Every signal should include a source label, such as:

- `obd2`
- `can`
- `gpio`
- `vehicle-controller`
- `mock`

---

## 20. Integration Acceptance Criteria

Vehicle integration is acceptable for v1.0 when:

- Rover boots reliably with the car.
- Rover shuts down safely when the car turns off.
- Audio works through the vehicle speakers.
- Backup camera works reliably.
- Reverse trigger works reliably.
- OBD-II telemetry is stable.
- Steering wheel controls work for core media functions.
- Wiring is fused and documented.
- No factory safety systems are modified.
- The install can be reversed or serviced without major damage.
