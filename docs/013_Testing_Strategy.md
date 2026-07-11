# 013 — Testing Strategy

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft  

---

## 1. Purpose

This document defines the testing strategy for ARGO Rover OS. Rover OS combines user interface software, backend services, local data storage, hardware adapters, vehicle data, power-management behavior, and optional remote integrations. Testing must therefore cover both ordinary software correctness and failures unique to embedded vehicle use.

The project should be testable without the Prius, while still having a disciplined path for bench and in-vehicle validation.

---

## 2. Testing Principles

- Test core behavior without requiring hardware.
- Make mock adapters first-class implementations.
- Separate deterministic automated tests from hardware qualification.
- Test degraded states, not only happy paths.
- Test startup, shutdown, and reconnection repeatedly.
- Never use a moving vehicle as the first test environment.
- Keep vehicle-facing write operations out of scope unless explicitly approved.
- Record test evidence for installation-critical behavior.

---

## 3. Test Pyramid

Rover OS should use multiple test levels.

```text
                  In-Vehicle Validation
                Bench Integration Tests
             End-to-End Application Tests
          Frontend / Backend Integration Tests
       Component, Service, and Adapter Unit Tests
```

Most tests should run at the lower levels because they are faster, safer, and easier to repeat.

---

## 4. Unit Testing

### 4.1 Backend Unit Tests

Backend unit tests should cover:

- Settings validation.
- Service health state transitions.
- Vehicle telemetry normalization.
- Unit conversions.
- Trip calculations.
- Maintenance due calculations.
- Database repository behavior.
- Event envelope creation.
- Error mapping.
- Retry and timeout logic.
- Shutdown coordinator state machine.

Recommended framework:

- Vitest.

### 4.2 Frontend Unit Tests

Frontend unit tests should cover:

- State stores.
- Event reducers.
- Formatting utilities.
- Driving-mode logic.
- Capability checks.
- Notification deduplication.
- Route access rules.
- Theme selection.

### 4.3 Firmware Unit Tests

Future vehicle I/O controller firmware should test:

- Input debounce.
- Ignition-state transitions.
- Reverse signal handling.
- Watchdog behavior.
- Serial protocol parsing.
- Shutdown timeout logic.

Firmware tests may use host-side test frameworks or hardware-in-loop tests depending on the platform.

---

## 5. Component Testing

React component tests should use React Testing Library or an equivalent user-focused tool.

Required component scenarios:

- Normal data.
- Missing data.
- Stale data.
- Hardware unavailable.
- Service degraded.
- Critical warning.
- Day theme.
- Night theme.
- Large text mode.

Important components:

- Dashboard cards.
- Telemetry values.
- Status badges.
- Camera failure view.
- Media controls.
- Diagnostics code list.
- Shutdown overlay.
- Notification center.

---

## 6. API Integration Testing

Backend API tests should run against a temporary test database.

Test:

- Common success envelope.
- Common error envelope.
- Request validation.
- Settings read/update.
- Vehicle snapshot.
- Vehicle capabilities.
- Diagnostics status.
- Camera listing.
- Maintenance CRUD.
- Trip history.
- Shutdown request authorization.

Each test should reset or isolate database state.

---

## 7. WebSocket and Event Testing

Test the event system for:

- Valid event envelopes.
- Event versioning.
- Subscription behavior.
- Reconnection.
- Snapshot refresh after reconnection.
- Slow-client handling.
- Telemetry coalescing.
- Critical event preservation.
- Invalid adapter payload rejection.
- One handler failing without breaking other handlers.

Use scripted event fixtures for deterministic tests.

---

## 8. Database Testing

Database tests should cover:

- Clean migration from empty database.
- Migration from previous released schema.
- Foreign key enforcement.
- Vehicle profile activation.
- Trip start and completion.
- Diagnostic session persistence.
- Maintenance scheduling.
- Retention jobs.
- Backup and restore.
- Interrupted transaction recovery.

