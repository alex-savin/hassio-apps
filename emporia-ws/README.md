# Hass.io Emporia Vue Add-on (WebSocket bridge)

Thin Go service wrapping `go-emporia-vue` to serve a WebSocket API for the Home Assistant integration. Message contract lives in `docs/websocket-addon.md`.

## Current status (Dec 2025)
- WebSocket server at `/ws` with `hello`/`ping`/`pong`.
- Username/password authentication that instantiates a `go-emporia-vue` client per connection.
- Fetch endpoints: `get_devices`, `get_usage`, `get_properties`, `get_channel_types`.
- Controls: `toggle_outlet`, `toggle_evse` (with charging rate fields).
- Usage subscriptions: per-subscription ticker with immediate first payload; `unsubscribe` cancels.
- Writes are serialized per connection to avoid concurrent websocket writes.
- Aux HTTP: `/healthz` liveness and `/metrics` plain-text counters (connections/messages).

## Running (dev)
```bash
GO111MODULE=on go run ./hassio-emporia-vue-addon
```

Environment variables:
- `ADDON_PORT` (default `8283`)
- `ADDON_ADDR` (default `0.0.0.0`)
	(defaults updated: `ADDON_PORT` now `8080`)
- `CREDENTIALS_FILE` (default `/data/credentials.yaml`) sets where Emporia tokens are persisted (mode 0600). Internally forwarded as `EMPORIA_CREDENTIALS_FILE` for the client library.
- `WS_AUTH_TOKEN` (optional) enforces shared-secret auth; send `Authorization: Bearer <token>` or `?token=` when connecting to `/ws`.

## TODO
- Persist credentials/tokens to `/data` and prefer token reuse.
- Add health/metrics endpoints.
- Optional TLS or shared-secret socket auth.
