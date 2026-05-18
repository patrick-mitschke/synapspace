# SynapSpace – Lokales KI-Lernökosystem

> **Dieses Repository befindet sich im aktiven Aufbau.**  
> Phasen 0–3 sind abgeschlossen und vollständig dokumentiert. Phasen 4–6 folgen in den nächsten Wochen. Die README gibt zunächst den vollständigen Überblick – detaillierte Logs, Konfigurationen und Entscheidungsdokumentation werden schrittweise in separaten Dateien unter `docs/` ausgelagert.

---

## Was ist SynapSpace?

SynapSpace ist ein selbst deployetes KI-Ökosystem, das auf einem lokalen Heimserver läuft und personalisiertes, datengeschütztes Lernen ermöglicht – ohne Abhängigkeit von kommerziellen Cloud-Diensten für sensible Inhalte.

Im Kern verbindet SynapSpace mehrere Open-Source-Komponenten zu einem funktionierenden Ganzen: **Ollama** stellt lokale Sprachmodelle bereit, **Qdrant** ermöglicht semantische Suche über urheberrechtlich geschützte Fachliteratur (RAG), **n8n** orchestriert automatisierte Lern-Workflows, **Open WebUI** bietet eine Sprachschnittstelle für tägliche Check-ins, und **DEX** (powered by OpenClaw) fungiert als agentischer Bildungs- und Karrierecoach, der proaktiv agiert – nicht nur reagiert. Ergänzt wird das System durch eine hybride Cloud-Anbindung (Gemini, NotebookLM, Perplexity, Claude) für Aufgaben, bei denen lokale Echtzeit-Inferenz noch nicht wirtschaftlich sinnvoll ist.

Das Ziel: Ein Lernsystem, das meinen aktuellen Wissensstand kennt, Lücken erkennt, Inhalte aufbereitet – und mich dabei auch dann erinnert und begleitet, wenn ich es gerade nicht aktiv nutze.

---

## Motivation & Ziele

**Kurzfristig – Praxisreife:** In 5 Monaten beginnt mein Berufspraktikum. Ziel ist konkrete Erfahrung mit Linux-Administration, Docker, Netzwerktopologien, Backup-Strategien und KI-Infrastruktur – nicht nur theoretisch, sondern durch echtes Deployment.

**Mittelfristig – IHK-Prüfungsvorbereitung:** Effektive, personalisierte Vorbereitung auf die Abschlussprüfung zum Fachinformatiker Systemintegration (Sommer 2027) – durch ein System, das meinen Lernstand kennt, Lücken erkennt und Inhalte gezielt aufbereitet.

**Langfristig – Ein System, das mitwächst:** SynapSpace ist modular erweiterbar und soll weit über die Prüfung hinaus als persönlicher Lernassistent dienen – für jedes Thema, jedes Berufsfeld, jeden Lebensabschnitt. Die Vision: Ein System, das seinen Nutzer über Jahre kennenlernt, mit ihm wächst und zunehmend proaktiv unterstützt.

**Als Portfolio:** Dieses Projekt dokumentiert, wo mein Interesse und meine Motivation liegen – an der Schnittstelle von KI, Infrastruktur und autodidaktischem Lernen. Es richtet sich an Betriebe, die in dieser Arbeitsweise und dieser Dokumentation erkennen, was mir wichtig ist – und die jemanden suchen, der mit Eigeninitiative und echtem Interesse an die Sache herangeht.

---

## Tech-Stack (MVP)

