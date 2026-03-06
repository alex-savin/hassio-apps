# RTLAMR WebSocket Add-on for Home Assistant

[![CI](https://github.com/alex-savin/hassio-app-rtlamr-ws/actions/workflows/ci.yaml/badge.svg)](https://github.com/alex-savin/hassio-app-rtlamr-ws/actions/workflows/ci.yaml)
[![GitHub Release](https://img.shields.io/github/v/release/alex-savin/hassio-app-rtlamr-ws)](https://github.com/alex-savin/hassio-app-rtlamr-ws/releases)
[![License](https://img.shields.io/github/license/alex-savin/hassio-app-rtlamr-ws)](LICENSE)

A Home Assistant add-on that reads utility meters (water, gas, electric) using an RTL-SDR dongle and broadcasts readings over WebSocket. Built in Go for high performance and low resource usage.

## Features

- 📡 **RTL-SDR Integration** - Uses rtl_tcp and rtlamr to decode AMR (Automatic Meter Reading) signals
- 🔌 **WebSocket API** - Real-time meter readings broadcast to connected clients
- 🏠 **Home Assistant Native** - Designed as a Home Assistant add-on with full integration
- 📊 **Multiple Protocols** - Supports IDM, NetIDM, R900, R900BCD, SCM, and SCM+ meter protocols
- 💧 **Multi-Meter Support** - Configure multiple meters (water, gas, electric) simultaneously
- 🔄 **Auto-Reconnect** - Resilient process management with automatic restart on failure
- ❤️ **Health Monitoring** - Built-in health check endpoint for reliability
- 🏗️ **Multi-Architecture** - Supports amd64 and aarch64 (ARM64)

## Requirements

- Home Assistant OS or Supervised installation
- RTL-SDR USB dongle (e.g., RTL2832U-based)
- Compatible utility meters using AMR technology

## Quick Start

### Installation

1. Navigate to **Settings** → **Add-ons** → **Add-on Store**
2. Click the three dots (⋮) → **Repositories**
3. Add: `https://github.com/alex-savin/hassio-apps`
4. Find "RTLAMR WebSocket" and click **Install**

### Basic Configuration

```yaml
general:
  sleep_for: 60
  verbosity: info
  device_id: "0"

meters:
  - id: 12345678
    protocol: r900
    name: water
    format: "######.###"
    unit_of_measurement: "gal"
    device_class: water
    state_class: total_increasing
```

## Documentation

- [Configuration Guide](docs/CONFIGURATION.md) - Detailed configuration options
- [API Reference](docs/API.md) - WebSocket and REST API documentation
- [Architecture](docs/ARCHITECTURE.md) - Technical architecture overview
- [Troubleshooting](docs/TROUBLESHOOTING.md) - Common issues and solutions
- [Development](docs/DEVELOPMENT.md) - Building and contributing

## API Endpoints

| Endpoint | Description |
|----------|-------------|
| `GET /ws` | WebSocket connection for real-time readings |
| `GET /health` | Health check (returns 200 OK) |
| `GET /status` | Service status information |

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- [rtlamr](https://github.com/bemasher/rtlamr) - AMR packet decoder
- [rtl-sdr](https://osmocom.org/projects/rtl-sdr/wiki) - RTL-SDR library
- [Home Assistant](https://www.home-assistant.io/) - Home automation platform

## Support

- 🐛 [Report a Bug](https://github.com/alex-savin/hassio-app-rtlamr-ws/issues/new?template=bug_report.md)
- 💡 [Request a Feature](https://github.com/alex-savin/hassio-app-rtlamr-ws/issues/new?template=feature_request.md)
- 💬 [Discussions](https://github.com/alex-savin/hassio-app-rtlamr-ws/discussions)
