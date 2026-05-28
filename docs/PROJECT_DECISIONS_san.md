# PROJECT_DECISIONS_san.md

_Laufend aktualisiert – neue Entscheidungen, Konfigurationen, Lessons Learned. Stand: inkl. Phase 4._
_Sensible Infrastruktur-Details sind durch Platzhalter ersetzt (`<SERVER_IP>`, `<BACKUP_IP>`, `<HOME_SUBNET>`, `<USER>` etc.)._

---

## Technische Entscheidungen

| Thema | Wahl | Verworfen | Grund |
|-------|------|-----------|-------|
| OS | Ubuntu Server 24.04 LTS (Bare-Metal) | Proxmox, Ubuntu 26.04 | Battle-tested, NVIDIA-Support, 5 Jahre Updates |
| Storage | LVM (40GB root, 404GB docker) | Klassische Partitionierung | Flexible Größenanpassung, Snapshots |
| Orchestrierung | Docker Compose | Kubernetes | Single-Node, MVP-Komplexität |
| LLM lokal | Ollama (GGUF, quantisiert) + Qdrant RAG | – | Privacy, urheberrechtlich geschützte Daten |
| LLM cloud | Mistral Le Chat (Schüler-Abo, ab Oktober) + Perplexity Education Pro | Google AI Pro, API-Calls | Weg von US-Anbietern; Schüler-Abo 6€/Monat |
| Backup | rsync + Acer TravelMate 5735 + 500GB USB-HDD | Synology NAS DS216+II (150€) | 0€ (Hardware vorhanden), MVP-Fokus |
| LUKS | NEIN | – | Headless-Server: Passphrase bei Reboot nicht möglich |
| Monitoring | Netdata (Docker, network_mode: host) | Prometheus + Grafana | Einfachste Einrichtung, Open Source, Netdata Cloud-Integration |
| Netdata Netzwerk | network_mode: host | bridge (internal-ai) | Bridge hatte DNS-Auflösungsprobleme bei Netdata-Cloud-Verbindung |
| Netdata Image | latest (nicht gepinnt) | – | Rolling release – keine stabilen Tags; Netdata Cloud-Kompatibilität erfordert aktuelle Version |
| GPU (Phase 4) | MSI Aero GTX 1070 8GB OC (Compute 6.1) | Quadro K2200 (Compute 5.0) | K2200 inkompatibel mit Ollama in Docker |
| OpenClaw | Installiert im MVP (keine Custom Skills) | – | Portfolio-Relevanz: agentisches Gateway als Differenzierungsmerkmal |
| OpenClaw Port | 18789 | 8080 (Blueprint-Fehler) | Korrigiert nach offizieller Doku |
| OpenClaw Image | ghcr.io/openclaw/openclaw:latest (2026.5.12) | 2026.4.22 | Wizard hängt in 2026.4.22; latest enthält CVE-Patches |
| OpenClaw Reihenfolge | Phase 3 (NACH Ollama-Modelle gepullt) | Parallel mit Stack | Wizard benötigt laufendes Ollama-Backend mit Modell |
| OpenClaw Sandbox | Docker CLI-Mount + Socket-Mount + sandbox-setup.sh | OPENCLAW_INSTALL_DOCKER_CLI Flag | Flag funktioniert nicht in 2026.5.12; manuelle Einrichtung nötig |
| OpenClaw Wizard | tmux | Direkte SSH-Session | SSH-Disconnect unterbricht interaktiven Wizard |
| DEX Telegram-Kanal | Telegram Bot API (einziges User-Interface) | Open WebUI | Mobil immer verfügbar; Open WebUI nur für direkte Modell-Tests |
| DEX-Modell (aktiv) | hermes3:8b-dex (Custom Modelfile) | interstellarninja/hermes-3-llama-3.1-8b-tools | OpenClaw Bug #50712: think:false nicht gesendet → leere Antworten |
| interstellarninja-Modell | ❌ Inkompatibel mit OpenClaw 2026.5.12 | – | Bug #50712: thinking-fähige Modelle produzieren leere Antworten |
| SOUL.md Ansatz | Kurze Beispiel-Sätze (<800 Zeichen) | Abstrakte Beschreibungen, lange Beispiellisten | Länge >~1000 Zeichen triggert Bug #2731 bei hermes3:8b |
| SYSTEM in Modelfile | NICHT verwendet | Baked-in SYSTEM-Parameter | Konflikt mit OpenClaw SOUL.md-Injection |
| PARAMETER think false | Nicht möglich | – | Kein gültiger Ollama-Modelfile-Parameter (unknown parameter) |
| DEX tools-Profil | minimal + explizit deny: ["group:fs", ...] | coding (Standard) | Bug #42165: minimal lässt group:fs durch ohne explizites deny |
| bootstrapTotalMaxChars | 60000 | Standard 150000 | SOUL.md behält Gewicht wenn MEMORY.md wächst |
| DEX allow-Liste | ["memory_search", "memory_get", "message"] | session_status enthalten | session_status triggert unnötige Tool-Calls bei Konversationsfragen |
| iptables DOCKER-USER | 172.0.0.0/8 permanent in /etc/ufw/after.rules | Temporäre iptables-Regel | Persistent über Reboots – Docker-Container haben dauerhaft Internetzugang |
| Sandbox-Agent | qwen2.5:32b | qwen2.5:7b | Komplexere Szenarien erfordern besseres Reasoning |
| Ollama Healthcheck | CMD-SHELL `ollama list` | CMD curl | curl nicht im Ollama-Container vorhanden |
| containerd Speicherort | Symlink /var/lib/containerd → /var/lib/docker/containerd | Root-Partition | Root-Partition-Schutz (40GB-Limit) |
| Shutdown-Sequenz | `docker compose down` vor shutdown | Direkter shutdown | Verhindert RWLayer-Korruption bei Container-State |
| GVS-Rhythmus | Zähler-basiert (alle 7 = Weekly, alle 28 = Monthly) | Datum-basiert | Kalenderunabhängig – verpasste Nächte zerstören nicht den GVS-Rhythmus |
| Backup-Trigger | systemd Timer (`Persistent=true`) auf Backup-Node | Cron auf Hauptserver | Catch-up beim nächsten Boot |
| Phase-Struktur | 6 Phasen (Phase 5 = RAG/Workflows, Phase 6 = Hybrid) | 5 Phasen | Logische Reihenfolge: Modelle vor OpenClaw; RAG als eigene Phase |
| MVP-Definition | Funktionsfähiges System inkl. GTX 1070 – portfolio-reif | Business-MVP-Konzept | Persönliches Projekt: MVP = dokumentierter Baseline-Stand, danach Iteration |
| openclaw.json bearbeiten | Gezielte sed/awk-Befehle oder SFTP | nano | nano ist fehleranfällig bei komplexem JSON – Komma-Fehler schwer zu finden |

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

