# 020 — Hardware Abstraction Layer

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft

---

## 1. Purpose

This document defines the Hardware Abstraction Layer (HAL) for Rover OS. The HAL isolates application services from Raspberry Pi details, Linux device paths, OBD adapters, camera drivers, audio hardware, GPS hardware, vehicle wiring, and future compute platforms.

The HAL is the core mechanism that makes Rover OS hardware-agnostic and vehicle-agnostic.

---

## 2. Design Goals

The HAL must:

- Allow desktop development without vehicle hardware.
- Provide mock implementations for every interface.
- Keep frontend and application services free of device-specific code.
- Support multiple hardware implementations behind stable contracts.
- Normalize errors and health states.
- Support capability discovery.
- Allow replacement of Raspberry Pi with another compute platform.
- Prevent unsafe vehicle writes by default.

---

## 3. Layering

```text
Application Services
        ↓
HAL Interfaces
        ↓
Platform / Device Adapters
        ↓
Linux, USB, Serial, GPIO, OBD, CAN, Cameras
```

Vehicle-specific normalization may sit beside or above device adapters:

```text
Raw Device Adapter
        ↓
Vehicle Adapter
        ↓
Generic Vehicle HAL
```

---

## 4. Common Types

```ts
export type HealthState =
  | "starting"
  | "healthy"
  | "degraded"
  | "offline"
  | "error";

export interface AdapterHealth {
  state: HealthState;
  message?: string;
  updatedAt: string;
  details?: Record<string, unknown>;
}

export interface AdapterCapabilities {
  [capability: string]: boolean | string | number;
}

export interface RoverAdapter {
  readonly id: string;
  readonly name: string;
  initialize(): Promise<void>;
  getHealth(): Promise<AdapterHealth>;
  getCapabilities(): Promise<AdapterCapabilities>;
  shutdown(): Promise<void>;
}
```

All adapters should support lifecycle and health reporting.

---

## 5. Error Model

Adapters should throw normalized errors.

```ts
export type HardwareErrorCode =
  | "DEVICE_NOT_FOUND"
  | "DEVICE_BUSY"
  | "PERMISSION_DENIED"
  | "TIMEOUT"
  | "UNSUPPORTED"
  | "INVALID_RESPONSE"
  | "DISCONNECTED"
  | "IO_ERROR"
  | "CONFIGURATION_ERROR";

export class HardwareError extends Error {
  constructor(
    public readonly code: HardwareErrorCode,
    message: string,
    public readonly retryable: boolean,
    public readonly details?: Record<string, unknown>
  ) {
    super(message);
  }
}
```

Application services should not need to interpret vendor-specific exceptions.

---

## 6. Vehicle Adapter

```ts
export interface VehicleSnapshot {
  connected: boolean;
  speedMph?: number;
  gear?: string;
  fuelPercent?: number;
  rangeMiles?: number;
  battery12v?: number;
  hybridSocPercent?: number;
  coolantTempF?: number;
  engineRpm?: number;
  odometerMiles?: number;
  reverse?: boolean;
  ignition?: "off" | "accessory" | "ready" | "unknown";
  updatedAt: string;
}

export interface VehicleAdapter extends RoverAdapter {
  getSnapshot(): Promise<VehicleSnapshot>;
  subscribe(
    listener: (snapshot: VehicleSnapshot) => void
  ): Promise<() => void>;
}
```

Required implementations:

- `MockVehicleAdapter`
- `GenericObdVehicleAdapter`
- Future `ToyotaPriusGen3Adapter`

---

## 7. OBD Adapter

```ts
export interface DiagnosticCode {
  code: string;
  status: "active" | "pending" | "historic";
  description?: string;
  system?: string;
}

export interface ObdAdapter extends RoverAdapter {
  connect(): Promise<void>;
  disconnect(): Promise<void>;
  readPid(pid: string): Promise<unknown>;
  readCodes(): Promise<DiagnosticCode[]>;
}
```

Safety rules:

- Read-only by default.
- Code clearing excluded from the first implementation.
- Arbitrary commands must not be exposed through public APIs.