| Komponente | Technologie | Rolle |
|---|---|---|
| **Server** | Fujitsu Celsius M740b, Xeon E5-2620 v4, 64 GB ECC RAM | Zentrale Inferenz- & Orchestrierungseinheit |
| **GPU** | MSI Aero GTX 1070 8GB OC *(gekauft, Einbau ausstehend)* | GPU-Inferenz, Layer-Splitting |
| **OS** | Ubuntu Server 24.04 LTS (Bare-Metal) | Stabil, NVIDIA-kompatibel, 5 Jahre LTS |
| **Storage** | LVM (40 GB root / 404 GB Docker) | Flexible Größenanpassung, Snapshot-fähig |
| **Container** | Docker + Docker Compose | Single-Node-Orchestrierung |
| **Inferenz** | Ollama (GGUF, quantisiert) | Lokale LLMs – aktuell CPU-only, GPU nach Einbau |
| **Vektordatenbank** | Qdrant | RAG für urheberrechtlich geschützte Medien |
| **Agentik** | OpenClaw / DEX | Proaktiver Bildungscoach, Heartbeat-Workflows, Telegram |
| **Automatisierung** | n8n | Deterministisches Routing, ETL, Monitoring |
| **UI** | Open WebUI | Voice-Check-ins via Whisper |
| **Monitoring** | Netdata *(Phase 4, ausstehend)* | Echtzeit-Dashboard: CPU, RAM, Disk, GPU, Docker |
| **Backup** | rsync + GVS-Strategie (Großvater-Vater-Sohn) | Automatisiert, kalenderunabhängig |
| **VPN** | WireGuard via FritzBox 7530 | Sicherer Remote-Zugriff |
| **Firewall** | UFW | Whitelisting auf Subnetz-Ebene |

**LLM-Portfolio (lokal, ~97.5 GB):**

| Modell | Größe | Rolle |
|---|---|---|
| Llama-3.2-3B | 2.0 GB | Triage & Routing (DEX) |
| Hermes-3-Llama-3.1-8B | 4.7 GB | Tool-Calling (OpenClaw-Motor) |
| Qwen2.5-7B-Instruct | 4.7 GB | IT-Skripting, Linux-Administration |
| Mistral-Nemo-12B | 7.1 GB | Vorfilterung, 128k Kontext |
| Command-R (35B) | 18 GB | RAG-Spezialist, präzise Zitierung |
| Qwen2.5-32B-Instruct | 19 GB | Komplexe Logik, Mathematik |
| Llama-3.1-70B | 42 GB | Qualitätskontrolle (Senior-Endabnahme) |

---

## DEX – Dynamic Educational eXpander

DEX ist der agentische Kern von SynapSpace – powered by OpenClaw, verbunden via Telegram, läuft vollständig lokal auf dem Heimserver.

Was DEX kann, was andere Komponenten nicht können: **proaktiv agieren**. n8n führt Workflows aus wenn sie getriggert werden. Open WebUI wartet auf Eingabe. DEX hingegen kann eigenständig im Hintergrund arbeiten, Kontext aufbauen und von sich aus handeln – z.B. Lernstand überprüfen, Erinnerungen schicken oder Troubleshooting-Aufgaben in einer isolierten Sandbox erstellen.

**Aktuelle Einsatzszenarien:**
- Passiver Bildungs- und Karrierecoach via Telegram (MVP aktiv)
- Praxis-Trainer: erstellt eigenständig Troubleshooting-Aufgaben in isolierter Sandbox
- Didaktische Brücke: transformiert Alltagssituationen in IT-Projektideen

**Post-MVP:** Proaktive Erinnerungen via Heartbeat-Zyklus, autonome Lernplan-Überprüfung, Tool-Anbindung (Kalender, To-do).

**Sicherheit:** Sandbox-Modus aktiv, eingeschränkter Tool-Korridor, Gateway nur im VPN-Subnetz erreichbar, CVE-gepatchte Version, Telegram-Allowlist.

---

## Architektur-Übersicht

