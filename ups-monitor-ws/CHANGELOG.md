# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
- Multi-architecture support (aarch64, amd64, armv7, armhf, i386)
- Configurable log level (debug, info, warn, error)
- Configurable poll interval for UPS status updates
- Translations for 10 languages (EN, DE, ES, FR, IT, NL, PT, RU, UK, ZH)

[Unreleased]: https://github.com/alex-savin/ups-monitor-addon/compare/1.0.2...HEAD
[1.0.2]: https://github.com/alex-savin/ups-monitor-addon/compare/1.0.1...1.0.2
[1.0.1]: https://github.com/alex-savin/ups-monitor-addon/compare/1.0.0...1.0.1
[1.0.0]: https://github.com/alex-savin/ups-monitor-addon/releases/tag/1.0.0
