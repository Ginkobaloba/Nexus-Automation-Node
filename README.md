# Nexus Automation Node

A self-hosted automation engine for [Project Nexus](https://github.com/Ginkobaloba/ProjectNexus), powered by [n8n](https://n8n.io) and deployed via Docker. This service runs workflow execution, event processing, task orchestration, and the automated integrations between Nexus subsystems.

## Features

- Dockerized n8n stack for reproducible deployment
- Environment template (`.env.example`) for quick configuration
- Optional Traefik / Cloudflare / NGINX reverse proxy support
- Automatic backups (MariaDB or PostgreSQL optional)
- API-ready integration with:
  - Nexus brainstem (validation layer)
  - Nexus cortex (LLM reasoning engine)
  - NAS memory node (semantic / episodic storage)
  - Local or remote Ollama / TRT-LLM runtimes
- Secure external access via tokens and HTTPS
- Designed for long-running, stable uptime

## Repository structure

```
nexus-automation-node/
  docker-compose.yml
  Dockerfile                (optional, when not using official n8n image)
  .env.example
  .gitignore
  proxy/
    traefik.yml
    dynamic/
    certificates/
  workflows/                exported n8n workflow JSON
  scripts/                  backup, restore, maintenance utilities
  README.md
```

## Deployment

### Prerequisites

- Docker and Docker Compose
- A domain or subdomain if using HTTPS
- Optional: Cloudflare Tunnel or Traefik

### Quick start

```bash
cp .env.example .env
# edit .env as needed
docker compose up -d
```

Access n8n at `http://localhost:5678` or your reverse-proxied domain.

## Local LLM integration

`nexus-automation-node` can call:

- Ollama running on another machine or container
- Nexus cortex runtime
- Any OpenAI-compatible LLM server

Example HTTP Request node payload:

```http
POST http://<machine-ip>:11434/api/generate

{
  "model": "llama2",
  "prompt": "Hello from Nexus automation node!"
}
```

## Security

- Use strong JWT and Basic Auth tokens
- Reverse proxy behind HTTPS when exposed to the internet
- Rotate encryption keys yearly (scripts included)
- Keep secrets in `.env`, never in workflows

## Backups

Back up workflows and credentials:

```bash
docker exec n8n sh -c "tar czf /home/node/.n8n_backup.tar.gz /home/node/.n8n"
docker cp n8n:/home/node/.n8n_backup.tar.gz .
```

Restore:

```bash
docker cp .n8n_backup.tar.gz n8n:/home/node
docker exec n8n tar xzf /home/node/.n8n_backup.tar.gz -C /
```

## About Project Nexus

This repo is one node in [Project Nexus](https://github.com/Ginkobaloba/ProjectNexus), a modular, biologically-inspired distributed cognition system composed of:

- Cortex (4090): reasoning engine
- Brainstem (4070): validation and routing
- NAS memory node: semantic and episodic store
- Peripheral Jetson Nanos: sensory modules
- Automation node (this repo): workflow orchestration layer

Each node is isolated, autonomous, and composable.

## License

MIT.
