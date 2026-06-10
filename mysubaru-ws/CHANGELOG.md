# Changelog

All notable changes to this project will be documented in this file.

## [1.1.0] - 2026-03-05

### Added

- **Horn & Lights**: `horn_start`, `horn_stop`, `lights_start`, `lights_stop` endpoints
- **Cancel Commands**: `lock_cancel`, `unlock_cancel`, `engine_start_cancel`, `lights_cancel`, `horn_lights_cancel`
- **Valet Mode**: `valet_start`, `valet_stop`, `valet_status` (GET), `valet_settings` (GET), `save_valet_settings`
- **GeoFence Alerts**: `geofence_activate`, `geofence_deactivate`, `set_geofence`, `update_geofence`, `delete_geofence`, `geofence_settings` (GET), `save_geofence_settings`
- **Speed Fence Alerts**: `speedfence_activate`, `speedfence_deactivate`, `set_speedfence`, `speedfence_settings` (GET), `save_speedfence_settings`
- **Curfew Alerts**: `curfew_activate`, `curfew_deactivate`, `set_curfew`, `curfew_settings` (GET), `save_curfew_settings`
- **Trip Tracker**: `triplog_start`, `triplog_stop`, `trips` (GET), `delete_trip`
- **POI/Destinations**: `send_poi`, `favorite_pois` (GET), `save_favorite_poi`
- **EV Charge Management**: `ev_charge_settings` (GET), `save_ev_charge_settings`, `delete_ev_charge_schedule`
- **Climate Presets**: `save_climate_presets`, `delete_climate_preset`
- **Vehicle Info**: `recalls` (GET), `warning_lights` (GET), `model_info` (GET)
- **Roadside Assistance**: `roadside_assistance` (GET), `request_roadside_assistance`
- **Auth**: `POST /auth/refresh_vehicles` to force-refresh vehicle list
- **Integration**: Switch entities for Valet Mode, GeoFence, Speed Fence, Curfew toggles
- **Integration**: 11 new button entities (horn, lights, cancel ops, trip log)
- **Integration**: 22 HA services for parameterized commands and data queries
- **Integration**: `services.yaml` with full field schemas for HA UI

### Changed

- Updated go-mysubaru to v0.0.0-20260223102448-f886644cd93f
- Updated Go to 1.26, golang.org/x/net to 0.51.0
- Vehicle handler now accepts GET for data retrieval endpoints (previously POST-only)
- Integration manifest version bumped to 1.1.0

## [1.0.2] - 2026-01-13

### Fixed

- Added automatic session re-authentication when `InvalidToken` errors occur during polling
- Location polling now detects session expiration and recovers automatically
- Vehicle status polling now handles session expiration gracefully

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
