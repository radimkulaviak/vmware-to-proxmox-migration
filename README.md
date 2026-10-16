# VMware / ESXi to Proxmox Migration Playbook

**Language versions:** [Cesky](README_CZ.md) | [Deutsch](README_DE.md)

**Scope:** Validated migration workflow for Linux (Ubuntu Server 24.04) and Windows Server 2022 virtual machines from VMware format to Proxmox KVM/Ceph storage.
**Author:** Radim Kulaviak (isol.cz)

## Key Technical Steps & Requirements

### 1. Disk Controller Mapping (Critical for Windows)
- The target VM in Proxmox **must use the exact same disk controller type** (SATA, SCSI/LSI Logic, or VirtIO) as the source VM on VMware.
- Mismatching controllers immediately trigger a `BSOD 0x7B (INACCESSIBLE_BOOT_DEVICE)` on Windows boot.

### 2. Storage Format Handling
- Proxmox `LVM-thin` and `Ceph RBD` do not accept raw `.vmdk` files directly.
- **Workflow:** Import VMDK into a temporary directory storage (`local` / dir) where Proxmox converts it to `raw` or `qcow2`, or perform direct disk import using `qm importdisk`.

### 3. Data Integrity & Verification Checklist
Before and after migration, verify the environment using strict cryptographic hashes:
- Record original hostname and network configuration.
- Generate MD5/SHA256 checksums of test files (`md5sum /root/testdata/*.txt`).
- Confirm web services respond with identical HTTP status and content (`curl localhost`, Apache check).
