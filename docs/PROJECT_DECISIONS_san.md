# PROJECT_DECISIONS – Technische Entscheidungen (SynapSpace)
_Entscheidungshistorie: Entscheidungen, Begründungen, Umentscheidungen. Überholtes wird „⚠️ HISTORISCH" markiert statt gelöscht (Portfolio-Erhalt). Konfigurationen und Lessons Learned stehen in den Phasen-Dokumenten (`PHASE_*_san.md`)._

---

## Technische Entscheidungen

| Thema | Wahl | Verworfen | Grund |
|-------|------|-----------|-------|
| OS | Ubuntu Server 24.04 LTS (Bare-Metal) | Proxmox, Ubuntu 26.04 | Battle-tested, NVIDIA-Support, 5 Jahre Updates |
| Storage | LVM (40GB root, 374GB docker) | Klassische Partitionierung | Flexible Größenanpassung, Snapshots |
| Orchestrierung | Docker Compose | Kubernetes | Single-Node, MVP-Komplexität |
| LLM lokal | Ollama (GGUF, quantisiert) + Qdrant RAG | – | Privacy, urheberrechtlich geschützte Daten |
| LLM cloud | **Mistral Vibe Education-Pro (aktiv, 12 Monate im Voraus bezahlt)** – einzige Cloud-KI im Ökosystem | Google AI Pro, Gemini/Gemini Live, NotebookLM, Perplexity, API-Calls, „Schüler-Abo ab Oktober" (überholt) | Weg von US-Anbietern; Education = voller Pro-Umfang; No-Telemetry; EU-Datenresidenz |
| **Tool-Souveränität (Lernökosystem)** | nur **lokal / Open Source / DSGVO-konform (EU)**; Cloud-KI ausschließlich Mistral; **Boardmix** als bezahlte Whiteboard-Ausnahme (keine KI-Funktionen genutzt) | Gemini/Gemini Live, NotebookLM, Perplexity, Google AI Pro | DSGVO/EU-Souveränität; kein US-CLOUD-Act; Boardmix lebenslang bezahlt + laut Anbieter EU-konform |
| **Coach-Engine (Phase 4.5)** | **Mistral Vibe (Cloud, „Weg A")** – Persona „DEX" via Projekt-Custom-Instructions | lokaler OpenClaw/Hermes-Agent + 24GB-GPU-Upgrade | Cloud-Modell (Medium 3.5, 256k Kontext) schlägt jedes lokal lauffähige Modell; kein GPU-Upgrade nötig; schnellster Weg zu „funktioniert einfach" |
| **Coach-Brücke (Phase 4.5)** | **MCP** – n8n als MCP-Server, Vibe als MCP-Client | – | Tool-Antworten fließen ins Coach-Reasoning; lange Material-Generierung läuft **asynchron** (`tool_timeout` ~60 s → Auftrag annehmen, Ergebnis separat melden) |
| **Coach-Gedächtnis / Lernprofil (Phase 4.5)** | **lokal** (n8n + DB), Coach liest per MCP | reine Cloud-Haltung | Cloud (EU) wäre datenschutzrechtlich ok; lokal nur aus **Praktikabilität** (automatische Pflege via MCP) |
| **OpenClaw/Hermes als Agent-Framework** | **Post-MVP / Fallback** („Weg B", über Mistral-API) | Einsatz im MVP | API ist NICHT im Abo (Medium 3.5 ≈ **$1,50 In / $7,50 Out** pro 1 Mio. Token) → Zusatzkosten; im MVP nicht nötig |
| **MCP-Brücke: Mechanismus (Phase 5)** | **Tür B – „MCP Server Trigger"-Node** (ein Workflow = benannte Tools) | Tür A – Instance-level MCP / nativer n8n-Konnektor (= Fallback) | Semantisch benannte Tools (DEX wählt aus der Bedeutung), GA-stabil (Tür A noch Beta), n8n bleibt einziger Schreib-Gateway, passt zum Rollenmodell |
| **n8n-Version (Phase 5)** | **2.14.2 (live bestätigt)** | – | Beide MCP-Wege verfügbar (Instance-level MCP ab 2.13; Server-Trigger-Node im Core) |
| **Externe Domain (Phase 5/6)** | **deSEC / dedyn.io** (`<EXTERNAL_DOMAIN>`, kostenloser DynDNS) | DuckDNS (verworfen) | DuckDNS unzuverlässig (wiederholte Ausfälle, Port-80-Timeouts schon im Browser); deSEC: gemeinnützig/DE, Anycast (15 Standorte, IPv4+IPv6), DSGVO/souverän, FritzBox-nativ, DNS-01-fähig |
| **Reverse-Proxy (Phase 6)** | **Nginx Proxy Manager 2.15.1** (Container im Stack, Netz `internal-ai`) | direkte Port-Exposition der Dienste | TLS-Terminierung + LE-Verwaltung per GUI; Proxy-Ziel containerintern `http://n8n:5678` (WebSockets an); Ports 80/443/81 an `<SERVER_IP>`, Admin-GUI (81) nur LAN; named volumes `npm_data`/`npm_letsencrypt` |
| **TLS-Verfahren (Phase 6)** | **DNS-01 über deSEC-API (geplant, Verifikation offen)** | HTTP-01 (verworfen) | Vodafone blockt eingehend Port 80 → HTTP-01 unmöglich; DNS-01 braucht keinen offenen Port 80 |
| **Externer Zugang / Port (Phase 6)** | **hoher Port geplant (z. B. extern 8443 → intern 443)**, hoher-Port-Test offen | Standard-Port 443 | Vodafone blockt eingehend die Standard-Ports 80/443 für Privatanschlüsse; hohe Ports kommen i. d. R. durch |
| **Recherchestrategie (Deployment)** | aktuelle/nischige Deployment-Fakten: **Claude-Websuche an Primärquellen**; NotebookLM nur für kuratierte/große/stabile Quellen | NotebookLM für brandaktuelle Nischenthemen | NotebookLMs Web-Discovery liefert dort SEO-Guides statt Primärquellen → direkte Recherche spart Troubleshooting |
| Backup | rsync + Acer TravelMate 5735 + 500GB USB-HDD | Synology NAS DS216+II (150€) | 0€ (Hardware vorhanden), MVP-Fokus |
| **Backup-Platten-Aufteilung (Phase 4.5)** | **Server → externe HDD (`/mnt/backup-hdd`); Client (DevNode) → interne SSD (`/mnt/backup/ssd`)** | Repartitionieren der Node-Platten | Saubere Trennung, nutzt vorhandene Partition; Live-Shrink der Root-Partition riskant und unnötig |
| **DevNode-Backup (Phase 4.5)** | **verschlüsselter Cryptomator-Tresor „Wertvoll" → Node-SSD, 3 rollierende Hardlink-Schnappschüsse, manuell per Ein-Klick** | Nacht-Automatik / Auto-Wake | Laptop + Node nur zeitweise an; manueller Start zuverlässiger (Wecken nur aus Schlaf, nicht aus Aus) |
| **Cryptomator (Phase 4.5)** | **alle Cloud-Daten verschlüsselt; zwei Tresore („01_Wertvoll" + „Rest")** | unverschlüsselte Cloud-Ablage | DSGVO/Souveränität; arbeitet transparent → ~0 Mehraufwand → „alles verschlüsseln"; EU-Alternativen (Filen/Proton/Nextcloud) Post-MVP |
| **VPN-Hinweis-Tool (DevNode, Phase 4.5)** | **ereignisgesteuerte Aufgabe + Mehrschirm-Popup (Stufe 1: Erinnerung)** | Auto-Aktivierung (Stufe 2), AutoHotkey-Dauer-Polling | „laut + von dir bestätigt" bei einer Sicherheitsfunktion sicherer als „still + automatisch"; Event-Trigger statt hängendem Dauerprozess |
| **FritzBox UPnP (Phase 4.5)** | **„Selbstständige Portfreigabe" aus** | an (Default) | keine automatischen Portöffnungen durch Geräte; Härtung |
| LUKS | NEIN | – | Headless-Server: Passphrase bei Reboot nicht möglich |
| Monitoring | Netdata (Docker, network_mode: host) | Prometheus + Grafana | Einfachste Einrichtung, Open Source, Netdata Cloud-Integration |
| Netdata Netzwerk | network_mode: host | bridge (internal-ai) | Bridge hatte DNS-Auflösungsprobleme bei Netdata-Cloud-Verbindung |
| Netdata Image | latest (nicht gepinnt) | – | Rolling release – keine stabilen Tags; Netdata Cloud-Kompatibilität erfordert aktuelle Version |
| GPU (Phase 4) | MSI Aero GTX 1070 8GB OC (Compute 6.1) | Quadro K2200 (Compute 5.0) | K2200 inkompatibel mit Ollama in Docker |
| OpenClaw | Installiert im MVP (keine Custom Skills) | – | Traumarbeitgeber vertreibt OpenClaw-as-a-Service |
| OpenClaw Port | 18789 | 8080 (Blueprint-Fehler) | Korrigiert nach offiziellem Docs |
| OpenClaw Image | ghcr.io/openclaw/openclaw:latest (2026.5.12) | 2026.4.22 | Wizard hängt in 2026.4.22; latest enthält CVE-Patches |
| OpenClaw Reihenfolge | Phase 3 (NACH Ollama-Modelle gepullt) | Parallel mit Stack | Wizard benötigt laufendes Ollama-Backend mit Modell |
| OpenClaw Sandbox | Docker CLI-Mount + Socket-Mount + sandbox-setup.sh | OPENCLAW_INSTALL_DOCKER_CLI Flag | Flag funktioniert nicht in 2026.5.12; manuelle Einrichtung nötig |
| OpenClaw Wizard | tmux | Direkte SSH-Session | SSH-Disconnect unterbricht interaktiven Wizard |
| OpenClaw Port-Binding (Host) | 127.0.0.1:18789-18790 (Loopback) | 0.0.0.0/[::] (Compose-Default) | Docker-published Ports umgehen UFW; Gateway (RCE-CVE-Historie + Docker-Socket) nicht extern/über IPv6 exponieren |
| OpenClaw Config-Rechte | 700 (Verzeichnis) / 600 (openclaw.json) | 755 / 664 | Telegram-Bot-Token-Schutz (Container liest als root weiterhin) |
| **DEX Nutzungskonzept — ⚠️ HISTORISCH** | OPENCLAW_BRIEFING.md (lokaler Coach-Plan) | – | beschreibt den lokalen OpenClaw-DEX; durch Cloud-Coach überholt |
| **DEX Telegram-Kanal — ⚠️ HISTORISCH** | Telegram Bot API (einziges User-Interface) | Open WebUI | galt für lokalen DEX; mit Cloud-Coach (Vibe-Interface) gegenstandslos |
| **DEX-Modell (lokal) — ⚠️ HISTORISCH / Post-MVP** | hermes3:8b-dex (Custom Modelfile) | interstellarninja/hermes-3-llama-3.1-8b-tools | galt für den **lokalen** DEX (Bug #50712 bei interstellarninja); mit Cloud-Coach (Phase 4.5) **gegenstandslos** |
| **Mistral Nemo 12b (DEX-Eval) — ⚠️ HISTORISCH** | verworfen | – | KV-Cache-Overhead grenzwertig bei 8GB VRAM; lokale DEX-Motor-Frage durch Cloud-Coach ohnehin gegenstandslos; kein 10B-Modell geplant |
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
| Shutdown-Sequenz (alt) | entfällt – Server läuft 24/7 | täglicher `compose down` + shutdown | Siehe „Server-Betrieb" |
| GVS-Rhythmus | Zähler-basiert (alle 7 = Weekly, alle 28 = Monthly) | Datum-basiert | Kalenderunabhängig – verpasste Nächte zerstören nicht den GVS-Rhythmus |
| Backup-Trigger | systemd Timer (`Persistent=true`) auf Backup-Node | Cron auf Hauptserver | Catch-up beim nächsten Boot |
| Phase-Struktur | 6 Phasen + Phase 4.5 (Härtung & Remote-Zugriff) | – | Betriebs-/Security-Strang parallel zum Feature-Phasenplan |
| MVP-Definition | Funktionsfähiges System inkl. GTX 1070 – portfolio-reif | Business-MVP-Konzept | Persönliches Projekt: MVP = dokumentierter Baseline-Stand, danach Iteration |
| openclaw.json bearbeiten | Gezielte sed/awk-Befehle oder SFTP | nano | nano ist fehleranfällig bei komplexem JSON – Komma-Fehler schwer zu finden |
| **Server-Betrieb** | **24/7 Dauerbetrieb** | täglicher Shutdown-Cron | Hardware ist für Dauerbetrieb gebaut; Thermal-Cycling schädlicher als Dauerlauf; Shutdown-Cron entfernt |
| **Reboot-Strategie** | **Wöchentlicher Reboot So 05:00 (Cron)** | täglicher Shutdown / unattended-upgrades Auto-Reboot | Aktiviert über die Woche installierte Kernel-/Lib-Patches; planbare Zeit; Container via `restart: unless-stopped` |
| **SSH-Authentifizierung** | **key-only (Ed25519)** via Drop-in `01-hardening.conf` | Passwort-Auth | Härtung; cloud-init `50-cloud-init.conf` hatte `PasswordAuthentication yes` erzwungen |
| **UFW Port 22** | **nur `<HOME_SUBNET>`** | Anywhere (v4 + v6) | Externe/IPv6-SSH-Exposition geschlossen; WireGuard-Clients erscheinen als LAN → kompatibel |
| **lv-docker Größe** | **374G** (von 404G verkleinert) | 404G | VG-Reserve (~30G `VFree`) für LVM-Snapshots vor kritischen Änderungen |
| **Heimanschluss (Phase 4.5)** | **Vodafone DSL mit DualStack / öffentlicher IPv4** (aktiv seit 04.06.2026) | DS-Lite (bis Phase 4.5) | DS-Lite hatte keine öffentliche IPv4 → Server extern nur via IPv6; DualStack macht ihn von außen via IPv4 erreichbar (Voraussetzung für MCP-Brücke) |
| **Coach-Scope (Phase 5)** | **DEX = nur Lerncoach** (Fachmann fürs Lernen, nicht für IT-Inhalte) | Karriere-Coaching im selben Coach | Lernwissenschaft vs. Arbeitsplatz/Interpersonelles sind verschiedene Domänen; ein **separater** Karrierecoach kommt zum Praktikumsstart |
| **Persona-Mechanik (Phase 5)** | **Projekt-Custom-Instructions** (Markdown, kein hartes Längenlimit) | Skill | Skills sind aufrufbare Automatisierung (Slash-Befehle), nicht für reaktiven Dialog (Briefing §15); CI führt jeden Chat im Projekt automatisch als DEX |
| **Tutoren (Phase 5)** | **eigene Mistral-Projekte, themen-spezialisiert**; Lerntechnik **pro Auftrag von n8n injiziert** | Tutor = Technik-Spezialist; Tutoren im DEX-Projekt | Domänenwissen ist tief/stabil → eigene Persona; Technik ist übertragbar → weniger Tutoren, einfacheres Routing |
| **Schreib-Hoheit (Phase 5)** | **n8n = einziger Schreib-Gateway zur DB** (DEX schreibt nie direkt) | DEX schreibt direkt | Datenhoheit lokal + konsistent, Validierung an einer Stelle, saubere Freigaben |
| **Tutor-Routing (Phase 5)** | **über das lokale System vermittelt** (DEX → n8n erzeugt Tutor-Prompt → Nutzer öffnet Tutor) | direkter Cross-Projekt-Anstoß | automatisches Anstoßen über Mistral-Projekte hinweg unklar → beim Bauen verifizieren; vermittelt sicher machbar |
| **Coach-Gedächtnis (Phase 5)** | **lokale DB = Quelle der Wahrheit**; Mistral-Chat-Gedächtnis nur Komfort-Schicht | Mistral-Chat-Gedächtnis als Fundament | Mistral-Memory ist Legacy/im Umbau, intransparent, unstrukturiert; lokal = strukturiert/abfragbar/kontrolliert |
| **Lernprofil-DB (Phase 5) ✅ angelegt** | **PostgreSQL 16-alpine (lokal, im Stack), kein Host-Port; Zugangsdaten via `.env`; Adminer als LAN-Web-Oberfläche; 11-Tabellen-Schema** inkl. Feedback-Haken + Meta-Lern-Feld | reine Cloud-Haltung; DB-Port nach außen | relational sauber abfragbar; lokal aus Praktikabilität (Pflege via MCP); kein offener DB-Port = Härtung |
| **Wissensart-/Methoden-Modell (Phase 5)** | **Wissensart-Achse (PACER-Platzhalter) + `methode_wissensart` (n:m, Eignung)**; Methodenpool je Thema = **Abfrage** | „zugeordnete Methode" als Einzelfeld; manuelle Pool-Pflege je Subthema | Methode ändert sich je Thema/über Zeit + Wirksamkeit messbar; Pool aus Tags ableitbar spart Pflege bei ~100 Subthemen; Taxonomie/Eignung via Recherche (Phase 6) |
| **DB-Geheimnisse / `.env` (Phase 5)** | **DB-Passwort in `.env` (chmod 600)**; bestehende Secrets (WEBUI/Netdata) bleiben in der Compose | bestehende Secrets aktiv in `.env` auslagern | `.env` trennt Geheimnis von versioniertem Code = **Repo-Hygiene, keine Härtung** (lokale Dateien auf demselben Server) → nur nötig, falls die Compose je geteilt/committet wird |
| **Mistral Projekt-Ton (Phase 5)** | **Standard**, Checkbox „globale Anweisungen einbeziehen" **aus** | professionell / einfühlsam / direkt | die Custom Instructions tragen den Ton selbst; „aus" hält den Projekt-Kontext sauber |

> ⚠️ **STATUS-HINWEIS (Coach-Architektur-Umentscheidung, Phase 4.5):**
> Alle Zeilen oben zu **OpenClaw/DEX als lokalem Coach**, zum **DEX-Motor-Modell** (hermes3:8b-dex) und zu den **DEX-Modell-Evaluierungen** sind **HISTORISCH / Post-MVP**.
> **JETZT gilt:** Coach „DEX" = **Mistral Vibe (Cloud, Weg A)**. OpenClaw bleibt installiert (auf Loopback, kostet nichts), ist aber **nicht** die Coach-Engine im MVP. Einziger möglicher Post-MVP-Use-Case: Sandbox-Szenario-Builder.
> Vollständige Begründung + Ziel-Architektur (3 Schichten + MCP-Brücke): **PHASE_4_5_san.md, Schritt 2 (Coach-Engine-Entscheidung)**.
