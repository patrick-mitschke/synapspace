# SynapSpace – Lokales KI-Lernökosystem

> **Dieses Repository befindet sich im aktiven Aufbau.**
> Phasen 0–4.5 sind abgeschlossen und dokumentiert. Phase 5 (Hybrid-System mit Mistral) ist aktiv, Phase 6 folgt. Die README gibt den vollständigen Überblick – detaillierte Logs, Konfigurationen und die Entscheidungshistorie liegen in separaten Dateien unter [`docs/`](https://github.com/patrick-mitschke/synapspace/blob/main/docs).

---

## Was ist SynapSpace?

SynapSpace ist ein selbst deployetes KI-Ökosystem, das auf einem lokalen Heimserver läuft und personalisiertes, datengeschütztes Lernen ermöglicht – ohne Abhängigkeit von kommerziellen Cloud-Diensten für sensible Inhalte.

Im Kern verbindet SynapSpace mehrere Open-Source-Komponenten zu einem funktionierenden Ganzen: **Ollama** stellt lokale Sprachmodelle bereit, **Qdrant** ermöglicht semantische Suche über urheberrechtlich geschützte Fachliteratur (RAG), **n8n** orchestriert Lern-Workflows und ist zugleich der einzige Schreib-Gateway zur lokalen Datenbank, **PostgreSQL** hält das persönliche Lernprofil, **Open WebUI** dient dem direkten Testen der lokalen Modelle. Der strategische Lerncoach **DEX** läuft als Cloud-Dienst auf **Mistral Vibe** (EU, DSGVO-konform) und liest das lokale Lernprofil über eine **MCP-Brücke** – das System bleibt damit datensouverän, während der Coach von einem leistungsfähigen Cloud-Modell profitiert.

Ein zentrales Prinzip ist **Tool-Souveränität:** Im laufenden Ökosystem kommen ausschließlich lokale bzw. Open-Source-Komponenten zum Einsatz, und als einzige Cloud-KI Mistral (EU-Datenresidenz). Urheberrechtlich geschützte Inhalte verlassen den Server nie.

Das Ziel: Ein Lernsystem, das meinen aktuellen Wissensstand kennt, Lücken erkennt und Inhalte gezielt aufbereitet – verlässlich verfügbar, wann immer ich lernen will.

---

## Motivation & Ziele

**Kurzfristig – Praxisreife:** Die Praktikumsphase meiner Umschulung steht bevor. Ziel ist konkrete Erfahrung mit Linux-Administration, Docker, Netzwerktopologien, Backup-Strategien und KI-Infrastruktur – nicht nur theoretisch, sondern durch echtes Deployment.

**Mittelfristig – IHK-Prüfungsvorbereitung:** Effektive, personalisierte Vorbereitung auf die Abschlussprüfung zum Fachinformatiker Systemintegration (Sommer 2027) – durch ein System, das meinen Lernstand kennt, Lücken erkennt und Inhalte gezielt aufbereitet.

**Langfristig – Ein System, das mitwächst:** SynapSpace ist modular erweiterbar und soll weit über die Prüfung hinaus als persönlicher Lernassistent dienen – für jedes Thema, jedes Berufsfeld, jeden Lebensabschnitt. Die Vision: Ein System, das seinen Nutzer über Jahre kennenlernt und mit ihm wächst.

**Als Portfolio:** Dieses Projekt dokumentiert, wo mein Interesse und meine Motivation liegen – an der Schnittstelle von KI, Infrastruktur und autodidaktischem Lernen. Es richtet sich an Betriebe, die in dieser Arbeitsweise und dieser Dokumentation erkennen, was mir wichtig ist – und die jemanden suchen, der mit Eigeninitiative und echtem Interesse an die Sache herangeht.

---

## Tech-Stack (MVP)

| Komponente          | Technologie                                             | Rolle                                                   |
| ------------------- | ------------------------------------------------------- | ------------------------------------------------------- |
| **Server**          | Fujitsu Celsius M740b, Xeon E5-2620 v4, 64 GB ECC RAM   | Zentrale Inferenz- & Orchestrierungseinheit             |
| **GPU**             | MSI Aero GTX 1070 8GB OC *(eingebaut, verifiziert)*     | GPU-Inferenz, Layer-Splitting                           |
| **OS**              | Ubuntu Server 24.04 LTS (Bare-Metal)                    | Stabil, NVIDIA-kompatibel, 5 Jahre LTS                  |
| **Storage**         | LVM (40 GB root / 374 GB Docker, ~30 GB Snapshot-Reserve) | Flexible Größenanpassung, Snapshots vor kritischen Änderungen |
| **Container**       | Docker + Docker Compose                                 | Single-Node-Orchestrierung                              |
| **Container-Mgmt**  | Portainer                                               | Web-Verwaltung des Docker-Stacks                        |
| **Inferenz**        | Ollama (GGUF, quantisiert)                              | Lokale LLMs – GPU-beschleunigt, automatisches Layer-Splitting |
| **Vektordatenbank** | Qdrant                                                  | RAG für urheberrechtlich geschützte Medien              |
| **Lernprofil-DB**   | PostgreSQL 16 (+ Adminer als LAN-Web-Oberfläche)        | Quelle der Wahrheit für Lernstand, ROI-Katalog, Profil  |
| **Cloud-Coach**     | Mistral Vibe (Education-Pro, EU/DSGVO)                  | Strategischer Lerncoach „DEX" – einzige Cloud-KI im Ökosystem |
| **Automatisierung** | n8n                                                     | Workflow-Routing, ETL, einziger Schreib-Gateway zur DB, **MCP-Server** für den Coach |
| **Agentik (lokal)** | OpenClaw                                                | Loopback-Installation, Post-MVP-Kandidat (Sandbox-Szenario-Builder) |
| **Reverse-Proxy**   | Nginx Proxy Manager                                     | TLS-Terminierung für die MCP-Brücke (Phase 6)           |
| **UI**              | Open WebUI                                              | Direktes Testen der lokalen Modelle                     |
| **Monitoring**      | Netdata                                                 | Echtzeit-Dashboard: CPU, RAM, Disk, GPU, Docker         |
| **Backup**          | rsync + GVS-Strategie (Großvater-Vater-Sohn)            | Automatisiert, kalenderunabhängig                       |
| **Remote-Zugriff**  | WireGuard via FritzBox 7530 (DualStack / öffentliche IPv4, MyFRITZ) | Sicherer Fernzugriff ohne offene Portfreigaben          |
| **Firewall**        | UFW                                                     | Default-Deny, Whitelisting auf Subnetz-Ebene            |

**LLM-Portfolio (lokal, ~97,5 GB):**

| Modell                | Größe  | Rolle                                  |
| --------------------- | ------ | -------------------------------------- |
| Llama-3.2-3B          | 2,0 GB | Triage & Routing (lokale Workflows)    |
| Hermes-3-Llama-3.1-8B | 4,7 GB | Tool-Calling (lokal)                   |
| Qwen2.5-7B-Instruct   | 4,7 GB | IT-Skripting, Linux-Administration     |
| Mistral-Nemo-12B      | 7,1 GB | Vorfilterung, 128k Kontext             |
| Command-R (35B)       | 18 GB  | RAG-Spezialist, präzise Zitierung      |
| Qwen2.5-32B-Instruct  | 19 GB  | Komplexe Logik, Sandbox-Agent          |
| Llama-3.1-70B         | 42 GB  | Qualitätskontrolle (Senior-Endabnahme, über Nacht) |

---

## DEX – Dynamic Educational eXpander

DEX ist der strategische Lerncoach von SynapSpace. Er läuft als **Cloud-Dienst auf Mistral Vibe** (EU-Datenresidenz, DSGVO-konform); seine Persona ist über Projekt-Custom-Instructions verankert. Das **Lernprofil bleibt lokal** (PostgreSQL) – DEX liest es über eine **MCP-Brücke** (n8n als MCP-Server). So profitiert der Coach von einem leistungsfähigen Cloud-Modell, ohne dass das System die Datenhoheit verliert oder ein teures GPU-Upgrade nötig wird.

**Was DEX tut:**

- Behält den Lernplan im Blick, dokumentiert Stärken und Schwächen und erkennt Lücken über den **ROI-priorisierten** Prüfungskatalog
- **Empfiehlt und gibt Impulse, entscheidet aber nicht** – die Lernautonomie bleibt beim Nutzer
- Routet Aufträge ans lokale System (Material-Generierung, Tutor-Sessions), erklärt selbst aber keine Inhalte
- Bereitet strategisch auf das IHK-Fachgespräch und den Praktikumseinstieg vor

**Klare Aufgabenteilung:** DEX ist der Coach (Fachmann fürs *Lernen*), nicht der Tutor. Inhalts-Interaktion und Live-Übung übernehmen themen-spezialisierte Tutor-Chats; reale Troubleshooting-Übungen laufen in einer isolierten lokalen Sandbox, angebunden via MCP.

**Datenhoheit:** DEX schreibt nie direkt in die Datenbank – **n8n ist der einzige Schreib-Gateway**, sodass Validierung und Freigaben an einer Stelle liegen. Urheberrechtlich geschützte Fachliteratur bleibt strikt lokal (Qdrant) und gelangt nie in die Cloud.

**Bewusst nicht proaktiv:** Der Nutzer startet Gespräche selbst – der Coach soll nicht „treiben". Optionale proaktive Erinnerungen sind als Post-MVP-Erweiterung über n8n denkbar.

---

## Architektur-Übersicht

```
┌────────────────────────────────────────────────────────────────┐
│                      SynapSpace Heimnetz                        │
│                                                                 │
│  ┌──────────────────────┐      ┌────────────────────────┐       │
│  │  Fujitsu Celsius     │      │  Backup-Node           │       │
│  │  M740b (Hauptserver) │◄────►│  Acer TravelMate 5735  │       │
│  │                      │      │  rsync + NFS           │       │
│  │  Docker-Stack:       │      │  500 GB USB-HDD + SSD  │       │
│  │   n8n  (MCP-Server)  │      └────────────────────────┘       │
│  │   Ollama             │                                       │
│  │   Qdrant   (RAG)     │      ┌────────────────────────┐       │
│  │   PostgreSQL         │◄────►│  Dev-Node (Windows)    │       │
│  │   Open WebUI         │      │  Acer TravelMate P216  │       │
│  │   Portainer          │      │  MobaXterm, Git, Claude│       │
│  │   Nginx Proxy Mgr    │      └────────────────────────┘       │
│  │   OpenClaw (Loopback,│                                       │
│  │     Post-MVP)        │                                       │
│  └──────────┬───────────┘                                       │
│             │                                                   │
│             │            ┌────────────────────────┐             │
│             └───────────►│  FritzBox 7530         │             │
│                          │  WireGuard · DualStack │             │
│                          └─────┬──────────┬───────┘             │
└────────────────────────────────┼──────────┼────────────────────┘
        MCP-Brücke (TLS, n8n)     │          │   VPN-Tunnel
        ┌─────────────────────────▼──┐   ┌───▼──────────────────────┐
        │  Mistral Vibe (Cloud,      │   │  Mobile Endgeräte         │
        │  EU/DSGVO)                 │   │  Z Flip 7   (Interface)   │
        │  Lerncoach „DEX"           │   │  OnePlus 6  (Admin)       │
        │  liest Lernprofil per MCP  │   │  Tab S6 Lite (Lern-Tablet)│
        └────────────────────────────┘   └───────────────────────────┘
```

**Hybrid-Ansatz:** Urheberrechtlich geschützte Daten bleiben ausschließlich lokal. Als einzige Cloud-KI ergänzt **Mistral** das System – der Coach „DEX" (Vibe) und bei Bedarf Le Chat. Der Coach liest das lokale Lernprofil über die **MCP-Brücke** (n8n als MCP-Server, Vibe als MCP-Client, TLS-gesichert); lange Material-Generierung läuft **asynchron** und wird separat zurückgemeldet. Geschützte Inhalte werden nie in ein Cloud-Modell übertragen. **Claude** wird als Technical-Lead beim *Aufbau* genutzt, ist als US-gehostetes Tool aber bewusst **kein Bestandteil des laufenden Ökosystems** („Bauteam ≠ Ökosystem").

