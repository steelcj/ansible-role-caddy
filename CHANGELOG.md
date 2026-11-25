# Changelog

All notable changes to this repository are recorded here, in the format of [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). The repository follows [Semantic Versioning](https://semver.org/).

Write entries in the Unreleased section below as work happens. `scripts/cut-release.py` turns that section into a dated version section when a release is cut.

## [Unreleased]

## [0.1.0] - 2026-11-25

* Created repository
* Created version 0.1.0

## 0.1.0 - 2026-11-25

- Initial Debian 13 service foundation, authenticated APT installation and package-managed systemd service.
- Global Caddyfile, independent site and snippet directories, restrictive state permissions, staged validation and graceful activation with attempted rollback.
- Local Incus Molecule scenario adapted from the supplied Foomuuri role.
- Role selection findings and manual testing guide. Runtime validation pending.
