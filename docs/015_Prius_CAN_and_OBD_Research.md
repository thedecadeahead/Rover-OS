# 015 — Prius CAN and OBD Research

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Research Draft  
**Vehicle Target:** 2014 Toyota Prius / Third Generation  

---

## 1. Purpose

This document defines the research and validation plan for obtaining vehicle data from a 2014 Toyota Prius through OBD-II and, later, passive CAN capture.

The first goal is not deep vehicle control. The first goal is reliable, read-only telemetry suitable for diagnostics, trip logging, maintenance, and the Rover OS dashboard.

No signal, PID, arbitration ID, scaling formula, connector pin, or bus rate should be treated as confirmed until it has been validated against the exact vehicle and documented with a reproducible test.

---

## 2. Safety Boundary

Rover OS must begin with read-only diagnostics.

The initial implementation must not:

- Transmit arbitrary CAN frames.
- Send actuator commands.
- Control steering, braking, throttle, gear selection, airbags, ABS, traction control, or stability control.
- Connect directly to high-voltage hybrid wiring.
- Clear diagnostic trouble codes automatically.
- Poll unknown diagnostic requests aggressively.
- Assume community-decoded signals are correct for the exact model year and trim.

A passive listener or standard diagnostic request can still affect system safety if implemented incorrectly. Testing must occur while parked before any road validation.

---

## 3. Integration Stages

Vehicle data research should proceed in this order:

1. Mock vehicle adapter.
2. Standard OBD-II through a known-good USB adapter.
3. Manufacturer-specific diagnostic requests, only after documentation and rate limits are understood.
4. Passive CAN capture with listen-only configuration.
5. Signal correlation and validation.
6. Production vehicle adapter using only confirmed read paths.

Skipping directly to active raw CAN work is not justified for the first Rover build.

---

## 4. Standard OBD-II First

Standard OBD-II is the safest starting point because it provides a defined diagnostic request/response model.

The initial `generic-obd2` adapter should attempt only supported standard commands discovered from the vehicle and adapter.

Candidate standard data includes:

- Vehicle speed.
- Engine coolant temperature.
- Engine RPM when reported.
- Calculated engine load.
- Intake air temperature.
- Fuel-system status.
- Diagnostic trouble codes.
- Monitor status.
- Control-module voltage where supported.

Availability differs by vehicle. Rover OS must expose capabilities dynamically and must not represent missing data as zero.

---

## 5. Adapter Requirements

Preferred initial adapter:

- USB connection.
- Stable Linux serial driver.
- Documented chipset.
- Reliable command handling.
- Support for the vehicle's diagnostic protocol.
- Unique serial number if possible for stable udev naming.

Avoid relying on an unknown counterfeit ELM327 clone for production qualification. A cheap adapter may be useful for experimentation, but it should not define the architecture.

The adapter should appear through a stable alias such as:

```text
/dev/rover-obd
```

---

## 6. OBD Software Abstraction

The Rover backend should not expose adapter command strings to the frontend.

Recommended interface:

```ts
interface ObdAdapter {
  start(): Promise<void>;
  stop(): Promise<void>;
  getHealth(): AdapterHealth;
  getCapabilities(): Promise<ObdCapabilities>;
  readSnapshot(): Promise<ObdSnapshot>;
  readDiagnosticCodes(): Promise<DiagnosticCode[]>;
  subscribe(listener: (sample: ObdSample) => void): Unsubscribe;
}
```

Implementations may use a Node library, a small Python service, or direct serial communication. The API contract should remain independent of the library choice.

---

## 7. Polling Rules

Diagnostic polling should be conservative.

Initial rules:

- Discover supported standard PIDs before polling.
- Poll only values actively used.
- Use low update rates during bench validation.
- Apply per-command timeouts.
- Prevent overlapping command floods on serial adapters.
- Back off after repeated errors.
- Stop polling cleanly when ignition is off or the adapter disconnects.

Suggested starting rates:

- Speed: 2–5 Hz.
- Engine state/RPM: 2–5 Hz.
- Coolant temperature: 0.5–1 Hz.
- Voltage: 0.5–1 Hz.
- Trouble-code scan: manual or infrequent, never continuously.

These are engineering starting points, not confirmed vehicle limits.

---

## 8. Prius-Specific Data Goals

After standard OBD-II works reliably, useful Prius-specific research targets include:

- Hybrid battery state of charge.
- Hybrid battery block voltages.
- Battery temperature sensors.
- Inverter temperatures.
- Engine running state.
- Motor-generator speeds or temperatures where available.
- Brake regeneration data.
- Fuel level or estimated range.
- Gear state.
- READY state.
- Door or lighting state, if useful and safely available.

Each signal must include:

