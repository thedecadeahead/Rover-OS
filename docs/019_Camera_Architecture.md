# 019 — Camera Architecture

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft

---

## 1. Purpose

This document defines the camera subsystem for Rover OS, including backup-camera display, manual camera viewing, future dashcam recording, storage rotation, health monitoring, and privacy controls.

The reverse camera is a safety-supporting convenience feature and must be designed for low latency and graceful failure.

---

## 2. Camera Goals

The camera subsystem should:

- Display the rear camera automatically in reverse.
- Allow manual camera selection.
- Support multiple camera technologies.
- Report camera health.
- Avoid blocking the rest of Rover OS.
- Support future recording and event protection.
- Protect privacy by default.
- Remain replaceable through camera adapters.

---

## 3. Camera Types

Initial and future camera roles:

- Rear backup camera.
- Front parking camera.
- Cabin camera.
- Side camera.
- Cargo or trailer camera.

The rear camera is required for head-unit parity. Other cameras are optional.

---

## 4. Supported Input Paths

Potential input technologies:

- USB UVC camera.
- Analog composite camera through USB capture.
- HDMI camera through USB capture.
- IP camera stream.
- Existing factory camera through a compatible conversion interface.

The camera service should normalize these sources behind one interface.

---

## 5. Camera Adapter Interface

```ts
interface CameraAdapter {
  initialize(): Promise<void>;
  listCameras(): Promise<CameraDescriptor[]>;
  getStatus(cameraId: string): Promise<CameraStatus>;
  getStream(cameraId: string): Promise<CameraStreamDescriptor>;
  startRecording?(cameraId: string): Promise<void>;
  stopRecording?(cameraId: string): Promise<MediaAsset>;
  shutdown(): Promise<void>;
}
```

Implementations may include:

- UVC adapter.
- V4L2 capture adapter.
- Network stream adapter.
- Mock camera adapter.

---

## 6. Reverse Trigger Flow

```text
Vehicle reverse signal
    ↓
Vehicle I/O controller or adapter
    ↓
vehicle.reverse_changed event
    ↓
Camera service activates rear stream
    ↓
Frontend camera override appears
```

A physical reverse signal is preferred over slow polling when available.

---

## 7. Reverse Camera Requirements

The rear camera should:

- Be available shortly after system startup.
- Open automatically when reverse is engaged.
- Override the current screen without destroying its state.
- Restore the previous screen when reverse ends.
- Show a clear failure screen if unavailable.
- Permit manual activation when not in reverse.

The camera view should not depend on internet, ARGO Core, or cloud services.

---

## 8. Latency

Latency must be measured end-to-end:

- Reverse signal detection.
- Backend event publication.
- Capture startup.
- Browser rendering.

Target behavior should feel immediate. Exact acceptance thresholds should be established through bench and vehicle testing.

Persistent capture may reduce switching delay at the cost of power and CPU use.

---

## 9. Stream Delivery

Possible browser-compatible stream methods:

- MJPEG for simplicity.
- WebRTC for low latency.
- HLS for recorded or noncritical streams.
- Local native overlay if browser performance is inadequate.

The first bench implementation may use MJPEG or another simple local stream. The final reverse-camera path should prioritize latency and reliability over compression efficiency.

---

## 10. Camera State Model

Recommended states:

- `starting`
- `available`
- `streaming`
- `recording`
- `degraded`
- `offline`
- `error`

Status should include:

- Camera ID.
- Adapter type.
- Resolution.
- Frame rate.
- Last frame time.
- Recording state.
- Error detail.

---

## 11. Manual Camera Screen

The manual camera UI should support:

- Camera selection.
- Fullscreen view.
- Recording indicator.
- Snapshot action when parked.
- Camera health indicator.

Advanced multi-camera layouts may be added later.

---

## 12. Dashcam Recording

Dashcam recording is a future milestone, separate from basic backup display.

Recording features may include:

- Rolling segmented files.
- Front and rear recording.
- GPS metadata.
- Trip association.
- Manual clip protection.
- Automatic protection after detected impact if supported.
- Upload queue to ARGO Vault.

Recording must not make reverse viewing unreliable.

---

## 13. Storage Architecture

Video files should live outside SQLite.

Recommended layout:

```text
/var/lib/rover/media/
├── recordings/
│   ├── front/
│   ├── rear/
│   └── cabin/
├── protected/
├── snapshots/
└── exports/
```

The database stores metadata and file paths.

---

## 14. Retention

Rolling retention should be based on:

- Maximum allocated storage.
- Minimum free-space threshold.
- Clip age.
- Protected status.
- Upload status.

Protected clips must not be deleted by routine rotation.

If storage is critically low:

- Stop nonessential recording.
- Preserve reverse-camera display.
- Publish a critical storage warning.

---

## 15. Recording Segments

Recommended design:

- Record short segments rather than one giant file.
- Finalize segments cleanly.
- Keep an index in the database.
- Mark interrupted segments after abrupt shutdown.

Short segments improve recovery and retention management.

---

## 16. Privacy

Requirements:

- Recording disabled by default until configured.
- Clear recording indicator.
- Cabin camera disabled by default.
- Remote camera viewing disabled by default.
- Configurable retention.
- Manual deletion and export.
- No silent cloud upload.

---

## 17. Health Monitoring

The camera service should detect:

- Device missing.
- Stream stalled.
- No recent frames.
- Capture process crash.
- Storage unavailable.
- Recording failure.

Recovery should use bounded retries and avoid endless process-spawn loops.

---

## 18. Resource Management

Camera processing can consume substantial CPU, USB bandwidth, and storage.

The subsystem should:

- Prefer hardware-supported formats where possible.
- Avoid transcoding unless needed.
- Limit resolution and frame rate appropriately.
- Keep reverse viewing prioritized over recording.
- Track USB topology to avoid bandwidth conflicts.

---

## 19. Guidelines Overlay

Parking guidelines may be:

- Built into the camera.
- Added by capture hardware.
- Rendered by Rover OS.

Software-rendered dynamic guidelines require calibration and vehicle geometry research. Static guidelines may be supported first.

Guidelines must not imply precision they do not have.

---

## 20. Failure Behavior

If rear camera fails while reverse is active:

- Display an immediate clear warning.
- Do not show a frozen old frame as live.
- Keep the rest of Rover OS operational.
- Log the error and camera state.

The UI should distinguish between loading, offline, and stalled states.

---

## 21. Bench Testing

Test:

- Camera enumeration.
- Stream startup.
- Reverse trigger simulation.
- Device removal.
- Frozen stream detection.
- Multiple USB devices.
- Storage-full behavior.
- Recording recovery after abrupt power loss.

---

## 22. Acceptance Criteria

The camera subsystem is acceptable for v1.0 when:

- Rear camera opens reliably on reverse.
- Manual rear-camera viewing works.
- Camera failure is clearly shown.
- Reverse view remains available offline.
- Other subsystem failures do not block the camera.
- Recording, if enabled, does not compromise reverse-camera reliability.
