# Gitea Stack

Docker Compose setup for a self-hosted [Gitea](https://gitea.com) instance with PostgreSQL, Traefik reverse proxy, Act Runner for CI/CD, and automatic deployment via GitHub/Gitea Actions.

## Stack

- **Gitea** — self-hosted Git service
- **PostgreSQL** — database
- **Traefik** — reverse proxy with automatic TLS (Let's Encrypt)
- **Gitea Act Runner** — CI/CD runner compatible with GitHub Actions syntax

## Structure

```
.
├── docker-compose.yml            # Production: Gitea + PostgreSQL
├── docker-compose-traefik.yml    # Traefik reverse proxy with TLS
├── docker-compose-dev.yml        # Local development (no Traefik)
├── docker-compose-runner.yml     # Act Runner
├── .gitea.env.example            # Environment variables template
└── .github/workflows/deploy.yml  # Auto-deploy on push to main
```

## Setup

### 1. Configure environment variables

```bash
cp .gitea.env.example .gitea.env
```

Fill in `.gitea.env` with your real values (see [Variables](#variables)).

### 2. Traefik network

Make sure the external network `gitea` exists:

```bash
docker network create gitea
```

### 3. Start Traefik

```bash
docker compose -f docker-compose-traefik.yml up -d
```

The Traefik dashboard is available at `https://<TRAEFIK_HOSTNAME>` (protected by basic auth).

### 4. Start Gitea

**Production** (with Traefik):
```bash
docker compose -f docker-compose.yml up -d
```

**Local development** (exposed on port `3000`, no Traefik required):
```bash
docker compose -f docker-compose-dev.yml up -d
```

### 5. Start the Runner

Get the registration token from the Gitea admin panel:
`Site Administration → Actions → Runners → Create new runner`

Add it to `.gitea.env` as `GITEA_RUNNER_REGISTRATION_TOKEN`, then:

```bash
docker compose -f docker-compose-runner.yml up -d
```


## Auto-deploy

The workflow `.github/workflows/deploy.yml` triggers on every push to `main` and connects to the server via SSH to update the containers.

Configure the following secrets in the repository:

| Secret | Description |
|---|---|
| `SSH_HOST` | Server IP or hostname |
| `SSH_USER` | SSH user |
| `SSH_PRIVATE_KEY` | SSH private key |
| `SSH_PORT` | SSH port (default `22`) |
| `DEPLOY_PATH` | Project path on the server |
