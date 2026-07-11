# 014 — Deployment

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft  
**Initial Production Target:** Raspberry Pi 5  

---

## 1. Purpose

This document defines how Rover OS should be built, installed, configured, started, updated, backed up, and recovered on its initial Raspberry Pi 5 vehicle computer.

The deployment process should support three environments:

- Development computer.
- Raspberry Pi bench system.
- Installed vehicle system.

The final vehicle system must start automatically, run without a normal desktop workflow, and shut down cleanly when requested by the power-management subsystem.

---

## 2. Deployment Principles

- Build reproducibly from source.
- Keep environment-specific configuration outside application code.
- Run services as an unprivileged user where practical.
- Use systemd for process supervision.
- Serve a production frontend build, not a Vite development server.
- Start local core services before optional network integrations.
- Recover automatically from ordinary process crashes.
- Avoid automatic updates that can strand the vehicle interface.
- Back up user data before risky updates.
- Keep a documented rollback path.

---

## 3. Target Operating System

Initial recommendation:

- 64-bit Raspberry Pi OS Lite or another Debian-based 64-bit Linux distribution with Raspberry Pi 5 support.

A minimal base is preferred because Rover OS should own the visible interface.

Required platform capabilities:

- systemd.
- Chromium or another supported kiosk browser.
- Node.js LTS.
- SQLite.
- HDMI output.
- USB touchscreen input.
- ALSA/PipeWire audio support.
- V4L2 camera support.
- Serial and optional SocketCAN support.

The exact OS image and version should be pinned before the first repeatable bench deployment.

---

## 4. Deployment Layout

Recommended filesystem layout:

```text
/opt/rover-os/
├── releases/
│   ├── 0.1.0/
│   └── 0.1.1/
├── current -> /opt/rover-os/releases/0.1.1
├── shared/
│   ├── config/
│   ├── data/
│   ├── logs/
│   ├── media/
│   └── backups/
└── scripts/
```

Recommended ownership:

```text
user: rover
 group: rover
```

Device-access groups may also be required, such as `dialout`, `video`, or `audio`.

---

## 5. Application Build

The production build should produce:

- Compiled backend JavaScript.
- Static frontend assets.
- Shared schema package output.
- Database migrations.
- Deployment scripts.
- Version metadata.

Example build flow:

```bash
pnpm install --frozen-lockfile
pnpm lint
pnpm typecheck
pnpm test
pnpm build
```

The output should not require TypeScript transpilation on every vehicle boot.

---

## 6. Release Artifact

A release artifact should contain only runtime requirements.

Suggested contents:

```text
rover-os-0.1.0/
├── backend/
├── frontend/
├── shared/
├── migrations/
├── scripts/
├── package.json
├── pnpm-lock.yaml
├── VERSION
└── checksums.txt
```

Release artifacts should be checksummed.

Future releases may be distributed through GitHub Releases, ARGO Core, or a locally copied archive.

---

## 7. Configuration

Production configuration should live under:

```text
/opt/rover-os/shared/config/
```

Example files:

- `rover.env`
- `hardware.json`
- `vehicle.json`
- `logging.json`

File permissions should restrict secrets.

Example environment:

```dotenv
ROVER_MODE=vehicle
ROVER_HTTP_HOST=127.0.0.1
ROVER_HTTP_PORT=3746
ROVER_DATABASE_PATH=/opt/rover-os/shared/data/rover.db
ROVER_LOG_LEVEL=info
ROVER_VEHICLE_ADAPTER=toyota-prius-gen3
ROVER_MEDIA_PATH=/opt/rover-os/shared/media
```

---

## 8. Rover User Account

Create a dedicated system user.

Conceptual example:

```bash
sudo useradd --system --create-home --shell /usr/sbin/nologin rover
sudo usermod -aG dialout,video,audio rover
```

Exact groups depend on the selected OS and hardware.

Do not run the whole backend as root merely to access USB or video devices. Use udev rules and group permissions.

---

## 9. Backend systemd Service

The backend should run as a supervised systemd service.

Conceptual unit:

```ini
[Unit]
Description=ARGO Rover OS Backend
After=network.target local-fs.target
Wants=network.target

[Service]
Type=simple
User=rover
Group=rover
WorkingDirectory=/opt/rover-os/current/backend
EnvironmentFile=/opt/rover-os/shared/config/rover.env
ExecStart=/usr/bin/node dist/index.js
Restart=on-failure
RestartSec=2
TimeoutStopSec=30

[Install]
WantedBy=multi-user.target
```

The real service should include hardening options after device-access requirements are known.

---

## 10. Frontend Serving

Preferred options:

1. Backend serves the static frontend bundle.
2. A small local web server serves the bundle.

Serving through the backend is simpler for the first implementation.

The production frontend URL should be local, for example:

```text
http://127.0.0.1:3746/
```

---

## 11. Kiosk Session

Chromium should start in kiosk mode after the backend is healthy enough to serve the UI.

Desired behavior:

- No browser chrome.
- No restore-session dialog.
- No password prompt.
- Hide mouse cursor after inactivity.
- Disable screen blanking while the vehicle system is active.
- Reopen after browser crash.

The kiosk service should be separate from the backend so either can restart independently.

---

## 12. Startup Order

Target startup sequence:

```text
Regulated power becomes available
        ↓
Linux boots
        ↓
Filesystem and database become available
        ↓
Rover backend starts
        ↓
Core services initialize
        ↓
Kiosk browser starts
        ↓
Dashboard renders local state
        ↓
Optional network integrations connect
```

The dashboard must not wait for internet, ARGO Core, or Home Assistant.

