# UPS Monitor Documentation

## Overview

The UPS Monitor add-on provides real-time monitoring of UPS devices using apcupsd and NUT protocols. It exposes a REST API and WebSocket endpoint for integration with Home Assistant.

## Configuration

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `log_level` | string | `info` | Log verbosity: `debug`, `info`, `warn`, `error` |
| `poll_interval_seconds` | int | `10` | How often to poll UPS devices for status |

## API Endpoints

### Health Check

```http
GET /healthz
```

Returns `{"status": "ok"}` when the server is healthy.

### Get All UPS Statuses

```http
GET /api/status
```

Returns the current status of all registered UPS devices:

```json
[
  {
    "name": "cyberpower_01",
    "online": true,
    "attributes": {
      "battery_charge": "100",
      "battery_runtime": "3600",
      "input_voltage": "120.0",
      "ups_load": "25",
      "ups_status": "OL"
    }
  }
]
```

### Register a UPS Device

```http
POST /api/device
Content-Type: application/json
```

Request body:

```json
{
  "name": "cyberpower_01",
  "type": "apcupsd",
  "host": "10.10.10.51",
  "port": 3551,
  "ups_name": ""
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Unique identifier for the UPS |
| `type` | string | Yes | Protocol: `apcupsd` or `nut` |
| `host` | string | Yes | IP address or hostname |
| `port` | int | Yes | Port (3551 for apcupsd, 3493 for NUT) |
| `ups_name` | string | No | UPS name for NUT protocol |

## WebSocket

Connect to `ws://<host>:8080/ws` for real-time updates.

### Message Format

```json
[
  {
    "name": "cyberpower_01",
    "online": true,
    "attributes": {
      "battery_charge": "100",
      "ups_status": "OL"
    }
  }
]
```

### UPS Status Codes

| Status | Description |
|--------|-------------|
| `OL` | Online (utility power) |
| `OB` | On Battery |
| `LB` | Low Battery |
| `CHRG` | Charging |
| `DISCHRG` | Discharging |

## Network Resilience

The add-on implements smart exponential backoff:

- **Retry interval**: 10 seconds to 5 minutes max
- **Connection recovery**: Automatically reconnects when devices become available
- **Log reduction**: Reduces log spam during prolonged outages

## Ports

- **8080/tcp**: API and WebSocket server
