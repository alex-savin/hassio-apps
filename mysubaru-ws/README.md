# MySubaru Websocket App

Home Assistant app that exposes MySubaru vehicle state over a websocket for integration with Home Assistant.

## Features

- Real-time vehicle status via websocket
- Configurable polling intervals for vehicle and location data
- Full vehicle control: lock/unlock, remote start/stop, horn, lights, EV charge
- Safety alerts: geofence, speed fence, curfew with activate/deactivate controls
- Valet mode management
- Trip tracking: start/stop logging, retrieve history
- POI/destination management: send to vehicle nav, manage favorites
- EV charge schedule management
- Climate preset management
- Vehicle information: recalls, warning lights, model info
- Roadside assistance requests
- Cancel any pending command
- AppArmor security profile for hardened container security
- s6-overlay service management

## Configuration

| Option | Description | Default |
|--------|-------------|---------|
| `log_level` | Log verbosity (debug, info, warn, error) | `info` |
| `poll_interval_seconds` | How often to poll vehicle status | `300` (5 min) |
| `location_poll_interval_seconds` | How often to update GPS location (0 to disable) | `300` (5 min) |

## Endpoints

### Authentication & Configuration

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/healthz` | GET | Liveness check |
| `/ws` | GET | WebSocket vehicle status stream |
| `/auth/config` | POST | Set credentials |
| `/auth/status` | GET | Check auth state |
| `/auth/send_code` | POST | Request verification code |
| `/auth/verify` | POST | Verify code (`?code=123456`) |
| `/auth/refresh_vehicles` | POST | Force refresh vehicle list |

### Vehicle Commands (`POST /vehicle/{vin}/{action}`)

Lock/Unlock, Remote Start/Stop, Horn, Lights, EV Charge, Valet Mode, GeoFence, Speed Fence, Curfew, Trip Log, POI, Climate Presets, Roadside Assistance, and Cancel operations.

### Vehicle Data (`GET /vehicle/{vin}/{action}`)

Trips, Valet Status/Settings, GeoFence/Speed Fence/Curfew Settings, EV Charge Settings, Recalls, Warning Lights, Roadside Assistance Info, Favorite POIs, Model Info.

See [ARCHITECTURE.md](docs/ARCHITECTURE.md) for the complete endpoint reference.

## Port Configuration

The app listens on internal port `8080`. Map any host port in Supervisor, then configure your integration with `ws://<ha-host>:<host_port>/ws`.

## Documentation

See [DOCS.md](DOCS.md) for development setup, building, and architecture details.
