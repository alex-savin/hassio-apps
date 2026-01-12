# Changelog

All notable changes to this project will be documented in this file.

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
