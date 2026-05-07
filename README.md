# MerlinOS

MerlinOS is a from-scratch mobile operating system initiative built directly on the upstream Linux kernel, without Android, Ubuntu Touch, postmarketOS, or other mobile distro forks. The immediate goal is to define a realistic architecture and staged execution plan before writing platform code.

## Vision and constraints

- **Kernel-first approach:** stay as close as possible to upstream Linux and contribute missing pieces upstream when feasible.
- **No Android userspace dependency:** avoid Binder/HAL/systemd-android compatibility layers unless absolutely required for specific hardware enablement.
- **Phone-native UX:** prioritize touch-first interaction, suspend/resume reliability, cellular telephony, and battery life over desktop compatibility.
- **Security baseline:** design with sandboxing, signed updates, and least-privilege service boundaries from day one.
- **Incremental bring-up:** optimize for observable milestones on real hardware, not only emulators.

## High-level design decisions

### 1) Hardware strategy: start narrow, then generalize

- Target one **well-supported reference device** first (preferably with good mainline Linux support).
- Define a board support matrix early:
  - Boot chain status (unlockable bootloader, secure boot options)
  - Mainline kernel compatibility
  - Modem integration path
  - GPU/display stack maturity
- Build abstraction layers only after one device is stable.

### 2) Kernel and low-level platform

- Track a modern **LTS Linux kernel** branch.
- Maintain a minimal patch set with strict upstreaming policy.
- Use device tree driven hardware description.
- Power management is a first-class requirement:
  - Suspend-to-RAM correctness
  - Wake source policy
  - Thermal and charging governance

### 3) Boot and system composition

- UEFI/U-Boot/fastboot flow depending on target hardware.
- Immutable base image with A/B (or equivalent) update slots.
- Early userspace responsible for verified mounts, rollback handling, and hardware sanity checks.

### 4) Userspace architecture

- Prefer a small, composable Linux userspace (musl or glibc decision deferred).
- Service supervision and dependency model should be explicit and observable.
- Split platform services by trust domain:
  - Telephony/radio service
  - Connectivity service (Wi-Fi/Bluetooth)
  - Media/camera service
  - UI compositor and shell

### 5) Graphics and UI stack

- Wayland-first architecture with a lightweight compositor tailored for phones.
- Toolkit decision should prioritize:
  - Touch ergonomics
  - Low memory footprint
  - Reliable GPU acceleration
- Define a shell contract for notifications, quick settings, lock screen, multitasking, and power controls.

### 6) App model and developer platform

- Start with system apps only (dialer, messaging, settings, camera, browser).
- Add third-party app runtime once security boundaries are proven.
- Package format should support:
  - Signatures
  - Permission metadata
  - Transactional install/uninstall

### 7) Security model

- Mandatory access control (SELinux/AppArmor decision to be made with prototype data).
- Per-service and per-app privilege minimization.
- Verified boot + signed OTA updates.
- Secrets handling for credentials, device keys, and provisioning.

### 8) Connectivity and telephony

- Modem control path must be isolated and restart-safe.
- SMS, voice, and data workflows should have testable APIs.
- Offline-first behavior for core phone functionality.

### 9) Observability and reliability

- Structured logging across boot, kernel, and userspace.
- Crash reporting pipeline with symbolized stack traces.
- Built-in health checks for battery drain, wakeups, and service restarts.

## Development roadmap (draft)

1. **Architecture phase (current):** lock high-level decisions, risk register, and target device shortlist.
2. **Bring-up phase:** boot to framebuffer/Wayland on reference device, basic input, and power controls.
3. **Core services phase:** telephony, networking, audio, storage encryption, settings service.
4. **Phone UX phase:** shell, lock screen, notifications, dialer/messaging MVP.
5. **Security hardening phase:** verified updates, MAC policy, permission model enforcement.
6. **Developer platform phase:** SDK/tooling, packaging, documentation, and reproducible builds.
7. **Stabilization phase:** battery life tuning, crash reduction, OTA resilience, and release criteria.

## Immediate next steps

- Choose 1-2 candidate reference devices and document tradeoffs.
- Decide kernel baseline (exact LTS version and update cadence).
- Select initial init/service manager and root filesystem strategy.
- Draft a thin system architecture diagram (boot, services, UI, update pipeline).
- Define success criteria for the first hardware milestone.
