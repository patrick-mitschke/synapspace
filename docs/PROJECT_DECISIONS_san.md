# PROJECT_DECISIONS_san.md
_Gelegentlich aktualisiert – neue Entscheidungen, Konfigurationen, Lessons Learned._

---

## Technische Entscheidungen

| Thema | Wahl | Verworfen | Grund |
|-------|------|-----------|-------|
| OS | Ubuntu Server 24.04 LTS (Bare-Metal) | Proxmox, Ubuntu 26.04 | Battle-tested, NVIDIA-Support, 5 Jahre Updates |
| Storage | LVM (40GB root, 404GB docker) | Klassische Partitionierung | Flexible Größenanpassung, Snapshots |
| Orchestrierung | Docker Compose | Kubernetes | Single-Node, MVP-Komplexität |
| LLM lokal | Ollama (GGUF, quantisiert) + Qdrant RAG | – | Privacy, urheberrechtlich geschützte Daten |
| LLM cloud | Google AI Pro + Perplexity Education Pro | API-Calls | Kostenoptimierung (Pro-Abos statt API) |
| Backup | rsync + Acer TravelMate 5735 + 500GB USB-HDD | Synology NAS DS216+II (150€) | 0€ (Hardware vorhanden), MVP-Fokus |
| LUKS | NEIN | – | Headless-Server: Passphrase bei Reboot nicht möglich |
| OpenClaw | Installiert im MVP (keine Custom Skills) | – | Strategisch relevante Technologie; Erfahrungsaufbau |
| GVS-Rhythmus | Zähler-basiert (alle 7 = Weekly, alle 28 = Monthly) | Datum-basiert (Wochentag/1. des Monats) | Kalenderunabhängig – verpasste Nächte zerstören nicht den GVS-Rhythmus |
| Backup-Trigger | systemd Timer (`Persistent=true`) auf Backup-Node | Cron auf Hauptserver | Catch-up beim nächsten Boot; Cron hätte verpasste Jobs stillschweigend ignoriert |

---

## Konfigurationen

### LVM
- **VG:** vg-synapspace (444.078G, PV = Partition 3 der Intel SSD)
- **lv-root:** 40G, ext4, /
- **lv-docker:** 404G, ext4, /var/lib/docker

### Partitionen (Intel SSD)
- P1: 1.049G fat32 → /boot/efi (ESP)
- P2: 2G ext4 → /boot
- P3: 444.079G unformatiert → PV vg-synapspace

### Netzwerk
- **NIC:** enp0s25 (Intel I217-LM, PCI-Bus 0, Slot 25)
- **Hostname:** synapspace
- **IP (statisch):** `<SERVER_IP>`/24 (via Netplan 50-cloud-init.yaml)
- **Gateway/DNS:** `<GATEWAY_IP>` (FritzBox 7530)

### Backup-Node (synapspace-backup)
- **IP (statisch):** `<BACKUP_IP>`/24 (via Netplan 00-installer-config.yaml)
- **SSH-Keys authorized:** synapspace-admin + backup-trigger (Ed25519)
- **Sudo-Regel:** `/etc/sudoers.d/backup-trigger` → NOPASSWD nur für `backup-job1.sh`

### GVS-Backup (final)
- **Trigger:** systemd Timer auf Backup-Node (`/etc/systemd/system/backup-trigger.timer`)
- **Skript Backup-Node:** `/usr/local/bin/trigger-backup.sh` → SSH → Hauptserver
- **Skript Hauptserver:** `/usr/local/bin/backup-job1.sh` (GVS-Zähler-Logik)
- **Counter:** `/var/lib/synapspace/backup-counter`
- **Log Backup-Node:** `/var/log/backup-trigger.log`
- **Log Hauptserver:** `/var/log/backup-job1.log`

### Cronjobs Hauptserver (root-crontab, final)
```
10 22 * * 7   /usr/local/bin/sysconfig-export.sh   # Sonntags 22:10
30 23 * * *   /sbin/shutdown -h now                 # täglich 23:30
```

