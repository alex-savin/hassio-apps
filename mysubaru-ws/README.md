# MySubaru Websocket Add-on

Home Assistant add-on that exposes MySubaru vehicle state over a websocket for integration with Home Assistant.

## Features

- Real-time vehicle status via websocket
- Configurable polling intervals for vehicle and location data
- AppArmor security profile for hardened container security
- s6-overlay service management

## Configuration

| Option | Description | Default |
|--------|-------------|---------|
| `log_level` | Log verbosity (debug, info, warn, error) | `info` |
| `poll_interval_seconds` | How often to poll vehicle status | `300` (5 min) |
| `location_poll_interval_seconds` | How often to update GPS location (0 to disable) | `300` (5 min) |

## Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/health` | GET | Readiness check |
| `/ws` | WS | Vehicle status stream |
| `/auth/config` | POST | Set credentials |
| `/auth/status` | GET | Check auth state |
| `/auth/send_code` | POST | Request verification code |
| `/auth/verify` | POST | Verify code (`?code=123456`) |

## Port Configuration

The add-on listens on internal port `8080`. Map any host port in Supervisor, then configure your integration with `ws://<ha-host>:<host_port>/ws`.

## Documentation

See [DOCS.md](DOCS.md) for development setup, building, and architecture details.
