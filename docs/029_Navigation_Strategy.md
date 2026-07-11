# 029 — Navigation Strategy

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft  

---

## 1. Purpose

This document defines the navigation strategy for Rover OS. Navigation is a core head-unit feature, but it should be implemented in stages so the project can become useful before a full offline routing engine is complete.

---

## 2. Navigation Goals

Rover navigation should eventually provide:

- Fast access to a map.
- Destination search.
- Turn-by-turn routing.
- Traffic when online.
- Voice guidance.
- Home/work/favorite destinations.
- GPS-based trip logging.
- Offline fallback where practical.
- Safe interaction while driving.

---

## 3. Staged Implementation

### Stage 1 — Launcher / Handoff

For the first prototype, Rover may:

- Open browser-based mapping.
- Launch Android Auto navigation.
- Send a destination to the driver's phone.
- Show current GPS position and heading.

This stage provides immediate utility without blocking the rest of Rover OS.

### Stage 2 — Embedded Online Map

Add an embedded map view with:

- Current location.
- Pan and zoom.
- Search.
- Route display.
- Online tiles and traffic where licensed.

### Stage 3 — Offline Map Cache

Add cached regional map data and graceful offline behavior.

### Stage 4 — Native Offline Routing

Evaluate a local routing stack such as OpenStreetMap-based tooling, subject to hardware, storage, and license requirements.

---

## 4. Navigation Providers

Potential provider categories:

- Android Auto / CarPlay projection.
- Google Maps in a browser context.
- Mapbox or similar SDK.
- OpenStreetMap tiles with a compatible renderer.
- Local routing using GraphHopper, Valhalla, OSRM, or equivalent.

The final choice should consider:

- Licensing and cost.
- Offline support.
- Linux/browser support.
- Raspberry Pi performance.
- Traffic availability.
- Data privacy.
- Ease of destination search.

---

## 5. GPS Sources

Navigation location may come from:

1. USB GNSS receiver.
2. Driver phone location sharing.
3. LTE/GPS modem.
4. Future vehicle GPS source if available.

A dedicated USB GNSS receiver is preferred for independent trip logging and phone-free operation.

The location service should expose:

- Latitude.
- Longitude.
- Accuracy.
- Speed.
- Heading.
- Fix state.
- Satellite/fix age where available.

---

## 6. Location Abstraction

Navigation must consume a generic location interface rather than access a USB device directly.

Example:

```ts
export interface LocationSnapshot {
  latitude?: number;
  longitude?: number;
  accuracyMeters?: number;
  speedMph?: number;
  headingDegrees?: number;
  fix: "none" | "estimated" | "2d" | "3d";
  source: "usb-gps" | "phone" | "modem" | "mock";
  updatedAt: string;
}
```

Mock location playback is required for development.

---

## 7. Search and Favorites

Navigation should support:

- Address search.
- Place search.
- Recent destinations.
- Saved favorites.
- Home and work shortcuts.

Sensitive destination data must remain local by default.

The UI should avoid keyboard-heavy search while driving. Prefer:

- Voice input.
- Recent destinations.
- Favorites.
- Phone handoff.

---

## 8. Route State

The backend or navigation adapter should expose normalized route state:

- No route.
- Calculating.
- Active.
- Rerouting.
- Arrived.
- Error.

Useful route fields:

- Destination label.
- ETA.
- Remaining distance.
- Next instruction.
- Next-turn distance.
- Route polyline or map data reference.

---

## 9. Audio Guidance

Navigation prompts should integrate with the Rover audio service.

Desired behavior:

- Duck media during spoken instructions.
- Preserve call priority.
- Allow independent guidance volume.
- Respect mute settings.
- Resume prior media level smoothly.

---

## 10. UI Requirements

Driving navigation view should prioritize:

- Map.
- Next instruction.
- Remaining distance.
- ETA.
- Current speed where appropriate.
- One-tap route cancellation or overview.

Avoid:

- Small map controls.
- Dense search results while moving.
- Complex settings.
- Unnecessary animations.

---

## 11. Dashboard Integration

The dashboard navigation card should show:

- Current destination.
- ETA.
- Remaining distance.
- Next instruction, if useful.
- Start-navigation shortcut when inactive.

The navigation card should still function when the full navigation provider is phone projection.

---

## 12. Offline Behavior

When internet is unavailable:

- Current GPS position should continue if local GNSS exists.
- Existing active route should continue if provider supports it.
- Cached maps should be used where available.
- Search may be limited.
- Rover should clearly show offline status without replacing the whole UI.

---

## 13. Privacy

Location and route history are sensitive.

Requirements:

- Trip/location logging can be disabled.
- Precise history remains local by default.
- ARGO sync requires explicit configuration.
- Diagnostic exports should omit route history unless selected.
- Logs should not routinely contain precise coordinates.

---

## 14. Map Data Storage

Offline maps may require substantial storage.

The system should support:

- Region-based downloads.
- Storage size estimates.
- Deletion of unused regions.
- Update timestamps.
- Download only while parked or on approved networks.

Map storage must not consume space reserved for system updates, database safety, or critical camera recordings.

---

## 15. Failure Handling

Navigation failures should remain scoped.

Examples:

- GPS unavailable.
- Map provider offline.
- Route calculation failed.
- Phone disconnected.
- Offline tiles missing.

Rover should provide a clear fallback, such as reconnecting projection, using phone navigation, or showing last known position.

---

## 16. Acceptance Criteria

Navigation is acceptable for v0.1 when:

- A navigation screen exists.
- Mock location can be displayed.
- A destination can be launched or handed off to a supported provider.
- Navigation state can appear on the dashboard.
- Internet loss produces a scoped offline state.

Navigation is acceptable for v1.0 when:

- The daily navigation path is reliable.
- Audio guidance works with media.
- Phone connection loss is handled cleanly.
- Driving controls are large and simple.
- GPS and trip logging work independently if local GNSS is installed.
