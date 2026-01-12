# UPS Monitor Home Assistant Add-on

[![CI](https://github.com/alex-savin/hassio-addon-ups-monitor/actions/workflows/ci.yaml/badge.svg)](https://github.com/alex-savin/hassio-addon-ups-monitor/actions/workflows/ci.yaml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

This add-on packages the `go-ups-monitor` server for Home Assistant, providing real-time UPS monitoring capabilities.

## Features

- 🔌 Accepts UPS registrations via the integration (`/api/device`)
- 📡 Real-time WebSocket updates (`/ws`)
- ❤️ Health check endpoint (`/healthz`)
- 📊 Status API endpoint (`/api/status`)
- 💾 Persists configuration to `/data/config.yml`
- 🏗️ Multi-architecture support (amd64, aarch64)

## Installation

### From GitHub Container Registry

1. Add this repository as a Home Assistant add-on repository
2. Find "UPS Monitor" in the add-on store and install it
3. Configure the add-on options and start it

### Local Build

```bash
docker build -t local/ups-monitor .
```

## Configuration

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `log_level` | string | `info` | Log verbosity: `debug`, `info`, `warn`, `error` |
| `poll_interval_seconds` | int | `10` | How often to poll UPS devices for status |

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/ws` | WebSocket | Real-time UPS status updates |
| `/healthz` | GET | Health check |
| `/api/status` | GET | Current status of all UPS devices |
| `/api/device` | POST | Register a new UPS device |

## Ports

- **8080/tcp**: API server (must be exposed if accessed outside Supervisor network)

## Translations

The add-on UI is available in multiple languages:
- 🇺🇸 English, 🇩🇪 German, 🇪🇸 Spanish, 🇫🇷 French, 🇮🇹 Italian
- 🇳🇱 Dutch, 🇵🇹 Portuguese, 🇷🇺 Russian, 🇺🇦 Ukrainian, 🇨🇳 Chinese (Simplified)

Translation files are located in the `translations/` folder.

## Development

### Prerequisites

- Go 1.25+
- Docker (for building the add-on)

### Build

```bash
# Build Go binary
go build -o ups-server ./cmd/ups-server

# Run tests
go test -v ./...

# Build Docker image
docker build -t ups-monitor .
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