---

## 13. Readiness and Health

The backend should expose:

```http
GET /health/live
GET /health/ready
```

`live` indicates the process is running.

`ready` indicates the frontend and essential local APIs can be served.

Optional hardware may be degraded without making the entire backend unready.

---

## 14. Database Migration During Deployment

Deployment should follow:

1. Stop writes or stop backend.
2. Back up database.
3. Install release.
4. Run migrations.
5. Start backend.
6. Verify health.
7. Switch `current` symlink only when appropriate.

If migration fails:

- Do not continue silently.
- Preserve logs.
- Restore previous release and database backup if required.

---

## 15. Logging

Production logs should use structured output and rotation.

Potential destinations:

- systemd journal.
- `/opt/rover-os/shared/logs/` for selected application logs.

Requirements:

- Avoid unbounded log growth.
- Redact secrets.
- Preserve recent startup and shutdown errors.
- Make logs exportable through parked/developer mode.

---

## 16. Media Storage

Dashcam and camera media should live outside release directories.

Recommended path:

```text
/opt/rover-os/shared/media/
```

The application should monitor:

- Free space.
- Media retention policy.
- Protected clips.
- Failed uploads.

Low storage must not corrupt the main database or prevent the dashboard from loading.

---

## 17. Shutdown Sequence

Target shutdown sequence:

```text
Ignition-off signal received
        ↓
Power service starts countdown
        ↓
UI displays shutdown pending
        ↓
Camera recording finalizes
        ↓
Trip closes
        ↓
Database and logs flush
        ↓
Backend exits
        ↓
Linux shuts down
        ↓
Power controller removes power after timeout
```

If ignition returns during the countdown, the shutdown may be cancelled before the final system shutdown stage.

---

## 18. Process Recovery

systemd should restart ordinary process failures.

Recovery rules:

- Backend crash → restart with bounded delay.
- Kiosk crash → restart browser.
- Hardware adapter crash → mark service degraded and restart adapter where possible.
- Repeated crash loop → preserve logs and show a recovery screen rather than rebooting forever.

---

## 19. Update Strategy

Updates should initially be manual or explicitly approved while parked.

Recommended update modes:

- USB/local archive.
- GitHub release download while parked.
- ARGO Core-managed staged update in the future.

Do not install unreviewed updates automatically during driving.

Update flow:

1. Download or copy artifact.
2. Verify checksum/signature.
3. Confirm vehicle is parked.
4. Back up data.
5. Install new release directory.
6. Run migrations.
7. Restart services.
8. Run health checks.
9. Roll back if checks fail.

---

## 20. Rollback Strategy

Keep at least one previous release.

Rollback should:

- Repoint `current` to previous release.
- Restore compatible database backup when necessary.
- Restart services.
- Record rollback reason.

Database migrations should be designed conservatively because schema rollback may be harder than application rollback.

---

## 21. Backup Strategy

Back up:

- SQLite database.
- Configuration.
- Vehicle profile.
- Maintenance history.
- Trip history if enabled.
- Protected media metadata.

Optional:

- Protected media files.
- Logs relevant to a failure.

Backup destinations:

- Local backup directory.
- Removable storage.
- ARGO Vault when connected.

Use SQLite's safe backup mechanisms rather than copying an actively written database blindly.

---

## 22. Recovery Mode

Rover OS should eventually provide a recovery mode that can:

- Start without hardware adapters.
- Disable third-party plugins.
- Use default display settings.
- Inspect service health.
- Export logs.
- Restore a backup.
- Select a previous release.
- Shut down safely.

Recovery mode should remain usable with keyboard/mouse during bench service.

---

## 23. Device Rules

Stable device names are important because `/dev/ttyUSB0` and `/dev/video0` may change.

Use udev rules based on:

- USB vendor/product IDs.
- Serial number.
- Physical port path where necessary.

Desired aliases:

```text
/dev/rover-obd
/dev/rover-vehicle-controller
/dev/rover-camera-rear
```

Document each rule with the hardware revision.

---

## 24. Network Deployment Posture

Default backend binding:

```text
127.0.0.1
```

Remote access should require explicit configuration, authentication, and a secure transport.

Do not expose the Rover API directly to the public internet.

Future remote access should use an ARGO-managed tunnel or authenticated VPN-style path.

---

## 25. Bench Deployment Checklist

- OS installed and updated.
- Rover user created.
- NVMe storage verified.
- Node.js and runtime dependencies installed.
- Release copied to `/opt/rover-os/releases/`.
- Configuration installed.
- Database migrated.
- Backend service enabled.
- Kiosk service enabled.
- Touchscreen tested.
- Audio tested.
- Camera tested.
- Reboot tested.
- Graceful shutdown tested.

---

## 26. Vehicle Deployment Checklist

- Bench qualification completed.
- Power supply fused and secured.
- Device cables strain-relieved.
- Active cooling verified.
- Startup tested repeatedly.
- Shutdown tested repeatedly.
- OBD adapter reconnect tested.
- Backup camera tested.
- Audio tested with vehicle READY.
- Day/night display tested.
- Recovery method available.

---

## 27. Acceptance Criteria

Deployment is acceptable for the bench release when:

- A clean target system can be installed from documentation.
- Backend and kiosk start automatically.
- Dashboard appears after reboot.
- Configuration persists outside release directories.
- Database migrations and backups work.
- Services restart after ordinary crashes.

Deployment is acceptable for the vehicle release when:

- Startup and shutdown are reliable.
- Updates can be rolled back.
- Hardware devices use stable configuration.
- Logs and backups are recoverable.
- No public unauthenticated API is exposed.
