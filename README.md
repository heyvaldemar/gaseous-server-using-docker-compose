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

## Updating

`./update.sh` moves this checkout to the latest release tag — a combination this repository's CI has booted, upgraded from the previous release on the same volumes, and smoke-tested — and then runs `docker compose up -d`. It refuses to cross a major version unattended, refuses to run over local changes, and names any variable that became required since your version before anything has moved. `./update.sh --dry-run` says what would happen. Every release cut by fleet triage also carries what upstream changed, read from its release notes against this compose file.

## Supply chain trust

Three images ([`traefik`](https://hub.docker.com/_/traefik), [`gaseousgames/gaseousserver`](https://hub.docker.com/r/gaseousgames/gaseousserver), [`mariadb`](https://hub.docker.com/_/mariadb)) pinned to `tag@sha256:<digest>` as interpolation defaults in the compose `x-images` block. `git pull` alone delivers the tested combination; an `*_IMAGE_TAG` variable in `.env` overrides deliberately.

The weekly `check-pin-freshness` CI job re-resolves each pin against its registry and compares the pinned versions against the latest upstream releases. GitHub Actions are pinned by commit SHA; Dependabot keeps those fresh.

## Production checklist

- [ ] **Register the admin account immediately after deploy.**
- [ ] **Strong secrets**: both DB passwords at 24+ random characters; regenerate the Traefik dashboard hash.
- [ ] **Host-mount the backup volumes** for disaster recovery: the library data includes your ROMs.
- [ ] **Mind the legal side**: only store ROMs you have the right to.

## Backups and restore

The `backups` service dumps the database and archives the application data on its interval (`BACKUP_INTERVAL`, default 24h), reads each file back before naming it a backup, and prunes by age. Restore with the two scripts next to the compose file:

```bash
./gaseous-server-restore-database.sh            # list the database dumps and ask which
./gaseous-server-restore-database.sh <file>     # restore that dump
./gaseous-server-restore-application-data.sh    # the same for application data
```

Both stop Gaseous while they work and start it again afterwards, and both take every path and file name from the running backups container, so they cannot disagree with where the stack writes. Set `COMPOSE_PROJECT_NAME` if you started the stack with a `-p` other than `gaseous`. CI runs these exact scripts on every push: it writes a marker after a backup, restores the backup, and requires the marker to be gone.

## Testing

The [Deployment Verification](https://github.com/heyvaldemar/gaseous-server-using-docker-compose/actions/workflows/deployment-verification.yml?query=branch%3Amain) workflow runs on every push, pull request, and every Monday at 06:00 UTC: shellcheck + actionlint, Trivy scans of all three pinned images, the weekly freshness check, and a deploy-and-test job that boots the stack with ephemeral credentials and requires the UI to answer through Traefik.

## Security notes

- Credentials are read from `.env` at deploy time; `.env` is gitignored and compose fails fast on missing required variables.
- `.env` carries only secrets and deliberate overrides; every image version is pinned in the compose file. `.env.example` lists what has to be set.
- MariaDB listens only on the internal network.

## What this repository does not contain

No ROM, BIOS, firmware or other copyrighted game file is included, linked to, or described how to obtain. Gaseous organises and plays a library you already own: dump your own cartridges and discs, and check that doing so is lawful where you live. This repository deploys the upstream [gaseous-server](https://github.com/gaseous-project/gaseous-server) image (AGPL-3.0) unmodified.

---

## About the maintainer

<div align="center">

**Maintained by [Vladimir Mikhalev](https://github.com/heyvaldemar)** — Docker Captain · IBM Champion · AWS Community Builder

[YouTube](https://www.youtube.com/channel/UCf85kQ0u1sYTTTyKVpxrlyQ?sub_confirmation=1) · [Blog](https://heyvaldemar.com) · [LinkedIn](https://www.linkedin.com/in/heyvaldemar/)

</div>
