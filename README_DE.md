# Migration von VMware / ESXi zu Proxmox: Notizen

**Sprachversionen:** [English](README.md) | [Česky](README_CZ.md)

**Umfang:** Migrationsablauf für virtuelle Maschinen mit Linux (Ubuntu Server 24.04) und Windows Server 2022 von VMware zu Proxmox KVM mit Ceph-Speicher. Getestet auf Labor-VMs (simulierte VMware-Quelle, siehe unten).
**Autor:** Radim Kulaviak (isol.cz)

## Wie der Test durchgeführt wurde
- **Datum:** 24.-26. August 2026, Lab mit 3 Proxmox-VE-9.2-/Ceph-Knoten (alte Hardware Xeon E5410 / E5645).
- **Es wurde kein echtes ESXi verwendet.** ESXi 8.0U3e und 6.7U3 starteten auf dem E5410 nicht (kein AVX, der Kernel hielt auch mit `allowLegacyCPU=true` an). Stattdessen erstellte ich die „Quell"-VMs in Proxmox mit **VMDK**-Platten (dem Format, das ESXi exportiert), installierte Betriebssystem und Dienst und importierte sie wie bei einer echten Migration.
- Das übt Formatkonvertierung, Import, Start und Datenprüfung. Es deckt **nicht** das Entfernen der VMware Tools oder etwas auf der ESXi-Seite ab.

| Test | Quelle | Ziel | Ergebnis |
|---|---|---|---|
| Ubuntu Server 24.04 (nginx) | VM 8000, `local` (VMDK) | VM 8001 auf Ceph `vm-pool` | funktioniert: nginx, Prüfsumme der Testdatei und Hostname unverändert |
| Windows Server 2022 Evaluation (IIS) | VM 8202, `local` (VMDK, SATA) | VM 8302, `dir`-Speicher (RAW, SATA) | funktioniert: IIS, Prüfsumme der Testdatei und Hostname unverändert |

Windows startete ohne Treiberinjektion, da auf Quelle und Ziel derselbe Controller-Typ (SATA) verwendet wurde.

## 1. Festplatten-Controller (kritisch bei Windows)
- Windows startet nur mit einem Treiber für den Controller, den es sieht. Bei Abweichung folgt `BSOD 0x7B (INACCESSIBLE_BOOT_DEVICE)`.
- Entweder für den ersten Start denselben Controller-Typ wie unter VMware beibehalten (SATA/SCSI) oder die VirtIO-Treiber **vor** der Migration installieren und dann auf VirtIO umstellen.

## 2. Import der Festplatte
- Eine VMDK wird mit `qm disk import` (Alias `qm importdisk`) konvertiert und importiert. Sie wird in das Format des Zielspeichers umgewandelt.
- Proxmox VE 8.2 und neuer hat zusätzlich einen integrierten Import-Assistenten für ESXi.

```bash
# Beispiel, VM-ID, Pfad und Speichername anpassen
qm create 120 --name migrated-vm --memory 4096 --cores 2 \
  --net0 virtio,bridge=vmbr0 --scsihw virtio-scsi-single
qm disk import 120 /var/lib/vz/import/vm.vmdk ceph-pool
qm set 120 --scsi0 ceph-pool:vm-120-disk-0 --boot order=scsi0
```

## 3. Prüfliste
- Vor der Migration Hostname, IP-Konfiguration und installierte Dienste festhalten.
- Vor und nach der Migration MD5 oder SHA-256-Prüfsummen von Testdateien erstellen: `sha256sum /root/testdata/*`.
- Prüfen, dass Webdienste denselben HTTP-Status und Inhalt liefern (`curl -I localhost`).
- Soll die VM nach einem Ausfall automatisch neu starten, dem HA-Manager hinzufügen.

*Umfang: im Labor getestete Notizen, keine Beschreibung von Kundenprojekten.*
