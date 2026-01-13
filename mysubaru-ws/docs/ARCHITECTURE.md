# WebSocket Add-on Architecture for Home Assistant

This document describes the Home Assistant add-on that wraps `go-mysubaru` and exposes a WebSocket API for the HA integration to exchange credentials, discover vehicles, and stream state updates.

## Goals
- Run as a lightweight add-on/container alongside Home Assistant.
- Accept credentials (username/password/PIN) from the HA integration.
- Discover vehicles and expose metadata (VIN, doors, locks, climate profiles).
- Push-based updates: broadcast snapshots to all connected clients at configurable intervals.
- Allow control actions (lock/unlock, remote start/stop, EV charge) via HTTP endpoints.

## Components
- **Addon service (Go)**: Wraps `go-mysubaru`, opens a WebSocket server (default `ws://0.0.0.0:8080/ws`). Handles auth, polling, and vehicle commands.
- **HA integration (Python)**: Connects to the socket, sends credentials/config, receives push updates, and maps messages to entities.
- **Storage**: Credentials are persisted to `/data/credentials.json` and reused across restarts.
- **Aux HTTP**: `/healthz` (liveness), `/auth/status` (authentication state), `/auth/config` (credential configuration).

## Current implementation (Jan 2026)
- WebSocket server with push-based vehicle state snapshots.
- Username/password/PIN authentication with 2FA support.
- Credential persistence for seamless restarts.
- Push-based snapshots: server polls MySubaru API and broadcasts to all connected clients.
- Vehicle commands: lock, unlock, remote start/stop, EV charge.
- Location polling with configurable interval.

## WebSocket Protocol (JSON messages)

### Server → Client (Push)
The server broadcasts vehicle state snapshots to all connected WebSocket clients:

```json
{
  "timestamp": "2026-01-12T10:30:00Z",
  "vehicles": [
    {
      "vin": "JF2...",
      "doors": {...},
      "climate_profiles": [...],
      ...
    }
  ]
}
```

### Command Status Broadcasts
When vehicle commands are executed, status updates are broadcast:

```json
{
  "type": "command_status",
  "vin": "JF2...",
  "command": "remote_start",
  "status": "started|finished|error|timeout",
  "message": "",
  "time": "2026-01-12T10:30:00Z"
}
```

## HTTP Endpoints

### Authentication & Configuration

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/healthz` | GET | Liveness check (returns `ok`) |
| `/auth/status` | GET | JSON authentication status |
| `/auth/config` | POST | Configure credentials |
| `/auth/send_code` | POST | Request 2FA verification code |
| `/auth/verify` | POST | Submit 2FA code (`?code=123456`) |
| `/ws` | GET | WebSocket connection |

### Vehicle Commands

All vehicle commands use `POST /vehicle/{vin}/{action}`:

| Action | Description | Request Body |
|--------|-------------|--------------|
| `lock` | Lock vehicle doors | - |
| `unlock` | Unlock vehicle doors | - |
| `remote_start` | Start engine with climate | `{"run_time": 10, "delay": 0, "horn": false, "profile": ""}` |
| `remote_stop` | Stop engine | - |
| `ev_charge` | Start EV charging | - |
| `poll` | Force status refresh with location update | - |

#### Remote Start Options
- `run_time`: 0, 1, 5, or 10 minutes
- `delay`: 0-60 minutes before starting
- `horn`: Honk horn on start
- `profile`: Climate profile name (optional)

## Push-based Polling
- The server maintains a global poll loop that fetches vehicle status from the MySubaru API.
- Poll interval is configurable via `POLL_INTERVAL_SECONDS` environment variable (default: 300s).
- Location poll interval is configurable via `LOCATION_POLL_INTERVAL_SECONDS` (default: 300s, 0 = disabled).
- Snapshots are broadcast to all connected WebSocket clients.
- Newly connected clients receive the last snapshot immediately.

## Data Model
Vehicle data includes:
- **VIN**: Vehicle identification number
- **Doors**: Door states with lock status
- **Climate Profiles**: User and factory climate presets
- **Vehicle Condition**: Battery, fuel, odometer
- **Vehicle Health**: Maintenance alerts
- **Location**: GPS coordinates (when polled)

## Authentication Flow

### Initial Setup
1. Client calls `POST /auth/config` with credentials:
   ```json
   {
     "username": "email@example.com",
     "password": "...",
     "pin": "1234",
     "device_id": "unique-device-id",
     "device_name": "Home Assistant",
     "region": "USA"
   }
   ```
2. If 2FA is required, response includes `"requires_2fa": true`
3. Client calls `POST /auth/send_code` to request verification code
4. Client calls `POST /auth/verify?code=123456` to complete verification
5. On success, vehicles are fetched and snapshots begin

### Credential Persistence
Credentials are stored in `/data/credentials.json` (mode 0600) and loaded on startup for seamless restarts.

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `WS_LISTEN_ADDR` | Listen address | `:8080` |
| `POLL_INTERVAL_SECONDS` | Polling interval for MySubaru API | `300` |
| `LOCATION_POLL_INTERVAL_SECONDS` | GPS location update interval (0 = disabled) | `300` |
| `LOG_LEVEL` | Logging level (debug, info, warn, error) | `info` |

## Retry & Backoff
API operations use exponential backoff for resilience:
- Max attempts: 5
- Initial delay: 1 second
- Max delay: 60 seconds
- Multiplier: 2.0
- Jitter: ±20%

## Operational Notes
- Graceful shutdown on SIGINT/SIGTERM with 5-second grace period.
- WebSocket connections are maintained until client disconnects.
- Last snapshot is cached and sent to newly connected clients.
- Structured logging with request context.

## Future Enhancements
- TLS support for secure WebSocket connections.
- Per-vehicle polling subscriptions.
- WebSocket-based command interface (currently HTTP-only).
- Token-based authentication for WebSocket connections.
