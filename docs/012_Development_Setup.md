# 012 — Development Setup

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft  

---

## 1. Purpose

This document defines the initial local development workflow for ARGO Rover OS. The project must be easy to run on a normal development computer before any Raspberry Pi, touchscreen, OBD-II adapter, camera, or Prius hardware is available.

The default developer experience should use mock services and one command to start the frontend and backend together.

---

## 2. Development Goals

The development environment should:

- Run on macOS, Linux, and Windows where practical.
- Require no vehicle hardware for normal feature work.
- Use mock vehicle, camera, network, and ARGO services.
- Support fast frontend refresh.
- Support backend restart on code changes.
- Share API and event types across packages.
- Provide deterministic test scenarios.
- Keep secrets outside source control.
- Match the production architecture closely enough to avoid deployment surprises.

---

## 3. Recommended Repository Layout

```text
Rover-OS/
├── frontend/
├── backend/
├── shared/
├── firmware/
├── hardware/
├── docs/
├── scripts/
├── data/
│   ├── mock/
│   └── development/
├── .github/
├── package.json
├── pnpm-workspace.yaml
├── tsconfig.base.json
├── .env.example
└── README.md
```

The project should use a JavaScript/TypeScript monorepo.

Recommended package manager:

- pnpm

Alternative:

- npm workspaces

The initial implementation should choose one and document it consistently.

---

## 4. Prerequisites

Recommended developer prerequisites:

- Git.
- Node.js current LTS.
- pnpm.
- A modern Chromium-based browser.
- SQLite command-line tools, optional but useful.
- Docker, optional for isolated tooling or future services.

Hardware-development prerequisites may later include:

- Python 3.
- `can-utils` on Linux.
- Serial terminal software.
- Raspberry Pi Imager.
- PlatformIO or Arduino tooling for microcontroller firmware.

---

## 5. Node Version Management

The repository should pin the expected Node.js major version.

Recommended files:

- `.nvmrc`
- `package.json` `engines` field
- Optional Volta configuration

Example:

```text
22
```

The exact LTS version should be selected when implementation begins and updated deliberately.

---

## 6. Workspace Packages

Recommended packages:

```text
@rover/frontend
@rover/backend
@rover/shared
```

Potential future packages:

```text
@rover/vehicle-mock
@rover/vehicle-generic-obd2
@rover/vehicle-toyota-prius-gen3
@rover/plugin-sdk
```

---

## 7. Environment Configuration

The repository should include `.env.example` with non-secret defaults.

Example variables:

```dotenv
ROVER_MODE=mock
ROVER_HTTP_HOST=127.0.0.1
ROVER_HTTP_PORT=3746
ROVER_DATABASE_PATH=./data/development/rover.db
ROVER_LOG_LEVEL=debug
ROVER_VEHICLE_ADAPTER=mock-vehicle
ROVER_ARGO_ENABLED=false
ROVER_HOME_ASSISTANT_ENABLED=false
```

Rules:

- Never commit `.env`.
- Never commit access tokens.
- Validate environment variables at backend startup.
- Log the active mode without logging secrets.

---

## 8. Installation Workflow

Target developer workflow:

```bash
git clone https://github.com/thedecadeahead/Rover-OS.git
cd Rover-OS
pnpm install
cp .env.example .env
pnpm dev
```

The root `pnpm dev` command should start:

- Backend development server.
- Frontend Vite development server.
- Mock vehicle scenario.

---

## 9. Root Scripts

Recommended root scripts:

```json
{
  "scripts": {
    "dev": "run frontend and backend development servers",
    "build": "build all packages",
    "test": "run all tests",
    "test:unit": "run unit tests",
    "test:e2e": "run end-to-end tests",
    "lint": "lint all packages",
    "typecheck": "type-check all packages",
    "format": "format repository files",
    "db:migrate": "apply database migrations",
    "db:reset": "reset development database",
    "mock:record": "record mock event fixture",
    "mock:replay": "replay mock event fixture"
  }
}
```

Implementation may use `concurrently`, Turborepo, or pnpm recursive commands.

---

## 10. Mock Development Mode

Mock mode is the default development mode.

It should simulate:

- Vehicle speed.
- Gear state.
- Fuel level.
- 12V battery voltage.
- Hybrid state of charge.
- Reverse activation.
- Ignition on/off.
- Camera availability.
- Network availability.
- ARGO Core availability.
- Service failures.

Mock mode should support named scenarios.

Example scenarios:

- `parked-idle`
- `city-drive`
- `highway-drive`
- `reverse-camera`
- `obd-disconnected`
- `camera-failure`
- `argo-offline`
- `low-voltage-shutdown`

---

