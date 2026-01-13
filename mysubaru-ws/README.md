# MySubaru Websocket Add-on

[![Add add-on to Home Assistant](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Falex-savin%2Fhassio-apps)
[![GitHub Release](https://img.shields.io/github/v/release/alex-savin/hassio-addon-mysubaru-ws)](https://github.com/alex-savin/hassio-addon-mysubaru-ws/releases)
[![License](https://img.shields.io/github/license/alex-savin/hassio-addon-mysubaru-ws)](LICENSE)

Home Assistant add-on that exposes MySubaru vehicle state over a websocket for integration with Home Assistant.

## Requirements

This add-on requires the [MySubaru Custom Integration](https://github.com/alex-savin/hassio-integration-mysubaru) to be installed in Home Assistant. The add-on acts as a websocket bridge to the MySubaru API, while the integration provides the Home Assistant entities (sensors, device trackers, etc.) that consume the data.

**Installation order:**
1. Install this add-on and start it
2. Install the [MySubaru Custom Integration](https://github.com/alex-savin/hassio-integration-mysubaru) via HACS or manually
3. Configure the integration to connect to this add-on's websocket endpoint

## Features

- Real-time vehicle status via websocket
- Configurable polling intervals for vehicle and location data
- AppArmor security profile for hardened container security
- s6-overlay service management

## Installation

1. Add this repository to your Home Assistant add-on store
2. Install the "MySubaru Websocket" add-on
3. Start the add-on
4. Configure the integration to connect to `ws://<ha-host>:8080/ws`

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

The add-on listens on internal port `8080`. Map any host port in Supervisor, then configure your integration with `ws://<ha-host>:<host_port>/ws`.

## Documentation

See [DOCS.md](DOCS.md) for development setup, building, and architecture details.
