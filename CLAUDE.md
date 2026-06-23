# Claude Code + OmniRoute (Docker)

This repo contains configuration for running [OmniRoute](https://github.com/diegosouzapw/OmniRoute) in Docker and connecting Claude Code to it as an Anthropic-compatible endpoint.

## What's here

```
omniroute/
├── docker-compose.yml              # OmniRoute + Redis services
├── .env.example                    # Environment variable template (secrets)
├── settings.json.example           # Claude Code project settings template
└── settings.local.json.example     # Claude Code local overrides template
```

## Quick start

```bash
cd omniroute
cp .env.example .env
# Fill in JWT_SECRET, API_KEY_SECRET, INITIAL_PASSWORD
docker compose up -d
# Open http://localhost:20128 → dashboard → configure providers
```

Then copy `omniroute/settings.json.example` → `.claude/settings.json` in any project where you want Claude Code to route through OmniRoute.

## Architecture

- **Claude Code** runs locally
- **OmniRoute** runs in Docker (port 20128, endpoint `/v1`)
- Provider keys are managed in the OmniRoute dashboard — Claude Code only needs the base URL
- Default model is `auto` — OmniRoute picks the best available provider with auto-fallback

## Key environment variables (in `.env`)

| Variable | Purpose | Generate with |
|----------|---------|---------------|
| `JWT_SECRET` | Dashboard session tokens | `openssl rand -base64 48` |
| `API_KEY_SECRET` | Encrypts stored API keys | `openssl rand -hex 32` |
| `INITIAL_PASSWORD` | First-boot admin password | pick a strong password |

## Claude Code settings

The template sets:
- `ANTHROPIC_BASE_URL=http://localhost:20128/v1`
- `ANTHROPIC_AUTH_TOKEN=dummy` (OmniRoute handles auth internally)
- `ANTHROPIC_MODEL=auto` (OmniRoute auto-routing)

To use a specific model, change `ANTHROPIC_MODEL` (e.g. `auto/cheap`, `auto/coding`, `deepseek/deepseek-chat`).
