# 004 — Hardware Specification

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft  
**Primary Vehicle Target:** 2014 Toyota Prius  
**Initial Compute Target:** Raspberry Pi 5  

---

## 1. Purpose

This document defines the initial hardware architecture for ARGO Rover OS. It describes the major hardware subsystems required to build a custom vehicle computing platform that can replace or exceed a commercial aftermarket head unit while remaining modular, serviceable, and upgradeable.

This document is not a final shopping list. Specific parts are tracked in `hardware/bill_of_materials.md`.

---

## 2. Hardware Design Principles

Rover hardware should follow these principles:

- Prefer non-invasive installation.
- Avoid cutting the factory vehicle harness when practical.
- Use fused power distribution.
- Separate vehicle-facing electrical signals from the main Linux computer.
- Treat the Raspberry Pi as replaceable compute, not the whole system.
- Use standard interfaces where possible: HDMI, USB, Ethernet, serial, OBD-II.
- Keep high-write workloads off microSD storage.
- Design for heat, vibration, voltage fluctuation, and service access.
- Preserve vehicle safety systems.

---

## 3. Hardware System Overview

```text
Vehicle 12V / ACC / Signals
        ↓
Fused Power + DC-DC Regulation
        ↓
Raspberry Pi 5 / Future Mini PC
        ↓
USB Hub / HDMI / Audio / Storage
        ↓
Touchscreen, DAC, OBD, Cameras, GPS, Microphone
```

Recommended subsystem boundaries:

- Compute subsystem.
- Display subsystem.
- Power subsystem.
- Audio subsystem.
- Vehicle interface subsystem.
- Camera subsystem.
- Network subsystem.
- Storage subsystem.
- Optional microcontroller subsystem.

---

## 4. Compute Subsystem

### 4.1 Initial Target

The initial compute target is Raspberry Pi 5.

Recommended configuration:

- Raspberry Pi 5, 8 GB preferred.
- Active cooling.
- NVMe HAT or equivalent SSD solution.
- Reliable USB-C or GPIO power input through an automotive-grade DC-DC regulator.
- Case or mount suitable for vehicle installation.

### 4.2 Future Compute Targets

The design should allow later migration to:

- Intel N100 mini PC.
- AMD Ryzen mini PC.
- Ruggedized fanless PC.
- ARM single-board computer.
- NVIDIA Jetson or similar edge AI device.

No vehicle integration should depend on Raspberry Pi-only behavior unless isolated behind a platform adapter.

---

## 5. Display Subsystem

### 5.1 Requirements

The primary display should be a large capacitive touchscreen.

Target specs:

- Size: approximately 13.3 inches.
- Resolution: 1920x1080 minimum.
- Input: HDMI preferred.
- Touch: USB capacitive touch.
- Orientation: landscape by default.
- Brightness: high enough for daytime cabin use.
- Mounting: compatible with a custom dashboard bezel.

### 5.2 Display Integration

The display should connect to the compute device via:

- HDMI for video.
- USB for touch.
- Separate regulated power if required.

The display should not be powered from an unstable cigarette lighter adapter in the final install.

### 5.3 Mounting

The final display mount will likely require a custom bezel or bracket.

Design priorities:

- Does not obstruct airbags.
- Does not block critical controls.
- Does not create sharp impact hazards.
- Allows service access.
- Looks intentional rather than temporary.

---

## 6. Power Subsystem

### 6.1 Power Sources

The system may need access to:

- Constant 12V.
- Switched accessory / ignition signal.
- Ground.
- Optional illumination / dimmer signal.
- Optional reverse signal.

### 6.2 Power Conversion

The compute and accessories require regulated power.

Likely outputs:

- 5V high-current output for Raspberry Pi.
- 5V USB power for peripherals.
- 12V accessory output for display or cameras if needed.

Power conversion should use automotive-rated DC-DC regulators where practical.

### 6.3 Protection

The system should include:

- Appropriate fuse near source power.
- Separate fused circuits for compute, display, and accessories where practical.
- Common ground strategy.
- Strain relief.
- Proper wire gauge.
- Protection from shorts.

### 6.4 Shutdown

The power subsystem should support graceful shutdown.

Recommended strategy:

1. Vehicle I/O controller detects ignition off.
2. Controller notifies backend or shutdown daemon.
3. Rover OS closes services and syncs storage.
4. Linux shuts down cleanly.
5. Power controller cuts compute power after a delay.

---

## 7. Storage Subsystem

### 7.1 Primary Storage

The system should use NVMe or SSD storage for the operating system.

MicroSD may be acceptable for early testing, but should not be the long-term storage target for a daily-driver vehicle system.

### 7.2 Data Types

Storage must account for:

- OS files.
- Application code.
- Logs.
- Settings database.
- Trip history.
- Diagnostic history.
- Cached maps or assets.
- Optional dashcam footage.

### 7.3 Dashcam Storage

Dashcam footage should be stored as files, not inside SQLite.

The database should store metadata such as:

- File path.
- Start time.
- End time.
- Camera source.
- GPS location if available.
- Event marker.

---

## 8. Audio Subsystem

### 8.1 Audio Goals

The audio subsystem should support:

- Media playback.
- Navigation prompts.
- Voice assistant output.
- System sounds.
- Potential phone audio integration.

