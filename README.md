# VMware / ESXi to Proxmox Migration Playbook

**Language versions:** [Česky](README_CZ.md) | [Deutsch](README_DE.md)

**Scope:** Migration workflow for Linux (Ubuntu Server 24.04) and Windows Server 2022 VMs from VMware to Proxmox VE (Ceph storage). Tested in my lab environment.
**Author:** Radim Kulaviak ([isol.cz](https://www.isol.cz))

## 1. Choose the import path
- **Proxmox ESXi import wizard** (Proxmox VE 8.2+): add the ESXi host as storage and import VMs directly. Fastest for a small number of VMs.
- **Manual:** export OVF/VMDK, then import the disk with `qm disk import <vmid> <file.vmdk> <storage>` (older name: `qm importdisk`). Proxmox converts to the target storage format.

## 2. Disk controller and drivers (critical for Windows)
- Windows needs a driver for the boot disk controller. Switching a Windows VM straight to VirtIO SCSI without the driver gives `0x7B INACCESSIBLE_BOOT_DEVICE`.
- Safe sequence: boot first on **SATA** (or a controller the guest already has a driver for) → install the VirtIO drivers and QEMU guest agent → shut down → switch the disk to VirtIO SCSI → boot again.
- Uninstall VMware Tools after the move; install `qemu-guest-agent`.
- Linux guests usually boot with VirtIO out of the box; verify that the initramfs contains the virtio modules and check network interface names (they often change).

## 3. Verification checklist
Before and after migration:
- Record hostname, IP configuration, services, listening ports (`ss -tulpn`).
- Checksums of test data: `sha256sum /root/testdata/* > before.sha256` and `sha256sum -c before.sha256` after.
- Service checks (`curl -I`, application login, DB connectivity).
- Boot, reboot, guest agent reporting, backup job runs.

## 4. Planning notes
Measure downtime per VM, keep the source VM powered off (not deleted) until acceptance, and agree a rollback window with the owner.
