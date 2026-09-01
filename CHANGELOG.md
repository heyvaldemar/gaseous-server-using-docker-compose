# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

_(no unreleased changes yet)_

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

[Unreleased]: https://github.com/heyvaldemar/gaseous-server-using-docker-compose/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/heyvaldemar/gaseous-server-using-docker-compose/releases/tag/v1.0.0
