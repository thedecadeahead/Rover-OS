# 006 — UI / UX Guide

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft  
**Primary Interface:** 13.3 inch vehicle touchscreen  

---

## 1. Purpose

This document defines the user interface and user experience principles for ARGO Rover OS. The goal is to create an interface that feels like a purpose-built vehicle operating environment rather than a generic desktop, tablet launcher, or web page.

Rover OS must be usable as a daily driver infotainment system while supporting advanced ARGO features when parked.

---

## 2. UX Principles

### 2.1 Glanceable First

Driving UI must prioritize information that can be understood quickly.

The driver should be able to understand core status in a glance:

- Navigation state.
- Media state.
- Vehicle status.
- Camera status.
- Warnings.
- Connectivity.

### 2.2 Parked Depth, Driving Simplicity

Rover OS should have two UX modes:

- **Driving mode:** simplified, large controls, minimal interaction.
- **Parked mode:** full access to settings, diagnostics, projects, logs, and configuration.

Driving mode should avoid deep menus and text entry.

### 2.3 Touch Target Safety

Touch targets used while driving should be large.

Recommended minimums:

- Primary driving controls: 72px or larger.
- Secondary controls: 56px or larger.
- Avoid dense toolbars.
- Avoid tiny close buttons.

### 2.4 System Status Always Visible

The system should always make critical status visible or quickly accessible:

- Vehicle connection.
- Network connection.
- Audio state.
- Camera availability.
- ARGO Core connection.
- System alerts.

### 2.5 No Desktop Leakage

The driver should not normally see:

- Linux desktop.
- Browser address bar.
- Terminal.
- Cursor-heavy UI.
- File manager.
- Generic app launcher.

Rover OS should feel like the operating environment.

---

## 3. Main Navigation Model

The UI should use a small set of top-level modes.

Recommended primary screens:

- Dashboard
- Navigation
- Media
- Cameras
- Diagnostics
- Maintenance
- Home
- Projects
- Settings

The dashboard should be the default landing screen.

---

## 4. Dashboard Requirements

The dashboard is the home screen for Rover OS.

It must provide:

- Current time.
- Vehicle state summary.
- Network status.
- ARGO Core status.
- Media summary.
- Navigation shortcut.
- Camera shortcut.
- Diagnostics shortcut.
- Settings shortcut.

It should provide:

- Weather when online.
- Fuel/range when available.
- 12V battery voltage when available.
- Current trip summary.
- Maintenance alert if due.

---

## 5. Driving Mode

Driving mode should activate when the car is moving, in drive, or when a driving state is inferred from vehicle data.

Driving mode should:

- Increase touch target size.
- Hide advanced settings.
- Reduce animations.
- Suppress non-critical notifications.
- Disable text entry.
- Prefer voice and steering wheel controls.
- Keep navigation, media, and cameras easily accessible.

Driving mode should not hide safety-relevant warnings.

---

## 6. Parked Mode

Parked mode can expose advanced controls.

Parked mode may include:

- Full diagnostics.
- Logs.
- Hardware status.
- Plugin management.
- ARGO integration settings.
- Home Assistant controls.
- Project dashboard.
- Developer tools.
- Update controls.

Parked mode should still be clear and organized, not a cluttered debug console.

---

## 7. Visual Style

Rover OS should look like an engineering-grade vehicle interface.

Style direction:

- Dark primary theme.
- High contrast text.
- Minimal chrome.
- Calm accent colors.
- Clear iconography.
- Large type.
- Subtle borders and panels.
- No excessive gradients or novelty effects.

The system should feel modern but not distracting.

---

## 8. Theme Requirements

Required themes:

- Dark cockpit theme.
- Daylight high-contrast theme.
- Night mode.

Optional future themes:

- ARGO prototype theme.
- Minimal monochrome theme.
- Accessibility high-contrast theme.

Theme switching should be controlled by:

- Manual setting.
- Illumination/dimmer signal if available.
- Time of day if configured.

---

## 9. Typography

Typography should prioritize readability in a moving vehicle.

