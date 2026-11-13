# Migrace z VMware / ESXi na Proxmox: poznámky

**Jazykové verze:** [English](README.md) | [Deutsch](README_DE.md)

**Rozsah:** postup migrace virtuálních strojů Linux (Ubuntu Server 24.04) a Windows Server 2022 z VMware na Proxmox KVM s úložištěm Ceph. Otestováno na laboratorních VM (simulovaný zdroj VMware, viz níže).
**Autor:** Radim Kulaviak (isol.cz)

## Jak test probíhal
- **Datum:** 24.-26. srpna 2026, laboratoř se 3 uzly Proxmox VE 9.2 / Ceph (starý hardware Xeon E5410 / E5645).
- **Skutečné ESXi jsem nepoužil.** ESXi 8.0U3e ani 6.7U3 na E5410 nenastartovalo (chybí AVX, jádro se zastavilo i s `allowLegacyCPU=true`). Místo toho jsem „zdrojové" VM vytvořil v Proxmoxu s disky **VMDK** (formát, který ESXi exportuje), nainstaloval OS a službu a naimportoval je stejným postupem jako při skutečné migraci.
- Takto se nacvičí převod formátu disku, import, start a ověření dat. **Nepokrývá** odstranění VMware Tools ani nic na straně ESXi.

| Test | Zdroj | Cíl | Výsledek |
|---|---|---|---|
| Ubuntu Server 24.04 (nginx) | VM 8000, `local` (VMDK) | VM 8001 na Ceph `vm-pool` | fungovalo: nginx, kontrolní součet souboru a hostname beze změny |
| Windows Server 2022 Evaluation (IIS) | VM 8202, `local` (VMDK, SATA) | VM 8302, `dir` úložiště (RAW, SATA) | fungovalo: IIS, kontrolní součet souboru a hostname beze změny |

Windows nastartoval bez vkládání ovladačů, protože na zdroji i cíli byl stejný typ řadiče (SATA).

## 1. Řadič disku (kritické u Windows)
- Windows nastartuje jen s ovladačem pro řadič disku, který vidí. Nesoulad vyvolá `BSOD 0x7B (INACCESSIBLE_BOOT_DEVICE)`.
- Buď ponech pro první start stejný typ řadiče jako ve VMware (SATA/SCSI), nebo nainstaluj ovladače VirtIO **před** migrací a poté přepni na VirtIO.

## 2. Import disku
- VMDK převedeš a naimportuješ příkazem `qm disk import` (alias `qm importdisk`). Převede se do formátu cílového úložiště.
- Proxmox VE 8.2 a novější má navíc vestavěný importní průvodce pro ESXi.

```bash
# příklad, uprav ID VM, cestu a název úložiště
qm create 120 --name migrated-vm --memory 4096 --cores 2 \
  --net0 virtio,bridge=vmbr0 --scsihw virtio-scsi-single
qm disk import 120 /var/lib/vz/import/vm.vmdk ceph-pool
qm set 120 --scsi0 ceph-pool:vm-120-disk-0 --boot order=scsi0
```

## 3. Kontrolní seznam
- Před migrací si poznamenej hostname, nastavení IP a nainstalované služby.
- Před a po migraci vytvoř MD5 nebo SHA-256 kontrolní součty testovacích souborů: `sha256sum /root/testdata/*`.
- Ověř, že webové služby vrací stejný HTTP stav a obsah (`curl -I localhost`).
- Pokud se má VM sama restartovat po výpadku, přidej ji do HA manageru.

*Rozsah: poznámky otestované v laboratoři, nejde o popis klientských projektů.*
