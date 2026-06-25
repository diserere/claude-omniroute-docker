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
- Provider keys are configured in the OmniRoute web dashboard — Claude Code only needs the base URL

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

If you want local overrides (not committed to git):

```bash
cp omniroute/settings.local.json.example .claude/settings.local.json
```

### 5. Run Claude Code

```bash
cd your-project
claude
```

Claude Code now routes through OmniRoute. The model is set to `auto` by default — OmniRoute picks the best available provider for each request.

## Switching to a specific model

In `.claude/settings.json`, change `ANTHROPIC_MODEL`:

```json
"ANTHROPIC_MODEL": "auto/cheap"          // cheapest viable provider
"ANTHROPIC_MODEL": "auto/coding"        // quality-first for coding
"ANTHROPIC_MODEL": "auto/fast"          // lowest latency
"ANTHROPIC_MODEL": "deepseek/deepseek-chat"  // specific provider/model
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
