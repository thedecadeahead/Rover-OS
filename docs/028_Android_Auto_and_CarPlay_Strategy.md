# 028 — Android Auto and CarPlay Strategy

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft  

---

## 1. Purpose

This document defines how Rover OS should approach Android Auto and Apple CarPlay compatibility without making phone projection the architectural center of the project.

Rover OS is intended to be the primary vehicle interface. Phone projection is an optional compatibility layer for navigation, communications, and media applications that are difficult or impractical to reproduce safely.

---

## 2. Product Position

Rover OS should support three levels of phone integration:

1. **Native Rover features** for dashboard, telemetry, cameras, diagnostics, maintenance, ARGO, and local media.
2. **Phone companion integration** for hotspot connectivity, notifications, location sharing, and media control.
3. **Optional Android Auto / CarPlay projection** for supported phone apps.

The UI must remain usable when no phone is connected.

---

## 3. Initial Priority

Because the primary driver uses Android, Android support should be developed first.

Priority order:

1. Android phone hotspot and Bluetooth audio.
2. Reliable media controls and navigation handoff.
3. Android Auto receiver research/prototype.
4. CarPlay receiver research/prototype.
5. Seamless integration into the Rover shell.

Neither Android Auto nor CarPlay is required for the first desktop mockup.

---

## 4. Implementation Paths

### 4.1 Open-Source Receiver Software

Potential approach:

- Run receiver software on Linux.
- Render projection inside a controlled Rover view.
- Route audio through Rover's audio service.
- Map steering wheel inputs to projection controls.

Any selected project must be reviewed for:

- Current maintenance status.
- Licensing.
- Raspberry Pi 5 support.
- Wireless and wired stability.
- Audio routing.
- Touch input.
- Legal and protocol limitations.

### 4.2 Commercial USB Projection Dongle

A USB dongle with Linux-compatible receiver software may provide a simpler compatibility path.

Tradeoffs:

- Faster prototype.
- Proprietary firmware.
- Vendor dependence.
- Potentially weak support.
- Security and update concerns.

### 4.3 Separate Android Device

A dedicated Android device could provide projection or native apps, with Rover OS acting as the surrounding shell.

This adds hardware complexity and is not preferred for v0.1.

---

## 5. Integration Model

Projection should appear as a Rover screen, not replace the full Rover environment.

```text
Rover Application Shell
├── Persistent status and alerts
├── Projection view
├── Camera override layer
├── Power/shutdown overlay
└── Critical vehicle notifications
```

Rover must retain control of:

- Reverse-camera override.
- Critical alerts.
- Volume policy.
- Power state.
- Display brightness.
- Return-to-dashboard behavior.

---

## 6. Android Auto Requirements

Desired support:

- Wired USB connection.
- Wireless connection if stable.
- Touch input.
- Navigation audio.
- Media audio.
- Phone call audio where feasible.
- Steering wheel media controls.
- Fast reconnect after vehicle startup.

Android Auto should not require a dedicated car phone. The driver's phone is the intended source.

---

## 7. CarPlay Requirements

CarPlay is a secondary compatibility goal.

Desired support:

- Wired connection first.
- Wireless later if stable.
- Touch input.
- Audio routing.
- Steering wheel controls.
- Automatic reconnect.

CarPlay support must not delay the core Rover OS prototype.

---

## 8. Audio Routing

Projection audio categories may include:

- Media.
- Navigation prompts.
- Calls.
- Voice assistant.

The Rover audio service should own final output routing and volume policy where technically possible.

Desired behavior:

- Navigation temporarily ducks media.
- Calls take priority.
- Reverse-camera display does not interrupt audio unless configured.
- Rover alerts can play over or alongside projected audio according to severity.

---

## 9. Microphone Routing

Hands-free calls and phone assistants may require microphone access.

Potential approaches:

- USB microphone presented to projection receiver.
- Bluetooth HFP managed separately by Linux.
- Phone microphone used directly in early versions.

Call-quality development should not block dashboard, media, camera, or diagnostic work.

---

## 10. Connection States

Rover should expose clear states:

- No phone.
- Phone detected.
- Pairing.
- Projection starting.
- Connected.
- Degraded.
- Disconnected.
- Unsupported device.

Connection failure must not freeze the UI.

---

## 11. Driving Safety

Projection systems already limit many unsafe interactions, but Rover should still enforce its global driving rules.

Rover must retain:

- Critical warning overlays.
- Camera override.
- Simplified global navigation.
- Ability to return home.

---

## 12. Offline and Phone-Free Operation

Without a phone, Rover must still provide:

- Dashboard.
- Vehicle telemetry.
- Cameras.
- Diagnostics.
- Maintenance records.
- Local media.
- Settings.

Phone projection is an enhancement, not a dependency.

---

## 13. Legal and Licensing Notes

Android Auto and CarPlay are proprietary ecosystems.

Before distributing receiver functionality, confirm:

- License compatibility of any receiver software.
- Restrictions on proprietary protocol implementations.
- Distribution rights for required binaries.
- Branding and trademark rules.

The repository should not include proprietary Apple or Google software without clear permission.

---

## 14. Fallback Strategy

If integrated projection proves unreliable, Rover should still provide a useful phone workflow:

- Automatic hotspot connection.
- Bluetooth audio.
- Phone-mounted navigation or browser-based map handoff.
- Media metadata/control where available.
- Voice commands through the phone or ARGO.

Feature parity does not require forcing an unstable projection implementation into v1.0.

---

## 15. Acceptance Criteria

Projection integration is acceptable for beta when:

- Rover remains stable with no phone connected.
- A supported phone can connect consistently.
- Touch input works.
- Audio is routed correctly.
- Reverse camera and critical Rover overlays take priority.
- Disconnecting the phone returns cleanly to Rover UI.
- Projection failure does not require rebooting the vehicle computer.
