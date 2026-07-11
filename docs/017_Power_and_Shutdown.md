# 017 — Power and Shutdown

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft

---

## 1. Purpose

This document defines the power-management, ignition-sense, startup, shutdown, low-voltage, and recovery behavior for Rover OS.

Vehicle power is noisy and transient. The system must not assume desktop-style continuous power.

---

## 2. Goals

The power system must:

- Provide stable regulated power to compute, display, and peripherals.
- Detect ignition or accessory state.
- Start Rover OS automatically.
- Shut Linux down gracefully.
- Prevent excessive 12V battery drain.
- Recover from interrupted power.
- Avoid storage corruption.
- Expose power state to the backend and UI.

---

## 3. Power Architecture

```text
Vehicle 12V
   ↓
Primary Fuse
   ↓
Automotive DC-DC / Power Controller
   ├── 5V Compute Rail
   ├── Display Rail
   ├── Peripheral Rail
   └── Ignition Sense / Shutdown Control
```

The production design should separate stable compute power from raw accessory power.

---

## 4. Required Inputs

The power controller may use:

- Constant 12V.
- Accessory or ignition signal.
- Ground.
- Optional READY-state signal if safely available.
- Optional low-voltage measurement.

Exact Prius wiring must be confirmed before installation.

---

## 5. Power States

Recommended normalized states:

- `off`
- `starting`
- `running`
- `accessory`
- `shutdown_pending`
- `shutting_down`
- `maintenance`
- `fault`

The frontend should consume these normalized states rather than raw GPIO levels.

---

## 6. Startup Sequence

```text
Ignition/ACC becomes active
    ↓
Power controller enables regulated rails
    ↓
Compute device boots
    ↓
Backend starts
    ↓
Frontend kiosk starts
    ↓
Power service reports running
```

The system should tolerate the display becoming ready before or after the Pi.

---

## 7. Shutdown Sequence

```text
Ignition/ACC becomes inactive
    ↓
Controller starts configurable delay
    ↓
Backend receives shutdown request
    ↓
UI displays shutdown pending
    ↓
Recording and write-heavy services stop
    ↓
Database and logs flush
    ↓
Linux shuts down
    ↓
Controller removes compute power
```

Recommended initial delay: configurable between 15 and 120 seconds.

---

## 8. Cancellation

If ignition returns during the shutdown delay:

- Cancel pending shutdown if Linux shutdown has not started.
- Publish `power.shutdown_cancelled`.
- Restore normal service operation.

Once the OS enters final shutdown, the controller should allow reboot on the next clean power cycle rather than attempting unsafe cancellation.

---

## 9. Delayed Accessory Mode

Rover may optionally remain active briefly after the car is turned off.

Use cases:

- Finish writing trip data.
- Upload small sync records.
- Preserve media playback during a brief stop.

This delay must be bounded and must not depend on internet availability.

---

## 10. Low-Voltage Protection

The controller should monitor the 12V supply if hardware supports it.

Behavior:

- Warn at a configurable threshold.
- Stop nonessential peripherals first.
- Request immediate safe shutdown at a critical threshold.
- Cut power after shutdown or timeout.

Thresholds must be selected based on measured Prius behavior and power-controller accuracy, not guessed in software.

---

## 11. Prius READY Mode

When the Prius is in READY, the DC-DC converter normally supports the 12V system from the hybrid system.

Rover should distinguish, where safely possible, between:

- Vehicle fully off.
- Accessory mode relying on the 12V battery.
- READY mode with active DC-DC support.

Until reliable detection exists, Rover should use conservative shutdown timers.

---

## 12. Power Budget

The hardware specification should track estimated and measured load for:

- Raspberry Pi 5.
- Touchscreen.
- NVMe storage.
- Powered USB hub.
- Cameras.
- USB DAC.
- OBD adapter.
- GPS and microphone.

The DC-DC supply should include reasonable headroom above measured peak load.

---

## 13. Peripheral Control

Future power controllers may switch peripherals independently.

Potential rails:

- Compute.
- Display.
- Cameras.
- USB accessories.
- Amplifier remote turn-on.

Benefits:

- Reduce standby drain.
- Restart failed peripherals.
- Keep compute alive briefly while display powers down.

---

## 14. Abrupt Power Loss Recovery

Rover must assume power may disappear without warning.

Mitigations:

- Use a journaling filesystem.
- Use SQLite WAL mode where appropriate.
- Keep frequent writes bounded.
- Use NVMe rather than microSD for production.
- Perform startup integrity checks.
- Mark interrupted trips as recoverable.
- Keep a last-known-good application release.

---

## 15. Watchdog

A watchdog should detect a hung compute system.

Options:

- Linux hardware watchdog.
- Microcontroller heartbeat over serial.
- Power-controller timeout.

The controller should not reboot continuously without limits. Repeated failures should enter a fault state.

---

## 16. Maintenance Mode

Maintenance mode should allow the system to remain powered while the car is stationary for development or service.

Requirements:

- Explicit manual activation.
- Visible indicator.
- Configurable maximum duration.
- Low-voltage protection remains active.
- No accidental permanent bypass of shutdown logic.

---

## 17. Power Service API

The backend power service should expose:

- Current normalized power state.
- Raw ignition state where available.
- Input voltage where available.
- Shutdown countdown.
- Maintenance mode state.
- Last shutdown reason.

Commands may include:

- Request shutdown.
- Cancel pending shutdown.
- Enter maintenance mode.
- Exit maintenance mode.

Sensitive commands should be restricted while driving.

---

## 18. Failure Modes

The system must handle:

- Ignition signal bouncing.
- Controller disconnected.
- Serial communication loss.
- Display power failure.
- USB hub overload.
- Repeated boot loops.
- Linux failing to acknowledge shutdown.

The controller should use debounce, timeouts, and a final power-cut policy.

---

## 19. Bench Testing

Before vehicle installation, test:

- Cold boot.
- Rapid ignition on/off cycles.
- Shutdown cancellation.
- Forced power removal.
- Low-voltage simulation.
- Peripheral overload.
- Controller communication loss.
- Repeated reboot recovery.

---

## 20. Acceptance Criteria

Power management is acceptable for v1.0 when:

- Rover starts reliably with the vehicle.
- Rover shuts down cleanly after ignition off.
- A short restart cancels shutdown when safe.
- Low-voltage behavior is documented and tested.
- Abrupt power loss does not normally corrupt the system.
- The controller cuts power after shutdown or timeout.
- Standby drain is measured and acceptable.
