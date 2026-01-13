# UPS Monitor Home Assistant Add-on

[![Add add-on to Home Assistant](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Falex-savin%2Fhassio-apps)
[![GitHub Release](https://img.shields.io/github/v/release/alex-savin/hassio-addon-ups-monitor)](https://github.com/alex-savin/hassio-addon-ups-monitor/releases)
[![CI](https://github.com/alex-savin/hassio-addon-ups-monitor/actions/workflows/ci.yaml/badge.svg)](https://github.com/alex-savin/hassio-addon-ups-monitor/actions/workflows/ci.yaml)
[![License](https://img.shields.io/github/license/alex-savin/hassio-addon-ups-monitor)](LICENSE)

Real-time UPS monitoring for Home Assistant with support for apcupsd and NUT protocols.

## Requirements

This add-on requires the [UPS Monitor Custom Integration](https://github.com/alex-savin/hassio-integration-ups-monitor) to be installed in Home Assistant. The add-on acts as a websocket bridge to your UPS devices (via apcupsd or NUT), while the integration provides the Home Assistant entities (sensors for battery level, load, runtime, etc.) that consume the data.

**Installation order:**
1. Install this add-on and start it
2. Install the [UPS Monitor Custom Integration](https://github.com/alex-savin/hassio-integration-ups-monitor) via HACS or manually
3. Configure the integration to connect to this add-on's websocket endpoint

## Features

- 🔌 Multi-protocol support (apcupsd, NUT)
- 📡 Real-time WebSocket updates
- 🔄 Smart exponential backoff for network resilience
- 🏗️ Multi-architecture support (amd64, aarch64)

## Installation

1. Add this repository to Home Assistant add-on repositories
2. Install "UPS Monitor" from the add-on store
3. Start the add-on

## Configuration

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `log_level` | string | `info` | Log verbosity: `debug`, `info`, `warn`, `error` |
| `poll_interval_seconds` | int | `10` | How often to poll UPS devices |

## Documentation

See [DOCS.md](DOCS.md) for API reference and configuration details.

## License

MIT License - see [LICENSE](LICENSE) for details.