### 8.2 Prius JBL Consideration

Many Prius trims include a JBL amplified audio system. The hardware design should attempt to retain the factory amplifier and speakers where practical.

The system may require:

- Toyota/JBL integration harness.
- Line-level audio interface.
- Steering wheel control interface.
- USB DAC from compute device.

### 8.3 Audio Path Options

Possible early audio paths:

1. HDMI audio during bench testing.
2. USB DAC to auxiliary input or test amplifier.
3. USB DAC to factory JBL integration interface.
4. Dedicated DSP/amplifier in a later version.

The production target should avoid noisy, low-quality analog output from the Pi headphone path because Raspberry Pi 5 does not include a traditional analog audio jack.

---

## 9. Vehicle Interface Subsystem

### 9.1 OBD-II

OBD-II is the safest initial vehicle data path.

Supported adapter types:

- USB OBD-II adapter.
- Bluetooth OBD-II adapter.
- Wi-Fi OBD-II adapter.

USB is preferred for reliability in the installed system.

### 9.2 CAN Bus

CAN bus integration may be useful later, but should be treated carefully.

Rules:

- Read-only by default.
- No safety-critical writes.
- Document every PID/message before use.
- Keep CAN functionality behind vehicle adapters.

### 9.3 Vehicle Signals

Useful vehicle-facing signals may include:

- Ignition / ACC.
- Reverse.
- Illumination / dimmer.
- Parking brake.
- Steering wheel controls.

These should be handled through harness adapters or a vehicle I/O controller where practical.

---

## 10. Vehicle I/O Controller

A microcontroller-based vehicle I/O controller is recommended.

Responsibilities may include:

- Reading ignition state.
- Reading reverse trigger.
- Reading illumination state.
- Reading button inputs.
- Sending safe shutdown signal.
- Acting as watchdog.
- Translating simple vehicle events to USB serial messages.

Potential platforms:

- RP2040.
- ESP32.
- STM32.

The controller should expose a simple documented protocol to the Linux host.

---

## 11. Camera Subsystem

### 11.1 Required Camera

The system must support at least one rear camera for backup view.

### 11.2 Optional Cameras

Future cameras may include:

- Front camera.
- Cabin camera.
- Side camera.
- Auxiliary cargo / trailer camera.

### 11.3 Camera Interface Options

Possible interfaces:

- USB UVC camera.
- HDMI camera through USB capture.
- Analog camera through USB capture.
- IP camera stream.

Early development should choose the simplest reliable path.

---

## 12. Networking Subsystem

### 12.1 Initial Network Strategy

A dedicated car phone is not required for v0.1.

Initial connectivity should use:

- Driver phone hotspot.
- Home Wi-Fi when parked nearby.
- Manual offline mode when no network is available.

### 12.2 Future Network Strategy

Future connectivity may include:

- Dedicated LTE phone.
- USB LTE modem.
- 5G hotspot.
- Remote tunnel.
- ARGO-managed sync.

Always-on remote connectivity is a future feature, not a requirement for the first install.

---

## 13. GPS Subsystem

GPS may be provided by:

- USB GPS receiver.
- Phone-provided location.
- Future LTE/GPS modem.

A USB GPS receiver is preferred for independent local logging if trip history becomes a core feature.

---

## 14. Microphone and Voice Subsystem

The system should support a USB microphone for:

- Voice commands.
- ARGO assistant.
- Possible hands-free audio experiments.

Placement should minimize road noise.

Potential locations:

- Near factory microphone position.
- Overhead console area.
- A-pillar area, avoiding airbag interference.

---

## 15. USB Topology

A powered USB hub is strongly recommended.

Potential USB devices:

- Touchscreen touch controller.
- USB DAC.
- OBD-II adapter.
- GPS receiver.
- Microphone.
- Camera capture device.
- Keyboard/trackpad for maintenance.
- Vehicle I/O controller.

The hub should be powered from a stable regulated supply.

---

## 16. Bench Build Requirements

Before installation in the vehicle, the following should be tested on a bench:

- Pi boots from target storage.
- Touchscreen works.
- Kiosk mode launches.
- USB hub remains stable.
- Audio output works.
- Mock telemetry displays.
- OBD-II adapter can connect to test software.
- Camera input works.
- Shutdown behavior is tested.

---

## 17. Vehicle Install Requirements

Initial vehicle install should prioritize reversibility.

Preferred:

- Plug-and-play harnesses.
- Add-a-fuse or properly tapped circuits only where appropriate.
- No factory harness cutting unless documented and unavoidable.
- No modification of airbag components.
- No obstruction of driver visibility.

---

## 18. Hardware Acceptance Criteria

The v0.1 bench hardware is acceptable when:

- Display works reliably.
- Touch input works.
- Pi boots from stable storage.
- Backend and frontend run.
- At least one camera can be displayed.
- Audio output can be tested.
- OBD-II adapter is recognized.

The v1.0 vehicle hardware is acceptable when:

- System boots consistently in the car.
- System shuts down safely.
- Audio works through the vehicle speakers.
- Backup camera works reliably.
- OBD-II telemetry works reliably.
- Wiring is fused, secured, and documented.
- Installation can be serviced without tearing the whole dash apart.
