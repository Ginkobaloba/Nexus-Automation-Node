# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Nexus Automation Node is the workflow orchestration layer of **Project Nexus** — a modular, biologically-inspired distributed cognition system. This repo deploys an **n8n** automation engine via Docker, fronted by Traefik reverse proxy and Cloudflare Tunnel for secure external access.

This is an infrastructure-as-code project with no application source code, build steps, or test suites. The primary development artifacts are Docker configuration and n8n workflow JSON files.

## Common Commands

```bash
# Start the full stack (Traefik + Cloudflared + n8n)
docker compose up -d

# Stop all services
docker compose down

# View logs
docker compose logs -f           # all services
docker compose logs -f n8n       # n8n only

# Restart a single service
docker compose restart n8n

# Backup n8n data (workflows + credentials)
docker exec n8n sh -c "tar czf /home/node/.n8n_backup.tar.gz /home/node/.n8n"
docker cp n8n:/home/node/.n8n_backup.tar.gz .

# Restore n8n data
docker cp .n8n_backup.tar.gz n8n:/home/node
docker exec n8n tar xzf /home/node/.n8n_backup.tar.gz -C /
```

## Architecture

Three Docker services on a shared `nexus` bridge network:

- **Traefik v2.11** — Reverse proxy. Routes `n8n.projectnexuscode.org` to the n8n container (port 5678). Config split between static (`traefik/traefik.yml`) and dynamic (`traefik/dynamic/config.yml`) with hot-reload. Docker provider discovers services via labels; `exposedByDefault: false` requires explicit opt-in.
- **Cloudflared** — Cloudflare Tunnel for secure ingress without exposing ports publicly. Token provided via `CLOUDFLARED_TUNNEL_TOKEN` env var.
- **n8n** — The automation engine. Persists data in a named Docker volume (`n8n_data` mounted at `/home/node/.n8n`). Runners enabled for workflow execution.

## Environment & Secrets

All secrets live in `.env` (gitignored). Required variables:
- `CLOUDFLARED_TUNNEL_TOKEN` — Cloudflare tunnel authentication
- `GOOGLE_OAUTH_CLIENT_ID` / `GOOGLE_OAUTH_CLIENT_SECRET` / `GOOGLE_OAUTH_REDIRECT` — Google OAuth2 for Gmail/Drive/Sheets/Docs
- `OPENAI_API_KEY` — OpenAI API access

n8n credentials are overwritten via `N8N_CREDENTIALS_OVERWRITE_DATA` in docker-compose.yml, defining two Gmail OAuth profiles: `gmail_main` (full Google suite scope) and `gmail_alerts` (Gmail-only scope).

`NEXUS_LABELS` env var defines email categorization labels (urgent, financial, legal, personal, kids, project, junk) used by n8n workflows.

## Key Configuration Details

- Production domain: `n8n.projectnexuscode.org`
- Traefik dashboard: `traefik.projectnexuscode.org`
- Timezone: `US/Central`
- TLS minimum version: 1.2 (enforced in `traefik/dynamic/config.yml`)
- `N8n_BLOCK_ENV_ACCESS_IN_NODE=false` — allows n8n Code nodes to read environment variables
- n8n workflow templates go in the repo root as `.json` files

## Project Nexus Integration Points

This node connects to other Nexus components:
- **Cortex (4090 GPU)** — LLM reasoning engine
- **Brainstem (4070 GPU)** — validation and routing layer
- **NAS Memory Node** — semantic/episodic knowledge store
- **Ollama / TRT-LLM** — local LLM runtimes (HTTP on port 11434)
- **OpenAI API** — external LLM access
