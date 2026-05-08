# MerlinOS

MerlinOS is a from-scratch mobile operating system initiative now targeting an **OpenBSD-based** platform (not Linux, Android, Ubuntu Touch, or postmarketOS). The immediate goal is to define a realistic architecture and staged execution plan before implementation.

## Vision and constraints

- **OpenBSD-first approach:** stay close to upstream OpenBSD and keep local divergences minimal.
- **No Android userspace dependency:** avoid Binder/HAL compatibility layers.
- **Phone-native UX:** prioritize touch interaction, suspend/resume reliability, telephony behavior, and battery life.
- **Security baseline:** design with least privilege, signed updates, and strict process isolation.
- **Incremental bring-up:** optimize for visible milestones on real hardware.

## What changes when moving from Linux to OpenBSD?

This is a major architectural pivot. The most important implications are:

1. **Driver and hardware support risk increases**
   - OpenBSD has excellent security posture, but mobile SoC/modem/GPU support is usually narrower than Linux mainline.
   - Reference device selection becomes the #1 gating factor.

2. **Userspace and service model changes**
   - Linux-specific assumptions (systemd, cgroups, namespaces, SELinux/AppArmor, etc.) no longer apply.
   - Process isolation strategy should center on OpenBSD-native mechanisms (`pledge`, `unveil`, privilege separation).

3. **Graphics stack strategy must be revalidated**
   - Wayland/compositor/toolkit choices must be filtered by OpenBSD compatibility and performance.

4. **App ecosystem strategy changes**
   - Flatpak/Flathub assumptions need validation because they are Linux-centric.
   - Third-party app model may need a custom packaging/runtime approach for OpenBSD.

5. **Virtualization/dev workflow changes**
   - For early bring-up, QEMU remains the preferred VM path; VirtualBox is less aligned for ARM/mobile workflows.

## High-level design decisions (OpenBSD track)

### 1) Hardware strategy: start narrow, then generalize

- Target one **well-supported reference device** first.
- Define a board support matrix early:
  - Boot chain status (unlockable bootloader, secure boot options)
  - OpenBSD kernel and driver viability
  - Modem integration path
  - GPU/display stack maturity
- Build abstraction layers only after one device is stable.

### 2) Kernel and low-level platform

- Track OpenBSD release cadence and `-current` risk policy.
- Maintain a minimal local patch set with upstream-first intent.
- Prioritize power management correctness early (suspend, wake behavior, thermals, charging).

### 3) Boot and system composition

- Define a hardware-specific boot chain (U-Boot/UEFI/bootloader path per target).
- Design for atomic updates and rollback safety.
- Early userspace verifies mounts, rollback state, and hardware sanity checks.

### 4) Userspace architecture

- Keep base userspace small and auditable.
- Service supervision and dependencies should be explicit and observable.
- Split platform services by trust domain:
  - Telephony/radio service
  - Connectivity service (Wi-Fi/Bluetooth)
  - Media/camera service
  - UI compositor and shell

### 5) Graphics and UI stack

- Validate a compositor strategy that is realistic on OpenBSD.
- Toolkit selection should prioritize touch ergonomics, memory efficiency, and GPU reliability.
- Define shell contract for notifications, quick settings, lock screen, multitasking, and power controls.

### 6) App model and developer platform

- Start with system apps only (dialer, messaging, settings, camera, browser).
- Add third-party app runtime only after security boundaries are proven.
- Packaging/distribution model should support signatures, permission metadata, and transactional lifecycle.

### 7) Security model

- Build around OpenBSD-native hardening patterns (privilege separation, `pledge`, `unveil`).
- Minimize privileges per service and per app.
- Require verified boot/update path and strong secrets handling.

### 8) Connectivity and telephony

- Modem control path must be isolated and restart-safe.
- SMS, voice, and data flows need testable APIs.
- Preserve offline-first behavior for core phone features.

### 9) Observability and reliability

- Structured logs across boot, kernel, and userspace.
- Crash diagnostics pipeline with symbolized traces where possible.
- Built-in health checks for battery drain, wakeups, and service restarts.

## Development roadmap (draft)

1. **Architecture phase (current):** lock OpenBSD-specific design decisions, risk register, and device shortlist.
2. **Bring-up phase:** boot to framebuffer/compositor on reference target, basic input, and power controls.
3. **Core services phase:** telephony, networking, audio, storage encryption, settings service.
4. **Phone UX phase:** shell, lock screen, notifications, dialer/messaging MVP.
5. **Security hardening phase:** strict service isolation, update integrity, permission model enforcement.
6. **Developer platform phase:** tooling, packaging/distribution docs, reproducible builds.
7. **Stabilization phase:** battery tuning, crash reduction, update resilience, release criteria.

## Immediate next steps

- Pick 1-2 OpenBSD-viable reference targets and document hard constraints.
- Decide OpenBSD release tracking policy (`stable` vs selective `-current` components).
- Validate compositor/toolkit feasibility on chosen hardware.
- Decide third-party app strategy (ports only vs custom runtime/packaging).
- Define success criteria for first hardware milestone.

## Stage 1 decision checklist (must-answer before implementation)

### Product direction and target segment

- Primary user segment: **privacy-conscious mainstream users** (not only technical users).
- Product values:
  - Highly lightweight
  - Privacy by default
  - Strong battery life
  - High-quality camera outcomes

### Open decisions to lock now

1. **Reference hardware path**
   - VM-first only, or immediate real-device bring-up?
   - If VM-first, confirm QEMU/virtio as primary target.

2. **Application distribution baseline**
   - Keep Linux-style Flatpak goals, or pivot to OpenBSD-native package/runtime strategy?

3. **Privacy posture (default UX policy)**
   - Defaults for telemetry, crash reporting, location, tracker blocking, and cloud sync.

4. **Security vs convenience tradeoffs**
   - Bootloader unlock policy for consumer builds.
   - App signing/sideloading/developer mode strictness.

5. **Camera strategy**
   - Primary quality definition: computational imaging, latency, low-light, or color accuracy.
   - Objective MVP metrics.

6. **Authenticity / anti-deepfake concept**
   - MVP or post-MVP?
   - Blockchain required, or signed provenance sufficient?

7. **Battery-life targets**
   - Explicit standby drain and screen-on targets.
   - Feature deferrals allowed to protect battery goals.

8. **Non-technical user experience requirements**
   - Setup flow simplicity requirements.
   - Which advanced controls are hidden behind developer mode.

### Stage 1 exit criteria

Stage 1 is complete when the project has:
- One chosen reference device and one fallback target.
- A written privacy-by-default policy for MVP.
- A locked app distribution decision.
- Measurable camera and battery goals.
- A decision on provenance/anti-deepfake scope (MVP vs post-MVP).
