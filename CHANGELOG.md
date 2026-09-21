# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

_(no unreleased changes yet)_

## [1.1.2] - 2026-09-21

### Security

- **`traefik:3.7` and `mariadb:11.4` repushed upstream; the pins follow them.** The same refresh was pushed on 2026-09-21 and reverted by fleet triage, which read the run's overall conclusion as a verdict on the change. Everything that verifies the deployment had passed — compose up, the HTTPS smoke test, all three Trivy scans, the linter — and the only red job was the freshness alarm, which goes red whenever any pin here lags upstream and says nothing about the digest just written. The triage no longer judges that way.

## [1.1.1] - 2026-09-14

### Security

- **`mariadb:11.4` was rebuilt upstream**; the pin moved from `sha256:611a2fcc5fa7…` to `sha256:80494b981069…`. Same version, same tag, a rebuilt base image — the usual shape of a security fix in a base layer.

## [1.1.0] - 2026-09-04

### Fixed

- **A failed database dump reported success.** The backup ran under `sh` with
  no `pipefail`, so the exit status of `mariadb-dump | gzip` was gzip's, and
  gzip is perfectly happy compressing an error message into a valid archive.
  Nothing was logged either way. The loop now runs under `bash -c` with
  `set -o pipefail`, names every outcome with a timestamp and a byte count,
  and writes each file to `<name>.partial`, renaming it only once the write
  succeeded, so the real name never exists unless the file behind it is whole.

### Added

- **The hardening this repository had missed.** It carried the digest pins and
  the CI, but not the container hardening the rest of the fleet has been
  running for two months: `no-new-privileges` on all four services,
  `cap_drop: ALL` with a minimal `cap_add` on MariaDB, Traefik and the backup
  sidecar, memory and CPU limits as `.env`-overridable defaults, and a sixty
  second shutdown grace period for MariaDB so an InnoDB flush is not cut short
  by SIGKILL.
- **Backup and restore tested end to end in CI.** The same suite the rest of
  the fleet runs, against the live stack: the backup is produced, readable and
  contains real dump content, a failure is detected when the database is down,
  a restore genuinely replaces state, and pruning keeps the recent files.

## [1.0.0] - 2026-09-01

First semver release. Brings this template to the fleet standard established
in [keycloak-traefik-letsencrypt-docker-compose](https://github.com/heyvaldemar/keycloak-traefik-letsencrypt-docker-compose)
v1.2.0.

### Changed

- **Gaseous Server pinned to v1.7.14, MariaDB to 11.4 LTS, Traefik to
  3.7** (3.2's Docker client cannot talk to Docker Engine 29), all by
  `tag@sha256:digest` in the compose `x-images` block. `git pull`
  delivers the tested combination; `.env` carries only secrets and
  deliberate overrides. IGDB credentials are optional now.

### Fixed

- Backup-loop variables are `$$`-escaped so the container shell resolves
  them at runtime; shellcheck findings in both restore scripts.

### Added

- **Deployment Verification workflow**: shellcheck + actionlint; Trivy
  scans of all three pinned images; weekly `check-pin-freshness` (digest
  drift + Gaseous and Traefik release lag); and a deploy-and-test job
  that boots the stack and requires the UI to answer through Traefik.
- `.env.example`; `.env` gitignored.

[Unreleased]: https://github.com/heyvaldemar/gaseous-server-using-docker-compose/compare/v1.1.2...HEAD
[1.1.2]: https://github.com/heyvaldemar/gaseous-server-using-docker-compose/compare/v1.1.1...v1.1.2
[1.1.1]: https://github.com/heyvaldemar/gaseous-server-using-docker-compose/compare/v1.1.0...v1.1.1
[1.1.0]: https://github.com/heyvaldemar/gaseous-server-using-docker-compose/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/heyvaldemar/gaseous-server-using-docker-compose/releases/tag/v1.0.0