- Source protocol.
- Request or arbitration ID.
- Response ID where relevant.
- Byte layout.
- Scaling.
- Unit.
- Expected range.
- Vehicle/model-year evidence.
- Validation procedure.
- Confidence level.

---

## 9. Signal Confidence Levels

Use the following confidence levels:

### Unverified

Found in a community source or inferred from another model. Not used in production UI.

### Observed

Seen on the target vehicle, but meaning or scaling is not fully validated.

### Correlated

Changes consistently with a known vehicle state or trusted scan-tool reading.

### Validated

Meaning, scaling, unit, and range have been reproduced across multiple sessions.

### Production Approved

Validated and reviewed for inclusion in the Rover Prius adapter.

The UI should not present low-confidence data as authoritative.

---

## 10. CAN Access Strategy

Linux should use SocketCAN for raw CAN interfaces where supported.

Expected interface model:

```text
USB or SPI CAN adapter
        ↓
Linux CAN network device
        ↓
SocketCAN interface such as can0
        ↓
Capture tools / Rover adapter
```

The Linux kernel SocketCAN stack exposes CAN devices as network interfaces. `can-utils` provides tools such as `candump`, `cansniffer`, and replay utilities for development.

Before vehicle connection, test with a virtual CAN device:

```bash
sudo modprobe vcan
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
candump vcan0
```

---

## 11. Listen-Only CAN Capture

The first physical CAN experiments must be passive.

Requirements:

- Use hardware or driver settings that support listen-only/silent mode where possible.
- Do not run replay or transmit tools on the vehicle interface.
- Capture while parked.
- Record exact ignition/READY state.
- Record which physical bus and connector pins are used.
- Record bitrate only after reliable identification.
- Stop if vehicle warnings or abnormal behavior appear.

The capture machine should have no application configured to transmit on the vehicle bus.

---

## 12. OBD Connector Caution

The standardized diagnostic connector can provide access to power, ground, and one or more communication paths, but exact use must be confirmed from authoritative vehicle wiring information before attaching raw CAN hardware.

Do not assume:

- Every Prius network is exposed at the diagnostic connector.
- All networks use the same bitrate.
- A community wiring diagram matches the exact 2014 vehicle.
- An OBD breakout device is passive simply because it has no visible controls.

A fused, protected, purpose-built adapter or breakout harness is preferred.

---

## 13. Capture Metadata

Every capture file should have a matching metadata record.

Example:

```yaml
vehicle: 2014 Toyota Prius
vehicle_profile_id: rover-001
capture_time_utc: 2026-07-10T03:30:00Z
interface: can0
mode: listen-only
physical_connection: OBD breakout
bus_description: unconfirmed
bitrate: unconfirmed
vehicle_state: READY, parked
scenario: press steering-wheel volume-up five times
software_commit: abc1234
notes: no warning lights observed
```

Never store a capture without enough metadata to reproduce or interpret it.

---

## 14. Research Scenarios

Use controlled scenarios to correlate messages.

Examples:

- Ignition off to accessory.
- Accessory to READY.
- READY to off.
- Shift P to R while stationary with brake applied.
- Shift R to N or P.
- Headlights off/on.
- Dimmer adjustment.
- Steering-wheel volume-up presses.
- Steering-wheel volume-down presses.
- Door open/closed.
- Climate fan level changes.

Only perform safe stationary actions during initial capture.

---

## 15. Correlation Method

For each scenario:

1. Start a baseline capture.
2. Leave the vehicle unchanged for a defined period.
3. Perform one controlled action.
4. Return to baseline.
5. Repeat the same action several times.
6. Compare changed arbitration IDs and bytes.
7. Validate candidates against a second session.

Avoid changing multiple controls simultaneously during signal discovery.

---

## 16. Community Data Use

Community projects may accelerate research, but they are reference material, not automatic truth.

Potential references include:

- comma.ai `opendbc` vehicle definitions.
- `can-utils` examples and tooling.
- Open-source Prius telemetry projects.
- Scan-tool PID documentation.

Before using a community signal:

- Confirm supported model years.
- Confirm the bus source.
- Confirm message direction.
- Confirm scaling and signedness.
- Reproduce it on the target vehicle.
- Check the source license before copying definitions into the repository.

Rover OS should document attribution and licensing for imported database definitions.

---

## 17. DBC and Signal Files

If validated CAN signals are represented in DBC format, keep them separate from core code.

Suggested structure:

```text
hardware/vehicles/toyota-prius-gen3/
├── README.md
├── captures/
├── research/
├── dbc/
└── validation/
```

Do not commit private trip captures or sensitive location data to the public repository.

A DBC entry should not be promoted to production until its confidence level is `Production Approved`.

---

## 18. Read-Only Enforcement