Destructive migration tests should run only against disposable test databases.

---

## 9. Mock Scenario Testing

Every major user flow should have at least one named mock scenario.

Required scenarios:

- Parked and healthy.
- City driving.
- Highway driving.
- Reverse selected.
- Rear camera unavailable.
- OBD-II disconnected.
- Network offline.
- ARGO Core offline.
- 12V voltage warning.
- Storage nearly full.
- Ignition off and shutdown countdown.
- Backend restart and UI reconnect.

Mock scenarios should be reusable in manual testing and automated end-to-end tests.

---

## 10. End-to-End Testing

Recommended framework:

- Playwright.

Required v0.1 flows:

1. Boot application into dashboard.
2. Receive mock telemetry.
3. Navigate between primary screens.
4. Update and persist settings.
5. Trigger reverse-camera override.
6. Simulate camera failure.
7. Lose and restore WebSocket connection.
8. Switch from parked to driving mode.
9. Verify restricted controls in driving mode.
10. Trigger shutdown countdown.

Tests should use a controlled mock backend rather than external services.

---

## 11. Visual Regression Testing

Visual regression testing is useful for a fixed display target.

Capture key screens at:

- 1920x1080 day theme.
- 1920x1080 night theme.
- Driving mode.
- Parked mode.
- Critical alert.
- Rear camera overlay.
- Offline state.

Visual tests should allow small rendering tolerances and avoid unstable dynamic timestamps where possible.

---

## 12. Performance Testing

Performance tests should measure:

- Time from frontend load to usable dashboard.
- Time from WebSocket event to visible update.
- Route transition latency.
- Reverse event to camera overlay latency.
- CPU and memory use on Raspberry Pi 5.
- Event throughput.
- Database write overhead.
- Camera stream impact.

Performance targets should be finalized after the first Pi bench build.

Initial desired outcomes:

- UI remains responsive during telemetry updates.
- Camera switching feels immediate.
- Long-running operation does not leak memory continuously.

---

## 13. Reliability and Soak Testing

Bench soak tests should run Rover OS continuously for increasing durations:

- 2 hours.
- 8 hours.
- 24 hours.
- 72 hours before daily-driver status.

Monitor:

- Memory growth.
- CPU temperature.
- USB disconnects.
- WebSocket stability.
- Database errors.
- Log growth.
- Camera stability.
- Audio stability.

A soak test should include repeated simulated ignition and reverse events.

---

## 14. Power-Cycle Testing

Power-cycle testing is essential.

Test cases:

- Normal ignition on.
- Normal ignition off.
- Rapid on/off cycle.
- Power restored during shutdown countdown.
- Abrupt power loss.
- Recovery after abrupt loss.
- Low-voltage shutdown.
- USB peripheral reconnect after boot.

Record whether:

- Filesystem remains healthy.
- Database opens successfully.
- Services restart.
- UI reconnects.
- No manual login is required.

---

## 15. Hardware Adapter Testing

Each hardware adapter should have a contract test suite.

Required adapter contracts:

- Start.
- Stop.
- Health state.
- Capabilities.
- Current snapshot.
- Event subscription.
- Error behavior.

Mock and real adapters should satisfy the same contract where applicable.

---

## 16. Camera Testing

Camera validation should cover:

- Device discovery.
- Stream start.
- Stream stop.
- Reverse-trigger switching.
- Manual switching.
- Device disconnect.
- Device reconnect.
- Black or frozen frames.
- Low-light usability.
- Boot-time availability.

Do not treat a successful one-time preview as sufficient qualification.

---

## 17. Audio Testing

Audio testing should cover:

- Playback.
- Volume changes.
- Mute.
- Navigation prompts.
- System sounds.
- Source switching.
- Reboot recovery.
- USB DAC disconnect/reconnect.
- Noise, hum, and alternator-related interference.
- Factory JBL amplifier wake behavior if retained.

