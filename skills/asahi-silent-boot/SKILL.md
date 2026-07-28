---
name: asahi-silent-boot
description: Comprehensive guide for achieving a 100% text-free, silent boot and shutdown experience on Asahi Linux / Arch Linux ARM (Apple Silicon Macs).
---

# Asahi Linux Silent Boot & Shutdown Guide

This skill provides the exact source code modifications, kernel parameters, service masks, and hook configurations required to achieve a completely silent boot and shutdown on Apple Silicon Macs running Asahi Linux / Arch Linux ARM.

## Boot Text Message Source & Solution Map

| Message / Artifact | Origin / Cause | Exact Solution |
|---|---|---|
| `Model: Apple MacBook...` | Hardcoded `printf("Model: %s\n", model)` in U-Boot `common/board_info.c` | Comment out `printf` lines in U-Boot C source and recompile `u-boot-nodtb.bin`. |
| `starting USB...` | `cmd/usb.c:634` in U-Boot source | Comment out `printf("starting USB...\n")` in C source and recompile. |
| `No working controllers found` | `drivers/usb/host/usb-uclass.c:392` in U-Boot source | Comment out `printf("No working controllers found\n")` in C source and recompile. |
| `Hit any key to stop autoboot` | `common/autoboot.c:375` in U-Boot source | Comment out `printf` in C source and recompile. |
| `** Booting bootflow '<NULL>'` | `boot/bootflow.c:524` in U-Boot source | Comment out `printf` in C source and recompile. |
| `Booting: nvme 0` | `lib/efi_loader/efi_bootmgr.c:690` in U-Boot source | Comment out `log_info("Booting: %ls\n", lo.label)` in C source and recompile. |
| **`63538 blocks`** | `/usr/lib/initcpio/hooks/asahi` — `cpio -i` prints block counts to stderr on vendorfw unpack | Add `>/dev/null 2>&1` to `( cd /; cpio -i < "$VENDORFW/firmware.cpio" )`. Create pacman hook to re-apply after updates. |
| `[ OK ] Mounted System ESP` | `asahi-scripts/functions.sh` `info()` function | Patched `info()` in `functions.sh` to check `/proc/cmdline` for `quiet`. |
| `watchdog: watchdog0: did not stop` | Kernel `apple_wdt` module on poweroff | Add `nowatchdog` & `modprobe.blacklist=apple_wdt,softdog,watchdog` to kernel cmdline + blacklist in `/etc/modprobe.d/nowatchdog.conf`. |
| `broadcast message from... reboot` | `systemd-wall` broadcast on shutdown | Create `/etc/systemd/system.conf.d/no-wall.conf` with `[Manager]\nWallMessage=no`. |
| `[ OK ] Mounted /boot/efi` / journal size | `systemd-journald` output on start | Create `/etc/systemd/journald.conf.d/99-no-console.conf` with `[Journal]\nForwardToConsole=no`. |

---

## Core Configuration Reference

### 1. `/etc/default/grub`
```ini
GRUB_TIMEOUT=0
GRUB_TIMEOUT_STYLE=hidden
GRUB_CMDLINE_LINUX_DEFAULT="nowatchdog modprobe.blacklist=apple_wdt,softdog quiet splash loglevel=0 systemd.show_status=0 rd.udev.log_level=0 vt.global_cursor_default=0 fsck.mode=skip appledrm.show_notch=1"
```

### 2. `/etc/mkinitcpio.conf`
> **CRITICAL**: `kms` MUST precede `plymouth` in `HOOKS`.
```sh
HOOKS=(base asahi udev autodetect microcode modconf kms plymouth keyboard keymap consolefont block filesystems)
```

### 3. `/etc/systemd/system.conf`
```ini
ShowStatus=no
ShutdownWatchdogSec=0
RebootWatchdogSec=0
KExecWatchdogSec=0
```

### 4. `/etc/fstab`
Ensure `/boot/efi` pass number is set to `0 0` (disables auto fsck report on boot):
```text
UUID=5ABE-18A7 /boot/efi vfat rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro 0 0
```

### 5. Masked Services
```bash
sudo systemctl mask systemd-fsck-root.service
sudo systemctl mask systemd-fsck@.service
sudo systemctl mask getty@tty1.service
```

### 6. Pacman Hook for Asahi Firmware Unpack (`/etc/pacman.d/hooks/asahi-cpio-quiet.hook`)
```ini
[Trigger]
Operation = Install
Operation = Upgrade
Type = Package
Target = asahi-scripts

[Action]
Description = Silence cpio block count in asahi initramfs hook
When = PostTransaction
Exec = /bin/sed -i 's|( cd /; cpio -i < "$VENDORFW/firmware.cpio" )$|( cd /; cpio -i < "$VENDORFW/firmware.cpio" ) >/dev/null 2>\&1|' /usr/lib/initcpio/hooks/asahi
NeedsTargets
```

---

## Maintenance & Updating m1n1 Payload
After building custom silent U-Boot binary (`u-boot-nodtb.bin`):
```bash
sudo cp /path/to/u-boot-silent/u-boot-nodtb.bin /usr/lib/asahi-boot/u-boot-nodtb.bin
sudo update-m1n1
sudo mkinitcpio -P
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