---

## Projektphasen

| Phase         | Inhalt                                                                                  | Status          | Aufwand   |
| ------------- | --------------------------------------------------------------------------------------- | --------------- | --------- |
| **Phase 0**   | Hardware-Inventur, RAM-Upgrade, Memtest, ISO-Setup                                      | ✅ Abgeschlossen | ~6h       |
| **Phase 1**   | Ubuntu Server, LVM, NVIDIA-Driver, Docker, UFW, SSH                                     | ✅ Abgeschlossen | ~5h       |
| **Phase 2**   | Laptop-Wartung, Netzwerk, Backup-Node, NFS, rsync/GVS, WireGuard, etckeeper             | ✅ Abgeschlossen | ~12h      |
| **Phase 3**   | Container-Stack, LLM-Modelle, OpenClaw                                                  | ✅ Abgeschlossen | ~6h       |
| **Phase 4**   | GPU-Integration, Netdata Monitoring, Modelfiles                                         | ✅ Abgeschlossen | ~10h      |
| **Phase 4.5** | Systemhärtung, Remote-Zugriff (DualStack/WireGuard), Coach-Architektur, Backup-Ausbau   | ✅ Abgeschlossen | ~20h      |
| **Phase 5**   | Hybrid-System mit Mistral: Coach „DEX" (Vibe), Lernprofil-DB, n8n als MCP-Server        | 🔄 In Arbeit    | ~12–15h   |
| **Phase 6**   | Lokales Wissens-Backend: RAG (Qdrant), Tutoren, Concept Map, Material-/Monitoring-Workflows | ⏳ Ausstehend | ~10–12h   |

