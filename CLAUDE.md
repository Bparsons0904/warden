# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Vaultwarden (Bitwarden-compatible password manager) deployment project using Docker Compose with Traefik reverse proxy integration and Drone CI for automated deployment.

## Architecture

**Core Components:**
- **Vaultwarden Service**: Self-hosted Bitwarden server running on `vaultwarden/server:latest`
- **Traefik Integration**: Reverse proxy with automatic SSL/TLS via Let's Encrypt
- **WebSocket Support**: Real-time notifications via dedicated WebSocket routing
- **Drone CI**: Automated deployment pipeline triggered on main branch pushes

**Network Architecture:**
- External Traefik network for reverse proxy communication
- HTTPS-only access via `warden.bobparsons.dev`
- WebSocket endpoint: `/notifications/hub` on port 3012
- Main service on port 80 (internal)

## Deployment Commands

```bash
# Local development/testing
docker-compose up -d
docker-compose down
docker-compose pull  # Update to latest image

# View logs
docker-compose logs -f vaultwarden

# Rebuild after configuration changes
docker-compose down && docker-compose up -d
```

## Configuration Management

**Environment Variables:**
- `VAULTWARDEN_ADMIN_TOKEN`: Admin panel access token (stored in Drone CI secrets)
- `DOMAIN`: Set to `https://warden.bobparsons.dev`
- Additional config via `.env` file (not tracked in git)

**Persistent Data:**
- Volume mount: `/home/server/vw-data/` → container `/data/`
- Contains database, attachments, and configuration

## CI/CD Pipeline

**Drone CI Workflow:**
1. Triggers on push to `main` branch
2. Pulls latest Docker images
3. Performs rolling deployment: `down` → `pull` → `up -d`
4. Uses Docker socket mounting for container management
5. Retrieves admin token from Drone secrets: `VAULTWARDEN_ADMIN_TOKEN`

**Secret Management:**
- Admin token must be configured in Drone CI repository secrets
- Environment variables are injected during deployment

## Traefik Configuration

**Routing Rules:**
- Main service: `Host(warden.bobparsons.dev)` → port 80
- WebSocket: `Host(warden.bobparsons.dev) && Path(/notifications/hub)` → port 3012
- HTTPS enforced with Let's Encrypt certificates
- Automatic certificate resolution via `letsencrypt` resolver

## Important Notes

- Database is pre-existing and persistent (restoration scenario)
- Service depends on external Traefik network being available
- WebSocket support required for real-time sync across Bitwarden clients
- Always verify Traefik network exists before deployment