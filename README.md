# VMware / ESXi to Proxmox Migration Notes

**Language versions:** [Česky](README_CZ.md) | [Deutsch](README_DE.md)

**Scope:** migration workflow for Linux (Ubuntu Server 24.04) and Windows Server 2022 virtual machines from VMware to Proxmox KVM with Ceph storage. Tested on lab VMs (simulated VMware source, see below).
**Author:** Radim Kulaviak (isol.cz)

## How the test was run
- **Date:** 24-26 August 2026, 3-node Proxmox VE 9.2 / Ceph lab (old Xeon E5410 / E5645 hardware).
- **No real ESXi was used.** ESXi 8.0U3e and 6.7U3 would not boot on the E5410 (no AVX; the kernel halted even with `allowLegacyCPU=true`). Instead I created the "source" VMs inside Proxmox with **VMDK** disks (the format ESXi exports), installed the OS and a service, and then imported them the same way as a real migration.
- This rehearses the disk format conversion, import, boot and data verification. It does **not** cover removing VMware Tools or anything on the ESXi side.

| Test | Source | Target | Result |
|---|---|---|---|
| Ubuntu Server 24.04 (nginx) | VM 8000, `local` (VMDK) | VM 8001 on Ceph `vm-pool` | worked: nginx, test file checksum and hostname unchanged |
| Windows Server 2022 Evaluation (IIS) | VM 8202, `local` (VMDK, SATA) | VM 8302, `dir` storage (RAW, SATA) | worked: IIS, test file checksum and hostname unchanged |

Windows booted without driver injection because the same controller type (SATA) was used on source and target.

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
- Create MD5 or SHA-256 checksums of test files before and after: `sha256sum /root/testdata/*`.
- Check that web services answer with the same HTTP status and content (`curl -I localhost`).
- Add the VM to the HA manager if it should restart automatically.

*Scope: lab-tested notes, not a description of client projects.*
