# Migrations-Playbook: VMware / ESXi → Proxmox VE

**Sprachversionen:** [English](README.md) | [Česky](README_CZ.md)

**Umfang:** Migrations-Workflow für VMs mit Linux (Ubuntu Server 24.04) und Windows Server 2022 von VMware auf Proxmox VE (Ceph-Speicher). In meiner Laborumgebung getestet.
**Autor:** Radim Kulaviak ([isol.cz](https://www.isol.cz))

## 1. Importweg wählen
- **ESXi-Import-Assistent von Proxmox** (Proxmox VE 8.2+): ESXi-Host als Storage hinzufügen und VMs direkt importieren. Am schnellsten bei wenigen VMs.
- **Manuell:** OVF/VMDK exportieren, dann Disk-Import mit `qm disk import <vmid> <datei.vmdk> <storage>` (älterer Name: `qm importdisk`). Proxmox konvertiert in das Format des Zielspeichers.

## 2. Disk-Controller und Treiber (kritisch bei Windows)
- Windows benötigt einen Treiber für den Controller der Bootplatte. Ein direkter Wechsel auf VirtIO SCSI ohne Treiber führt zu `0x7B INACCESSIBLE_BOOT_DEVICE`.
- Sichere Reihenfolge: zuerst mit **SATA** booten (oder einem Controller, für den der Gast einen Treiber hat) → VirtIO-Treiber und QEMU-Guest-Agent installieren → herunterfahren → Disk auf VirtIO SCSI umstellen → erneut booten.
- Nach dem Umzug VMware Tools deinstallieren und `qemu-guest-agent` installieren.
- Linux-Gäste booten meist ohne Anpassung mit VirtIO; initramfs auf virtio-Module prüfen und Netzwerkschnittstellennamen kontrollieren (ändern sich oft).

## 3. Prüfliste
Vor und nach der Migration:
- Hostname, IP-Konfiguration, Dienste, lauschende Ports festhalten (`ss -tulpn`).
- Prüfsummen von Testdaten: `sha256sum /root/testdata/* > before.sha256`, danach `sha256sum -c before.sha256`.
- Dienstprüfungen (`curl -I`, Anwendungs-Login, DB-Verbindung).
- Boot, Neustart, Guest-Agent-Meldung, Lauf des Backup-Jobs.

## 4. Planungshinweise
Ausfallzeit pro VM messen, Quell-VM bis zur Abnahme ausgeschaltet (nicht gelöscht) lassen und mit dem Eigentümer ein Rollback-Fenster vereinbaren.
