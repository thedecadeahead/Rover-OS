# 016 — Security and Privacy

**Project:** ARGO Rover OS  
**Version:** 0.1 Alpha  
**Status:** Draft

---

## 1. Purpose

This document defines the initial security and privacy model for Rover OS. The system handles vehicle data, location history, diagnostic information, integration credentials, camera footage, and remote connectivity. Security must be designed in from the beginning rather than added after deployment.

---

## 2. Security Goals

Rover OS should:

- Default to local-only operation.
- Continue functioning without cloud services.
- Protect credentials and private vehicle data.
- Avoid exposing unsafe vehicle-control interfaces.
- Limit the damage caused by a compromised service.
- Provide secure update and rollback paths.
- Make remote access opt-in and authenticated.

---

## 3. Threat Model

Primary threats include:

- Unauthorized access over Wi-Fi or hotspot networks.
- Theft of the compute unit or storage device.
- Malicious or compromised USB peripherals.
- Exposed API tokens.
- Vulnerable third-party packages.
- Unsafe remote tunnels.
- Corrupted or malicious updates.
- Accidental disclosure of trip history or camera footage.
- Untrusted CAN or OBD devices.

Rover OS is not intended to defend against a fully compromised vehicle network, but it should avoid increasing risk.

---

## 4. Trust Boundaries

Trust boundaries include:

- Frontend to local backend.
- Backend to vehicle adapters.
- Backend to ARGO Core.
- Backend to Home Assistant.
- Rover device to driver phone hotspot.
- Rover device to public or home Wi-Fi.
- Rover device to removable USB hardware.

Each boundary should use explicit authentication, validation, or isolation appropriate to the risk.

---

## 5. Local API Security

Default behavior:

- Bind backend APIs to `127.0.0.1` only.
- Do not expose control APIs on the LAN by default.
- Require explicit configuration for remote access.
- Separate read-only status endpoints from command endpoints.
- Validate every request body.
- Apply request size and rate limits.

If LAN access is enabled:

- Use HTTPS where practical.
- Require authentication.
- Restrict allowed origins.
- Log failed access attempts.

---

## 6. Authentication and Authorization

The first local kiosk may use trusted localhost access.

Future remote access should support:

- Device-specific credentials.
- Short-lived access tokens.
- Role separation between viewer, operator, and administrator.
- Revocation.
- Audit logs.

Sensitive commands such as shutdown, settings reset, remote sync, or update installation should require elevated authorization.

---

## 7. Secrets Management

Secrets must not be committed to Git.

Examples:

- ARGO Core API token.
- Home Assistant token.
- Remote tunnel credentials.
- Wi-Fi credentials.
- Encryption keys.

Recommended storage:

- Root-readable environment file.
- systemd credentials.
- OS keyring or encrypted local secret store.

The frontend must never receive secrets that it does not need.

---

## 8. Data Classification

### Public

- Application version.
- Open-source documentation.
- Generic hardware capability data.

### Internal

- Service health.
- Non-sensitive settings.
- Hardware configuration.

### Sensitive

- VIN.
- GPS and trip history.
- Home/work destinations.
- Camera footage.
- Diagnostic history.
- Network names.

### Secret

- Tokens.
- Passwords.
- Private keys.
- Remote access credentials.

Storage and logging rules should reflect this classification.

---

## 9. Privacy Controls

Users should be able to:

- Disable GPS history.
- Disable trip history.
- Disable camera recording.
- Delete trips and recordings.
- Export local data.
- Disable ARGO sync.
- Disable remote access.

The system should make clear when cameras, GPS, or remote sync are active.

---

## 10. Camera Privacy

Camera requirements:

- Display a recording indicator.
- Store footage locally by default.
- Use configurable retention.
- Mark protected clips explicitly.
- Avoid cabin recording by default.
- Require explicit enablement for remote camera access.

Remote live camera viewing should be disabled until authentication and encryption are implemented.

---

## 11. Network Security

Rover OS should assume hotspot and public Wi-Fi networks are untrusted.

Requirements:

- Block unsolicited inbound traffic by default.
- Use a host firewall.
- Disable unused services.
- Avoid passwordless SSH.
- Use SSH keys if SSH is enabled.
- Disable developer ports in vehicle mode.
- Prefer outbound authenticated tunnels over inbound port forwarding.

---

## 12. Vehicle Interface Safety

OBD and CAN interfaces should be treated as untrusted inputs.

Rules:

- Read-only by default.
- Validate adapter responses.
- Time out stalled devices.
- Do not permit arbitrary CAN writes through public APIs.
- Keep vehicle-control functions out of the first release.
- Document every supported message.

---

## 13. Update Security

Production updates should eventually support:

- Signed release artifacts.
- Checksum verification.
- Versioned packages.
- Rollback to last known-good version.
- Clear separation between stable and development channels.

Do not auto-install unverified code while the car is in use.

---

## 14. Dependency Security

The project should:

- Pin major dependency versions.
- Run automated vulnerability scanning.
- Use lockfiles.
- Review high-risk native packages.
- Minimize privileged dependencies.
- Keep Linux security updates current.

---

## 15. Logging Rules

Logs must not contain:

- Full access tokens.
- Passwords.
- Private keys.
- Unredacted authorization headers.
- Full location history unless explicitly required.

Security-relevant logs should include:

- Failed authentication.
- Remote access enablement.
- Update installation.
- Settings reset.
- Unexpected service privilege errors.

---

## 16. Physical Security

Recommended controls:

- Mount compute hardware out of obvious sight.
- Secure storage devices.
- Encrypt sensitive partitions if boot performance permits.
- Require credentials for maintenance access.
- Avoid exposing USB maintenance ports publicly.

The system should remain recoverable if hardware is stolen, but private data should not be trivially readable.

---

## 17. Incident Response

The project should document procedures to:

- Revoke ARGO and Home Assistant tokens.
- Disable remote access.
- Rotate credentials.
- Restore from known-good images.
- Export logs for investigation.
- Wipe sensitive local data before hardware disposal.

---

## 18. Acceptance Criteria

Security is acceptable for v0.1 when:

- APIs bind to localhost by default.
- Secrets are excluded from source control.
- Mock and development modes do not require production credentials.
- Input validation exists for APIs and events.
- No unsafe vehicle write functions are exposed.
- Privacy-sensitive features are disabled or clearly disclosed.