```
┌──────────────────────────────────────────────────────────┐
│                   SynapSpace Heimnetz                    │
│                                                          │
│  ┌─────────────────┐      ┌────────────────────────┐    │
│  │  Fujitsu        │      │  Backup-Node           │    │
│  │  Celsius M740b  │◄────►│  Acer TravelMate 5735  │    │
│  │  (Hauptserver)  │      │  rsync + NFS           │    │
│  │                 │      │  500 GB USB-HDD        │    │
│  │  Docker-Stack:  │      └────────────────────────┘    │
│  │  n8n            │                                     │
│  │  OpenClaw/DEX   │      ┌────────────────────────┐    │
│  │  Ollama         │◄────►│  Dev-Node              │    │
│  │  Qdrant         │      │  Acer TravelMate P216  │    │
│  │  Open WebUI     │      │  SSH/SFTP, Git, Claude │    │
│  │  Portainer      │      └────────────────────────┘    │
│  └─────────────────┘                                     │
│           │                                              │
│           │         ┌────────────────────────┐          │
│           └────────►│  FritzBox 7530         │          │
│                     │  WireGuard VPN Gateway │          │
│                     └───────────┬────────────┘          │
└─────────────────────────────────┼────────────────────────┘
                                  │ VPN-Tunnel
                     ┌────────────▼─────────────────────┐
                     │  Mobile Endgeräte                │
                     │  Samsung Z Flip 7 (Interface)    │
                     │  OnePlus 6 (Administration)      │
                     └──────────────────────────────────┘
```

**Hybrid-Ansatz:** Urheberrechtlich geschützte Daten bleiben ausschließlich lokal. Cloud-LLMs (Claude, Le Chat von Mistral) ergänzen das System für Aufgaben, die lokal aktuell noch nicht in Echtzeit wirtschaftlich sinnvoll lösbar sind. Prompts werden lokal via n8n generiert und manuell übergeben. Kein API-Schlüssel, kein Cloud-Transfer geschützter Inhalte.

---

## Projektphasen

| Phase | Inhalt | Status | Aufwand |
|---|---|---|---|
| **Phase 0** | Hardware-Inventur, RAM-Upgrade, Memtest, ISO-Setup | ✅ Abgeschlossen | ~6h |
| **Phase 1** | Ubuntu Server, LVM, NVIDIA-Driver, Docker, UFW, SSH | ✅ Abgeschlossen | ~5h |
| **Phase 2** | Laptop-Wartung, Netzwerk, Backup-Node, NFS, rsync/GVS, WireGuard, etckeeper | ✅ Abgeschlossen | ~12h |
| **Phase 3** | Container-Stack, LLM-Modelle, DEX/OpenClaw | ✅ Abgeschlossen | ~6h |
| **Phase 4** | GPU-Integration, Netdata Monitoring, Modelfiles | 🔄 In Arbeit | ~8–10h |
| **Phase 5** | RAG & n8n Workflows | ⏳ Ausstehend | ~8–10h |
| **Phase 6** | Hybrid-Integration Cloud | ⏳ Ausstehend | ~2–3h |

Detaillierte Logs mit Befehlen, Konfigurationen, Troubleshooting und Lessons Learned: [`docs/`](./docs/)

---

## Ausgewählte technische Entscheidungen

Jede Entscheidung ist bewusst getroffen und begründet dokumentiert. Drei Beispiele:

**Bare-Metal statt Proxmox:** GPU-Passthrough-Fehlerquellen im MVP-Zeitrahmen eliminiert. Stabilität vor Flexibilität.

**GVS-Backup mit Zähler-Logik statt Datumslogik:** Der Backup-Node verfügt über kein BIOS-seitiges RTC-Wake – er muss manuell eingeschaltet werden. Wird eine Nacht vergessen, darf das nicht den gesamten GVS-Rhythmus zerstören. Zählerbasierte Logik (alle 7 Läufe = differenziell, alle 28 = Vollbackup) ist kalenderunabhängig und robust. `systemd`-Timer mit `Persistent=true` holt verpasste Jobs beim nächsten Boot automatisch nach.

**Keine LUKS-Festplattenverschlüsselung:** LUKS verschlüsselt die gesamte Festplatte und verlangt beim Booten eine Passphrase zum Entsperren. Auf einem Headless-Server, der nachts automatisch startet und unbeaufsichtigt läuft, wäre das ein Single Point of Failure – der Server würde ohne manuelle Eingabe hängen bleiben. Bewusst dagegen entschieden zugunsten stabilem Unattended-Boot.

