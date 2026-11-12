# VMware / ESXi to Proxmox Migration Notes

**Language versions:** [Česky](README_CZ.md) | [Deutsch](README_DE.md)

**Scope:** migration workflow for Linux (Ubuntu Server 24.04) and Windows Server 2022 virtual machines from VMware to Proxmox KVM with Ceph storage. Tested on lab VMs.
**Author:** Radim Kulaviak (isol.cz)

## 1. Disk controller (critical for Windows)
- Windows boots only with a driver for the disk controller it sees. A mismatch gives `BSOD 0x7B (INACCESSIBLE_BOOT_DEVICE)`.
- Either keep the same controller type as on VMware (SATA/SCSI) for the first boot, or install the VirtIO drivers **before** the migration and then switch to VirtIO.

## 2. Importing the disk
- Convert and import a VMDK with `qm disk import` (alias `qm importdisk`). It converts to the target storage format.
- Proxmox VE 8.2 and later also has a built-in import wizard for ESXi.

```bash
# example, adjust VM ID, path and storage name
qm create 120 --name migrated-vm --memory 4096 --cores 2 \
  --net0 virtio,bridge=vmbr0 --scsihw virtio-scsi-single
qm disk import 120 /var/lib/vz/import/vm.vmdk ceph-pool
qm set 120 --scsi0 ceph-pool:vm-120-disk-0 --boot order=scsi0
```

## 3. Verification checklist
- Record hostname, IP configuration and installed services before the migration.
- Create SHA-256 checksums of test files before and after: `sha256sum /root/testdata/*`.
- Check that web services answer with the same HTTP status and content (`curl -I localhost`).
- Add the VM to the HA manager if it should restart automatically.

*Scope: lab-tested notes, not a description of client projects.*
