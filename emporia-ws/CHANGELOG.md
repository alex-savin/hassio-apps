# Changelog

All notable changes to the Emporia Vue Websocket Add-on will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2026-06-10

### Added

- Cross-site WebSocket hijacking protection: browser connections must be
  same-origin or listed in the new `WS_ALLOWED_ORIGINS` setting (native
  clients without an `Origin` header are unaffected)
- Build-time version injection so the binary self-reports the add-on version
- Unit and concurrency (`-race`) tests for the WebSocket server

### Changed

- Updated `go-emporia-vue` to a concurrency-safe release. The upstream client
  is now safe for concurrent use, honors `EMPORIA_CREDENTIALS_FILE` at runtime,
  and no longer ignores the configured EV charging rate
- In-flight Emporia API calls are now cancelled on shutdown
- Aligned build tooling on Go 1.26

### Fixed

- Data races in the WebSocket hub (client auth flag, client count, auth status)
- A slow client could block broadcasts and new connections; network writes no
  longer happen while holding the hub lock
- Poll failures no longer broadcast stale/empty snapshots
- EV charger toggles now apply the requested charging rate
- Token cache now persists to `/data` across restarts
- Corrected the `poll_interval_seconds` fallback default (was 10, now 60)

## [1.0.1] - 2026-01-12

### Changed

- Migrated to hassio-addons/base:19.0.0 base image
- Restructured rootfs to use modern s6-overlay v3 (s6-rc.d) service management
- Binary path changed from /usr/bin to /usr/local/bin
- Updated AppArmor profile with simplified single-profile design

### Removed

- Deprecated services.d and cont-init.d directories

## [1.0.0] - 2026-01-12

### Added

- Translations for 8 languages (English, German, Spanish, French, Dutch, Portuguese, Italian, Polish)
- AppArmor security profile
- Initial release
- WebSocket server exposing Emporia Vue energy data
- Push-based updates with configurable poll interval
- Persistent authentication across restarts
- Device and channel data support
- Outlet/EVSE control functionality
- Health endpoint for monitoring
- Configurable log level