## 11. Mock Scenario Format

Mock scenarios should be stored as JSON or TypeScript fixtures.

Example:

```json
{
  "name": "city-drive",
  "durationSeconds": 120,
  "events": [
    {
      "atSeconds": 0,
      "type": "vehicle.ignition_changed",
      "payload": { "state": "on" }
    },
    {
      "atSeconds": 5,
      "type": "vehicle.telemetry",
      "payload": { "speedMph": 22, "gear": "D" }
    }
  ]
}
```

The exact format should support repeatable automated tests.

---

## 12. Frontend Development

The frontend should run through Vite in development mode.

Requirements:

- Hot module replacement.
- Proxy API calls to local backend.
- Configurable backend URL.
- Mock mode indicator in developer builds.
- Responsive preview for the target 1920x1080 display.

Recommended viewport presets:

- 1920x1080 primary vehicle display.
- 1280x720 lower-resolution test.
- Laptop viewport for normal development.

---

## 13. Backend Development

The backend development server should:

- Restart automatically on source changes.
- Use a local SQLite database.
- Load mock adapters by default.
- Emit structured console logs.
- Validate configuration at startup.
- Provide a health endpoint.

Suggested development command:

```bash
pnpm --filter @rover/backend dev
```

---

## 14. Shared Package Development

The `shared` package should contain API and event contracts.

It should export:

- TypeScript types.
- Runtime validation schemas.
- Event constants.
- Error codes.
- Units and formatting contracts where appropriate.

Frontend and backend should both depend on the package through the workspace.

---

## 15. Database Setup

Development startup should automatically create the development database if missing.

Required commands:

```bash
pnpm db:migrate
pnpm db:reset
```

`db:reset` must only operate on development/test databases and should require an explicit environment check.

Seed data should include:

- Rover vehicle profile.
- Default settings.
- Example maintenance records.
- Optional sample trip history.

---

## 16. Code Quality

Recommended tools:

- ESLint.
- Prettier.
- TypeScript strict mode.
- Husky or another optional Git-hook manager.
- lint-staged, optional.

Required checks before merging implementation work:

- Formatting.
- Linting.
- Type checking.
- Unit tests.
- Production build.

---

## 17. Logging

Development logs should be readable and structured.

Each backend log should include where relevant:

- Timestamp.
- Level.
- Service.
- Event or operation.
- Error code.
- Correlation ID.

Sensitive tokens and GPS history should not be printed casually in development logs.

---

## 18. Hardware Development Mode

Hardware development should be opt-in.

Example configuration:

```dotenv
ROVER_MODE=bench
ROVER_VEHICLE_ADAPTER=generic-obd2
ROVER_OBD_DEVICE=/dev/ttyUSB0
ROVER_CAMERA_REAR=/dev/video0
```

The backend should fail individual hardware services gracefully if devices are missing.

The entire application should not fail to start because one optional USB device is absent.

---

## 19. Linux Permissions

Bench and vehicle mode may require access to:

- Serial ports.
- Video devices.
- SocketCAN interfaces.
- Audio devices.
- GPIO or USB serial controller.

Use Linux groups and udev rules rather than running the whole application as root.

Potential groups:

- `dialout`
- `video`
- `audio`

Exact rules should be documented with each hardware adapter.

---

## 20. Git Workflow

Recommended workflow:

- `main` remains buildable.
- Use feature branches for implementation.
- Keep commits focused.
- Use pull requests for substantial code changes.
- Reference relevant documentation or issues.

Suggested branch names:

- `feat/mock-vehicle-service`
- `feat/dashboard-shell`
- `hardware/obd-usb-adapter`
- `docs/power-management`
- `fix/websocket-reconnect`

---

## 21. GitHub Actions

Initial CI should run on pushes and pull requests.

Required jobs:

- Install dependencies.
- Lint.
- Type check.
- Unit tests.
- Build frontend.
- Build backend.

Later jobs may include:

- Playwright tests.
- Raspberry Pi cross-build or ARM runner.
- Container build.
- Release artifact creation.

---

## 22. Developer Onboarding Checklist

A new developer should be able to:

1. Clone the repository.
2. Install dependencies.
3. Copy `.env.example`.
4. Run migrations.
5. Start mock mode.
6. See the dashboard.
7. Trigger reverse-camera simulation.
8. Run tests.
9. Build production assets.

No physical car should be required.

---

## 23. Acceptance Criteria

The development setup is acceptable for v0.1 when:

- A clean clone can be started with documented commands.
- Frontend and backend start together.
- Mock telemetry appears on the dashboard.
- The development database is created automatically or through one command.
- Lint, type-check, test, and build commands work from the repository root.
- Hardware access is optional.
