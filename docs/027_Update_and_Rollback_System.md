# 027 — Update and Rollback System

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft  

---

## 1. Purpose

This document defines how Rover OS software is updated safely and how the system recovers when an update fails.

A vehicle computer must not become unusable because of a broken release, lost network connection, incomplete package install, or power interruption.

---

## 2. Goals

The update system should:

- Support manual and automatic update checks.
- Download updates without blocking normal driving functions.
- Verify update integrity.
- Install only while parked unless explicitly overridden for development.
- Preserve settings and user data.
- Roll back to a known-good release after failed startup.
- Record update history.
- Work through driver phone hotspot or home Wi-Fi.
- Remain optional during early development.

---

## 3. Update Units

Rover OS consists of several updateable units:

- Frontend bundle.
- Backend application.
- Shared schemas/types.
- Database migrations.
- systemd service definitions.
- Platform scripts.
- Vehicle adapter packages.
- Vehicle I/O controller firmware.

The host operating system should be updated separately from the Rover application unless a future image-based deployment is adopted.

---

## 4. Release Channels

Recommended channels:

- `development`: frequent, unstable builds.
- `beta`: bench and limited vehicle testing.
- `stable`: daily-driver releases.

Vehicle mode should default to stable once stable releases exist.

---

## 5. Versioning

Use semantic versioning where practical:

```text
MAJOR.MINOR.PATCH
```

Pre-release examples:

```text
0.1.0-alpha.1
0.2.0-beta.3
1.0.0
```

Every running installation should expose:

- Application version.
- Git commit SHA.
- Build date.
- Database schema version.
- Active vehicle adapter version.

---

## 6. Release Artifact

Recommended initial artifact:

- Signed or checksummed compressed release bundle.
- Manifest describing files, versions, migrations, and compatibility.

Example manifest:

```json
{
  "version": "0.2.0-beta.1",
  "commit": "abcdef123456",
  "minimumSchemaVersion": 4,
  "targetPlatforms": ["linux-arm64", "linux-x64"],
  "sha256": "..."
}
```

---

## 7. Update Workflow

```text
Check for release
    ↓
Download to staging directory
    ↓
Verify checksum/signature
    ↓
Validate platform compatibility
    ↓
Back up database and configuration
    ↓
Install into inactive release slot
    ↓
Run preflight checks
    ↓
Switch active release pointer
    ↓
Restart Rover services
    ↓
Run startup health checks
    ↓
Confirm success or roll back
```

---

## 8. Release Slots

Recommended application layout:

```text
/opt/rover-os/releases/0.1.0/
/opt/rover-os/releases/0.2.0/
/opt/rover-os/current -> /opt/rover-os/releases/0.2.0/
/opt/rover-os/previous -> /opt/rover-os/releases/0.1.0/
```

Updates should install into a new directory rather than modifying the active release in place.

---

## 9. Preflight Checks

Before activation, validate:

- Bundle checksum/signature.
- Correct CPU architecture.
- Minimum free storage.
- Required runtime version.
- Configuration readability.
- Database backup success.
- Migration compatibility.
- Frontend/backend build completeness.

Do not activate if preflight fails.

---

## 10. Database Migrations

Database migrations are the highest-risk update component.

Rules:

- Back up SQLite before migration.
- Use transactional migrations where possible.
- Never silently discard user data.
- Mark irreversible migrations explicitly.
- Test migration from every supported previous release.
- Roll back application and database together when feasible.

A release with an irreversible migration should require explicit confirmation during beta and early stable stages.

---

## 11. Startup Health Confirmation

A newly activated release should not be considered successful immediately.

Required checks:

- Backend starts.
- Database opens.
- API health endpoint responds.
- Frontend bundle is available.
- WebSocket endpoint starts.
- Core service health is not `error`.

The release should confirm healthy operation within a configured window, such as 60–120 seconds.

---

## 12. Automatic Rollback

Rollback should occur when:

- Backend repeatedly crashes after activation.
- Health check never succeeds.
- Database cannot open after migration.
- Frontend assets are missing.
- Watchdog sees repeated boot failure.

Rollback steps:

1. Stop failed services.
2. Restore previous release pointer.
3. Restore database backup when required.
4. Restart previous release.
5. Record failure details.
6. Disable automatic retry of the failed release.

---

## 13. Power Loss During Update

The update system must assume power can disappear.

Requirements:

- Never overwrite the only working release.
- Download to temporary files.
- Use atomic rename/symlink changes.
- Sync critical filesystem changes.
- Defer activation until all files are complete.
- Resume or discard incomplete downloads safely.

---

## 14. Driving Restrictions

While driving:

- Update checks may run.
- Downloads may run if bandwidth settings permit.
- Installation and restart must not run.

Activation should require:

- Parked interaction mode.
- Adequate voltage/power state.
- Explicit confirmation, or a configured maintenance window.

---

## 15. User Experience

Settings should show:

- Current version.
- Release channel.
- Last check time.
- Available version.
- Download progress.
- Installation state.
- Rollback availability.
- Recent update history.

A failed update should explain that Rover restored the prior working version.

---

## 16. Firmware Updates

Vehicle I/O controller firmware should use a separate process.

Requirements:

- Validate firmware target and checksum.
- Require parked mode.
- Keep existing firmware unless flashing succeeds.
- Avoid firmware updates during first vehicle prototype unless necessary.
- Document physical recovery/flashing procedure.

---

## 17. Security

Production updates should eventually support cryptographic signing.

The updater must:

- Use HTTPS for remote release metadata.
- Reject checksum mismatch.
- Reject incompatible artifacts.
- Avoid executing arbitrary downloaded scripts without validation.
- Protect update configuration from untrusted network users.

---

## 18. Development Workflow

During initial development, deployment may use Git and package-manager commands.

That workflow is acceptable for desktop and bench mode but should not be the final daily-driver update mechanism.

---

## 19. Acceptance Criteria

The update system is acceptable for beta when:

- A new application release installs into an inactive slot.
- Checksums are verified.
- Settings and database are preserved.
- Health checks confirm activation.
- A simulated failed release automatically rolls back.
- Power interruption before activation does not damage the active release.
