# ARGO Rover OS

ARGO Rover OS is a Linux-based vehicle operating environment for the 2014 Toyota Prius, designed to replace or exceed a premium aftermarket head unit while acting as a mobile ARGO node.

## Initial Goals

- Run on Raspberry Pi 5 first, with a future upgrade path to mini PCs.
- Use a large HDMI capacitive touchscreen as the primary interface.
- Provide feature parity with store-bought Android Auto / CarPlay head units.
- Add custom Rover features: vehicle telemetry, diagnostics, ARGO integration, Home Assistant, cameras, and local-first services.
- Keep the architecture modular, hardware-agnostic, and vehicle-agnostic.

## Initial Stack

- Linux host OS
- Chromium kiosk mode
- React + TypeScript frontend
- Node.js backend services
- Local database
- WebSocket live event bus
- OBD-II / CAN integration layer
- Hardware abstraction layer

## Status

Design and documentation phase.
