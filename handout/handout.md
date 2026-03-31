# Autonomous Agents Workshop
### OpenClaw & ZeroClaw — Hands-On für Entwickler

> Workshop by Oskar · execute008 · 2026

---

## 01 — Was ist ein autonomer Agent?

Ein **autonomer Agent** ist ein KI-System, das nicht nur antwortet — es **handelt**. Es kann Werkzeuge verwenden, Entscheidungen treffen und proaktiv auf seine Umgebung reagieren — auch ohne User-Input.

### Chatbot vs. Agent

| | Klassischer Chatbot | Autonomer Agent |
|---|---|---|
| Interaktion | Frage → Antwort | Ziel → Plan → Ausführung |
| Gedächtnis | Keins | Persistent (Dateien / SQLite) |
| Systemzugriff | Nein | Shell, Browser, APIs, Dateien |
| Proaktivität | Reaktiv | Aktiv — auch ohne User |

### Die Agent-Schleife

```
👁 Wahrnehmen → 🧠 Denken → 🔧 Handeln (Tools) → 💾 Merken → 🔄 Wiederholen
```

### Tools (Beispiele)

- `web_search` / `web_fetch` — Internet-Recherche in Echtzeit
- `exec` / `shell` — Befehle auf dem Host ausführen
- `read` / `write` / `edit` — Dateisystem-Operationen
- `browser` — Echten Browser steuern (Playwright)
- `message` — Telegram, Discord, Slack senden
- `memory_search` — Semantische Suche im Gedächtnis

### Memory-System

Agenten sind zustandslos — Memory-Dateien sind das Persistenz-Layer:

- `MEMORY.md` — Langzeitgedächtnis, semantisch durchsuchbar
- `memory/YYYY-MM-DD.md` — Tagesnotizen, rohe Logs
- `SOUL.md` — Persönlichkeit & Werte des Agenten
- `USER.md` — Kontext über den Menschen
- `HEARTBEAT.md` — Periodische Aufgaben-Checkliste

> **Key Insight:** LLM = Gehirn · Tools = Hände · Memory-Dateien = Langzeitgedächtnis

---

## 02 — OpenClaw vs ZeroClaw

Beide sind **vollständige Agent-Runtimes** — der Unterschied liegt nicht im Deployment-Ort, sondern im **Technologie-Stack**. Beide können lokal *und* in der Cloud laufen.

> ⚠️ OpenClaw läuft nicht nur in der Cloud, ZeroClaw nicht nur lokal. **Beide können beides.**

| Feature | 🐾 OpenClaw (Node.js) | 🦀 ZeroClaw (Rust) |
|---|---|---|
| Stack | Node.js / TypeScript | Rust — 3MB Binary, kein Runtime |
| Deployment | Lokal + Server + Cloud | Lokal + Server + Cloud + Android |
| AI Provider | ~10 (Anthropic, OpenAI, Ollama…) | **65 Provider** — Ollama, LM Studio, vLLM, DeepSeek… |
| 100% lokal | ✅ mit Ollama | ✅ Ollama / LM Studio / llama.cpp |
| Channels | Telegram, Discord, Signal, WhatsApp, Slack, iMessage… | 20+ inkl. Nostr, Matrix, IRC, DingTalk, Lark, Reddit, Bluesky |
| Memory | Markdown + semantische Suche | SQLite (auto-save) + Markdown |
| Skills | ClawHub Marketplace | Built-in + manuell |
| Multi-Agent | ✅ MQTT, Gateway, subagents, ACP | ✅ Daemon + WebSocket Gateway |
| Hardware | — | ✅ STM32, Raspberry Pi GPIO, USB |
| Safety | Tool policies, approval flows | E-Stop, OTP, max actions/hour, cost/day |
| Footprint | Node.js + npm | 3MB Binary — kein Node, kein Python |

### Wann was?

**OpenClaw** wenn: JS/TS-Ökosystem, ClawHub Skills, MQTT Multi-Agent, Telegram/Discord, ACP Coding-Agents

**ZeroClaw** wenn: Minimaler Footprint, Hardware (GPIO/STM32), 65 Provider, Niche Channels (Nostr/Matrix/IRC), Migration von OpenClaw

---

## 03 — ZeroClaw Deep Dive

### Installation

```bash
# Interaktiv — guided setup
curl -fsSL https://zeroclawlabs.ai/install.sh | bash

# Lokal ohne API Key (Ollama)
./install.sh --provider ollama --prefer-prebuilt

# Mit Anthropic
./install.sh --api-key "sk-ant-..." --provider anthropic
```

