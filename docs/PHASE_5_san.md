# PHASE_5 – Hybrid-System mit Mistral (Coach-Layer + MCP-Brücke)
**Status: ✅ COMPLETE** — Hybrid-Brücke end-to-end verifiziert, **beidseitig** (Lesen + Schreiben). FritzBox-51443 geschlossen → Heim rein ausgehend.

---

## Konfigurationen

### Architektur (Hybrid-Brücke, live)
- **Pfad:** Mistral (Coach „DEX") → **EU-VPS:443 (Caddy)** → **NetBird-Mesh** → **Heim-n8n:5678** → PostgreSQL.
- **Einziger öffentlicher Eingang = EU-VPS** (Caddy:443, nur `/mcp` mit Bearer). Das Heim erreicht den VPS rein **ausgehend** über das NetBird-Mesh; FritzBox-Freigaben geschlossen.
- **Rollenmodell:** DEX = Cloud-Coach (empfiehlt/routet, erklärt keine Inhalte, schreibt nie direkt in die DB). **n8n = einziger Schreib-Gateway.** Tutoren (eigene Mistral-Projekte) + lokale Produktion (Ollama/Qdrant) = Phase 6. **Lokale DB = SSoT** (DEX liest per MCP).

### Coach „DEX" (Mistral Vibe, Cloud)
- Projekt **„DEX"** in **Vibe Work**; Persona als **Projekt-Custom-Instructions** (Datei `DEX_Persona_CustomInstructions.md`) → jeder Chat ist automatisch DEX, kein Prompt-Einfügen.
- **Einstellungen:** Projektton **Standard**; Checkbox **„globale Anweisungen einbeziehen" = aus** (Kontext sauber halten).
- **Persona-Kern:** Lerncoach (kein Tutor), Deutsch/du, sachlich-direkt, leicht trockener Humor, Lob nur überdurchschnittlich; IT-Wording nur prüfungsrelevant auf Nutzer-Niveau (unbekanntes Wort → nicht erklären, routen); **Anti-Halluzinations-Mandat** (Lücken nie raten → einfordern); **empfiehlt, entscheidet nicht** (Lernautonomie beim Nutzer).
- Mechanik: Custom Instructions für Persona/Dialog; **Skills** = aufrufbare Automatisierung, nicht für laufenden Dialog. Mistral teilt Gesprächsinhalt projektweit über Chats — Komfort, **kein Fundament** (Legacy/intransparent).

### Lernprofil-DB (PostgreSQL + Adminer)
Stack-Erweiterung `/opt/synapspace/docker-compose.yml`:
- **postgres** `postgres:16-alpine`, `container_name: postgres`, **kein Host-Port** (nur Netz `internal-ai` → DB nur containerintern), Volume `postgres_data:/var/lib/postgresql/data`, Healthcheck `pg_isready -U $POSTGRES_USER -d $POSTGRES_DB`.
- **adminer** `adminer:4.8.1`, Port `<SERVER_IP>:8081->8080` (LAN-only), `ADMINER_DEFAULT_SERVER=postgres`, `depends_on: postgres (service_healthy)`.
- **`.env`** (`/opt/synapspace/.env`, **chmod 600**): `POSTGRES_USER=<DB_USER>`, `POSTGRES_PASSWORD` (`openssl rand -hex 24`), `POSTGRES_DB=synapspace_lernprofil`; Compose referenziert über `${VAR}`.
- **UFW:** `8081/tcp` ALLOW `<HOME_SUBNET>`. PostgreSQL braucht keine Regel (kein Host-Port).
- Schema-Einspielung: SFTP → Server, `docker exec -i postgres psql -U <DB_USER> -d synapspace_lernprofil < lernprofil_schema.sql` (kein DB-Port nach außen).

**Schema `lernprofil_schema.sql` — 11 Tabellen** (Skelett + Beispielzeilen; echte Inhalte = Phase 6):
- *Stammdaten/Kataloge:* `nutzer` (Mehrbenutzer-Platzhalter, `nutzer_id` überall FK) · `themen` (Hierarchie via Selbstreferenz `parent_id`, `wissensart_id` FK, ROI-Felder `roi_aufwand`/`roi_punkte`/`roi_wahrscheinlichkeit` **nullable**) · `wissensarten` (PACER als Platzhalter, finale Taxonomie via Recherche) · `methoden`.
- *themenbezogen:* `lernstand` (Verständnisgrad/Status/`zuletzt_geuebt`/`naechster_termin`) · `thema_methode` (n:m, Wirksamkeit objektiv/subjektiv + `status`, Wirksamkeit **nullable**).
- *personenbezogen:* `profil` (1 Zeile/Nutzer) · `methoden_verinnerlichung` (Verinnerlichungsgrad + `eingefuehrt_am`) · `gewohnheiten` (Tagesprotokoll, bewusst **kein** Streak-Druck) · `feedback` (CHECK: `typ ∈ {workflow_ergebnis, fehler, qualitaetsbewertung}`).
- *Methoden-Eignung:* `methode_wissensart` (n:m, `eignung` gut/neutral/ungeeignet) → **Methodenpool je Thema = Abfrage** (Wissensart des Themas ⋈ Eignung der Methode), keine manuelle Pflege je Subthema.
- UNIQUE auf Katalog-Schlüsseln + sinnvollen Kombinationen (ein Profil/Nutzer, ein Lernstand/Nutzer+Thema).

### n8n als MCP-Server — „DEX MCP Bridge (MVP)" (Tür B)
- **Tür B = „MCP Server Trigger"-Node** (GA-stabil, semantisch). Tür A (Instance-level MCP, Beta) = Fallback.
- **MCP-Server-Trigger:** interner Pfad `http://<SERVER_IP>:5678/mcp/<MCP_PATH_ID>`; **Authentication: Bearer Auth**, Token = `openssl rand -hex 32` (Passwortmanager). Spricht SSE **und** Streamable HTTP. **Path-Feld bleibt die ID** (Token NICHT dort eintragen). Fehlender/falscher Token → **403**.
- **Postgres-Credential in n8n:** Host `postgres`, Port 5432, DB `synapspace_lernprofil`, User `<DB_USER>`, SSL **Disable** (containerinternes Netz).
- **Tool 1 `lernstand_lesen`** (Postgres Tool, *Execute Query*):
  ```sql
  SELECT t.titel AS thema, l.verstaendnisgrad, l.status, l.zuletzt_geuebt, l.naechster_termin
  FROM lernstand l JOIN themen t ON t.id = l.thema_id WHERE t.titel ILIKE $1;
  ```
  Query-Parameter (**Expression**): `{{ $fromAI('thema', 'Titel des Themas …', 'string') }}`.
- **Tool 2 `logbuch_schreiben`** (Postgres Tool, *Insert*, Tabelle `feedback`): `nutzer_id = 1` (fest); `typ = {{ $fromAI('typ','Art des Eintrags. Erlaubt: workflow_ergebnis, fehler, qualitaetsbewertung','string') }}`; `inhalt = {{ $fromAI('inhalt','…','string') }}`; `zeitstempel = {{ $now }}`. **`id` (GENERATED ALWAYS) und `thema_id` NICHT zuordnen.**
- „Save executions" **aktiv** (MCP-Tool-Calls erscheinen nur in der Execution-Liste, nicht im Editor = objektiver Beleg).

### EU-VPS-Vermittler (öffentlicher 443-Eingang)
**VPS-Härtung** — Hetzner **CX23** (x86, 2 vCPU/4 GB/40 GB), DE, Ubuntu 24.04, Hostname `synapspace-vps`, IPv4 `<VPS_IP>`, NetBird-IP `<MESH_IP_VPS>`:
- **SSH key-only:** `/etc/ssh/sshd_config.d/99-hardening.conf` → `PasswordAuthentication no`, `PermitRootLogin prohibit-password`, `KbdInteractiveAuthentication no`. Key `synapse-base-admin`.
- **UFW:** default deny in / allow out; offen `22`, `80/tcp`, `443/tcp`, `51820/udp` (NetBird-WireGuard).
- **fail2ban:** `/etc/fail2ban/jail.local` `[sshd]` `backend=systemd`, `maxretry=5`, `bantime=1h`, `findtime=10m`.
- **unattended-upgrades** aktiv.

**NetBird-Mesh** (NetBird Cloud, Client 0.72.4, Interface `wt0`, WireGuard-Port 51820):
- Peers: VPS `synapspace-vps` `<MESH_IP_VPS>` · Heimserver `synapspace` `<MESH_IP_HEIM>`. Direkte P2P-Verbindung (~16 ms, kein Relay).
- Install je Peer: `curl -fsSL https://pkgs.netbird.io/install.sh | sh` + `netbird up` (Aktivierungs-URL im Browser bestätigen).
- **Login-Expiration pro Server-Peer ausgeschaltet** (sonst Mesh-Ausfall alle 24 h).

**„Networks"-Route (Weg A, ersetzt abgekündigte „Network Routes"):**
- **Network** `heim-lan` · **Resource** `n8n-host` = `<SERVER_IP>/32` in **Gruppe** `heim-n8n` · **Routing Peer** `synapspace`, **Masquerade an**, Metric 9999.
- **Policy** `all-zu-n8n`: Source Gruppe `All` → Destination Gruppe `heim-n8n`, TCP 5678. (Quelle später auf enge `vps`-Gruppe eindampfen — offen.)
- Trägt **ohne** neue Firewall-Regel: Masquerade schreibt die Quelle auf die LAN-IP `<SERVER_IP>` um, die DOCKER-USER bereits erlaubt. Weg B (n8n auf NetBird-IP veröffentlichen) unnötig.

