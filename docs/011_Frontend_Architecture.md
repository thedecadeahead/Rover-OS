# 011 — Frontend Architecture

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft  
**Frontend Stack:** React + TypeScript + Vite  

---

## 1. Purpose

This document defines the frontend architecture for ARGO Rover OS. The frontend is the touchscreen interface presented to the driver and passenger. It must remain responsive, readable, safe, and functional even when vehicle hardware, internet access, or optional ARGO services are unavailable.

The frontend should behave like an embedded vehicle interface, not a generic website.

---

## 2. Frontend Goals

The frontend must:

- Run in Chromium kiosk mode on Raspberry Pi 5.
- Run in a normal desktop browser for development.
- Consume backend REST APIs and WebSocket events.
- Work with real or mock vehicle data.
- Tolerate partial and missing telemetry.
- Show degraded subsystem states clearly.
- Support driving and parked interaction modes.
- Avoid direct hardware access.
- Remain portable to future ARM or x86 hardware.

---

## 3. Recommended Stack

Initial stack:

- React.
- TypeScript.
- Vite.
- React Router.
- TanStack Query for REST server state.
- Zustand or Redux Toolkit for local/live application state.
- Zod for runtime payload validation where shared schemas are practical.
- CSS Modules, Tailwind CSS, or a token-driven styling system.
- Vitest and React Testing Library.
- Playwright for end-to-end testing.

The implementation should avoid unnecessary framework complexity during v0.1.

---

## 4. Frontend Directory Structure

Recommended structure:

```text
frontend/
├── src/
│   ├── app/
│   │   ├── App.tsx
│   │   ├── router.tsx
│   │   └── providers.tsx
│   ├── components/
│   │   ├── controls/
│   │   ├── layout/
│   │   ├── status/
│   │   └── feedback/
│   ├── features/
│   │   ├── dashboard/
│   │   ├── navigation/
│   │   ├── media/
│   │   ├── cameras/
│   │   ├── diagnostics/
│   │   ├── maintenance/
│   │   ├── home/
│   │   ├── projects/
│   │   └── settings/
│   ├── services/
│   │   ├── apiClient.ts
│   │   ├── eventClient.ts
│   │   └── bootstrap.ts
│   ├── state/
│   ├── hooks/
│   ├── types/
│   ├── styles/
│   ├── assets/
│   └── main.tsx
├── public/
├── tests/
├── package.json
└── vite.config.ts
```

Feature-specific components, hooks, and types should stay inside their feature directory unless broadly reused.

---

## 5. Application Shell

The application shell should provide:

- Persistent status region.
- Main content region.
- Primary navigation rail or dock.
- Global notification layer.
- Camera override layer.
- Shutdown overlay.
- Error boundary.

Suggested hierarchy:

```text
App
├── GlobalErrorBoundary
├── RoverProviders
├── ApplicationShell
│   ├── StatusBar
│   ├── MainRouterOutlet
│   ├── PrimaryNavigation
│   ├── NotificationCenter
│   ├── CameraOverride
│   └── PowerOverlay
```

---

## 6. Routing

Recommended initial routes:

```text
/
/navigation
/media
/cameras
/diagnostics
/maintenance
/home
/projects
/settings
/settings/display
/settings/audio
/settings/vehicle
/settings/network
/settings/argo
/settings/developer
```

The root route should render the dashboard.

Route loading should not block global status updates or reverse-camera behavior.

---

## 7. State Categories

Frontend state should be divided into three categories.

### 7.1 Server State

Data fetched and cached through REST:

- Settings.
- Vehicle capabilities.
- Maintenance records.
- Trip history.
- Diagnostic code history.
- Camera configuration.

Use TanStack Query or an equivalent request cache.

### 7.2 Live State

Data received from WebSocket events:

- Vehicle telemetry.
- Media state.
- Service health.
- Network status.
- Camera status.
- Power state.
- ARGO status.

Use a small normalized live-state store.

### 7.3 Local UI State

Transient interface state:

- Open panel.
- Selected camera.
- Temporary dialog.
- Navigation rail expansion.
- User-dismissed notification.

