# 024 — Claude Build Prompt

Use this document as the primary implementation prompt for Claude Code or another coding agent working in this repository.

---

## Role

You are the lead software engineer implementing ARGO Rover OS from the specifications in this repository.

Your job is to build a reliable, modular, local-first vehicle operating environment that initially runs on a development computer with mock data and later deploys to a Raspberry Pi 5 installed in a 2014 Toyota Prius.

Do not treat this as a generic dashboard demo. Build production-quality foundations suitable for gradual hardware integration.

---

## Required Reading Order

Before writing code, read:

1. `README.md`
2. `docs/001_Project_Vision.md`
3. `docs/002_Product_Requirements.md`
4. `docs/003_System_Architecture.md`
5. `docs/006_UI_UX_Guide.md`
6. `docs/007_Backend_Architecture.md`
7. `docs/008_API_Design.md`
8. `docs/009_Database_Schema.md`
9. `docs/010_Event_System.md`
10. `docs/011_Frontend_Architecture.md`
11. `docs/012_Development_Setup.md`
12. `docs/013_Testing_Strategy.md`
13. `docs/016_Security_and_Privacy.md`
14. `docs/020_Hardware_Abstraction_Layer.md`
15. `docs/023_Roadmap.md`

Read hardware-specific documents before implementing related adapters.

---

## Immediate Assignment

Implement Phase 1 and the minimum viable portion of Phase 2:

- TypeScript monorepo.
- React/Vite frontend.
- Node.js/Fastify backend.
- Shared TypeScript types and Zod schemas.
- SQLite migration foundation.
- Mock vehicle adapter.
- REST system status and settings endpoints.
- WebSocket event stream.
- Basic dashboard and route shell.
- Service-health model.
- Automated tests.
- Linting, formatting, and CI.

The result must run on a normal laptop without vehicle hardware.

---

## Required Repository Structure

Use a workspace-based monorepo:

```text
frontend/
backend/
shared/
firmware/
scripts/
docs/
hardware/
.github/workflows/
```

Use the package manager specified by the repository setup documents. If none is already configured, prefer pnpm workspaces.

---

## Architecture Rules

- Frontend must never access vehicle hardware directly.
- Backend hardware access must use interfaces from the hardware-abstraction design.
- Every real adapter must have a mock equivalent.
- Prius-specific behavior must not be embedded in generic services.
- ARGO Core and internet access must remain optional.
- A failed optional subsystem must not crash the application.
- Do not implement vehicle-control writes.
- Do not implement braking, steering, throttle, airbag, ABS, traction-control, or hybrid high-voltage control.

---

## Frontend Requirements

Build:

- Application shell.
- Dashboard route.
- Navigation, Media, Cameras, Diagnostics, Maintenance, Home, Projects, and Settings placeholder routes.
- Persistent status area.
- Primary navigation.
- Connection/degraded-state indicators.
- Mock driving and parked mode.
- Reverse-camera override simulation.

Use large touch targets and the UI guidelines in `docs/006_UI_UX_Guide.md`.

Do not expose a generic desktop or app-launcher experience.

---

## Backend Requirements

Build:

- Fastify server.
- `/health/live`.
- `/health/ready`.
- `/api/system/status`.
- `/api/settings` GET/PATCH.
- `/api/vehicle/snapshot`.
- `/api/vehicle/capabilities`.
- WebSocket endpoint at `/events`.
- In-process typed event bus.
- Mock vehicle service.
- Service-health registry.
- Structured logging.
- Graceful process shutdown.

Default bind address should be localhost unless development configuration explicitly changes it.

---

## Shared Package Requirements

Create shared schemas/types for:

- Common API response envelope.
- Error envelope and error codes.
- Rover event envelope.
- Vehicle snapshot.
- Vehicle capabilities.
- Vehicle telemetry event.
- Service health.
- Settings.
- Interaction mode.

Use Zod for runtime validation and infer TypeScript types where practical.

---

## Mock Vehicle Requirements

The mock adapter must simulate:

- Speed.
- Gear.
- Fuel percentage.
- Range.
- 12V voltage.
- Coolant temperature.
- Connection state.
- Ignition state.
- Reverse state.
- Day/night illumination state.
- Configurable service failures.

Provide a development control mechanism for changing mock state.

Do not require the frontend to know that the source is mock beyond a visible development indicator.

---

## Database Requirements

Use SQLite with migrations.

For the initial implementation, create at minimum:

- `system_settings`.
- `vehicle_profiles`.
- `service_health_events`.
- `system_events`.

Seed a default mock vehicle profile in development mode.

Do not store secrets in the database without encryption.

---

## Testing Requirements

Include:

- Shared-schema tests.
- Backend endpoint tests.
- Settings persistence tests.
- Event-envelope validation tests.
- Mock adapter tests.
- Frontend component tests for missing/degraded data.
- At least one end-to-end smoke test for boot to dashboard.

All tests must run without hardware.

---

## Quality Requirements

- TypeScript strict mode.
- No unexplained `any` types.
- Clear error handling.
- Structured logs.
- Useful comments for non-obvious decisions.
- No secrets committed.
- No placeholder code that silently pretends to work.
- README instructions kept current.
- Small, reviewable commits when operating with Git access.

---

## Implementation Order

1. Inspect repository and existing files.
2. Propose a concise implementation plan.
3. Create monorepo and tooling.
4. Create shared schemas.
5. Create backend health/status foundation.
6. Create mock adapter and event bus.
7. Create frontend shell and dashboard.
8. Connect REST and WebSocket clients.
9. Add SQLite settings persistence.
10. Add tests and CI.
11. Update README with exact commands.
12. Report completed work, test results, and remaining gaps.

Do not skip tests to make the UI appear complete.

---

## Completion Criteria

The assignment is complete when a fresh clone can:

```bash
pnpm install
pnpm dev
```

and produce:

- A running backend.
- A running frontend.
- A dashboard receiving live mock telemetry.
- Working settings persistence.
- Visible service-health and connection states.
- Passing type checks, lint, and tests.

Document any deviations from the specifications and explain why they were necessary.
