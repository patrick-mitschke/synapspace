# PHASE_0 – Vorbereitung
**Status: ✅ COMPLETE**
**Zeitraum:** 2026-04 | **Aufwand:** ~6h (inkl. 1.5h Memtest)

---

## Schritte
- ✅ Projektdokumente erstellt (SSoT, Deployment-Reihenfolge, Anforderungen)
- ✅ Hardware-Inventur abgeschlossen
- ✅ KI-Toolchain eingerichtet (Claude Pro + Project, Custom Instructions)
- ✅ RAM-Module empfangen (4x16GB ECC DDR4, 157€)
- ✅ RAM installiert (Dual-Channel B+D)
- ✅ BIOS: 64GB erkannt, 2133MHz
- ✅ Ubuntu 24.04 LTS ISO heruntergeladen (~2GB)
- ✅ Rufus konfiguriert (GPT, UEFI, FAT32)
- ✅ Memtest86 USB erstellt (imageUSB.exe)
- ✅ Memtest86 Pass 0: NULL FEHLER (64GB stabil)

---

## Konfigurationen

### Hardware (Hauptserver)
- **Gerät:** Fujitsu Celsius M740b
- **CPU:** Xeon E5-2620 v4 (8C/16T)
- **RAM:** 64GB ECC DDR4 (4x16GB) – 2x SK Hynix RA0, 1x Crucial RB0, 1x Crucial RBB
- **RAM-Modus:** Dual-Channel B+D, 2133MHz
- **GPU:** Quadro K2200 (4GB VRAM)
- **Storage:** 480GB SSD (99% Health, CrystalDiskInfo)
- **Memtest86:** Pass 0 – NULL Fehler

---

## Dokumentation

### Server-Innenraum (Ausgangszustand)
![Server-Innenraum Gesamtansicht](../assets/phase-0/01_server_interior_complete_overview.jpg)

### Originaler RAM-Riegel (1x 8GB, ausgebaut)
![HP 8GB RAM-Modul Vorderseite](../assets/phase-0/02_ram_module_hp_8gb_front.jpg)

### SSD-Zustand vor Installation
![CrystalDiskInfo – 480GB SSD 99% Health](../assets/phase-0/03_crystaldiscinfo_ssd.jpg)

### Neue RAM-Riegel eingebaut
![RAM-Module eingebaut – rechte Slots](../assets/phase-0/04_ram_module_eingebaut_rechts.jpg)
![RAM-Module eingebaut – linke Slots](../assets/phase-0/05_ram_module_eingebaut_links.jpg)

### Memtest86 – Ergebnis
![Memtest86 abgeschlossen – 0 Fehler](../assets/phase-0/06_Memtest_Complete_0_Errors.jpg)

---

## Lessons Learned

### Hardware & Vorbereitung
- ✅ Systematische Dokumentation (Screenshots, BIOS-Checks) zahlt sich später aus
- ✅ Memtest validiert Stabilität trotz gemischter RAM-Revisionen
- ⚠️ eBay-Risiko: Immer RAM-Revisionen VOR dem Kauf prüfen
- ⚠️ ZIP-Archive IMMER entpacken vor Installer-Ausführung