Do not persist transient UI state unless it improves startup continuity.

---

## 8. Bootstrap Process

Frontend startup should follow this sequence:

1. Render a lightweight branded boot screen.
2. Load local frontend assets.
3. Connect to backend WebSocket.
4. Request bootstrap or snapshot APIs.
5. Populate settings and current state.
6. Render dashboard.
7. Continue loading noncritical history and integrations.

The user should not wait for ARGO Core, weather, or internet before seeing the local dashboard.

---

## 9. WebSocket Client

The event client should:

- Maintain one primary WebSocket connection.
- Parse and validate event envelopes.
- Dispatch events to typed handlers.
- Reconnect automatically.
- Expose connection health.
- Ignore unsupported event versions safely.
- Prevent one bad handler from breaking all event processing.

Reconnect behavior:

- Immediate first retry.
- Exponential backoff.
- Maximum retry delay around 30 seconds.
- Refresh REST snapshots after reconnection.

---

## 10. REST Client

The REST client should:

- Use a configurable backend base URL.
- Default to the local backend.
- Parse the common response envelope.
- Throw typed application errors.
- Support cancellation and timeouts.
- Avoid infinite retries for hardware-unavailable responses.

Error categories should map to user-friendly states rather than exposing raw stack traces.

---

## 11. Shared Types

The frontend and backend should share stable API and event types through a `shared/` package.

Recommended shared content:

- Event envelope types.
- Vehicle snapshot type.
- Service health type.
- Settings schemas.
- API response types.
- Error codes.
- Adapter capability types.

Do not share backend implementation classes or database entities directly with the frontend.

---

## 12. Dashboard Feature

The dashboard should be composed of independent cards or regions.

Suggested components:

- `ClockCard`
- `VehicleSummaryCard`
- `NavigationCard`
- `MediaCard`
- `ConnectivityCard`
- `ArgoStatusCard`
- `TripSummaryCard`
- `MaintenanceAlertCard`
- `QuickActions`

Each card should handle missing data gracefully.

Example:

- Fuel unavailable → show `Unavailable`, not `0%`.
- ARGO offline → show local operation continues.
- OBD disconnected → show degraded diagnostics status without breaking dashboard.

---

## 13. Driving Mode Architecture

Driving mode should be derived from normalized backend state.

Possible inputs:

- Speed greater than zero.
- Gear state.
- Parking brake state.
- Vehicle adapter driving-state flag.

The frontend should consume a single state such as:

```ts
type InteractionMode = "driving" | "parked" | "unknown";
```

Driving mode effects:

- Hide or disable text entry.
- Simplify navigation.
- Increase touch target size.
- Suppress low-priority notifications.
- Restrict developer tools.
- Keep navigation, media, cameras, and critical alerts available.

Do not rely only on CSS to enforce restricted actions. Backend authorization rules should also protect sensitive commands.

---

## 14. Camera Override Architecture

Reverse camera display must work regardless of the current route.

The camera override should be rendered above normal route content.

```text
Normal Route
    ↓
Global Camera Override Layer
    ↓
Rear Camera shown when reverse=true
```

Requirements:

- React immediately to `vehicle.reverse_changed`.
- Show a loading state only briefly.
- Show a clear camera failure state.
- Restore the prior screen when reverse ends.
- Permit manual camera selection when not in automatic reverse mode.

The override should not destroy route state.

---

## 15. Notification Architecture

Notifications should include:

- Unique ID.
- Severity.
- Title.
- Message.
- Source.
- Created time.
- Dismissibility.
- Optional action.

Severity levels:

- Info.
- Advisory.
- Warning.
- Critical.

Rules:

- Critical alerts must remain visible until resolved or acknowledged.
- Low-priority messages should not obscure navigation while driving.
- Repeated identical hardware errors should be grouped or deduplicated.

---

## 16. Component Design System

The UI should use reusable components with design tokens.

Core tokens:

- Spacing scale.
- Typography scale.
- Touch target sizes.
- Border radii.
- Surface elevations.
- Semantic colors.
- Animation durations.