**Caddy als 443-Reverse-Proxy** (Apt-Repo cloudsmith), Domain `vps.<EXTERNAL_DOMAIN>` (A-Record → VPS-IP), **TLS via HTTP-01** (Port 80 offen, kein DNS-01-Token nötig). `/etc/caddy/Caddyfile`:
```
vps.<EXTERNAL_DOMAIN> {
	@mcp_noauth { path /mcp*; not header Authorization * }
	handle @mcp_noauth { header WWW-Authenticate Bearer; respond 401 }
	@mcp_auth { path /mcp*; header Authorization * }
	handle @mcp_auth { reverse_proxy <SERVER_IP>:5678 { flush_interval -1 } }
	handle { respond 403 }
}
```
- `/mcp` ohne Token → **401 + `WWW-Authenticate: Bearer`** (Mistrals Auto-Detect braucht 401; n8ns 403 würde als Sperre gewertet). Mit Token → reverse_proxy an n8n (`flush_interval -1` hält den SSE-Stream offen). Sonst → 403.
- Anwenden: `caddy validate --config /etc/caddy/Caddyfile && systemctl reload caddy`.

**Mistral-Connector** (Web-GUI, *Connectors → Add → Custom MCP Connector*):
- **Name** `synapspace-mcp` (eindeutig, ohne Leer-/Sonderzeichen).
- **Server-URL** `https://vps.<EXTERNAL_DOMAIN>/mcp/<MCP_PATH_ID>`.
- **Auth:** Header `Authorization`, Typ Bearer, Token aus Passwortmanager.
- ⚠️ **Pro Chat anhängen** (Plus-Button) — kein dauerhaftes Anhängen auf Projektebene auffindbar.

