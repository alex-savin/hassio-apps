# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.1.1] - 2026-07-17

### Fixed

- **rtlamr never reached the main reading loop (entities stuck unavailable)**:
  the readiness probe waited for the literal `GainCount:`, but rtlamr's
  `v0.9.1 → v0.9.5` bump (in 1.1.0) switched its logging to logfmt, so it now
  prints `GainCount=29` (no colon). `start()` never matched the signal, blocked
  in its startup scan loop forever, and the parse/broadcast loop never ran —
  meters decoded but nothing was ever sent to WebSocket clients. The readiness
  markers are now constants that match both the old (`GainCount: 29`) and new
  (`GainCount=29`) formats, guarded by a regression test.
- **Unit tests were never tracked or run in CI**: a bare `rtlamr-ws` line in
  `.gitignore` (meant for the repo-root build binary) also matched the
  `cmd/rtlamr-ws/` source directory, so `main_test.go` could never be committed
  and `go test ./...` ran with no tests. The pattern is now anchored to the root
  binary (`/rtlamr-ws`) and the test suite is tracked.

### Documentation

- Corrected the "High CPU Usage" guidance: lowering rtl_tcp's `-s` has no effect
  because rtlamr commands its own sample rate on connect. The effective knob is
  `custom_parameters.rtlamr: "-symbollength=N"` (sample rate = `32768 × N`;
  default 72 ≈ 2.36 MHz).

## [1.1.0] - 2026-06-24

### Fixed

- **rtlamr auto-restart**: the crash-recovery path never triggered because
  process liveness was checked via `cmd.ProcessState` (only set by `Wait()`),
  so a crashed rtlamr left the reader loop spinning at 100% CPU and never
  restarted. Liveness is now determined by stdout EOF and the process is
  reaped before restart (no more zombies).
- **Lost readings on restart**: a second `bufio.Scanner` was created on the
  process pipe after startup, discarding any output the first scanner had
  buffered. A single scanner is now reused for the process lifetime.
- **`formatNumber` leading zeros**: a value narrower than the format width
  rendered as `001234.567` instead of the documented `1234.567`. Leading
  zeros in the integer part are now stripped.
- **`WS_LISTEN_ADDR` ignored**: the listen address env var was set in config
  but never read; the port was hard-coded to `:8080`. It is now honored.
- **`RTLAMR_CONFIG_PATH` ignored**: the env var set in the Dockerfile is now
  used as a fallback when `-config` is not passed.
- **Dead `log_level` option**: the s6 run script read a non-existent
  `log_level` option; it now surfaces the real `general.verbosity`.

### Added

- `expire_after` and `force_update` meter options are now parsed and forwarded
  in the WebSocket `attributes` (previously validated by the schema but
  dropped by the service).
- WebSocket keepalive (ping/pong with read deadline) so dead clients are
  detected promptly.
- Go unit tests for config loading, parsing, formatting, and argument builders.

### Changed

- Health endpoint (`/health`, `/healthz`) now returns a plain-text `ok` body
  with a `200` status, matching the sibling mysubaru-ws add-on (was JSON
  `{"status":"ok"}`). Clients should rely on the status code, not the body.
- Broadcasts no longer hold the hub lock during network writes; each
  connection has its own write mutex, so one slow client can't stall others.
- Updated `rtlamr` to `v0.9.5` (was `v0.9.1`).
- Migrated CI/CD off the deprecated `home-assistant/builder` action to the
  `prepare-multi-arch-matrix` + `build-image` reusable actions (which run
  `docker buildx` directly, with no builder container image); CD now signs
  published images with cosign.
- Pinned rtl-sdr to `v2.0.2` in the Dockerfile (was an unpinned `master`
  clone) for reproducible builds.
- Aligned Go toolchain to 1.26 across `go.mod`, the Dockerfile, and `build.sh`.
- Rewrote `build.sh` for the current (flat) repository layout.

## [1.0.1] - 2026-01-12

### Changed

- Migrated to s6-overlay v3 service management (s6-rc.d structure)
- Updated base image to `ghcr.io/hassio-addons/base:19.0.0`
- Refactored Dockerfile with separate build stages for rtl-sdr and Go service
- Moved binary to `/usr/local/bin/rtlamr-ws`
- Updated AppArmor profile for s6-overlay v3 compatibility

### Removed

- Deprecated s6-overlay v2 directories (cont-init.d, services.d)

## [1.0.0] - 2026-01-11

### Added

- Initial release of RTLAMR WebSocket Add-on for Home Assistant
- RTL-SDR integration using rtl_tcp and rtlamr
- WebSocket server for real-time meter reading broadcasts
- Support for multiple meter protocols:
  - IDM (Interval Data Message)
  - NetIDM (Network IDM)
  - R900 (Badger Orion)
  - R900BCD (R900 with BCD encoding)
  - SCM (Standard Consumption Message)
  - SCM+ (Enhanced SCM)
- Multi-meter configuration support
- Configurable reading format with decimal point placement
- Health check endpoint at `/health`
- WebSocket endpoint at `/ws` with automatic client management
- Cached readings for new WebSocket clients
- S6-overlay service management
- Multi-architecture support (amd64, aarch64)
- GitHub Actions CI/CD workflows
- Comprehensive configuration schema validation

### Configuration Options

- `general.sleep_for` - Interval between reading cycles
- `general.verbosity` - Log level (debug, info, warning, critical, none)
- `general.device_id` - RTL-SDR device index
- `general.rtltcp_host` - Optional external rtl_tcp server
- `custom_parameters.rtltcp` - Additional rtl_tcp arguments
- `custom_parameters.rtlamr` - Additional rtlamr arguments
- `meters` - List of meter configurations with protocol, format, and Home Assistant attributes

[Unreleased]: https://github.com/alex-savin/hassio-app-rtlamr-ws/compare/1.1.1...HEAD
[1.1.1]: https://github.com/alex-savin/hassio-app-rtlamr-ws/compare/1.1.0...1.1.1
[1.1.0]: https://github.com/alex-savin/hassio-app-rtlamr-ws/compare/1.0.1...1.1.0
[1.0.1]: https://github.com/alex-savin/hassio-app-rtlamr-ws/compare/1.0.0...1.0.1
[1.0.0]: https://github.com/alex-savin/hassio-app-rtlamr-ws/releases/tag/1.0.0
