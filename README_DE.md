# VMware / ESXi auf Proxmox VE Migrations-Playbook

**Sprachversionen:** [English](README.md) | [Cesky](README_CZ.md)

**Umfang:** Validierter Migrations-Workflow fuer Linux (Ubuntu Server 24.04) und Windows Server 2022 virtuelle Maschinen vom VMware-Format auf Proxmox KVM / Ceph-Speicher.
**Autor:** Radim Kulaviak (isol.cz)

## Wichtige technische Schritte und Anforderungen

### 1. Festplatten-Controller-Mapping (Kritisch fuer Windows)
- Die Ziel-VM in Proxmox **muss exakt denselben Controller-Typ** (SATA, SCSI/LSI Logic oder VirtIO) verwenden wie die Quell-VM auf VMware.
- Nicht uebereinstimmende Controller fuehren beim Windows-Start sofort zu einem `BSOD 0x7B (INACCESSIBLE_BOOT_DEVICE)`.

### 2. Handhabung von Speicherformaten
- Proxmox `LVM-thin` und `Ceph RBD` akzeptieren keine direkten `.vmdk`-Dateien.
- **Workflow:** VMDK in einen temporaeren Verzeichnisspeicher (`local` / dir) importieren, wo Proxmox es in `raw` oder `qcow2` umwandelt, oder direkten Festplattenimport mit `qm importdisk` durchfuehren.

### 3. Datenintegritaet und Pruefliste
Vor und nach der Migration die Umgebung mit kryptografischen Hashes ueberpruefen:
- Urspruenglichen Hostnamen und Netzwerkkonfiguration aufzeichnen.
- MD5/SHA256-Pruefsummen von Testdateien generieren (`md5sum /root/testdata/*.txt`).
- Bestaetigen, dass Webdienste mit identischem HTTP-Status und Inhalt antworten (`curl localhost`, Apache-Pruefung).
