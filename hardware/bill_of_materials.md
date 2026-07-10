# Hardware Bill of Materials

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft / Living Document  
**Primary Vehicle Target:** 2014 Toyota Prius  

---

## 1. Purpose

This document tracks the hardware required to build ARGO Rover OS. It separates required parts, recommended parts, optional upgrades, and future research items.

Prices, exact models, and vendor links should be updated during purchasing research. This file should remain a living document as the hardware design matures.

---

## 2. Purchasing Philosophy

Before buying parts:

- Confirm the exact Prius trim.
- Confirm whether the car has JBL audio.
- Confirm whether it has a factory backup camera.
- Confirm dashboard/radio configuration.
- Confirm available mounting space.
- Bench-test the compute and display stack.

Prefer parts that are:

- Documented.
- Replaceable.
- Standard-interface.
- Available from multiple vendors.
- Compatible with Linux.
- Reversible in the vehicle.

---

## 3. Required for Bench Prototype

These parts are required to build the first working Rover OS bench prototype.

| Category | Item | Requirement | Status | Notes |
|---|---|---|---|---|
| Compute | Raspberry Pi 5 | 8 GB preferred | Needed | Initial runtime target |
| Cooling | Pi 5 active cooler | Required | Needed | Vehicle cabin heat makes cooling important |
| Storage | NVMe HAT / SSD adapter | Strongly recommended | Needed | Avoid relying on microSD for heavy writes |
| Storage | NVMe SSD | 256 GB minimum, 1 TB preferred | Needed | OS, logs, cache, future video metadata |
| Display | 13.3 inch HDMI capacitive touchscreen | 1080p minimum | Needed | Primary UI display |
| Display Cable | HDMI cable | Short, reliable | Needed | Use right-angle if useful |
| Touch Cable | USB cable | For touchscreen input | Needed | Connects display touch controller |
| USB | Powered USB hub | Stable power | Needed | Required for DAC, touch, OBD, GPS, camera |
| Audio | USB DAC | Linux compatible | Needed | Bench and vehicle audio output |
| Camera | USB camera or USB capture device | UVC compatible preferred | Needed | Used to prototype camera screen |
| Power | Bench power supply | Stable 5V/USB-C | Needed | For desk testing |
| Input | USB keyboard/trackpad | Maintenance only | Optional | Useful during setup |

---

## 4. Required for Vehicle Prototype

These parts are needed before installing Rover OS in the Prius.

| Category | Item | Requirement | Status | Notes |
|---|---|---|---|---|
| Power | Automotive DC-DC converter | Stable 5V high-current | Needed | Must support Pi 5 power needs |
| Power | Fuse holder / add-a-fuse / fuse tap | Vehicle appropriate | Needed | Exact method TBD |
| Power | Fused distribution block | Recommended | Needed | Keeps added circuits organized |
| Power | Automotive wire | Proper gauge | Needed | Use stranded automotive wire |
| Power | Ring terminals / connectors | Crimped properly | Needed | Avoid loose temporary wiring |
| Mounting | Pi enclosure or mount | Secure installation | Needed | Should allow airflow |
| Mounting | Display mount / bezel | Custom likely | Needed | Final design TBD |
| Audio | Toyota radio harness adapter | Prius compatible | Needed | No-cut install strategy |
| Audio | JBL retention interface | Required if JBL present | Unknown | Confirm vehicle audio system |
| Audio | Line-level interface | May be required | Unknown | Depends on JBL interface path |
| Controls | Steering wheel control interface | Desired | Needed | Required for head-unit parity |
| Camera | Backup camera | Existing or aftermarket | Unknown | Confirm if car has one |
| Camera | Camera capture interface | Depends on camera type | Needed | USB capture likely |
| Vehicle Data | USB OBD-II adapter | Linux compatible | Needed | USB preferred over Bluetooth |
| Safety | Wire loom / heat shrink / zip ties | Required | Needed | Secure and protect wiring |

---

## 5. Recommended Parts by Subsystem

### 5.1 Compute

Recommended initial compute:

- Raspberry Pi 5, 8 GB.
- Official or high-quality active cooler.
- NVMe HAT.
- NVMe SSD.
- Rugged or ventilated case.

Future compute upgrade candidates:

- Intel N100 mini PC.
- AMD Ryzen mini PC.
- Fanless industrial PC.

### 5.2 Display

Target display requirements:

- 13.3 inch class.
- HDMI input.
- USB capacitive touch.
- 1920x1080 minimum.
- High brightness preferred.
- Mountable without blocking controls.

Research candidates:

- Waveshare-style 13.3 inch HDMI capacitive touchscreen.
- Lilliput-style open-frame industrial touchscreen.
- Other HDMI/USB touch monitor compatible with Linux.

### 5.3 Power

Required characteristics:

- Automotive input tolerance.
- Stable 5V output.
- Enough current for Pi 5 and attached peripherals.
- Fuse protection.
- Safe shutdown support, either built-in or through external controller.

