# 018 — Audio Architecture

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft

---

## 1. Purpose

This document defines how Rover OS should produce, mix, route, and control audio while preserving the 2014 Prius factory speaker system where practical.

The audio subsystem must support media, navigation prompts, system sounds, voice-assistant responses, and future phone integration without tying Rover OS to one DAC, amplifier, or vehicle.

---

## 2. Audio Goals

The audio system should:

- Match normal head-unit usability.
- Use a clean digital-to-analog path.
- Retain the factory JBL amplifier when practical.
- Provide software and physical volume control.
- Support navigation ducking.
- Support steering-wheel buttons.
- Recover from missing audio devices.
- Avoid startup pops and shutdown noise.
- Remain replaceable through an adapter interface.

---

## 3. Recommended Initial Signal Path

```text
Rover OS
   ↓
Linux Audio Stack
   ↓
USB DAC
   ↓
Line-Level / JBL Retention Interface
   ↓
Factory JBL Amplifier
   ↓
Factory Speakers
```

Bench testing may use HDMI audio or powered desktop speakers.

---

## 4. Factory JBL Retention

Before purchasing interfaces, confirm:

- Exact Prius trim.
- Factory radio model.
- JBL branding and amplifier presence.
- Existing camera and steering-wheel wiring.

A compatible retention interface should preserve the factory amplifier while accepting line-level output from Rover.

The exact part must be validated against the specific vehicle before installation.

---

## 5. Audio Adapter Interface

The backend should define a generic audio adapter.

```ts
interface AudioAdapter {
  initialize(): Promise<void>;
  getState(): Promise<AudioState>;
  setMasterVolume(level: number): Promise<void>;
  setMuted(muted: boolean): Promise<void>;
  setSource(source: AudioSource): Promise<void>;
  playSystemSound(id: string): Promise<void>;
  shutdown(): Promise<void>;
}
```

Implementations may include:

- Linux PipeWire adapter.
- ALSA adapter.
- Mock audio adapter.
- Future DSP adapter.

---

## 6. Linux Audio Stack

PipeWire is the preferred long-term Linux audio layer if stable on the selected OS image.

Advantages:

- Device routing.
- Mixing.
- Bluetooth integration.
- Per-application volume.
- Future microphone processing.

ALSA may be used directly for early bench work.

The frontend should never invoke `amixer`, `pactl`, or hardware commands directly.

---

## 7. Audio Sources

Rover should normalize these sources:

- Local media.
- Bluetooth media.
- Navigation prompt.
- System alert.
- Voice assistant.
- Phone-call audio.
- External auxiliary input.

The active source and playback state should be visible through the media service.

---

## 8. Mixing and Priority

Suggested priority order:

1. Critical system alert.
2. Phone call.
3. Navigation instruction.
4. Voice assistant.
5. Media playback.
6. Non-critical system sound.

Navigation and voice prompts should duck media rather than stop it.

Critical alerts may temporarily override all other sources.

---

## 9. Volume Model

Rover should maintain:

- Master volume.
- Mute state.
- Optional source-relative levels.
- Maximum startup volume.

Recommended range:

```text
0–100 logical volume
```

The audio adapter maps logical volume to the actual device.

Startup behavior should restore the last volume but clamp it to a configurable safe maximum.

---

## 10. Physical Controls

Required control paths:

- Touchscreen volume controls.
- Steering-wheel volume buttons.

Recommended future control:

- Physical rotary encoder or knob.

All controls should publish generic input events and call the same media/audio service commands.

---

## 11. Steering-Wheel Integration

Generic events:

- `input.volume_up`
- `input.volume_down`
- `input.mute_toggle`
- `input.media_next`
- `input.media_previous`
- `input.play_pause`
- `input.voice_toggle`

Button repeat behavior must be rate-limited and tested while driving.

---

## 12. Bluetooth Media

Bluetooth media support should eventually provide:

- A2DP sink behavior.
- Automatic reconnection to the driver phone.
- Metadata where available.
- Play/pause/next/previous through AVRCP.

Bluetooth audio should be treated as one source behind the media service.

The first software milestone may defer full Bluetooth sink support in favor of local playback or phone projection.

---

## 13. Phone Calls

Hands-free calling is one of the hardest parity features.

A complete solution requires:

- Bluetooth HFP support.
- Microphone capture.
- Echo cancellation.
- Noise reduction.
- Call-state UI.
- Reliable routing to vehicle speakers.

This should be treated as a dedicated milestone rather than assumed to work automatically through Linux Bluetooth.

Android Auto or CarPlay projection may provide the most reliable early call experience.

---

## 14. Microphone Architecture

Initial microphone path:

```text
USB Microphone
   ↓
PipeWire / ALSA
   ↓
Voice Service or Phone Service
```

Requirements:

- Selectable capture device.
- Gain configuration.
- Noise testing in the moving vehicle.
- Clear privacy indicator when actively recording.

Do not place microphones in a way that interferes with curtain airbags.

---

## 15. Navigation Prompts

The navigation service should request transient audio focus.

Expected behavior:

1. Reduce media volume.
2. Play prompt.
3. Restore media volume smoothly.

Prompts should remain audible even when media is paused, unless master mute applies.

---

## 16. Startup and Shutdown Behavior

Startup:

- Initialize the DAC before enabling amplifier output where possible.
- Start muted or at a safe level.
- Avoid startup chimes until audio is stable.

Shutdown:

- Fade or mute audio.
- Stop active playback.
- Disable amplifier remote output if controlled.
- Close the audio device cleanly.

---

## 17. Device Failure Handling

If the configured DAC disappears:

- Mark audio service degraded.
- Attempt bounded reconnection.
- Do not crash the backend.
- Show a clear UI warning.
- Select a configured fallback only if safe.

Rover should not unexpectedly route private audio to an unintended Bluetooth or HDMI device.

---

## 18. Audio Quality Requirements

The installed system should be evaluated for:

- Ground-loop noise.
- Alternator or converter noise.
- Digital interference.
- Clipping.
- Channel balance.
- Startup pops.
- Latency.

Use isolated or differential interfaces if needed. Avoid cheap unshielded analog paths in the final install.

---

## 19. Testing

Bench tests:

- DAC detection.
- Volume control.
- Mute.
- Source switching.
- Navigation ducking.
- Device removal/reconnect.

Vehicle tests:

- Noise with car off, accessory, and READY.
- Steering-wheel response.
- Voice intelligibility.
- Startup/shutdown noise.
- Maximum safe volume.

---

## 20. Acceptance Criteria

Audio is acceptable for v1.0 when:

- Media plays reliably through vehicle speakers.
- Volume and mute work from touchscreen and steering wheel.
- Navigation prompts can duck media.
- Startup and shutdown do not produce harmful pops.
- Audio device failures are reported and recoverable.
- The factory JBL system is retained or a documented replacement path is installed.
