# Aegis OTA

Aegis OTA is a fault-tolerant Over-The-Air update system designed for edge devices and physical fleet hardware running custom storage software in volatile environments.

The project focuses on safe deployment for embedded systems where a failed update can brick a node permanently. Instead of patching a live filesystem, the platform uses immutable OS images, A/B partition switching, cryptographic verification, and bootloader-driven rollback to keep devices operational even when power loss, network instability, or kernel failures occur.

## Problem it solves

Pushing updates to cloud infrastructure is straightforward; pushing updates to deployed hardware in the field is significantly more dangerous. When edge devices carry custom storage software and rely on persistent local state, even a minor update bug can render the node unreachable or permanently unusable.

Aegis OTA is built around the assumption that the environment can fail at any time. The design prioritizes recovery, verification, and self-healing behavior before convenience.

## Core architecture

### 1. Immutable image pipeline
- Build read-only OS images from Yocto or Buildroot
- Package the Go-based OTA agent together with the system software
- Generate a cryptographic signature for each image artifact
- Treat payload integrity as independent from TLS

### 2. Secure and memory-efficient delivery
- The OTA agent downloads the update directly to the inactive partition
- Stream the image to a raw block device instead of buffering it in RAM
- Validate the payload using the signed manifest before activating it

### 3. A/B partition activation
- Keep three critical partitions: boot, active root, and passive root
- Use a U-Boot-style bootloader to switch between partitions safely
- Set upgrade state flags before rebooting to prepare the rollback path

### 4. Post-boot health verification
- Reboot into the new slot
- Run a strict health check from a systemd service or init step
- Validate required mounts, network reachability, and service health
- Only mark the update as successful after the platform is confirmed healthy

### 5. Automatic rollback
- Do not rely on application-layer rollback alone
- If the kernel panics or the system fails during boot, the bootloader must restore the previous partition automatically
- The rollback path is triggered at the hardware and bootloader level without manual intervention

## System goals

- Prevent permanent bricking of field nodes
- Ensure every deployed image is cryptographically verified
- Minimize RAM usage during image transfer
- Support robust boot switching with minimal operator intervention
- Recover automatically from failed upgrades or bad kernel state

## Recommended project names

If you want a stronger brand identity, a few good options are:

- Aegis OTA
- Aegis Fleet
- Ashvatha Edge Guard
- NodeShield OTA
- BootLock Fleet
- SafeSwap Edge

## Typical flow

1. CI builds an immutable root filesystem and signs the artifact
2. The edge node downloads and stream-writes the update to the inactive partition
3. The agent validates the image against the signed manifest
4. U-Boot is updated to boot the new partition on the next reboot
5. The device reboots into the updated slot
6. The health-check service verifies the environment
7. On success, the update is committed
8. On failure, the bootloader rolls back to the previous slot automatically

## Tech stack

- Go
- Linux embedded systems
- Yocto or Buildroot
- U-Boot
- systemd
- Signed artifact delivery
- Raw block-device flashing

## Repository structure

This repository is intended to evolve into a complete OTA platform with:

- agent/ for the Go-based edge updater
- ci/ for signing and artifact generation
- boot/ for bootloader configuration and rollback logic
- scripts/ for deployment and validation tools
- docs/ for architecture and operational runbooks

## Status

This project is currently in the design and foundation stage, with the focus on creating a resilient, self-healing OTA deployment model for embedded fleet devices.

---

Aegis OTA is designed for the harsh reality of field hardware: when the system fails, the device must recover without human intervention.