Core components:

- `RoverButton`
- `IconButton`
- `StatusBadge`
- `TelemetryValue`
- `RoverCard`
- `AlertBanner`
- `Modal`
- `ConfirmDialog`
- `ScreenHeader`
- `NavigationItem`
- `LoadingState`
- `UnavailableState`

Avoid feature teams creating visually inconsistent duplicate controls.

---

## 17. Styling and Themes

Theme values should be exposed through CSS custom properties or a token system.

Required semantic tokens:

- Background.
- Surface.
- Elevated surface.
- Primary text.
- Secondary text.
- Accent.
- Success.
- Advisory.
- Warning.
- Critical.
- Focus outline.

Do not encode meaning only through color. Pair status colors with text or icons.

Day and night modes must preserve readable contrast.

---

## 18. Performance Requirements

The frontend should target smooth operation on Raspberry Pi 5.

Guidelines:

- Avoid excessive rerenders from high-frequency telemetry.
- Select only required state slices in components.
- Coalesce telemetry updates when appropriate.
- Lazy-load parked-only feature screens.
- Avoid large animation libraries for basic transitions.
- Optimize camera rendering separately from normal UI state.
- Keep dashboard bundle and boot path small.

Target behavior:

- Dashboard remains responsive during telemetry updates.
- Route changes feel immediate.
- Camera override appears with minimal UI delay.

---

## 19. Offline Behavior

The frontend must distinguish between:

- Backend unavailable.
- Internet unavailable.
- ARGO Core unavailable.
- Vehicle adapter unavailable.

Internet loss should not produce a system-wide failure screen.

Core local screens should remain available:

- Dashboard.
- Media controls.
- Cameras.
- Diagnostics.
- Maintenance records.
- Settings.

Online-only content should show a scoped offline state.

---

## 20. Error Boundaries

Use layered error boundaries:

- Global application boundary.
- Route-level boundaries.
- High-risk feature boundaries for camera or integration views.

A broken optional feature should not replace the entire dashboard with a blank screen.

The global fallback should offer:

- Retry UI.
- Backend status.
- Restart frontend option if supported.
- Access to basic shutdown command when safe.

---

## 21. Accessibility

Frontend implementation should support:

- High contrast.
- Large text.
- Reduced motion.
- Keyboard navigation for maintenance.
- Visible focus states.
- Labels for icon-only buttons.
- Status communication beyond color.

The primary interaction remains touch, but accessible markup improves maintainability and testing.

---

## 22. Testing Strategy

### Unit Tests

Test:

- State reducers/stores.
- Event handlers.
- Formatters.
- Capability logic.
- Driving-mode rules.

### Component Tests

Test:

- Missing telemetry.
- Degraded services.
- Critical alerts.
- Theme changes.
- Large touch controls.

### End-to-End Tests

Test:

- Boot to dashboard.
- WebSocket reconnect.
- Reverse camera override.
- Media command flow.
- Settings persistence.
- Driving-mode restrictions.

Use mock backend fixtures for repeatable tests.

---

## 23. Development Tools

Development mode should provide optional tools for:

- Selecting mock scenarios.
- Simulating speed and gear.
- Triggering reverse.
- Disconnecting mock cameras.
- Simulating ARGO Core loss.
- Viewing event traffic.
- Testing day/night themes.

These tools should not be visible in normal vehicle mode.

---

## 24. Build and Deployment

The production frontend should build to static assets.

Expected flow:

```text
TypeScript source
    ↓
Vite production build
    ↓
Static frontend bundle
    ↓
Served by Rover backend or local web server
    ↓
Chromium kiosk mode
```

The frontend should not require a development server in vehicle mode.

---

## 25. Acceptance Criteria

The frontend architecture is acceptable for v0.1 when:

- The app runs on a development computer.
- Dashboard renders mock data.
- Routes exist for primary features.
- REST snapshot loading works.
- WebSocket events update live state.
- Connection loss and reconnection are visible.
- Missing hardware data does not crash components.
- Reverse-camera override can be simulated.
- Driving and parked modes can be demonstrated with mock data.
