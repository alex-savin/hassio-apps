# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.1] - 2026-01-12

### Fixed
- Updated s6-overlay shebang paths from `/usr/bin/with-contenv` to `/command/with-contenv` for s6-overlay v3 compatibility
- Fixed AppArmor profile to allow s6-overlay init system (`/init`, `/command/**`), shells, and correct binary path
- Fixed "Permission denied" error on container shutdown

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

[Unreleased]: https://github.com/alex-savin/ups-monitor-addon/compare/1.0.1...HEAD
[1.0.1]: https://github.com/alex-savin/ups-monitor-addon/compare/1.0.0...1.0.1
[1.0.0]: https://github.com/alex-savin/ups-monitor-addon/releases/tag/1.0.0
