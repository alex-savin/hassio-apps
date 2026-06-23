# Xiaomi Home Websocket (Home Assistant Add-on)

A Home Assistant **add-on** that bridges the [Xiaomi Home / MIoT cloud][miot] to
the companion [Xiaomi Home integration][integration]. It wraps the
[`go-xiaomi-home`][lib] client, authenticates to your Mi account, discovers your
robot vacuums, and:

- streams each vacuum's live state (status, battery, fan speed, rooms) over a
  websocket,
- streams the **live map** while a clean is running (rendered server-side to
  PNG, with room labels), and
- accepts vacuum commands over HTTP (start/stop/pause/dock/locate, fan speed,
  clean specific rooms, raw RPC).

> This is the server half. Install the companion
> [`hassio-integration-xiaomi`][integration] custom integration to get HA
> entities (vacuum, map camera, sensors, buttons).

## How it works

```
Mi cloud  ⇄  [ this add-on: ws-server ]  ⇄  ws://…/ws + HTTP  ⇄  [ HA integration ]  ⇄  HA entities
```

The add-on holds your Mi account session (persisted in `/data`) and is the only
component that talks to the cloud. The integration never sees a cloud token — it
pushes credentials to the add-on once, then consumes the websocket and issues
commands.

## Configuration

| Option | Default | Description |
| --- | --- | --- |
| `log_level` | `info` | `debug`, `info`, `warn`, or `error`. |
| `poll_interval_seconds` | `300` | How often to poll vacuum status. |
| `map_poll_interval_seconds` | `5` | How often to re-render the live map **while cleaning** (minimum 2; `0` disables live map streaming). |

Credentials (Mi account username/password/region) are **not** add-on options —
they are sent securely by the integration during setup and stored in
`/data/credentials.json` (plus the auth session in `/data/session.json`), both
mode `0600`.

## Ports

The websocket and HTTP API listen on internal port `8080` (mapped to a
configurable host port). The integration connects to
`ws://<add-on-hostname>:8080/ws`.

## Endpoints

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for the full websocket protocol
and HTTP API. In short:

- `GET /ws` — websocket: device snapshots, `command_status`, and `map_frame`
  events.
- `GET /healthz` — readiness probe.
- `POST /auth/config`, `POST /auth/send_code`, `POST /auth/verify?code=…`,
  `GET /auth/status`, `POST /auth/refresh_devices` — onboarding + 2FA.
- `POST /device/{did}/{action}` — `start`, `stop`, `pause`, `dock`, `locate`,
  `set_fan_speed`, `clean_rooms`, `refresh_map`, `poll`, `rpc`.
- `GET /device/{did}/status`, `GET /device/{did}/rooms`,
  `GET /device/{did}/map.png` — read-only data + the rendered map image.

## A note on the upstream library

`go-xiaomi-home` is documented as **safe for sequential use only** (it is not
goroutine-safe). The add-on therefore serializes *every* call into the client
through a single mutex, including the live-map renderer — so concurrent commands,
polls, and map downloads never race the shared session.

[miot]: https://github.com/XiaoMi/ha_xiaomi_home
[lib]: https://github.com/alex-savin/go-xiaomi-home
[integration]: https://github.com/alex-savin/hassio-integration-xiaomi
