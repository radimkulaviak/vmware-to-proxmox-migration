# Migration von VMware / ESXi zu Proxmox: Notizen

**Sprachversionen:** [English](README.md) | [Česky](README_CZ.md)

**Umfang:** Migrationsablauf für virtuelle Maschinen mit Linux (Ubuntu Server 24.04) und Windows Server 2022 von VMware zu Proxmox KVM mit Ceph-Speicher. Getestet auf Labor-VMs.
**Autor:** Radim Kulaviak (isol.cz)

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
- Vor und nach der Migration SHA-256-Prüfsummen von Testdateien erstellen: `sha256sum /root/testdata/*`.
- Prüfen, dass Webdienste denselben HTTP-Status und Inhalt liefern (`curl -I localhost`).
- Soll die VM nach einem Ausfall automatisch neu starten, dem HA-Manager hinzufügen.

*Umfang: im Labor getestete Notizen, keine Beschreibung von Kundenprojekten.*