### Heim-seitige Härtung — Schritt 6 (im Live-Pfad durch den VPS ersetzt; Configs am VPS wiederverwendbar)
> Der frühere externe Heim-Eingang (FritzBox 51443 → NPM:443) ist **geschlossen**. Caddy am VPS übernimmt die Türsteher-Logik. Folgende Configs bleiben als wiederverwendbares Muster / für LAN-/Mesh-interne Nutzung dokumentiert.
- **deSEC/DynDNS:** Domain `<EXTERNAL_DOMAIN>`; FritzBox-DynDNS „Benutzerdefiniert" Update-URL `https://update.dedyn.io/?myipv4=<ipaddr>` (nur IPv4), Benutzer = Domain, Kennwort = deSEC-Token. **AAAA-Record entfernt** (sonst validiert LE bevorzugt über IPv6).
- **Zertifikat (DNS-01 in NPM):** *Let's Encrypt via DNS*, Domain `<EXTERNAL_DOMAIN>`, Key ECDSA 256, Provider deSEC. Credentials-File:
  ```
  dns_desec_token = <DESEC_API_TOKEN>
  dns_desec_endpoint = https://desec.io/api/v1/
  ```
  **Propagation Seconds: 180** (60 s → „secondary validation: NXDOMAIN"). deSEC-Token plugin-bedingt als Klartext im `npm_data`-Volume → Master im Passwortmanager.
- **NPM file-based Custom-Config** (GUI-„Custom Location" killt in NPM 2.15.1 still die Host-Conf → file-based statt GUI). `/data/nginx/custom/server_proxy.conf`:
  ```
  location /mcp {
      proxy_pass http://n8n:5678;
      proxy_http_version 1.1;
      proxy_set_header Host $host;
      proxy_set_header Connection "";
      proxy_buffering off; proxy_cache off; gzip off;
      proxy_read_timeout 3600; proxy_send_timeout 3600;
  }
  location ~ ^/(?!mcp(/|$)) { return 403; }
  ```
  Anwenden: `docker exec npm nginx -t && docker exec npm nginx -s reload`.
- **DOCKER-USER (Docker-FORWARD-Ebene)** — UFW-verwaltet in `/etc/ufw/after.rules` (SSoT, kein `iptables-persistent`). UFW-INPUT greift **nicht** für durchgereichte Container-Pakete. Freigabe **vor** dem NEW-DROP, an den **Port** gebunden:
  ```
  -A DOCKER-USER -p tcp --dport 443 -m conntrack --ctstate NEW -j RETURN
  ```
  Aktiv via `sudo ufw reload` (Backup `after.rules.bak`).
- **FritzBox:** 51443→443-Freigabe **gelöscht** (Heim rein ausgehend). WireGuard-Lifeline (Reiter „VPN (WireGuard)") unberührt.

### Vertraulichkeit / Secrets
- **Copyright-Fachliteratur:** strikt lokal (Qdrant, Phase 6), „Secret" — **nie** Cloud. **Lernprofil:** lokal (PostgreSQL), per MCP gelesen, über n8n geschrieben (darf in EU-Cloud, liegt lokal aus Praktikabilität).
- **Secrets:** DB-Passwort in `.env` (chmod 600); MCP-Bearer-Token + deSEC-Token im Passwortmanager (deSEC zusätzlich Klartext im `npm_data`-Volume). Alle **nie ins Repo/Screenshot**; `docker compose config` löst `.env` im Klartext auf → solche Ausgaben nicht teilen.
- **No-Telemetry** in Mistral aktiv; EU-Datenresidenz.

---

## Lessons Learned

**Netzwerk / NetBird**
- ⚠️ **NetBird-Modell:** Eine Network Route erreicht das Netz **hinter** dem Routing-Peer, nicht den Peer selbst (ein Dienst **auf** dem Peer ist Input-Chain → bräuchte Peer-to-Peer-Policy). **Masquerade** muss an bleiben (sonst keine ACL-Gruppen), greift aber erst **nach** Dockers FORWARD-Kette. **Korrektur per Live-Test:** für n8n in Docker trägt Weg A **ohne** neue DOCKER-USER-Regel, weil Masquerade die Quelle auf die LAN-IP umschreibt, die DOCKER-USER schon erlaubt. **Eigene Messung schlägt Bericht-Schlussfolgerung.**
- ⚠️ **NetBird-Resource braucht Gruppe + Policy** (Resources sind nicht in `All`). Sauberste Policy = Gruppe → Gruppe; **kein** Resource-Bezug in der Regel. Fehler **422 „specify either sources or source resources, not both"** entsteht, wenn man die Policy im Access-Control-Reiter der Resource baut → Resource und Policy **getrennt** anlegen.
- ⚠️ **Login-Expiration** (Default 24 h) für 24/7-Server-Peers ausschalten, sonst täglicher Mesh-Ausfall. Re-Auth: `netbird up` → URL auf `login.netbird.io/activate` (nicht `docs.netbird.io`).
- ℹ️ Direkte P2P-Verbindung braucht eingehend **UDP 51820** offen, sonst (langsamerer) Relay-Fallback. „Networks" (v0.72) ersetzt abgekündigte „Network Routes".

**Firewall / Docker / Proxy**
- ⚠️ **DOCKER-USER (FORWARD)** ist UFW-verwaltet in `/etc/ufw/after.rules`; UFW-INPUT-Regeln greifen nicht für durchgereichte Container-Pakete → Freigabe in DOCKER-USER vor dem NEW-DROP, an den **Port** gebunden (nicht die wechselnde Container-IP).
- ⚠️ **NPM-GUI-„Custom Location" bricht in 2.15.1 still die Host-Conf-Erzeugung** (DB-Status bleibt trügerisch „online") → file-based `/data/nginx/custom/server_proxy.conf`, mit `nginx -t` validieren.
- ℹ️ **NPM lauscht auf der LAN-IP** (nicht `127.0.0.1`) → curl-Test mit `--resolve hostname:443:<SERVER_IP>` (Hostname für SNI), sonst `tlsv1 unrecognized name`. **SSE-Endpoint** hält offen → `--max-time`.

**TLS / Ports**
- ⚠️ **deSEC + DNS-01:** Propagation Seconds **≥ 180** (sonst NXDOMAIN durch LEs Mehr-Perspektiven-Prüfung); deSEC-Token braucht DNS-Schreibrechte. Cert via „Let's Encrypt via DNS" → kein Port 80 nötig.
- ℹ️ **Vodafone DSL** blockt eingehend nur 80/443; hohe Ports kommen durch. Öffentliche IPv4 echt (kein CGNAT). **FritzBox** sperrt 80/443 als externen Freigabe-Port (Warnung + stille Umbiegung auf hohen Port).
- ℹ️ **Hetzner CAX11 (ARM)** chronisch ausverkauft → x86-Äquivalent **CX23** (für Reverse-Proxy/Tunnelende egal, nimmt ARM-Fragen raus).
- ℹ️ **fail2ban auf Ubuntu 24.04:** `backend=systemd` nötig (SSH-Logs im Journal).

**MCP / Mistral / n8n**
- ⚠️ **Mistral Custom MCP Connector verbindet nur auf Standard-443** (A/B-belegt). „beliebige Ports" war Modell-Schlussfolgerung, keine Doku-Zusage. → daher der EU-VPS als 443-Vermittler.
- ⚠️ **Mistral-Auto-Detect erwartet 401 + `WWW-Authenticate`**; n8ns 403 wird als Sperre gewertet → Caddy liefert für token-loses `/mcp` ein 401. n8n-Bearer weist fehlenden/falschen Token mit **403** ab.
- ⚠️ **Mistral-MCP-Ausführung wackelig:** Tool wird erkannt, erster Aufruf kann scheitern (kein Execution-Eintrag). Fix: **frischer Chat + Connector neu anhängen + konkreter Parameter**. **Objektiver Beleg = n8n-Execution-Liste**, nicht DEX' Erzählung.
- ⚠️ **Persona-/SSoT-Falle:** bei Tool-Fehlschlag wich DEX auf ein Library-Dokument aus und gab es als DB-Daten aus → Persona muss bei Fehlschlag **melden**, nie aus Library/Gedächtnis substituieren. **Mistral-Libraries sind account-weit sichtbar** (Kontextvermischung).
- ℹ️ **n8n MCP:** Tür B = MCP-Server-Trigger (gewählt). Workflow-als-Tool: Tool-Name = Node-Name, Parameter via `$fromAI('key','beschreibung','typ')` im Expression-Modus, Tool Description = Fixed. **Insert-Tool** nimmt `id` automatisch mit → bei `GENERATED ALWAYS` entfernen. **CHECK-Constraints** vorab prüfen und erlaubte Werte in die `$fromAI`-Beschreibung schreiben. „Publish" = Speichern + Aktivieren. **„Save executions" aktivieren**, sonst kein Beleg für MCP-Tool-Calls.
- ℹ️ **Human-in-the-loop greift:** Schreibaktion löst in Vibe eine Freigabe aus. Reines Anhängen unkritisch („immer erlauben" ok), solange das Tool nichts ändert/löscht — bei künftigen Schreib-Tools vor dem Bestätigen prüfen.

**Plattform / Härtung**
- ℹ️ **DB ohne Host-Port** (nur internes Docker-Netz) = Härtung; Schema via `docker exec -i … psql` einspielen statt einen DB-Port zu öffnen. `nutzer_id` von Anfang an als FK (Mehrbenutzerfähigkeit als Gewohnheit). Später befüllte Felder (ROI, Wirksamkeit) **nullable** halten, sonst blockiert `NOT NULL` den Katalog-Import.
- ℹ️ **Persona-Mechanik:** Projekt-Custom-Instructions für „jeder Chat automatisch DEX"; Mistral-Chat-Gedächtnis ist Legacy/intransparent → lokale DB bleibt SSoT.

---

## Abgrenzung
- **Hybrid-Integration = Mistral** (diese Phase). Alte Idee „Cloud-Workflows über Gemini/NotebookLM/Perplexity" mit der Tool-Souveränität gestrichen.
- **In Phase 6 verschoben:** lokales RAG (Qdrant) + Material-/Monitoring-Workflows, echte Tutoren, Concept Map (`thema_beziehung`-Tabelle, Netz statt nur Baum), Glossar, Sandboxes, Nutzungspaket; Wissensart-Taxonomie (PACER) final + `methode_wissensart`-Matrix, **ROI-Werte je Thema**, **Themenkatalog-Import** in `themen`.
