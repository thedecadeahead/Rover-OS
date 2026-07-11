# 023 — Project Roadmap

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Active Planning  

## 1. Purpose

This roadmap converts the project vision into staged, testable milestones. Each milestone should leave Rover OS usable and documented rather than depending on one large final integration.

## 2. Phase 0 — Documentation Foundation

Status: in progress.

Deliverables:

- Product vision and requirements.
- System, frontend, backend, API, database, and event architecture.
- Hardware, Prius integration, power, audio, camera, security, and HAL specifications.
- Development, testing, deployment, recovery, and roadmap documents.
- Claude build prompt.

Exit criteria:

- A developer can understand the intended system without this chat.
- Major safety and architectural boundaries are explicit.

## 3. Phase 1 — Monorepo Scaffold

Deliverables:

- Package-manager workspace.
- `frontend/`, `backend/`, and `shared/` packages.
- TypeScript configuration.
- Linting and formatting.
- Environment example.
- Basic CI.
- Development scripts.

Exit criteria:

- One command installs dependencies.
- One command starts frontend and backend.
- CI validates formatting, types, and tests.

## 4. Phase 2 — Desktop Mock Prototype

Deliverables:

- Dashboard shell.
- Primary routes.
- Mock vehicle adapter.
- REST status/settings APIs.
- WebSocket telemetry.
- Service-health model.
- SQLite settings persistence.
- Driving/parked mode simulation.

Exit criteria:

- Rover OS runs on a laptop without hardware.
- Reverse-camera override can be simulated.
- Hardware failures can be simulated.

## 5. Phase 3 — Raspberry Pi Bench Prototype

Deliverables:

- Pi installation/deployment scripts.
- Chromium kiosk mode.
- 13.3-inch touchscreen support.
- NVMe boot/storage.
- USB DAC output.
- Test camera input.
- USB OBD recognition.
- systemd units.

Exit criteria:

- Pi boots directly into Rover OS.
- Touch, audio, camera, and mock telemetry operate together.
- Reboot and shutdown are reliable.

## 6. Phase 4 — Power and Vehicle-I/O Bench

Deliverables:

- Automotive DC-DC supply selected.
- Vehicle I/O controller prototype.
- Ignition and reverse test inputs.
- Graceful shutdown handshake.
- Voltage monitoring.
- Watchdog tests.

Exit criteria:

- Simulated ignition reliably starts and stops the system.
- Forced failures do not cause uncontrolled reboot loops.

## 7. Phase 5 — Non-Invasive Prius Prototype

Deliverables:

- Temporary secure display mounting.
- Fused vehicle power.
- OBD-II telemetry.
- Driver-phone hotspot connection.
- Temporary audio path.
- Manual camera view.

Exit criteria:

- Rover can be road-tested without replacing all factory functions.
- Wiring remains reversible and documented.

## 8. Phase 6 — Head-Unit Feature Parity

Deliverables:

- Factory/JBL audio integration.
- Reliable volume control.
- Steering-wheel controls.
- Rear camera and reverse trigger.
- Navigation strategy implemented.
- Bluetooth/phone strategy implemented.
- Day/night behavior.

Exit criteria:

- Rover can replace a normal aftermarket head unit for daily use.

## 9. Phase 7 — Vehicle Intelligence

Deliverables:

- Trip logging.
- Fuel-economy analytics.
- Diagnostic-code history.
- Maintenance tracking.
- Prius-specific read-only metrics where verified.
- GPS logging if enabled.

Exit criteria:

- Rover provides useful information beyond a commercial head unit.

## 10. Phase 8 — ARGO Integration

Deliverables:

- ARGO Core health/status.
- Authenticated sync.
- ARGO Vault trip/log backup.
- Home Assistant summary and parked controls.
- Remote-task hooks.

Exit criteria:

- Rover remains fully usable with Core offline.
- Sync resumes safely after reconnection.

## 11. Phase 9 — Dashcam and Security Features

Deliverables:

- Front/rear recording strategy.
- Rolling storage retention.
- Protected clips.
- Event/location metadata.
- Optional upload to Vault.
- Parked security design, subject to power budget.

Exit criteria:

- Recording does not impair UI or storage reliability.

## 12. Phase 10 — Daily-Driver Hardening

Deliverables:

- Finished bezel/mount.
- Thermal validation.
- Vibration and cable validation.
- Recovery testing.
- Signed/versioned releases.
- Update and rollback process.
- Installation manual.

Exit criteria:

- Four weeks of daily use without a critical reliability failure.
- Recovery procedures have been tested, not merely documented.

## 13. Phase 11 — Platform Expansion

Potential work:

- Mini-PC compute target.
- Additional vehicle adapters.
- Plugin SDK.
- Fleet management.
- Contributor documentation.

## 14. Immediate Next Actions

1. Finish remaining foundation and repository-support documents.
2. Create Claude implementation prompt.
3. Scaffold the TypeScript monorepo.
4. Implement shared schemas and mock telemetry.
5. Build the first dashboard prototype.

## 15. Release Targets

- `v0.1.0`: desktop mock prototype.
- `v0.2.0`: Pi bench prototype.
- `v0.3.0`: non-invasive Prius prototype.
- `v0.5.0`: integrated audio/camera/controls.
- `v1.0.0`: daily-driver release.

Dates should not be assigned until hardware availability and development capacity are known.