Rover OS should enforce read-only policy in multiple layers:

- No transmit methods in the initial Prius CAN adapter interface.
- Open CAN sockets without application transmit paths.
- Run capture tools with least privilege.
- Separate experimental tools from production services.
- Require explicit build-time or configuration changes to enable any future transmit feature.
- Log adapter mode at startup.

The production v1 Prius adapter should not transmit arbitrary raw CAN frames.

---

## 19. Data Quality Model

Every live vehicle field should carry quality state:

- `good`
- `stale`
- `estimated`
- `invalid`
- `unavailable`

Example:

```json
{
  "hybridSocPercent": 61.5,
  "quality": {
    "hybridSocPercent": "good"
  },
  "source": {
    "hybridSocPercent": "toyota-diagnostic"
  }
}
```

If a value times out, the backend should mark it stale before eventually marking it unavailable. It should not silently continue displaying old data as current.

---

## 20. Failure Handling

The vehicle adapter must handle:

- Adapter absent at boot.
- Adapter unplugged while running.
- Serial timeouts.
- Malformed responses.
- Unsupported commands.
- Vehicle ignition off.
- Partial telemetry availability.
- Reconnection without backend restart.

OBD or CAN loss must not crash Rover OS or block the backup camera and media features.

---

## 21. Security Considerations

The diagnostic connector and vehicle networks should be treated as security-sensitive.

Requirements:

- Do not bridge raw vehicle networks to the public internet.
- Do not expose raw transmit APIs.
- Do not permit remote arbitrary diagnostic commands.
- Authenticate any future remote diagnostics workflow.
- Keep experimental capture tooling disabled in normal vehicle mode.
- Review third-party dongles and libraries before installation.

---

## 22. Proposed Adapter Capability Model

```ts
interface VehicleCapabilities {
  standardObd: boolean;
  diagnosticCodes: boolean;
  speed: boolean;
  engineRpm: boolean;
  coolantTemperature: boolean;
  controlModuleVoltage: boolean;
  gear: boolean;
  reverseState: boolean;
  readyState: boolean;
  fuelLevel: boolean;
  hybridBatterySoc: boolean;
  hybridBatteryBlocks: boolean;
  steeringWheelControls: boolean;
  illuminationState: boolean;
}
```

Capabilities should be discovered or configured per vehicle profile.

---

## 23. Research Deliverables

Before implementing the production Prius adapter, create:

- Exact vehicle and trim record.
- OBD adapter qualification report.
- Standard PID support matrix.
- Read-rate and timeout measurements.
- Passive CAN connection diagram.
- Capture metadata template.
- Signal validation table.
- Licensing review for imported definitions.
- Safety review confirming read-only behavior.

---

## 24. Initial Milestones

### Milestone 1 — Generic OBD Bench Adapter

- Connect to USB adapter.
- Detect serial device.
- Query supported standard commands.
- Read a safe snapshot.
- Handle disconnect/reconnect.

### Milestone 2 — Parked Prius OBD Validation

- Validate standard PIDs against the target vehicle.
- Record supported/unsupported commands.
- Measure stable polling rates.
- Read trouble codes without clearing them.

### Milestone 3 — Virtual CAN Development

- Configure `vcan0`.
- Parse recorded frames.
- Test adapter contracts.
- Test event normalization.

### Milestone 4 — Passive Vehicle Capture

- Use listen-only hardware.
- Capture controlled stationary scenarios.
- Record complete metadata.

### Milestone 5 — Validated Prius Signals

- Correlate selected useful signals.
- Reproduce results across sessions.
- Add source and confidence metadata.
- Expose only approved values to Rover OS.

---

## 25. Primary Technical References

Use current primary documentation during implementation:

- Linux kernel SocketCAN documentation: `https://docs.kernel.org/networking/can.html`
- Linux CAN utilities: `https://github.com/linux-can/can-utils`
- comma.ai OpenDBC repository: `https://github.com/commaai/opendbc`
- python-OBD documentation, if evaluated for a Python adapter: `https://python-obd.readthedocs.io/`
- Toyota service information and electrical wiring documentation for the exact vehicle, obtained through an authorized Toyota technical information source where available.

Community forum posts may generate hypotheses, but they are not sufficient validation by themselves.

---

## 26. Acceptance Criteria

This research phase is complete enough for a v0.x Prius adapter when:

- A qualified USB OBD-II adapter works reliably on Linux.
- Supported standard PIDs are recorded for the target vehicle.
- Polling rates and timeout behavior are documented.
- OBD disconnect/reconnect works without restarting Rover OS.
- Any Prius-specific values used by the UI have a documented source and confidence level.
- Raw CAN work remains passive and reproducible.
- No production code exposes arbitrary vehicle-network transmission.