Research candidates:

- Automotive Raspberry Pi power supply with ignition sense.
- Dedicated DC-DC buck converter plus vehicle I/O controller.
- UPS HAT only if it handles vehicle startup/shutdown gracefully.

### 5.4 Audio

Required characteristics:

- Linux-compatible USB DAC.
- Clean line-level output.
- Integration path to Prius audio system.

Research candidates:

- USB DAC with line out.
- Toyota/JBL amplifier retention interface.
- PAC / Metra / iDatalink-style integration products, depending on compatibility.

### 5.5 Vehicle Data

Initial data path:

- USB OBD-II adapter.

Future data path:

- CAN interface.
- Vehicle I/O microcontroller.
- Prius-specific data research.

Preferred OBD-II adapter characteristics:

- USB connection.
- ELM327-compatible or better.
- Reliable Linux serial support.
- Known-good with Python/Node OBD libraries.

### 5.6 Cameras

Initial camera path:

- USB UVC camera for bench testing.
- USB capture device for backup camera integration.

Production camera path TBD after confirming vehicle equipment.

Possible sources:

- Existing factory rear camera.
- Aftermarket analog rear camera.
- USB rear camera.
- HDMI camera through USB capture.

### 5.7 GPS

GPS is optional for v0.1 but recommended for trip logging.

Possible options:

- USB GPS receiver.
- Phone-provided location.
- Future LTE/GPS modem.

### 5.8 Microphone

Recommended:

- USB microphone.
- Noise-resistant placement.
- Hidden near factory microphone area if practical.

Use cases:

- Voice assistant.
- Voice notes.
- Future hands-free experiments.

---

## 6. Optional Upgrades

These are not required for the first build but should be considered later.

| Item | Purpose | Priority |
|---|---|---|
| Dedicated LTE modem | Always-on connectivity | Later |
| Dedicated car phone | LTE, GPS, remote access | Later |
| Front camera | Parking/security/dashcam | Medium |
| Cabin camera | Security/logging | Low |
| Side cameras | Expanded visibility | Low |
| SDR receiver | Radio/scanner experiments | Low |
| LoRa module | Experimental communications | Low |
| Environmental sensor | Cabin temp/humidity | Medium |
| TPMS reader | Tire monitoring | Research |
| DSP amplifier | Better audio control | Later |
| Physical control knob | Volume / quick actions | Medium |

---

## 7. Dedicated Car Phone Decision

A dedicated car phone is not required for v0.1.

Initial network plan:

- Use driver phone hotspot when available.
- Allow Rover OS to operate offline when no network exists.

A dedicated phone or LTE modem becomes useful if Rover needs:

- Remote access while parked.
- Always-on GPS location.
- Automatic dashcam upload without driver phone present.
- Security mode notifications.
- Independent vehicle connectivity.

---

## 8. Open Research Items

These items must be researched before final purchase or installation.

- Exact 2014 Prius trim and audio configuration.
- Whether the target Prius has JBL audio.
- Whether the target Prius has a factory backup camera.
- Which Toyota harness adapters fit the exact trim.
- Best JBL retention interface for Linux/USB DAC source.
- Steering wheel control interface options usable by a custom computer.
- Best reverse trigger source.
- Best ignition-sense power supply for Pi 5.
- Display brightness and heat performance.
- Safe dash mounting strategy for a 13.3 inch screen.
- Prius Gen 3 OBD-II and hybrid telemetry support.

---

## 9. Versioned Hardware Targets

### Rover Hardware v0.1 — Bench

Goal: prove software and UI stack.

Includes:

- Pi 5.
- Touchscreen.
- USB DAC.
- Mock vehicle data.
- USB camera.
- Local storage.

### Rover Hardware v0.2 — Vehicle Non-Invasive

Goal: run in car without deep integration.

Includes:

- Vehicle power.
- OBD-II.
- Temporary mount.
- Basic audio output.
- Manual camera support.

### Rover Hardware v0.3 — Integrated Prototype

Goal: head-unit parity.

Includes:

- Factory speaker integration.
- Backup camera trigger.
- Steering wheel controls.
- Ignition-aware shutdown.
- Secure mounting.

### Rover Hardware v1.0 — Daily Driver

Goal: reliable long-term install.

Includes:

- Finished bezel.
- Documented wiring.
- Stable power.
- Serviceable install.
- Reliable audio/camera/diagnostics.

---

## 10. Notes for Purchasing

Do not buy all parts at once.

Recommended purchase order:

1. Pi 5 compute stack.
2. Display.
3. Storage and USB hub.
4. USB DAC and test camera.
5. OBD-II adapter.
6. Bench power and development accessories.
7. Vehicle-specific harnesses after exact Prius is confirmed.
8. Mounting and power hardware after physical test fitting.

This reduces the risk of buying Prius-specific parts before the exact car configuration is known.
