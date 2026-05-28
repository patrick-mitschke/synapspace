# Dokumentation – SynapSpace

Dieser Ordner enthält die vollständigen Deployment-Logs des SynapSpace-Projekts, aufgeteilt nach Projektphasen.

Jede Phasen-Datei dokumentiert:
- Durchgeführte Schritte mit Status
- Konfigurationen und verwendete Befehle
- Troubleshooting und Lösungswege
- Lessons Learned

Sensible Infrastruktur-Details (IP-Adressen, Subnetze) sind in allen Dateien durch Platzhalter ersetzt (`<SERVER_IP>`, `<BACKUP_IP>`, `<HOME_SUBNET>` etc.).

---

## Phasen-Übersicht

| Datei | Inhalt | Status |
|-------|--------|--------|
| [PHASE_0_san.md](./PHASE_0_san.md) | Hardware-Inventur, RAM-Upgrade, Memtest, ISO-Setup | ✅ Abgeschlossen |
| [PHASE_1_san.md](./PHASE_1_san.md) | Ubuntu Server, LVM, NVIDIA-Driver, Docker, UFW, SSH | ✅ Abgeschlossen |
| [PHASE_2_san.md](./PHASE_2_san.md) | Laptop-Wartung, Netzwerk, Backup-Node, NFS, rsync/GVS, WireGuard, etckeeper | ✅ Abgeschlossen |
| [PHASE_3_san.md](./PHASE_3_san.md) | Container-Stack, LLM-Modelle, DEX/OpenClaw | ✅ Abgeschlossen |
| [PHASE_4_san.md](./PHASE_4_san.md) | GPU-Integration, Netdata Monitoring, Modelfiles | ✅ Abgeschlossen |
| PHASE_5_san.md | RAG & n8n Workflows | ⏳ Ausstehend |
| PHASE_6_san.md | Hybrid-Integration Cloud | ⏳ Ausstehend |
| [PROJECT_DECISIONS_san.md](./PROJECT_DECISIONS_san.md) | Alle technischen Entscheidungen mit Begründungen | 📋 Laufend |

---

← Zurück zur [Projekt-Übersicht](../README.md)