### containerd
- **Speicherort:** /var/lib/docker/containerd (auf lv-docker)
- **Symlink:** /var/lib/containerd → /var/lib/docker/containerd

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
10 22 * * 7   /usr/local/bin/sysconfig-export.sh                    # Sonntags 22:10
30 23 * * *   cd /opt/synapspace && docker compose down && cd /opt/openclaw && docker compose -f docker-compose.yml down && /sbin/shutdown -h now   # täglich 23:30
```

### NVIDIA
- **Driver:** `<NVIDIA_DRIVER_VERSION>` (via ubuntu-drivers install)
- **CUDA:** 12.2
- **Container Toolkit:** nvidia-container-toolkit (nvidia.github.io Repository)
- **Docker Runtime Config:** /etc/docker/daemon.json (via nvidia-ctk runtime configure)
- **GPU:** MSI Aero GTX 1070 8GB OC (Compute 6.1) ✅ eingebaut + verifiziert (gpu-burn: 0 Fehler, 74°C Peak)

### Docker
- **Installation:** Docker CE aus offiziellem Docker-Repository (download.docker.com)
- **Pakete:** docker-ce, docker-ce-cli, containerd.io, docker-compose-plugin

### UFW
- **Status:** aktiv
- **Default:** deny incoming, allow outgoing
- **Port 22/tcp:** ALLOW IN Anywhere
- **Ports 5678, 3000, 9000, 11434, 6333, 18789:** ALLOW IN `<HOME_SUBNET>`
- **UFW/Docker-Bypass-Fix:** DOCKER-USER Chain in /etc/ufw/after.rules konfiguriert
- **Docker-interne Netze:** `-A DOCKER-USER -s 172.0.0.0/8 -j RETURN` permanent in /etc/ufw/after.rules

### SSH
- **Key-Typ:** Ed25519
- **Kommentar:** synapspace-admin
- **Public Key Speicherort Server:** /home/`<USER>`/.ssh/authorized_keys
- **PasswordAuthentication:** no (/etc/ssh/sshd_config)

### Docker Compose (aktiv)
- **Pfad:** /opt/synapspace/docker-compose.yml
- **Services:** n8n, Ollama, Qdrant, Open WebUI, Portainer, Netdata
- **Netzwerk:** synapspace_internal-ai (Bridge) – Netdata: network_mode: host
- **Volumes:** Persistent benannt für alle Services
- **Port-Binding:** Alle Ports an `<SERVER_IP>` gebunden (UFW-Bypass-Schutz)

### Service-Ports (aktiv)
- n8n: 5678 | Open WebUI: 3000 | Portainer: 9000 | Ollama: 11434 | Qdrant: 6333 | OpenClaw/DEX: 18789
- Netdata: 19999 (network_mode: host, Netdata Cloud verbunden – persönlicher Account)

### Image-Versionen (gepinnt)
- portainer/portainer-ce:**2.41.1**
- ollama/ollama:**0.24.0**
- qdrant/qdrant:**v1.17.0**
- ghcr.io/open-webui/open-webui:**v0.9.5**
- n8nio/n8n:**2.14.2**
- ghcr.io/openclaw/openclaw:**2026.5.12** (latest, enthält CVE-Patches)
- netdata/netdata:**latest** (aktiv, Rolling Release)

### Modelle (installiert)
- llama3.2:3b (2GB) | hermes3:8b (4.7GB) | **hermes3:8b-dex** (4.7GB, Custom Modelfile) | qwen2.5:7b (4.7GB)
- mistral-nemo:12b (7.1GB) | command-r:35b (18GB) | qwen2.5:32b (19GB)
- llama3.1:70b (42GB)
- **Gesamt:** ~102GB

### Modelfiles (aktiv)
- **Pfad:** /opt/synapspace/modelfiles/
- **hermes3-dex.Modelfile:** FROM hermes3:8b, temperature 0.3, top_p 0.9, repeat_penalty 1.1, num_ctx 8192
- **KEIN SYSTEM-Parameter** im Modelfile – Konflikt mit OpenClaw SOUL.md-Injection

### DEX / OpenClaw (aktiv)
- **Name:** DEX – Dynamic Educational eXpander
- **Repo:** /opt/openclaw
- **Image:** ghcr.io/openclaw/openclaw:2026.5.12
- **Port:** 18789
- **Config:** /home/`<USER>`/.openclaw/openclaw.json
- **Workspace:** /home/`<USER>`/.openclaw/workspace/
- **Primäres Modell:** ollama/hermes3:8b-dex
- **Ollama-URL:** http://`<SERVER_IP>`:11434 (KEIN /v1-Suffix)
- **Heartbeat MVP:** every: "0m"
- **Sandbox:** mode: non-main, scope: agent
- **Sandbox-Image:** openclaw-sandbox:bookworm-slim
- **DOCKER_GID:** `<DOCKER_GID>`
- **Telegram-Bot:** DEX (Bot-Handle redigiert), Allowlist: nur autorisierter Nutzer (numerische User-ID, unquoted)
- **tools.profile:** minimal
- **tools.deny:** group:fs, exec, process, browser, cron, gateway, sessions_spawn
- **tools.allow:** memory_search, memory_get, message
- **bootstrapTotalMaxChars:** 60000
- **loopDetection:** enabled, criticalThreshold: 10
- **SOUL.md:** ~750 Zeichen (kürzer = besser bei hermes3:8b; >~1000 Zeichen triggert Bug #2731)

---

## Lessons Learned

### Hardware & Vorbereitung
- ✅ Systematische Dokumentation (Screenshots, BIOS-Checks)
- ✅ Memtest validiert Stabilität trotz gemischter RAM-Revisionen
- ⚠️ eBay-Risiko: Immer Revisionen VOR Kauf prüfen
- ⚠️ ZIP-Archive IMMER entpacken vor Installer-Ausführung
- ⚠️ PCIe-Kabelkompatibilität vorab prüfen – Fujitsu M740b braucht 6+2-auf-8-Pin Adapter für GTX 1070
- ℹ️ FurMark + GPU-Z vor Gebrauchtkauf: Temperaturen, Artefakte, VRAM-Verifizierung
- ℹ️ gpu-burn im Docker-Container (nvidia/cuda:12.2.0-devel) ist zuverlässiger Stresstest für eingebaute GPU
- ℹ️ Erdung: Netzkabel ziehen + Heizungsrohr reicht als Erdungspunkt

### Installation
- ⚠️ Ethernet-Kabel VOR dem Boot einstecken – DHCP sonst nicht möglich
- ⚠️ Ubuntu 24.04: SSH trotz Installer-Auswahl nicht automatisch aktiv → `sudo systemctl enable ssh --now`
- ⚠️ LUKS auf Headless-Servern vermeiden – Passphrase bei Reboot nicht möglich
- ℹ️ Headless-Betrieb ab Phase 1 vollständig – Monitor/Tastatur am Server nicht mehr erforderlich

### Docker & NVIDIA
- ⚠️ Ollama Compute 5.0 (Quadro K2200) inkompatibel mit aktuellem Ollama in Docker → CPU-only bis GPU-Upgrade
- ⚠️ containerd speichert Layer-Daten auf Root-Partition → Symlink auf Docker-Volume Pflicht
- ⚠️ Shutdown ohne `docker compose down` → RWLayer-Korruption beim nächsten Start
- ⚠️ `<USER>` nicht automatisch in docker-Gruppe → `sudo usermod -aG docker <USER> && newgrp docker`
- ℹ️ Ollama curl-Healthcheck schlägt fehl → `CMD-SHELL ollama list || exit 1` verwenden
- ℹ️ Layer-Splitting erfolgt automatisch durch Ollama – kein manueller Eingriff nötig

### Netdata
- ⚠️ Netdata im Bridge-Netzwerk hat DNS-Auflösungsprobleme → `network_mode: host` nötig
- ℹ️ iptables DOCKER-USER Regel `172.0.0.0/8 RETURN` muss permanent in `/etc/ufw/after.rules` stehen (nicht nur temporär)

### SSH-Hardening
- ℹ️ Reihenfolge: Key-Login verifizieren → erst dann PasswordAuthentication deaktivieren
- ℹ️ Ed25519 bevorzugen gegenüber RSA

### OpenClaw & DEX
- ⚠️ Wizard in Version 2026.4.22 hängt → latest (2026.5.12) verwenden
- ⚠️ Wizard immer in tmux ausführen – SSH-Disconnect unterbricht interaktiven Wizard
- ⚠️ Sandbox benötigt: Docker CLI (Mount) + Docker Socket + Sandbox-Image (sandbox-setup.sh)
- ⚠️ OPENCLAW_INSTALL_DOCKER_CLI Flag funktioniert nicht → manueller Mount via EXTRA_MOUNTS
- ⚠️ Telegram Bot-Token nach mehreren Wizard-Läufen ungültig → via BotFather revoken + neu setzen
- ⚠️ DOCKER-USER Chain: 172.0.0.0/8-Regel PERMANENT in /etc/ufw/after.rules eintragen – nach Reboot sonst weg
- ⚠️ SOUL.md Länge >~1000 Zeichen triggert hermes3:8b Bug #2731 zuverlässig → unter 800 Zeichen halten
- ⚠️ interstellarninja/hermes-3-llama-3.1-8b-tools inkompatibel mit OpenClaw 2026.5.12 (Bug #50712: think:false nicht gesendet → payloads=0)
- ⚠️ PARAMETER think false ist kein gültiger Ollama-Modelfile-Parameter → unknown parameter
- ⚠️ OpenClaw löst custom Modellnamen auf Basis-Modell auf – Renaming umgeht thinking-Erkennung nicht
- ⚠️ tools.profile: "minimal" lässt group:fs durch → explizit deny: ["group:fs"] setzen (Bug #42165)
- ⚠️ session_status aus allow-Liste entfernen – triggert unnötige Tool-Calls bei Konversationsfragen
- ⚠️ openclaw.json Bearbeitung per nano ist fehleranfällig → gezielte sed/awk-Befehle oder SFTP bevorzugen
- ⚠️ `python3 -m json.tool` nach jeder JSON-Änderung – verhindert stundenlange Fehlersuche
- ℹ️ Ollama baseUrl OHNE /v1-Suffix – sonst bricht Tool-Calling komplett
- ℹ️ OpenClaw Provider-Prefix `ollama/` zwingend – sonst Fallback auf openai/
- ℹ️ AGENTS.md ist System-Prompt-Erweiterung, kein technischer Enforcer – harte Grenzen via Config
- ℹ️ MAS: Keine native DAG-Orchestrierung – Blackboard-Pattern via Dateisystem
- ℹ️ 8B-Modelle haben begrenzte Persona-Treue – mit 30B-Modellen deutlich besser