Alle Entscheidungen: [`docs/PROJECT_DECISIONS_san.md`](./docs/PROJECT_DECISIONS_san.md)

---

## Sicherheitskonzept

- SSH ausschließlich via Ed25519-Key (Passwort-Login deaktiviert)
- UFW: Default-Deny, alle Service-Ports nur im Heimsubnetz erreichbar
- WireGuard VPN für jeden externen Zugriff
- Docker-Netzwerk `internal-ai` ohne externe Internet-Exposition
- UFW/Docker-iptables-Lücke explizit über `DOCKER-USER`-Chain geschlossen
- Secrets ausschließlich in `.env`-Dateien (nie in Git)
- Urheberrechtlich geschützte Daten: Read-Only-Mounts, kein Cloud-Transfer
- **DEX/OpenClaw:** Sandbox-Modus, eingeschränkter Tool-Korridor, Gateway nur im VPN-Subnetz, DM-Allowlist, CVE-gepatchte Version (2026.5.12)

---

## Was kommt als nächstes

**Phase 4 – GPU-Integration (in Arbeit):**
- GTX 1070 8GB OC einbauen, Layer-Splitting verifizieren
- Netdata Echtzeit-Monitoring einrichten
- Modelfiles konfigurieren (Temperatur, System-Prompts, Kontextlänge)

**Phase 5 – RAG & Workflows:**
- n8n RAG-Workflows: Fachliteratur → Qdrant indexieren
- Anki-Export-Automatisierung, LVM-Monitoring-Alarm

**Phase 6 – Hybrid-Integration:**
- Cloud-Workflows einrichten, Prompt-Routing, State-Transfer

**Post-MVP:**
- DEX Heartbeat aktivieren: proaktive Lernbegleitung via Telegram
- Hardware-Upgrade: mehr VRAM und RAM für Echtzeitkommunikation

**Längerfristige Vision:**
- Physisches Companion-Device (Raspberry Pi + Kamera + Mikrofon + Lautsprecher): verbunden mit dem lokalen System, gibt Bewegungserinnerungen, erkennt ergonomische Probleme
- Smart-Glasses-Integration: Der Agent sieht live, woran gearbeitet wird – vollständig lokal

---

## Über mich

Ich bin Patrick Mitschke, Umschüler zum Fachinformatiker Systemintegration (Abschluss Sommer 2027). Vor dieser Umschulung habe ich in völlig anderen Bereichen gearbeitet – SynapSpace ist mein erstes eigenes IT-Projekt. Es ist aus dem Antrieb entstanden, Dinge nicht nur zu nutzen, sondern von Grund auf zu verstehen und selbst zu gestalten.

Ich lerne am besten autodidaktisch: durch reale Projekte, eigenverantwortliches Arbeiten und den Freiraum, meinen eigenen Weg zu gehen. SynapSpace ist der Beweis, dass das funktioniert.

Mein Fokusthema: Generative KI, agentische Systeme und KI-Infrastruktur.

**KI-Kompetenzprofil und -Anwendungsdokumentation:** [patrick-mitschke.github.io/ki-portfolio](https://patrick-mitschke.github.io/ki-portfolio)  
Dort dokumentiere ich, wie ich KI-Tools einsetze – vor allem während der Entwicklung von SynapSpace. Hier wird kaum was ausgelassen, was mit den großen Cloud-Modellen möglich ist. Tool-Orchestrierung, fortgeschrittenes Prompt-Engineering, Workflows, modellübergreifende Kommunikation zwischen Chats und mit mir als Postbote und andere Techniken und Strategien. Coding-Tools habe ich jedoch bewusst ausgelassen, damit ich bei der Entwicklung auch was lerne.

---

*Dokumentation, Configs und Phasen-Logs wachsen mit dem Projekt. Stand: Mai 2026.*