### Config (`~/.zeroclaw/config.toml`)

```toml
# Lokal — komplett offline
default_provider = "ollama"
default_model    = "qwen3:8b"

# Cloud
default_provider = "anthropic"
api_key          = "sk-ant-..."
default_model    = "claude-opus-4"
```

### Daemon + Service

```bash
zeroclaw daemon          # Gateway + Channels + Heartbeat in einem Prozess
zeroclaw service install # Als systemd User Service registrieren
zeroclaw status
zeroclaw doctor
```

### Safety Controls

- **E-Stop** — Notfall-Stopp
- **OTP** — Einmal-Passwort für sensitive Actions
- `max_actions_per_hour` — Rate limit
- `max_cost_per_day` — Budget limit ($)
- `allowed_commands` — Shell-Whitelist
- `workspace_only` — Dateisystem-Sandbox

### Hardware (einzigartig — OpenClaw hat das nicht)

STM32 · Raspberry Pi GPIO · USB device discovery → IoT-Agents möglich

```bash
zeroclaw hardware    # scan
zeroclaw peripheral  # manage
```

### Migration von OpenClaw

```bash
zeroclaw migrate openclaw
# Importiert MEMORY.md, SOUL.md, USER.md und alle .md Dateien
```

---

## 04 — Skills & Multi-Agent

### Skills — Modulare Erweiterungen

Skills sind Ordner mit einer `SKILL.md` — kein Code-Deploy, nur Text-Instruktionen.

```
my-skill/
├── SKILL.md       # Trigger-Bedingungen + Anleitung
├── references/    # API-Specs, Docs
└── scripts/       # Hilfs-Skripte
```

**OpenClaw:** `npx clawhub install weather`
**ZeroClaw:** `zeroclaw skills install`

Beispiele: `weather`, `notion`, `coding-agent`, `openai-whisper-api`, `healthcheck`

### Multi-Agent Kommunikation

**MQTT (OpenClaw)**
- Broker: Mosquitto (Podman/Docker)
- Topics: `hive/agent/inbox`
- Push-basiert, event-driven
- Agents auf verschiedenen Maschinen

**WebSocket Gateway (ZeroClaw)**
- Daemon öffnet HTTP + WS Server
- `ws://host:port/ws/chat`
- REST API für externe Trigger

### Design-Prinzipien

- **Separation of concerns** — Jeder Agent hat eine klare Rolle
- **Skepticism** — Agents vertrauen sich nicht blind
- **Async by default** — Push statt Polling
- **Human-in-the-loop** — Für destructive Actions immer Bestätigung

---

## 05 — Quickstart & Ressourcen

### OpenClaw

```bash
npm install -g openclaw
openclaw configure     # Provider, API Key, Channels
openclaw gateway start
openclaw status
```

Voraussetzungen: Node.js ≥ 18 · API Key · optional: Telegram Bot Token

### ZeroClaw

```bash
curl -fsSL https://zeroclawlabs.ai/install.sh | bash
zeroclaw daemon
zeroclaw status
```

Voraussetzungen: Nichts (Rust wird mitinstalliert). Für lokale Modelle: Ollama.

### Workspace-Struktur (beide)

```
~/.openclaw/workspace/  OR  ~/.zeroclaw/workspace/
├── SOUL.md        # Persönlichkeit des Agenten
├── USER.md        # Kontext über den Menschen
├── MEMORY.md      # Langzeitgedächtnis (semantisch durchsuchbar)
├── HEARTBEAT.md   # Periodische Aufgaben-Checkliste
├── AGENTS.md      # Workspace-Konventionen
└── memory/YYYY-MM-DD.md
```

### Links

| | |
|---|---|
| OpenClaw Docs | docs.openclaw.ai |
| ZeroClaw | zeroclawlabs.ai |
| Skill Registry | clawhub.com |
| Community Discord | discord.com/invite/clawd |
| Workshop Repo | github.com/execute008/autonomous-agents-workshop |
| ZeroClaw npm | npmjs.com/package/zeroclaw |

---

> **Take-Away:** OpenClaw und ZeroClaw lösen dasselbe Problem auf zwei Arten — Node.js mit ClawHub und ACP vs. Rust mit 65 Providern und Hardware-Support. Beide lokal oder Cloud. Beide mit Ollama für komplette Datenprivacy.
>
> *The real shift: you give the LLM hands, memory, and a persistent identity. That's when it stops being a chatbot and starts being an agent.*
