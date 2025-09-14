# n8n Stack Deployment

This repository provides a ready-to-use deployment of **n8n** with **Caddy** as a reverse proxy and automatic HTTPS certificates.  
Deployment is automated through **GitHub Actions** and executed on a remote server.

---

## Overview

- **n8n** workflow automation platform
- **Caddy** reverse proxy with automatic TLS via Let’s Encrypt
- **GitHub Actions** pipeline for zero-downtime deployment
- Secure environment with encryption key and basic auth support

---

## Architecture

### Services (`docker-compose.yml`)
- **n8n**
  - Image: `n8nio/n8n:latest`
  - Exposed on port `5678`
  - Persistent volume: `n8n_data:/home/node/.n8n`
  - Configured with environment variables from `.env`

- **caddy**
  - Image: `caddy:latest`
  - Exposed on ports `80` and `443`
  - Uses `Caddyfile` for reverse proxy configuration
  - Handles HTTPS certificates automatically
  - Persistent volumes: `caddy_data`, `caddy_config`

### Volumes
- `n8n_data`
- `caddy_data`
- `caddy_config`

---

## Reverse Proxy (`Caddyfile`)

Caddy is configured to:

- Serve **${DOMAIN}** on HTTPS
- Automatically issue and renew TLS certificates via Let’s Encrypt
- Proxy traffic securely to the `n8n` container at port `5678`

---

## Environment Variables (`.env`)

The following variables are required:

```env
DOMAIN=yourdomain.com
LE_EMAIL=you@example.com
N8N_ENCRYPTION_KEY=your_encryption_key
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=strongpassword
```

---

## Setup

### Where to work
You can use this repository directly or fork it to your own GitHub account. If you plan to customize the stack, forking is recommended so you can push to your own `main` branch.

### Configure GitHub Secrets

Add the following repository **Secrets** in GitHub:
1. Open your repo → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**.
2. Create these secrets:

Required for SSH:
- `SSH_HOST` — server IP or hostname
- `SSH_USER` — SSH username (with permissions to run Docker)
- `SSH_PASSWORD` — SSH password for that user

Required for n8n/Caddy:
- `DOMAIN` — your domain (e.g. `automation.example.com`)
- `LE_EMAIL` — email for Let’s Encrypt
- `N8N_ENCRYPTION_KEY` — strong random string (used to encrypt credentials in n8n)
- `N8N_BASIC_AUTH_ACTIVE` — `true` or `false`
- `N8N_BASIC_AUTH_USER` — login for Basic Auth
- `N8N_BASIC_AUTH_PASSWORD` — password for Basic Auth

> Names must match exactly as above — the workflow reads them via `${{ secrets.NAME }}`.

### One-time server prerequisites
- A Linux host reachable via SSH (ports 22, 80, 443 open).
- User from `SSH_USER` can run Docker (either in `docker` group or via `sudo`).
- DNS A/AAAA record for `DOMAIN` points to your server’s IP.

---

## Deployment

### How to trigger the deploy
The workflow runs on **push to `main`**. To deploy:
1. Commit your changes.
2. Push to `main`:
   ```bash
   git push origin main
   ```
3. Go to **Actions** → **Deploy n8n** to watch the run.

### Optional: manual trigger
If you want a manual “Run workflow” button, add this to the top of `.github/workflows/deploy.yml`:
```yaml
on:
  push:
    branches: ["main"]
  workflow_dispatch: {}
```
Then open **Actions** → **Deploy n8n** → **Run workflow**.

---

## Manual Server Preparation

Ensure the server has:
- Docker installed
- Docker Compose plugin (`docker compose`) or legacy binary (`docker-compose`)
- Open ports 80 and 443

---

## Usage

After deployment:
- Access n8n via: https://yourdomain.com/
- Login with Basic Auth credentials (`N8N_BASIC_AUTH_USER` / `N8N_BASIC_AUTH_PASSWORD`)
- All workflows and credentials are stored in `n8n_data` volume

---

## Updating

To redeploy:

```bash
git push origin main
```

This will trigger the GitHub Actions workflow and update the remote stack.

---

## License

This repository is provided as-is for deploying n8n with Docker and Caddy.
