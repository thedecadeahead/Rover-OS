# 025 — Coding Standards

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft  

## 1. Purpose

This document defines coding standards for Rover OS. The objective is predictable, maintainable code that can be reviewed by humans and coding agents and moved between development hardware and the in-vehicle system.

## 2. General Principles

- Prefer clarity over cleverness.
- Keep modules focused.
- Make failure states explicit.
- Validate external input.
- Avoid hidden global state.
- Keep hardware-specific code behind adapters.
- Write code that runs with mocks.
- Document decisions that are not obvious.
- Do not claim a feature works when it is only stubbed.

## 3. TypeScript

Requirements:

- Enable strict mode.
- Avoid `any`; use `unknown` and validate.
- Prefer discriminated unions for state and events.
- Export stable public types from `shared/`.
- Avoid frontend imports from backend implementation modules.
- Use explicit return types on exported functions.
- Use `readonly` where mutation is not intended.

## 4. Naming

- Files and directories: consistent lowercase convention selected by package; prefer `kebab-case` for feature files.
- React components: `PascalCase`.
- Functions and variables: `camelCase`.
- Types and interfaces: `PascalCase`.
- Constants: `UPPER_SNAKE_CASE` only for true constants.
- Event names: lowercase dot notation.
- Database columns: `snake_case`.
- API JSON fields: `camelCase`.

Use domain names rather than hardware implementation names in core code.

## 5. Module Boundaries

- `frontend/` may depend on `shared/`.
- `backend/` may depend on `shared/`.
- `shared/` must not depend on frontend or backend.
- Hardware adapters belong in backend or dedicated adapter packages.
- Firmware is independent and communicates through documented protocols.
- Vehicle-specific adapters must not leak into generic UI components.

## 6. Error Handling

- Never silently swallow errors.
- Convert external-library failures into domain errors.
- Include actionable context in logs.
- Do not expose stack traces or secrets to the normal UI.
- Hardware absence should usually produce degraded state, not process termination.
- Use typed error codes defined in `shared/`.

## 7. Validation

Validate at trust boundaries:

- REST requests.
- WebSocket messages.
- Serial controller messages.
- OBD/CAN adapter output.
- Environment variables.
- Configuration files.
- Integration responses.

Prefer shared Zod schemas where contracts cross package boundaries.

## 8. Logging

Use structured logging.

Every significant log should include relevant fields such as:

- Service.
- Event type.
- Adapter ID.
- Correlation ID.
- Error code.
- Device path when appropriate.

Do not log:

- Tokens.
- Passwords.
- Full sensitive configuration.
- Unredacted location history in ordinary debug logs.

## 9. React Standards

- Prefer functional components and hooks.
- Keep route components focused on composition.
- Place domain behavior in feature hooks/services.
- Do not perform hardware or raw WebSocket work inside visual components.
- Handle loading, unavailable, degraded, and error states.
- Avoid rerendering entire screens for high-frequency telemetry.
- Use semantic HTML and accessible labels.

## 10. Backend Standards

- Keep route handlers thin.
- Put domain behavior in services.
- Put device behavior in adapters.
- Do not let adapters write directly to HTTP responses.
- Use dependency injection or factories so mocks can replace hardware.
- Support graceful startup and shutdown.
- Use transactions for multi-step database writes.

## 11. API Standards

- Follow `docs/008_API_Design.md`.
- Use the common response envelope.
- Return stable error codes.
- Use REST for commands/snapshots and WebSocket for live events.
- Avoid breaking response contracts without versioning.
- Do not expose unsafe vehicle-control endpoints.

## 12. Database Standards

- Use migrations.
- Never modify a released migration.
- Use UTC timestamps.
- Enable foreign keys.
- Back up before destructive migrations.
- Keep large media outside SQLite.
- Avoid querying raw JSON when a normalized field is frequently needed.

## 13. Tests

Every new feature should include the most appropriate tests:

- Unit test for domain logic.
- Contract/schema test for API or events.
- Component test for UI states.
- Integration test for database or adapter boundary.
- End-to-end test for critical user flows.

Bug fixes should include a regression test when practical.

## 14. Formatting and Linting

Use repository-wide automated tooling.

Recommended:

- Prettier.
- ESLint with TypeScript rules.
- EditorConfig.
- CI checks for formatting, lint, typecheck, and tests.

Do not debate formatting in code review; let the formatter decide.

## 15. Comments and Documentation

Comments should explain why, not restate what the code does.

Required documentation areas:

- Public adapter interfaces.
- Serial protocol.
- Safety-related logic.
- Shutdown timing.
- Non-obvious unit conversions.
- Workarounds for hardware or platform bugs.

Update relevant Markdown documents when architecture or behavior changes.

## 16. Units

Internal domain models should use explicit unit names.

Examples:

- `speedMph`
- `batteryVoltage`
- `coolantTempF`
- `distanceMiles`

Do not use ambiguous fields such as `speed`, `temp`, or `distance` in cross-service contracts.

A future unit-conversion layer may support metric display.

## 17. Security

- No secrets in source control.
- Bind services locally by default.
- Apply least privilege to files and processes.
- Validate file paths and uploads.
- Review third-party dependencies.
- Do not add remote execution features as convenience shortcuts.

## 18. Git Practices

- Use focused commits.
- Use imperative commit messages.
- Avoid committing generated build output unless required for deployment.
- Keep unrelated refactors separate from feature work.
- Document breaking changes.
- Do not force-push shared branches without agreement.

## 19. Definition of Done

A change is done when:

- Behavior matches the relevant specification.
- Types pass.
- Lint and formatting pass.
- Tests pass.
- Failure states are handled.
- Documentation is updated.
- No secrets or machine-specific files are committed.
- The feature works in mock/development mode when hardware is not required.
