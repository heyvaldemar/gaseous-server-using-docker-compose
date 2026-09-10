# Gaseous Server + Traefik + Let's Encrypt on Docker Compose

[![Deployment Verification](https://github.com/heyvaldemar/gaseous-server-using-docker-compose/actions/workflows/deployment-verification.yml/badge.svg?branch=main)](https://github.com/heyvaldemar/gaseous-server-using-docker-compose/actions/workflows/deployment-verification.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

This repository deploys Gaseous Server (a self-hosted ROM manager and in-browser retro game player) behind Traefik with automatic Let's Encrypt TLS, backed by MariaDB 11.4 LTS, with scheduled backups (database + library data) and companion restore scripts.

## Getting started

```bash
# 1. Clone
git clone https://github.com/heyvaldemar/gaseous-server-using-docker-compose
cd gaseous-server-using-docker-compose

# 2. Create the two Docker networks the stack expects
docker network create traefik-network
docker network create gaseous-server-network

# 3. Copy the environment template and fill in required values
cp .env.example .env
$EDITOR .env
# ^ Required: two generated DB passwords, GASEOUS_SERVER_HOSTNAME,
#   TRAEFIK_HOSTNAME, TRAEFIK_ACME_EMAIL, TRAEFIK_BASIC_AUTH.

# 4. Deploy
docker compose -f gaseous-server-traefik-letsencrypt-docker-compose.yml -p gaseous up -d
```

First start initializes the database. Give it a few minutes. The first account registered in the web UI becomes the admin, so open the site right after deploy. Add IGDB credentials in `.env` when you want covers and metadata.

### What success looks like

```bash
docker compose -f gaseous-server-traefik-letsencrypt-docker-compose.yml -p gaseous ps
curl -fskL -o /dev/null -w "%{http_code}\n" "https://${GASEOUS_SERVER_HOSTNAME}/"
```

### Common first-deploy issues

- **Cert issuance fails.** DNS hasn't propagated or port 80 isn't reachable from the internet.
- **504/timeouts in the first minutes.** Database initialization is still running; the healthcheck holds Traefik back until the server answers.
- **Networks not found.** Step 2 was skipped.

## Supply chain trust

Three images ([`traefik`](https://hub.docker.com/_/traefik), [`gaseousgames/gaseousserver`](https://hub.docker.com/r/gaseousgames/gaseousserver), [`mariadb`](https://hub.docker.com/_/mariadb)) pinned to `tag@sha256:<digest>` as interpolation defaults in the compose `x-images` block. `git pull` alone delivers the tested combination; an `*_IMAGE_TAG` variable in `.env` overrides deliberately.

The weekly `check-pin-freshness` CI job re-resolves each pin against its registry and compares the pinned versions against the latest upstream releases. GitHub Actions are pinned by commit SHA; Dependabot keeps those fresh.

## Production checklist

- [ ] **Register the admin account immediately after deploy.**
- [ ] **Strong secrets**: both DB passwords at 24+ random characters; regenerate the Traefik dashboard hash.
- [ ] **Host-mount the backup volumes** for disaster recovery: the library data includes your ROMs.
- [ ] **Mind the legal side**: only store ROMs you have the right to.

## Backups and restore

The `backups` container runs a `mariadb-dump | gzip` + `tar.gz`-of-library → prune → sleep loop (defaults: 30-minute warm-up, 24-hour interval, 7-day retention). Restore with the interactive scripts (`chmod +x *.sh` once): `./gaseous-server-restore-database.sh`, then `./gaseous-server-restore-application-data.sh`.

## Testing

The [Deployment Verification](https://github.com/heyvaldemar/gaseous-server-using-docker-compose/actions/workflows/deployment-verification.yml?query=branch%3Amain) workflow runs on every push, pull request, and every Monday at 06:00 UTC: shellcheck + actionlint, Trivy scans of all three pinned images, the weekly freshness check, and a deploy-and-test job that boots the stack with ephemeral credentials and requires the UI to answer through Traefik.

## Security notes

- Credentials are read from `.env` at deploy time; `.env` is gitignored and compose fails fast on missing required variables.
- MariaDB listens only on the internal network.


## Security notes

- **Pre-rotation advisory.** Earlier revisions of this repository tracked a
  `.env` carrying `GASEOUS_SERVER_DB_PASSWORD` and
  `GASEOUS_SERVER_DB_ADMIN_PASSWORD`. The file is untracked now and `.env` is
  gitignored, but the values are still in the git history and cannot be taken
  out of it. **Rotate both** if this deployment ever used them.
- `.env` carries only secrets and deliberate overrides; every image version is
  pinned in the compose file. `.env.example` lists what has to be set.

---

## About the maintainer

<div align="center">

**Maintained by [Vladimir Mikhalev](https://github.com/heyvaldemar)** — Docker Captain · IBM Champion · AWS Community Builder

[YouTube](https://www.youtube.com/channel/UCf85kQ0u1sYTTTyKVpxrlyQ?sub_confirmation=1) · [Blog](https://heyvaldemar.com) · [LinkedIn](https://www.linkedin.com/in/heyvaldemar/)

</div>
