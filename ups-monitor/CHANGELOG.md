# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.1.0] - 2026-06-24

### Added
- `poll_interval_seconds` now actually controls the polling cadence (it was previously ignored — polling was hard-coded to 10s)
- Per-device poller lifecycle management: adding, updating, or deleting a device at runtime now starts/stops the correct poller (fixes duplicate pollers on update and leaked pollers on delete)
- Docker `HEALTHCHECK` on `/healthz` so the Supervisor watchdog can detect and restart a hung add-on
- `WS_LISTEN_ADDR` is now honored as the server listen address

### Changed
- `poll_interval_seconds` is now range-validated to 1–3600
- AppArmor profile is enabled and updated for s6-overlay v3 (it was disabled in 1.0.1 to fix startup; it has since been re-enabled with v3-compatible permissions)
- Updated the Go directive to 1.26 and bumped `go-ups-monitor` to its latest build

### Fixed
- Data race on the cached WebSocket payload (`hub.last`) that was read without its mutex
- `DOCS.md` corrected to match the real API: status JSON uses `device_name` and has no `online` field, `/healthz` returns `healthy`, the NUT device name must match the server-side UPS name (there is no `ups_name` field), and the command/test endpoints are now documented
- Aligned Go toolchain version across `build.sh` (was `golang:1.23`, below the module requirement) and the Dockerfile
- Migrated CI/CD off the deprecated `home-assistant/builder` action to its reusable `build-image` / `prepare-multi-arch-matrix` actions (the old action's `2026.06.0` builder image is unpublished, which broke the Docker build)

## [1.0.2] - 2026-01-13

### Changed
- Improved exponential backoff with max 5-minute retry interval (was 60 seconds)
- Smart error logging: first 3 failures at ERROR, 4-10 at WARN, then DEBUG to reduce log spam
- Periodic summary logging every 10 failures during prolonged outages

### Added
- Consecutive failure tracking per device
- Recovery logging when device comes back online
- New error type detection to re-escalate logging for different errors

## [1.0.1] - 2026-01-12

### Changed
- Switched to community base image `ghcr.io/hassio-addons/base:19.0.0` for better s6-overlay v3 support
- Migrated from legacy `services.d` to modern `s6-rc.d` service structure
- Disabled custom AppArmor profile (was causing startup failures)

### Fixed
- Updated s6-overlay shebang paths from `/usr/bin/with-contenv` to `/command/with-contenv` for s6-overlay v3 compatibility
- Fixed "Permission denied" and "s6-svscan already running" errors on container startup

## [1.0.0] - 2026-01-11

### Added
- Initial release of UPS Monitor add-on
- WebSocket endpoint `/ws` for real-time UPS status updates
- Health check endpoint `/healthz`
- Status API endpoint `/api/status`
- Device registration API endpoint `/api/device`
- Configuration persistence to `/data/config.yml`
- Multi-architecture support (aarch64, amd64)
- Configurable log level (debug, info, warn, error)
- Configurable poll interval for UPS status updates
- Translations for 10 languages (EN, DE, ES, FR, IT, NL, PT, RU, UK, ZH)

[Unreleased]: https://github.com/alex-savin/hassio-app-ups-monitor/compare/1.1.0...HEAD
[1.1.0]: https://github.com/alex-savin/hassio-app-ups-monitor/compare/1.0.2...1.1.0
[1.0.2]: https://github.com/alex-savin/hassio-app-ups-monitor/compare/1.0.1...1.0.2
[1.0.1]: https://github.com/alex-savin/hassio-app-ups-monitor/compare/1.0.0...1.0.1
[1.0.0]: https://github.com/alex-savin/hassio-app-ups-monitor/releases/tag/1.0.0
