# Emporia Vue Websocket Add-on

[![Add add-on to Home Assistant](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Falex-savin%2Fhassio-apps)
[![GitHub Release](https://img.shields.io/github/v/release/alex-savin/hassio-addon-emporia-vue-ws)](https://github.com/alex-savin/hassio-addon-emporia-vue-ws/releases)
[![License](https://img.shields.io/github/license/alex-savin/hassio-addon-emporia-vue-ws)](LICENSE)

This Home Assistant add-on exposes Emporia Vue energy monitor data over a WebSocket interface.

## Requirements

This add-on requires the [Emporia Vue Custom Integration](https://github.com/alex-savin/hassio-integration-emporia-vue) to be installed in Home Assistant. The add-on acts as a websocket bridge to the Emporia API, while the integration provides the Home Assistant entities (sensors for energy usage, power consumption per circuit, etc.) that consume the data.

**Installation order:**
1. Install this add-on and start it
2. Install the [Emporia Vue Custom Integration](https://github.com/alex-savin/hassio-integration-emporia-vue) via HACS or manually
3. Configure the integration to connect to this add-on's websocket endpoint

## Features

- **Push-based updates**: The add-on polls the Emporia API and broadcasts snapshots to all connected clients
- **Persistent authentication**: Credentials are stored and reused across restarts
- **Device and channel data**: Provides both device-level and per-circuit usage information
- **Outlet/EVSE control**: Toggle smart outlets and EV chargers

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

Connect to `ws://<addon-ip>:8080/ws` and authenticate:

```json
{"type": "authenticate", "username": "your-emporia-email", "password": "your-emporia-password"}
```

## Documentation

- [Full Documentation](DOCS.md)
- [Architecture](docs/ARCHITECTURE.md)
- [Changelog](CHANGELOG.md)

## Building

```bash
# Build binary only
./build.sh

# Build Docker image
BUILD_IMAGE=1 TARGETARCH=amd64 ./build.sh
```

## Support

- [Report an issue](https://github.com/alex-savin/hassio-addon-emporia-vue-ws/issues)