---

## 8. Camera Adapter

```ts
export interface CameraDescriptor {
  id: string;
  name: string;
  role: "rear" | "front" | "cabin" | "side" | "auxiliary";
}

export interface CameraStreamDescriptor {
  url: string;
  transport: "mjpeg" | "webrtc" | "hls" | "native";
  width?: number;
  height?: number;
  frameRate?: number;
}

export interface CameraAdapter extends RoverAdapter {
  listCameras(): Promise<CameraDescriptor[]>;
  getStream(cameraId: string): Promise<CameraStreamDescriptor>;
  startRecording?(cameraId: string): Promise<void>;
  stopRecording?(cameraId: string): Promise<{ filePath: string }>;
}
```

Required implementation:

- `MockCameraAdapter`

Likely real implementations:

- `V4l2CameraAdapter`
- `NetworkCameraAdapter`

---

## 9. Audio Adapter

```ts
export interface AudioState {
  available: boolean;
  volume: number;
  muted: boolean;
  source?: string;
}

export interface AudioAdapter extends RoverAdapter {
  getState(): Promise<AudioState>;
  setMasterVolume(level: number): Promise<void>;
  setMuted(muted: boolean): Promise<void>;
  selectOutput?(outputId: string): Promise<void>;
  playSystemSound?(soundId: string): Promise<void>;
}
```

Implementations:

- `MockAudioAdapter`
- `PipeWireAudioAdapter`
- Optional `AlsaAudioAdapter`

---

## 10. Power Adapter

```ts
export interface PowerState {
  state:
    | "off"
    | "starting"
    | "running"
    | "accessory"
    | "shutdown_pending"
    | "shutting_down"
    | "maintenance"
    | "fault";
  ignition?: "off" | "accessory" | "ready" | "unknown";
  inputVoltage?: number;
  shutdownAt?: string;
}

export interface PowerAdapter extends RoverAdapter {
  getState(): Promise<PowerState>;
  requestShutdown(reason: string): Promise<void>;
  cancelShutdown?(): Promise<void>;
  setMaintenanceMode?(enabled: boolean): Promise<void>;
  subscribe(listener: (state: PowerState) => void): Promise<() => void>;
}
```

Implementations:

- `MockPowerAdapter`
- `SerialVehicleControllerPowerAdapter`
- Optional GPIO adapter for bench work

---

## 11. Input Adapter

```ts
export type InputAction =
  | "volume_up"
  | "volume_down"
  | "mute_toggle"
  | "media_next"
  | "media_previous"
  | "play_pause"
  | "voice_toggle"
  | "home"
  | "camera_toggle";

export interface InputEvent {
  action: InputAction;
  state: "pressed" | "released" | "repeated";
  source: string;
  timestamp: string;
}

export interface InputAdapter extends RoverAdapter {
  subscribe(listener: (event: InputEvent) => void): Promise<() => void>;
}
```

Sources may include:

- Steering-wheel interface.
- Rotary encoder.
- Vehicle I/O controller.
- Keyboard during development.

---

## 12. GPS Adapter

```ts
export interface PositionFix {
  latitude: number;
  longitude: number;
  altitudeMeters?: number;
  speedMps?: number;
  headingDegrees?: number;
  accuracyMeters?: number;
  fixType: "none" | "2d" | "3d";
  timestamp: string;
}

export interface GpsAdapter extends RoverAdapter {
  getCurrentFix(): Promise<PositionFix | null>;
  subscribe(listener: (fix: PositionFix) => void): Promise<() => void>;
}
```

Implementations:

- `MockGpsAdapter`
- `GpsdAdapter`
- Future phone-location bridge

---

## 13. Network Adapter

```ts
export interface NetworkState {
  online: boolean;
  interface?: string;
  connectionType?: "wifi" | "ethernet" | "cellular" | "unknown";
  networkName?: string;
  signalPercent?: number;
}

export interface NetworkAdapter extends RoverAdapter {
  getState(): Promise<NetworkState>;
  subscribe(listener: (state: NetworkState) => void): Promise<() => void>;
}
```

