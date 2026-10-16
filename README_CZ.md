# Migracni kucharce: VMware / ESXi na Proxmox VE

**Jazykove verze:** [English](README.md) | [Deutsch](README_DE.md)

**Rozsah:** Overeny migracni postup pro Linux (Ubuntu Server 24.04) a Windows Server 2022 virtualni stroje z formatu VMware na Proxmox KVM / Ceph uloziste.
**Autor:** Radim Kulaviak (isol.cz)

## Klicove technicke kroky a pozadavky

### 1. Mapovani diskovych radicu (Kriticke pro Windows)
- Cilova VM v Proxmoxu **musi pouzivat presne stejny typ diskoveho radice** (SATA, SCSI/LSI Logic nebo VirtIO) jako zdrojova VM na VMware.
- Nesoulad radicu okamzite muze zpusobit `BSOD 0x7B (INACCESSIBLE_BOOT_DEVICE)` pri startu Windows.

### 2. Prace s formaty uloziste
- Proxmox `LVM-thin` a `Ceph RBD` neakceptuji prirozeny `.vmdk` soubor primo.
- **Postup:** Importujte VMDK do docasneho adresaroveho uloziste (`local` / dir), kde ho Proxmox konvertuje na `raw` nebo `qcow2`, nebo pouzijte priamy import disku pomoci `qm importdisk`.

### 3. Kontrolni seznam datove integrity a overeni
Pred migraci a po ni overte prostredi pomoci kryptografickych hashu:
- Zaznamenejte původni hostname a sitovou konfiguraci.
- Vygenerujte MD5/SHA256 kontrolni soucty testovacich souboru (`md5sum /root/testdata/*.txt`).
- Overte, ze webove sluzby odpovidaji se stejnym HTTP stavem a obsahem (`curl localhost`, kontrola Apache).
