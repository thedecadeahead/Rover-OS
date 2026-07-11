# 021 — Vehicle I/O Controller

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft  

## 1. Purpose

The vehicle I/O controller is a dedicated microcontroller that isolates time-sensitive and vehicle-facing electrical work from the Linux host. It should read low-voltage vehicle signals, report them to Rover OS, assist with safe startup and shutdown, and remain simple enough to audit.

## 2. Initial Responsibilities

The controller may handle:

- Accessory/ignition state.
- Reverse signal.
- Illumination/headlight state.
- Parking-brake state where used.
- Steering-wheel button inputs where practical.
- Host watchdog status.
- Graceful shutdown request.
- Delayed power-cut control through a relay or power-enable line.
- Basic supply-voltage monitoring.

It must not control braking, steering, throttle, airbags, ABS, traction control, or hybrid high-voltage systems.

## 3. Candidate Platforms

Preferred initial candidates:

- RP2040.
- ESP32.
- STM32.

RP2040 is a strong first choice because it is inexpensive, well documented, supports USB serial, and does not require wireless features.

## 4. Electrical Boundaries

Vehicle inputs must not connect directly to microcontroller GPIO pins.

Required protections may include:

- Voltage dividers.
- Optocouplers where appropriate.
- Transient suppression.
- Reverse-polarity protection.
- Input filtering.
- Fused power.
- Common-ground design reviewed before installation.

Exact circuits must be documented in `hardware/wiring.md` before vehicle use.

## 5. Host Interface

Initial transport should be USB serial.

Recommended line-delimited JSON protocol:

```json
{"type":"vehicle_state","ignition":true,"reverse":false,"illumination":true,"voltage":13.9}
```

Host command example:

```json
{"type":"host_state","state":"shutdown_complete"}
```

Messages must include a protocol version once stabilized.

## 6. Event Types

Controller to host:

- `controller.hello`
- `controller.heartbeat`
- `vehicle.ignition_changed`
- `vehicle.reverse_changed`
- `vehicle.illumination_changed`
- `vehicle.parking_brake_changed`
- `vehicle.button`
- `power.voltage`
- `power.shutdown_requested`
- `controller.error`

Host to controller:

- `host.hello`
- `host.heartbeat`
- `host.shutdown_acknowledged`
- `host.shutdown_complete`
- `controller.configure`
- `controller.identify`

## 7. Startup Sequence

1. Ignition/accessory signal becomes active.
2. Controller enables regulated host power if it controls power switching.
3. Controller starts heartbeat messages.
4. Linux host boots.
5. Rover backend opens serial connection.
6. Host and controller exchange protocol versions and capabilities.
7. Normal event flow begins.

## 8. Shutdown Sequence

1. Ignition becomes inactive.
2. Controller sends `power.shutdown_requested`.
3. Backend begins graceful shutdown.
4. Host sends acknowledgement.
5. Linux completes shutdown.
6. Host signals `shutdown_complete` where hardware permits.
7. Controller waits a configurable safety delay.
8. Controller disables host power.

Timeout behavior must prevent indefinite battery drain if the host hangs.

## 9. Watchdog Behavior

The controller should expect host heartbeats while Rover OS is running.

If heartbeats stop:

- Do not immediately cut power.
- Wait through a configurable timeout.
- Record/reset reason where possible.
- Allow one controlled restart attempt if enabled.
- Avoid reboot loops.

## 10. Button Handling

Buttons should be debounced in firmware.

Normalized button events should include:

- Button identifier.
- Press/release state.
- Optional long-press event.
- Timestamp or sequence number.

The controller should not assign application meaning beyond stable button IDs.

## 11. Firmware Update Strategy

Initial firmware updates may require USB maintenance access.

Future update support may include:

- Bootloader-based local update.
- Signed firmware artifact.
- Version check from Rover backend.
- Rollback only if the platform safely supports it.

Remote firmware updates should not be enabled until recovery procedures are tested.

## 12. Logging and Diagnostics

The controller should expose:

- Firmware version.
- Protocol version.
- Reset reason.
- Uptime.
- Input states.
- Supply voltage.
- Error counters.
- Last shutdown reason.

## 13. Mock Controller

The backend must include a mock controller adapter that can simulate:

- Ignition on/off.
- Reverse transitions.
- Headlight transitions.
- Voltage warnings.
- Missing heartbeat.
- Button presses.

## 14. Acceptance Criteria

The controller design is acceptable for bench prototype when:

- USB serial connection is stable.
- Ignition and reverse can be simulated with low-voltage test inputs.
- Host receives normalized events.
- Shutdown request/acknowledgement flow works.
- Watchdog timeout can be tested without uncontrolled power cycling.
