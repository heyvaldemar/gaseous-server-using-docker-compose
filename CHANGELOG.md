# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

_(no unreleased changes yet)_

## [1.2.1] - 2026-09-23

### Fixed

- **Both restore scripts pointed at directories the stack does not use.** The
  database script listed `/srv/gaseous-server-mariadb/backups` while dumps are
  written to `/srv/gaseous-mariadb/backups`; the data script listed, filtered
  and cleared three paths that do not exist in this stack. Run on the day they
  were needed, both would have offered nothing to restore. They now take every
  path and name from the running backups container, accept the file name as an
  argument, and CI runs them: a marker written after a backup must be gone once
  that backup is restored, for the database and for the application data.
- **The version check called any difference "behind"**, including being ahead.
  It compares by order now, as the rest of the fleet does.

### Added

- **OpenSSF Scorecard**, which every public repository in the fleet carries.

## [1.2.0] - 2026-09-23

### Fixed

- **A data backup was named a backup on tar's exit code alone.** It is read back with `tar -tzf` before it is renamed into place; an exit code has never been a statement about whether the archive opens.
- **A Trivy scan that could not finish read as success.** `continue-on-error` hid exactly that case, and the SARIF upload after it was skipped too.
- **The README carried two Security notes sections**, one of them saying the old passwords could not be taken out of the history. They can, and have been.

### Changed

- **Published.** This repository was private while an earlier `.env` carrying two database passwords sat in its history. That file has been removed from every commit before publication, and a full-history secret scan finds nothing; the earlier tags keep their versions.
- **Checked daily**, as the security policy already said; the schedule was weekly.
- **The freshness check has its own workflow, Pin Freshness**, so the badge says whether the stack boots rather than whether a pin is one version behind.
- **`./update.sh`** moves a deployment between release tags, refuses a major version unattended and names any newly required variable before anything moves.
- **Every CI run upgrades from the previous release** on the same volumes before the smoke tests, so a release is proven on data a deployed host already has.
- **What the repository does not contain** is stated in the README: no ROM, BIOS or firmware, and no pointer to any.

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
