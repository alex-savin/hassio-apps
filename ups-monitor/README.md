# UPS Monitor Home Assistant Add-on

[![CI](https://github.com/alex-savin/hassio-addon-ups-monitor/actions/workflows/ci.yaml/badge.svg)](https://github.com/alex-savin/hassio-addon-ups-monitor/actions/workflows/ci.yaml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Real-time UPS monitoring for Home Assistant with support for apcupsd and NUT protocols.

## Features

- 🔌 Multi-protocol support (apcupsd, NUT)
- 📡 Real-time WebSocket updates
- 🔄 Smart exponential backoff for network resilience
- 🏗️ Multi-architecture support (amd64, aarch64)

## Quick Start

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