Guidelines:

- Use large font sizes.
- Avoid thin weights.
- Avoid dense paragraphs on driving screens.
- Use numeric displays for telemetry.
- Use short labels.

Suggested scale:

- Hero status: 48–72px.
- Major cards: 28–40px.
- Button text: 20–28px.
- Secondary labels: 16–20px.

---

## 10. Layout

The display target is approximately 13.3 inches at 16:9 landscape.

Recommended layout concepts:

- Persistent top or side status area.
- Large central content region.
- Bottom or side quick-action rail.
- Cards for status groups.
- Modal overlays only when necessary.

Avoid layouts that require precision scrolling while driving.

---

## 11. Alerts and Notifications

Alerts should be prioritized.

Alert levels:

- Info
- Advisory
- Warning
- Critical

Critical alerts should be visually obvious and remain visible until acknowledged or resolved.

Examples:

- Camera unavailable.
- OBD disconnected.
- Low 12V voltage.
- Storage nearly full.
- Shutdown pending.
- Backend service failure.

Non-critical alerts should not interrupt driving.

---

## 12. Camera UX

Camera views must be fast and obvious.

Rear camera behavior:

- Reverse trigger should immediately open the rear view when available.
- Manual camera button should be available from dashboard.
- Camera unavailable state should be clear.

Camera screen should support:

- Rear view.
- Front view.
- Cabin view.
- Auxiliary view.

Future support may include split views or recording indicators.

---

## 13. Diagnostics UX

Diagnostics should be understandable without requiring mechanic-level knowledge.

The diagnostics screen should show:

- Connection status.
- Current live data.
- Active trouble codes.
- Explanation of codes where available.
- Clear distinction between warning and informational data.

Advanced raw data should be hidden under an expert or developer section.

---

## 14. Media UX

Media controls must be simple.

Required controls:

- Play/pause.
- Previous.
- Next.
- Volume.
- Source selection.

The current media screen should show:

- Source.
- Track title when available.
- Artist when available.
- Playback state.

Steering wheel controls should map cleanly to media actions.

---

## 15. Settings UX

Settings should be grouped by category:

- Display.
- Audio.
- Vehicle.
- Network.
- ARGO.
- Cameras.
- Storage.
- Developer.

Dangerous or disruptive settings should require confirmation.

Examples:

- Reboot.
- Shutdown.
- Factory reset.
- Clear logs.
- Clear trip history.

---

## 16. Home Assistant UX

Home Assistant should be available, but not treated as a driving-critical feature.

Recommended approach:

- Summary card on dashboard.
- Full panel only when parked.
- Quick actions for common tasks.

Examples:

- Garage / door state.
- Lights.
- Security cameras.
- Climate.

---

## 17. ARGO UX

ARGO status should be visible without dominating the driving experience.

Dashboard should show:

- ARGO Core online/offline.
- Sync status.
- Pending tasks count.

Parked mode may expose:

- Project dashboard.
- Remote jobs.
- Core/Vault status.
- Logs.
- AI assistant.

---

## 18. Developer UX

Developer views should exist but be hidden from normal driving flows.

Developer tools may include:

- Service health.
- Event bus viewer.
- Raw telemetry.
- Logs.
- Hardware status.
- Mock data controls.

Developer mode should be disabled or hidden by default in vehicle mode.

---

## 19. Accessibility

Rover OS should support:

- High contrast.
- Large text.
- Minimal reliance on color alone.
- Clear focus states for keyboard/mouse maintenance.
- Reduced animation mode.

---

## 20. UI Acceptance Criteria

The UI is acceptable for v0.1 when:

- Dashboard renders with mock data.
- Primary screens exist.
- Navigation between screens is clear.
- Touch targets are large enough for bench testing.
- Settings persist.
- Degraded hardware states can be displayed.

The UI is acceptable for v1.0 when:

- It can be used daily while driving.
- Backup camera access is immediate.
- Media controls are reliable.
- Diagnostics are readable.
- Driving mode reduces distraction.
- Parked mode exposes advanced features safely.
