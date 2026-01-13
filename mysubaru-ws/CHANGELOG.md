# Changelog

All notable changes to this project will be documented in this file.

## [1.0.1] - 2026-01-12

### Changed

- Updated base image from `ghcr.io/home-assistant/base:latest` to `ghcr.io/hassio-addons/base:19.0.0`
- Migrated from s6-overlay v2 to v3 service structure
- Moved binary location from `/usr/bin/ws-server` to `/usr/local/bin/ws-server`
- Replaced `services.d` and `cont-init.d` with modern `s6-rc.d` structure
- Consolidated configuration loading into the service run script
- Updated AppArmor profile for s6-overlay v3 compatibility

## [1.0.0] - 2026-01-11

### Added

- Initial release
- WebSocket server for real-time vehicle status
- Health endpoint at `/health`
- Authentication endpoints (`/auth/config`, `/auth/status`, `/auth/send_code`, `/auth/verify`)
- Configurable options:
  - `log_level`: Log verbosity (debug, info, warn, error)
  - `poll_interval_seconds`: Vehicle status polling interval
  - `location_poll_interval_seconds`: GPS location update interval
- AppArmor security profile
- s6-overlay service management
- Multi-architecture support (amd64, aarch64)
