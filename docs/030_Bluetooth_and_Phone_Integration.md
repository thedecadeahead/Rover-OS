# 030 — Bluetooth and Phone Integration

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft  

---

## 1. Purpose

This document defines how Rover OS should connect to the driver's phone for internet access, media, calls, notifications, location sharing, and optional projection.

The initial design assumes the driver carries a phone. A dedicated car phone is optional and not required for v0.1.

---

## 2. Integration Goals

Phone integration should provide:

- Automatic hotspot connectivity.
- Bluetooth media playback where practical.
- Media transport controls.
- Optional call support.
- Optional contact and notification integration.
- Navigation handoff or projection.
- Clean connect/disconnect behavior.
- Phone-free fallback for core Rover functions.

---

## 3. Initial Connectivity Model

Recommended v0.1 model:

- Driver phone provides Wi-Fi hotspot.
- Rover automatically joins a trusted hotspot.
- Phone provides internet for maps, traffic, streaming, and ARGO sync.
- Rover continues offline when the phone is absent.

A dedicated phone or LTE modem should only be added when always-on remote connectivity becomes necessary.

---

## 4. Bluetooth Roles

Potential Bluetooth profiles include:

- A2DP sink for phone media audio.
- AVRCP for play/pause/next/previous and metadata.
- HFP/HSP for calls.
- BLE for companion-app communication.
- PAN tethering as an alternative to Wi-Fi hotspot.

Not all profiles need to be implemented at once.

Priority:

1. Reliable hotspot connection.
2. Media audio and controls.
3. Companion-app data channel.
4. Call audio.

---

## 5. Linux Bluetooth Stack

The initial Linux implementation will likely use:

- BlueZ.
- PipeWire for audio routing.
- WirePlumber or equivalent session management.
- D-Bus integration from the backend.

Rover services should wrap Bluetooth behavior behind a `PhoneAdapter` or `BluetoothAdapter` interface rather than embedding shell commands throughout the codebase.

---

## 6. Pairing UX

Pairing should only be available while parked.

Pairing flow:

1. Open Phone settings.
2. Enable discoverable/pairing mode for a limited time.
3. Select device.
4. Confirm code on both devices.
5. Choose trusted capabilities.
6. Store device record.
7. Attempt automatic reconnect on future starts.

The UI should show:

- Device name.
- Connection state.
- Supported capabilities.
- Last connected time.
- Remove/forget action.

---

## 7. Trusted Devices

Rover should maintain a list of trusted phones.

Each record may include:

- Stable device ID/address where appropriate.
- Friendly name.
- Preferred driver flag.
- Auto-connect setting.
- Media permission.
- Call permission.
- Companion-app permission.
- Hotspot configuration reference.

Sensitive credentials must not be committed to the repository or exposed to the frontend.

---

## 8. Connection Priority

Suggested priority:

1. Preferred driver's phone.
2. Previously connected trusted phone.
3. Manual selection.

Rover should avoid repeatedly switching between two nearby trusted phones without user intent.

Multi-phone behavior should be deferred until single-phone operation is reliable.

---

## 9. Hotspot Automation

Rover should automatically connect to the driver's trusted hotspot when available.

Requirements:

- Store Wi-Fi credentials securely through the operating system.
- Retry with backoff.
- Show online/offline status.
- Avoid blocking local startup while waiting.
- Prefer approved home Wi-Fi over metered hotspot for large updates, if configured.

Potential Android automation may use phone routines, Tasker, or a future Rover companion app to enable hotspot when Bluetooth or another trigger detects the car.

---

## 10. Media Audio

For Bluetooth media, the phone acts as A2DP source and Rover as A2DP sink.

The audio path should be:

```text
Phone
    ↓ Bluetooth A2DP
Linux / PipeWire
    ↓ Rover Audio Service
USB DAC
    ↓ Vehicle amplifier and speakers
```

Rover should expose:

- Play/pause.
- Next/previous.
- Track metadata where available.
- Source name.
- Connection quality/state.