Detaillierte Logs mit Befehlen, Konfigurationen, Troubleshooting und Lessons Learned: [`docs/`](https://github.com/patrick-mitschke/synapspace/blob/main/docs)

---

## Ausgewählte technische Entscheidungen

Jede Entscheidung ist bewusst getroffen und begründet dokumentiert. Vier Beispiele:

**Bare-Metal statt Proxmox:** GPU-Passthrough-Fehlerquellen im MVP-Zeitrahmen eliminiert. Stabilität vor Flexibilität.

**Coach in die Cloud (Mistral Vibe) statt lokalem Agenten:** Ursprünglich war DEX als lokaler Agent geplant – das hätte für ein brauchbares 32B-Modell ein teures GPU-Upgrade (24 GB VRAM) verlangt. Nach Klärung der Anforderungen (keine Proaktivität nötig, Cloud-Gedächtnis bei EU-Anbieter akzeptabel) fiel die Wahl auf Mistral Vibe: ein leistungsfähigeres Cloud-Modell, DSGVO-konform, ohne Hardware-Investition. Die Datenhoheit bleibt erhalten, weil das Lernprofil lokal liegt und nur über eine kontrollierte MCP-Brücke gelesen wird.

**GVS-Backup mit Zähler-Logik statt Datumslogik:** Der Backup-Node verfügt über kein BIOS-seitiges RTC-Wake – er muss manuell eingeschaltet werden. Wird eine Nacht vergessen, darf das nicht den gesamten GVS-Rhythmus zerstören. Zählerbasierte Logik (alle 7 Läufe = differenziell, alle 28 = Vollbackup) ist kalenderunabhängig und robust. `systemd`-Timer mit `Persistent=true` holt verpasste Jobs beim nächsten Boot automatisch nach.

**Keine LUKS-Festplattenverschlüsselung:** LUKS verschlüsselt die gesamte Festplatte und verlangt beim Booten eine Passphrase zum Entsperren. Auf einem Headless-Server, der unbeaufsichtigt läuft und automatisch neu startet, wäre das ein Single Point of Failure – der Server würde ohne manuelle Eingabe hängen bleiben. Bewusst dagegen entschieden, zugunsten eines stabilen Unattended-Boots.

Alle Entscheidungen: [`docs/PROJECT_DECISIONS_san.md`](https://github.com/patrick-mitschke/synapspace/blob/main/docs/PROJECT_DECISIONS_san.md)

---

## Sicherheitskonzept

- SSH ausschließlich via Ed25519-Key (Passwort-Login deaktiviert)
- UFW: Default-Deny, alle Service-Ports nur im Heimsubnetz erreichbar
- Externer Zugriff ausschließlich über WireGuard – keine offenen Portfreigaben, kein Exposed Host (UPnP deaktiviert)
- OpenClaw-Gateway nur auf Loopback (`127.0.0.1`) gebunden – nicht extern oder über IPv6 erreichbar
- Docker/UFW-iptables-Lücke explizit über die `DOCKER-USER`-Chain geschlossen
- PostgreSQL ohne Host-Port – nur containerintern erreichbar
- Secrets in `.env`-Dateien (chmod 600), getrennt vom versionierten Code; nie ins öffentliche Repo
- Urheberrechtlich geschützte Daten: strikt lokal in Qdrant, kein Cloud-Transfer
- Alle ins Cloud-Backup gehenden Privatdaten via Cryptomator verschlüsselt (verschlüsselt at rest)
- CVE-gepatchte OpenClaw-Version (2026.5.12), eingeschränkter Tool-Korridor, Telegram-Allowlist

---

## Was kommt als nächstes

**Phase 5 – Hybrid-System mit Mistral (in Arbeit):**

- Coach „DEX" in Mistral Vibe mit Persona, Lernprofil-Datenbank (PostgreSQL) lokal angelegt
- n8n als MCP-Server, MCP-Brücke zwischen Cloud-Coach und lokalem System (externe Härtung via Reverse-Proxy + TLS)

**Phase 6 – Lokales Wissens-Backend:**

- RAG-Pipeline: Fachliteratur → Qdrant, präzise Zitierung mit lokalem Modell
- Themen-spezialisierte Tutoren, Concept Map des Umschulungsstoffs, Glossar
- Deterministische n8n-Workflows: Material-Generierung, Anki-CSV-Export, Monitoring-/Backup-Alarme

**Post-MVP:**

- Optionale proaktive Erinnerungen über n8n
- Hardware-Upgrade-Pfad: 1 TB SSD (größere RAG-Basis) → 16 GB VRAM (Echtzeit-Sprache, 32B auf GPU) → 128 GB RAM (volle Multi-Agenten-Parallelität). Der Coach selbst braucht kein Upgrade – er läuft als Cloud-Dienst.

**Längerfristige Vision:**

- Physisches Companion-Device (Raspberry Pi + Kamera + Mikrofon + Lautsprecher): verbunden mit dem lokalen System, gibt Bewegungserinnerungen, erkennt ergonomische Probleme
- Smart-Glasses-Integration: Der Agent sieht live, woran gearbeitet wird – vollständig lokal

---

## Über mich

Ich bin Patrick Mitschke, Umschüler zum Fachinformatiker Systemintegration (Abschluss Sommer 2027). Vor dieser Umschulung habe ich in völlig anderen Bereichen gearbeitet – SynapSpace ist mein erstes eigenes IT-Projekt. Es ist aus dem Antrieb entstanden, Dinge nicht nur zu nutzen, sondern von Grund auf zu verstehen und selbst zu gestalten.

Ich lerne am besten autodidaktisch: durch reale Projekte, eigenverantwortliches Arbeiten und den Freiraum, meinen eigenen Weg zu gehen. SynapSpace ist der Beweis, dass das funktioniert.

Mein Fokusthema: Generative KI, agentische Systeme und KI-Infrastruktur.

**KI-Kompetenzprofil und -Anwendungsdokumentation:** [patrick-mitschke.github.io/ki-portfolio](https://patrick-mitschke.github.io/ki-portfolio)
Dort dokumentiere ich, wie ich KI-Tools einsetze – vor allem während der Entwicklung von SynapSpace. Tool-Orchestrierung, fortgeschrittenes Prompt-Engineering, Workflows, modellübergreifende Kommunikation zwischen Chats (mit mir als Postbote) und weitere Techniken. Coding-Tools habe ich bewusst ausgelassen, damit ich bei der Entwicklung selbst dazulerne.

---

*Dokumentation, Configs und Phasen-Logs wachsen mit dem Projekt. Stand: Juni 2026.*