### NVIDIA
- **Driver:** 535.288.01 (via ubuntu-drivers install)
- **CUDA:** 12.2
- **Container Toolkit:** nvidia-container-toolkit (nvidia.github.io Repository)
- **Docker Runtime Config:** /etc/docker/daemon.json (via nvidia-ctk runtime configure)

### Docker
- **Installation:** Docker CE aus offiziellem Docker-Repository (download.docker.com)
- **Pakete:** docker-ce, docker-ce-cli, containerd.io, docker-compose-plugin

### UFW
- **Status:** aktiv
- **Default:** deny incoming, allow outgoing
- **Port 22/tcp:** ALLOW IN Anywhere
- **Ports 5678, 3000, 9000, 11434, 6333, 8080:** ALLOW IN `<HOME_SUBNET>`

### SSH
- **Key-Typ:** Ed25519
- **Kommentar:** synapspace-admin
- **Public Key Speicherort Server:** /home/sysadmin/.ssh/authorized_keys
- **PasswordAuthentication:** no (/etc/ssh/sshd_config)

### Docker Compose (geplant)
- **Services:** n8n, OpenClaw, Ollama, Qdrant, Open WebUI, Portainer
- **Netzwerk:** internal-ai (Bridge)
- **Volumes:** Persistent für alle Services

### Service-Ports (geplant)
- n8n: 5678 | Open WebUI: 3000 | Portainer: 9000 | Ollama: 11434 | Qdrant: 6333 | OpenClaw: 8080

### Modelle (~100GB geplant)
- Llama-3.2-3B (2GB), Hermes-3-8B (5GB), Qwen2.5-7B (4GB)
- Mistral-Nemo-12B (7GB), Command-R-35B (20GB), Qwen2.5-32B (18GB)
- Llama-3.1-70B (40GB)

---

## Lessons Learned

### Hardware & Vorbereitung
- ✅ Systematische Dokumentation (Screenshots, BIOS-Checks)
- ✅ Memtest validiert Stabilität trotz gemischter RAM-Revisionen
- ⚠️ eBay-Risiko: Immer Revisionen VOR Kauf prüfen
- ⚠️ ZIP-Archive IMMER entpacken vor Installer-Ausführung

### Installation
- ⚠️ Ethernet-Kabel VOR dem Boot einstecken – DHCP sonst nicht möglich
- ⚠️ "Create bond" nur bei mehreren NICs – bei Einzelkarte immer abbrechen
- ⚠️ Ubuntu 24.04: SSH trotz Installer-Auswahl nicht automatisch aktiv → `sudo systemctl enable ssh --now`
- ⚠️ LUKS auf Headless-Servern vermeiden – Passphrase bei Reboot nicht möglich
- ℹ️ Predictable Network Naming: enp0s25 = Ethernet, PCI-Bus 0, Slot 25
- ℹ️ EFI-Partition im Subiquity Custom Layout: Disk → "Use as Boot Device" → ESP automatisch korrekt (fat32)
- ℹ️ vfat nicht im Format-Dropdown wählbar – wird automatisch gesetzt bei Mount /boot/efi
- ℹ️ Installer-Update überspringen ("Continue without updating") – kein MVP-Mehrwert
- ℹ️ sudo-Passwort-Prompt kann mitten in verketteten Befehlen erscheinen → Ausgabe erst vollständig nach Eingabe
- ℹ️ Headless-Betrieb ab Phase 1 vollständig – Monitor/Tastatur am Server nicht mehr erforderlich

### Docker & NVIDIA
- ⚠️ NVIDIA Container Toolkit kann vor Docker installiert werden – nvidia-ctk schreibt daemon.json erfolgreich, Docker-Dienst aber erst nach Docker-Installation startbar
- ℹ️ Docker immer aus offiziellem Docker-Repository installieren (nicht Ubuntu-Paketquellen) → aktuellere Version + docker-compose-plugin enthalten

### SSH-Hardening
- ℹ️ Reihenfolge: Key-Login verifizieren → erst dann PasswordAuthentication deaktivieren – verhindert Aussperren
- ℹ️ Ed25519 bevorzugen gegenüber RSA – kompakter, moderner, sicherer
