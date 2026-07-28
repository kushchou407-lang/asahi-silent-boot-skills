# Asahi Linux Silent Boot & Custom Plymouth Skills

This repository contains comprehensive skills and reference guides for achieving a 100% silent boot/shutdown on Apple Silicon Macs running Asahi Linux / Arch Linux ARM, as well as authoring custom Plymouth script splash themes.

## Included Skills

### 1. [`asahi-silent-boot`](skills/asahi-silent-boot/SKILL.md)
A complete source-level guide mapping every single boot/shutdown log line to its exact cause and solution on Apple Silicon:
- Commenting out U-Boot `printf` messages (machine model, USB initialization, autoboot, bootflow, EFI loader).
- Silencing initramfs `asahi-scripts` firmware extraction (`cpio -i`) with automated pacman hooks.
- Configuring kernel parameters, systemd wall notifications, watchdog blacklist, and GRUB.

### 2. [`plymouth-script-theme`](skills/plymouth-script-theme/SKILL.md)
An authoritative API reference for writing smooth Plymouth boot themes using Plymouth's `script` module:
- Correct API methods (`Image`, `Sprite`, `Scale`, `Plymouth.*`, `Math.*`).
- Critical rules to prevent Plymouth crashes and black screen fallbacks.
- Correct initramfs hook ordering (`kms` before `plymouth`).

---

## Installation
To install these skills into your Antigravity agent configuration:

```bash
mkdir -p ~/.gemini/config/skills/
cp -r skills/* ~/.gemini/config/skills/
```
