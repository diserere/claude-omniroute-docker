# Claude Code + OmniRoute (Docker)

Run [OmniRoute](https://github.com/diegosouzapw/OmniRoute) — a free AI gateway with 227 providers and 50+ free tiers — in Docker, and connect Claude Code to it through one Anthropic-compatible endpoint.

## Architecture

```
┌─────────────────┐     http://localhost:20128/v1     ┌──────────────────┐
│   Claude Code   │ ─────────────────────────────────▶│   OmniRoute      │
│   (local CLI)   │                                   │   (Docker)       │
└─────────────────┘                                   └────────┬─────────┘
                                                               │
                                                ┌──────────────┼──────────────┐
                                                ▼              ▼              ▼
                                           Free tiers     API keys      Subscriptions
                                           (50+ free)    (OpenRouter,  (Claude Code,
                                                            DeepSeek...)   Codex...)
```

- **Claude Code** runs locally on your machine
- **OmniRoute** runs in Docker, proxies requests to the best available provider
- Provider API keys are configured in the OmniRoute web dashboard
- **You need an OmniRoute API key** (generated in dashboard → Settings → API Keys) for Claude Code authentication

## Quick Start

### 1. Configure environment

```bash
cd omniroute
cp .env.example .env
```

Edit `.env` and fill in the required secrets:

```bash
# Generate secrets:
#   JWT_SECRET:        openssl rand -base64 48
#   API_KEY_SECRET:     openssl rand -hex 32
JWT_SECRET=your-jwt-secret-here
API_KEY_SECRET=your-api-key-secret-here
INITIAL_PASSWORD=a-strong-admin-password
```

### 2. Start OmniRoute

```bash
docker compose up -d
```

Check health:

```bash
curl http://localhost:20128/status
```

### 3. Configure providers

Open [http://localhost:20128](http://localhost:20128) in your browser.

Log in with the `INITIAL_PASSWORD` you set, then add your provider API keys (OpenRouter, DeepSeek, etc.) or use the 50+ free-tier providers that need no key.

### 4. Connect Claude Code

Copy the settings template to your project's `.claude/` directory:

```bash
# From the repo root:
cp omniroute/settings.json.example .claude/settings.json
```

Generate an OmniRoute API key in the dashboard (Settings → API Keys), then create local overrides:

```bash
cp omniroute/settings.local.json.example .claude/settings.local.json
# Edit .claude/settings.local.json and paste your API key into ANTHROPIC_AUTH_TOKEN
```

### 5. Run Claude Code

```bash
cd your-project
claude
```

Claude Code now routes through OmniRoute. The default model is `kr/minimax-m2.5` — change it in `.claude/settings.json` if needed.

## Switching to a specific model

In `.claude/settings.json`, change `ANTHROPIC_MODEL`. Find available models in OmniRoute dashboard → Models:

```json
"ANTHROPIC_MODEL": "kr/minimax-m2.5"      // default: MiniMax model via Kiro
"ANTHROPIC_MODEL": "kiro/glm-5"           // GLM-5 via Kiro
"ANTHROPIC_MODEL": "kiro/qwen3-coder-next" // Qwen3 Coder Next via Kiro
```

See [OmniRoute docs](https://github.com/diegosouzapw/OmniRoute) for the full model routing guide.

## Stopping

```bash
cd omniroute
docker compose down
```

Data persists in `omniroute/data/` across restarts.

## Updating

```bash
cd omniroute
docker compose pull
docker compose up -d
```

## Files

| File | Purpose |
|------|---------|
| `omniroute/docker-compose.yml` | OmniRoute + Redis services |
| `omniroute/.env.example` | Environment variable template |
| `omniroute/settings.json.example` | Claude Code project settings |
| `omniroute/settings.local.json.example` | Claude Code local overrides |
