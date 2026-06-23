# Changelog

All notable changes to this add-on are documented here. The format is based on
[Keep a Changelog](https://keepachangelog.com/en/1.0.0/) and this project adheres
to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-06-22

Initial release. Bridges the Xiaomi Home (MIoT) cloud to Home Assistant via
[`go-xiaomi-home`][lib].

### Added

- Mi account authentication with email/phone 2FA and persistent sessions
  (`/data/session.json`), so verification is a one-time step per device id.
- Robot-vacuum discovery and periodic status polling, broadcast as JSON
  snapshots over a websocket (`/ws`).
- Live map streaming while cleaning: the map is downloaded, parsed, and rendered
  to a labelled PNG server-side, with lightweight `map_frame` events on the
  socket and the image served at `GET /device/{did}/map.png`.
- HTTP command surface: `start`, `stop`, `pause`, `dock`, `locate`,
  `set_fan_speed`, `clean_rooms`, `refresh_map`, `poll`, and raw `rpc`.
- `command_status` events for command progress.
- Exponential backoff with jitter on cloud calls and silent re-authentication on
  session expiry.
- Multi-arch (amd64, aarch64) packaging on `ghcr.io/hassio-addons/base:19.0.0`
  with s6-overlay v3, an AppArmor profile, and keyless-signed GHCR images.

### Notes

- The upstream `go-xiaomi-home` client is not goroutine-safe; all cloud access is
  serialized through a single mutex, including the live-map renderer.
- `go-xiaomi-home` is a private module pinned by version in `go.mod`; CI fetches
  it with a short-lived GitHub App token and the container build vendors it
  ephemerally so no credential enters a Docker layer (see `DOCS.md`).

[lib]: https://github.com/alex-savin/go-xiaomi-home
