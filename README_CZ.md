# Migrační playbook: VMware / ESXi → Proxmox VE

**Jazykové verze:** [English](README.md) | [Deutsch](README_DE.md)

**Rozsah:** Postup migrace VM s Linuxem (Ubuntu Server 24.04) a Windows Server 2022 z VMware na Proxmox VE (úložiště Ceph). Ověřeno v mém laboratorním prostředí.
**Autor:** Radim Kulaviak ([isol.cz](https://www.isol.cz))

## 1. Volba způsobu importu
- **Import wizard pro ESXi v Proxmoxu** (Proxmox VE 8.2+): ESXi host se přidá jako storage a VM se importují přímo. Nejrychlejší pro malý počet VM.
- **Ručně:** export OVF/VMDK, pak import disku `qm disk import <vmid> <soubor.vmdk> <storage>` (starší název: `qm importdisk`). Proxmox převede disk do formátu cílového úložiště.

## 2. Diskový řadič a ovladače (kritické pro Windows)
- Windows potřebuje ovladač pro řadič bootovacího disku. Přepnutí Windows VM rovnou na VirtIO SCSI bez ovladače končí chybou `0x7B INACCESSIBLE_BOOT_DEVICE`.
- Bezpečný postup: nejdřív start na **SATA** (nebo řadiči, pro který hostovi ovladač existuje) → instalace ovladačů VirtIO a QEMU guest agenta → vypnutí → přepnutí disku na VirtIO SCSI → opětovný start.
- Po přesunu odinstalovat VMware Tools, nainstalovat `qemu-guest-agent`.
- Linuxoví hosté obvykle nabootují s VirtIO bez zásahu; ověřte, že initramfs obsahuje moduly virtio, a zkontrolujte názvy síťových rozhraní (často se mění).

## 3. Kontrolní seznam ověření
Před migrací i po ní:
- Zaznamenat hostname, IP konfiguraci, služby, naslouchající porty (`ss -tulpn`).
- Kontrolní součty testovacích dat: `sha256sum /root/testdata/* > before.sha256` a po migraci `sha256sum -c before.sha256`.
- Kontrola služeb (`curl -I`, přihlášení do aplikace, připojení k DB).
- Start, restart, hlášení guest agenta, běh zálohovací úlohy.

## 4. Poznámky k plánování
Měřit výpadek pro každou VM, zdrojovou VM ponechat vypnutou (ne smazanou) do akceptace a dohodnout s vlastníkem okno pro rollback.