The adapter should avoid exposing passwords or sensitive connection details.

---

## 14. Storage Adapter

```ts
export interface StorageState {
  path: string;
  totalBytes: number;
  freeBytes: number;
  writable: boolean;
}

export interface StorageAdapter extends RoverAdapter {
  getState(path?: string): Promise<StorageState>;
  reserveSpace?(bytes: number): Promise<boolean>;
}
```

Use cases:

- Dashcam retention.
- Database health.
- Update staging.
- Low-space warnings.

---

## 15. Adapter Registry

The backend should use a registry or dependency-injection container.

```ts
interface AdapterRegistry {
  vehicle: VehicleAdapter;
  obd?: ObdAdapter;
  cameras: CameraAdapter;
  audio: AudioAdapter;
  power: PowerAdapter;
  input: InputAdapter;
  gps?: GpsAdapter;
  network: NetworkAdapter;
  storage: StorageAdapter;
}
```

Adapter selection should come from validated configuration.

---

## 16. Capability Discovery

Services must not assume every adapter supports every feature.

Examples:

- Generic OBD may provide speed but not gear.
- A camera adapter may display but not record.
- Power adapter may expose ignition but not voltage.

The UI should receive normalized capabilities through the backend API.

---

## 17. Mock Implementations

Every interface must have a mock adapter.

Mocks should support scripted scenarios:

- Driving and parked states.
- Reverse engagement.
- Low voltage.
- OBD disconnect.
- Camera failure.
- Audio device loss.
- GPS movement.

Mocks are production architecture, not disposable test hacks.

---

## 18. Lifecycle

Adapter lifecycle:

1. Construct from validated configuration.
2. Initialize.
3. Report health.
4. Subscribe or poll.
5. Retry recoverable failures with bounds.
6. Shut down cleanly.

The backend should initialize adapters in dependency order and should not block all startup on optional hardware.

---

## 19. Polling and Subscription

Prefer subscriptions when the underlying hardware provides events.

Use polling when necessary, with:

- Configurable interval.
- Timeout.
- Cancellation.
- Backoff after repeated failure.
- No overlapping polls.

High-frequency device data should be normalized and rate-limited before frontend publication.

---

## 20. Platform Isolation

Platform-specific code belongs in adapters.

Examples:

- `/dev/ttyUSB0`
- `v4l2-ctl`
- `wpctl`
- GPIO libraries.
- systemd commands.

Core services must not contain hard-coded Linux device paths or Raspberry Pi GPIO numbers.

---

## 21. Configuration

Adapter configuration should support stable identifiers rather than relying only on transient device paths.

Prefer:

- USB vendor/product IDs.
- Serial numbers.
- `/dev/serial/by-id` paths.
- Camera logical IDs.
- Explicit fallback order.

Configuration must be validated before adapter initialization.

---

## 22. Observability

Every adapter should provide:

- Health state.
- Last successful operation.
- Last error.
- Retry count.
- Connected device identity where safe.
- Capabilities.

Raw debug detail should only appear in developer mode.

---

## 23. Safety Restrictions

The HAL must not provide generic unrestricted functions such as:

```ts
sendCanFrame(id, bytes)
runShellCommand(command)
writeObdCommand(command)
```

through application-facing interfaces.

Low-level development tools, if added, must be isolated, disabled in vehicle mode, and never remotely exposed by default.

---

## 24. Testing

Each real adapter should pass a shared contract test suite.

Tests should cover:

- Initialization.
- Capability reporting.
- Health transitions.
- Timeout behavior.
- Device disconnection.
- Clean shutdown.
- Contract-compatible mock behavior.

Integration tests should use recorded fixtures where physical hardware is unavailable.

---

## 25. Acceptance Criteria

The HAL is acceptable for v0.1 when:

- All major interfaces are defined in shared TypeScript.
- Mock adapters run the complete application without hardware.
- Services depend on interfaces rather than concrete devices.
- Hardware errors are normalized.
- Capabilities can be queried.
- One missing optional adapter does not prevent Rover from booting.