---

## 11. Volume Policy

Where possible:

- Phone media volume should remain at a stable high level.
- Rover should control final vehicle output volume.
- Steering wheel volume buttons should control Rover output.
- Track controls should be sent through AVRCP or projection.

This avoids conflicting phone and system volume scales.

---

## 12. Calls

Hands-free calling is desirable but technically more complex than media playback.

Required components may include:

- HFP support.
- USB microphone.
- Echo cancellation.
- Noise suppression.
- Audio routing between call and vehicle speakers.
- Call-priority policy.

Early fallback:

- Calls remain on the phone or phone earbuds.
- Rover lowers or pauses media when call state is detected, if available.

Call support should not delay camera, diagnostics, media, or power milestones.

---

## 13. Companion App Strategy

A future Android/iOS Rover companion app could provide:

- Hotspot automation assistance.
- GPS/location sharing.
- Destination handoff.
- Notification filtering.
- Contact lookup.
- Voice input.
- Remote status when the phone is near the car.
- Secure pairing with Rover.

The companion app should communicate through a documented local API over BLE, Wi-Fi, or both.

No companion app is required for the first desktop prototype.

---

## 14. Notifications

Phone notifications can be distracting and private.

If implemented, Rover should allow only selected categories:

- Navigation.
- Calls.
- Messages from approved contacts.
- Calendar reminders.
- Critical device alerts.

Full notification mirroring should be disabled by default.

Notification text should not be displayed extensively while driving.

---

## 15. Contacts and Messages

Contact and message integration should be optional.

Requirements:

- Explicit permission.
- Local caching minimized.
- Data deleted when device is forgotten.
- No contact/message sync to ARGO Core by default.
- Voice-first interaction while driving.

---

## 16. Location Sharing

The driver's phone may provide location when no USB GNSS receiver is installed.

The location service must indicate source and data age.

If phone location stops updating, Rover should mark it stale rather than continuing to present it as current.

---

## 17. Projection Integration

Android Auto and CarPlay are documented separately in `028_Android_Auto_and_CarPlay_Strategy.md`.

Phone integration must remain useful even when projection is unavailable.

---

## 18. Connection States

Normalize phone state:

```ts
export type PhoneConnectionState =
  | "absent"
  | "detected"
  | "pairing"
  | "connected"
  | "degraded"
  | "disconnecting"
  | "error";
```

Capability state should be separate:

- Internet available.
- Media available.
- Calls available.
- Location available.
- Projection available.

A phone may be connected for one capability and unavailable for another.

---

## 19. Failure Handling

Rover must handle:

- Phone leaves range.
- Hotspot turns off.
- Bluetooth restarts.
- Audio profile fails.
- Two trusted devices are present.
- Phone changes its friendly name.
- Projection disconnects during navigation.

The system should return to local Rover UI and continue core functions without rebooting.

---

## 20. Security and Privacy

Requirements:

- Pair only while parked or in developer mode.
- Authenticate companion-app communication.
- Do not expose contacts/messages to unauthenticated clients.
- Do not log message contents.
- Store minimal phone metadata.
- Allow the user to forget a device and remove its cached data.

---

## 21. Dedicated Car Phone Decision

A dedicated car phone is justified later if Rover needs:

- Internet while the driver is away.
- Always-on GPS tracking.
- Security alerts.
- Dashcam uploads without the driver's phone.
- Independent remote access.

Until then, the driver's phone is the preferred connectivity source.

---

## 22. Acceptance Criteria

Phone integration is acceptable for v0.1 when:

- Rover can automatically connect to a trusted phone hotspot.
- The UI clearly shows connectivity state.
- Rover remains functional without the phone.
- A mock phone adapter supports frontend development.

Phone integration is acceptable for v1.0 when:

- Reconnection is reliable after startup.
- Bluetooth media and transport controls work if enabled.
- Phone loss returns gracefully to local operation.
- Pairing and device removal are understandable.
- No private phone data is exposed without permission.
