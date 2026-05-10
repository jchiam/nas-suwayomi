# nas-suwayomi

Suwayomi manga server running on my UGreen NAS, with FlareSolverr for bypassing Cloudflare-protected manga sources.

## Purpose

Hosts a self-managed manga library and reader, accessible over HTTPS via the Caddy reverse proxy on the same NAS.

## Architecture

```
Internet → manga.in.<my-domain> → Tailscale (access control) → Caddy → localhost:4567
                                                                              ↓
                                                                         Suwayomi
                                                                              ↓
                                                                        FlareSolverr
                                                                     (Cloudflare bypass)
```

## Services

| Container | Image | Purpose |
|---|---|---|
| `suwayomi` | `ghcr.io/suwayomi/suwayomi-server:latest` | Manga server and reader UI |
| `flaresolverr` | `ghcr.io/flaresolverr/flaresolverr:latest` | Bypasses Cloudflare protection on manga sources |

## Prerequisites

- Docker and Docker Compose installed on the NAS
- [nas-caddy](../nas-caddy) running as the reverse proxy

## Setup

1. Clone the repository onto the NAS.

2. Start the containers:

   ```bash
   docker compose up -d
   ```

3. Access the UI at `http://localhost:4567` (or via the Caddy-proxied domain).

## Configuration

| Variable | Value | Purpose |
|---|---|---|
| `BACKUP_INTERVAL` | `1` | Auto-backup every 1 day |
| `UPDATE_INTERVAL` | `12` | Auto library refresh every 12 hours |
| `FLARESOLVERR_ENABLED` | `true` | Route requests through FlareSolverr |
| `FLARESOLVERR_URL` | `http://flaresolverr:8191` | FlareSolverr endpoint |
| `FLARESOLVERR_TIMEOUT` | `60` | FlareSolverr timeout in seconds |

## Persistent Data

The current directory is bind-mounted into the container as Suwayomi's data directory:

```yaml
volumes:
  - ./:/home/suwayomi/.local/share/Suwayomi-Server
```

This stores the library, downloads, and configuration alongside the compose file.

## Linting

Lint checks run automatically on every push and PR to `master` via GitHub Actions ([.github/workflows/lint.yaml](.github/workflows/lint.yaml)).

| Tool | Target |
|---|---|
| `yamllint` | `docker-compose.yaml` |

## Files

| File | Purpose |
|---|---|
| [docker-compose.yaml](docker-compose.yaml) | Service definitions for Suwayomi and FlareSolverr |
| [.github/workflows/lint.yaml](.github/workflows/lint.yaml) | GitHub Actions lint workflow |
