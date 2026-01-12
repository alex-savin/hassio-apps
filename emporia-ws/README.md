# Emporia Vue Websocket Add-on

[![GitHub Release][releases-shield]][releases]
[![License][license-shield]](LICENSE)

This Home Assistant add-on exposes Emporia Vue energy monitor data over a WebSocket interface.

## Features

- **Push-based updates**: The add-on polls the Emporia API and broadcasts snapshots to all connected clients
- **Persistent authentication**: Credentials are stored and reused across restarts
- **Device and channel data**: Provides both device-level and per-circuit usage information
- **Outlet/EVSE control**: Toggle smart outlets and EV chargers
- **Multi-language support**: Translated into 8 languages

## Installation

1. Add this repository to your Home Assistant add-on store
2. Install the "Emporia Vue Websocket" add-on
3. Start the add-on
4. Connect to the WebSocket endpoint and authenticate with your Emporia credentials

## Configuration

| Option | Description | Default |
|--------|-------------|---------|
| `log_level` | Logging verbosity (debug, info, warn, error) | `info` |
| `poll_interval_seconds` | How often to poll Emporia API (in seconds) | `60` |

## WebSocket API

Connect to `ws://<addon-ip>:8080/ws`

### Endpoints

| Endpoint | Description |
|----------|-------------|
| `/ws` | WebSocket connection |
| `/health` | Health check endpoint |

### Messages

**Hello** (server → client):
```json
{"type": "hello", "version": "1.0.0", "capabilities": ["usage", "control", "channels", "push"]}
```

**Authenticate** (client → server):
```json
{"type": "authenticate", "username": "your-emporia-email", "password": "your-emporia-password"}
```

**Auth Result** (server → client):
```json
{"type": "auth_result", "success": true}
```

**Snapshot** (server → client, push):
```json
{
  "type": "snapshot",
  "timestamp": "2026-01-12T12:00:00Z",
  "devices": [...],
  "usage": {...}
}
```

**Control** (client → server):
```json
{"type": "control", "device_gid": 12345, "outlet_on": true}
```

## Building

```bash
# Build binary only
./build.sh

# Build Docker image
BUILD_IMAGE=1 TARGETARCH=amd64 ./build.sh
```

## Support

- [Documentation](docs/websocket-addon.md)
- [Changelog](CHANGELOG.md)
- [Report an issue](https://github.com/alex-savin/hassio-addon-emporia-vue-ws/issues)

[releases-shield]: https://img.shields.io/github/v/release/alex-savin/hassio-addon-emporia-vue-ws
[releases]: https://github.com/alex-savin/hassio-addon-emporia-vue-ws/releases
[license-shield]: https://img.shields.io/github/license/alex-savin/hassio-addon-emporia-vue-ws