Subjective audio quality notes should be recorded during vehicle tests.

---

## 18. OBD-II and CAN Testing

Initial OBD-II tests should be read-only.

Validate:

- Adapter discovery.
- Connection timeout.
- Supported PID detection.
- Standard PID reads.
- Diagnostic trouble code reads.
- Adapter disconnect/reconnect.
- No uncontrolled request flooding.

CAN testing rules:

- Use a virtual CAN interface first.
- Use recorded logs second.
- Use listen-only vehicle capture third.
- No active transmission during initial research.
- Document bus, bitrate, connector pins, and capture conditions.

---

## 19. Vehicle I/O Controller Testing

Before connecting to the Prius, test the controller with simulated voltage inputs.

Validate:

- Input voltage conditioning.
- Ignition event timing.
- Reverse event timing.
- Serial message integrity.
- Loss of host connection.
- Host shutdown acknowledgement.
- Delayed power cutoff.
- Watchdog recovery.

Automotive voltage inputs must not be connected directly to low-voltage GPIO without proper interface circuitry.

---

## 20. Security Testing

Security testing should cover:

- APIs bound to localhost by default.
- Secrets absent from frontend bundles.
- Secrets absent from logs.
- Invalid API payload rejection.
- Authentication for any enabled remote access.
- File permissions for database and media.
- Dependency vulnerability review.
- No default public debug endpoints.

Remote vehicle control remains out of scope.

---

## 21. In-Vehicle Test Phases

### Phase 1 — Parked, Vehicle Off

- Verify physical installation.
- Verify no safety controls are obstructed.
- Verify manual power and display behavior.

### Phase 2 — Parked, Vehicle READY

- Verify regulated power.
- Verify audio.
- Verify OBD-II reads.
- Verify camera.
- Monitor heat and electrical noise.

### Phase 3 — Controlled Low-Speed Test

- Use a safe private area.
- Validate UI readability.
- Validate steering wheel controls.
- Validate reverse camera.
- Do not debug while driving.

### Phase 4 — Normal Road Validation

- A passenger records issues where possible.
- Driver focuses on driving.
- Validate daytime/nighttime visibility.
- Validate startup across repeated trips.

---

## 22. Release Gates

### v0.1 Desktop Alpha

Must pass:

- Lint.
- Type check.
- Unit tests.
- API integration tests.
- Core Playwright flows.

### v0.2 Bench Alpha

Must additionally pass:

- Pi build and startup.
- Touchscreen test.
- Camera test.
- Audio test.
- 24-hour soak test.

### v0.3 Vehicle Prototype

Must additionally pass:

- Safe power test.
- Graceful shutdown test.
- Parked vehicle integration test.
- Reverse-camera test.
- OBD-II reconnect test.

### v1.0 Daily Driver

Must additionally pass:

- 72-hour soak.
- Repeated power-cycle qualification.
- Day/night usability.
- No unresolved critical safety or data-loss defects.
- Documented installation and recovery procedure.

---

## 23. Defect Severity

- **Critical:** safety risk, data corruption, inability to shut down, or system unusable.
- **High:** core head-unit feature unreliable, such as audio or backup camera.
- **Medium:** important feature degraded with workaround.
- **Low:** cosmetic or minor usability issue.

Critical and high defects block daily-driver release.

---

## 24. Test Evidence

For hardware and vehicle qualification, record:

- Hardware revision.
- Software commit.
- Configuration.
- Test date.
- Test duration.
- Result.
- Logs.
- Photos or video where useful.
- Follow-up issue links.

---

## 25. Acceptance Criteria

The testing strategy is implemented adequately for v0.1 when:

- CI runs lint, type-check, tests, and builds.
- Mock scenarios cover the major screens and failures.
- API and event contracts have automated tests.
- Core Playwright flows run without hardware.
- Bench and vehicle tests have documented checklists.
