# UPS Monitor Documentation

## Overview

The UPS Monitor add-on provides real-time monitoring of UPS devices using the
apcupsd and NUT protocols. It exposes a REST API and a WebSocket endpoint for
integration with Home Assistant.

## Configuration

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `log_level` | string | `info` | Log verbosity: `debug`, `info`, `warn`, `error` |
| `poll_interval_seconds` | int (1–3600) | `10` | How often to poll UPS devices for status |

## Ports

- **8080/tcp**: REST API and WebSocket server (host port configurable in the add-on UI).

## REST API

All responses are JSON.

### Health check

```http
GET /healthz
```

```json
{ "status": "healthy", "timestamp": "2026-01-13T12:00:00Z" }
```

### Get all UPS statuses

```http
GET /api/status
```

Returns the current status of every registered device:

```json
[
  {
    "device_name": "office_ups",
    "type": "apcupsd",
    "attributes": {
      "status": "OL",
      "battery_charge": "100",
      "time_left": "3600",
      "input_voltage": "120.0",
      "load_percentage": "25"
    }
  }
]
```

> Attribute keys are normalized to a unified schema (e.g. `status`,
> `battery_charge`, `input_voltage`, `load_percentage`, `time_left`). The exact
> set depends on what the UPS reports and on `selected_attributes` (see below).
> There is no top-level `online` field — connectivity is reflected by whether a
> device appears with fresh attributes.

### List registered devices

```http
GET /api/devices
```

Returns the configured devices (without live status).

### Register or update a UPS device

```http
POST /api/device
Content-Type: application/json
```

```json
{
  "name": "office_ups",
  "type": "nut",
  "host": "10.10.10.51",
  "port": 3493,
  "username": "",
  "password": "",
  "selected_attributes": []
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Unique identifier for the device. For NUT this **must match the UPS name configured on the NUT server**. |
| `type` | string | Yes | Protocol: `apcupsd` or `nut` |
| `host` | string | Yes | IP address or hostname |
| `port` | int | No | Defaults to `3551` (apcupsd) or `3493` (NUT) |
| `username` | string | No | NUT username (for authenticated NUT servers) |
| `password` | string | No | NUT password |
| `selected_attributes` | string[] | No | Restrict reported attributes to this set (identity fields such as model/serial are always included) |

The device is validated by connecting to it before being saved; an unreachable
device returns `400`. Posting an existing `name` updates that device in place
and restarts its poller.

### Delete a UPS device

```http
DELETE /api/device?name=office_ups
```

Stops the device poller and removes the device from the configuration.

### Test a device connection

```http
POST /api/device/test
Content-Type: application/json
```

Same body as registration. Connects to the device without saving it and returns
the attributes it exposes:

```json
{ "success": true, "attributes": { "status": "OL", "battery_charge": "100" } }
```

On failure it returns `200` with `{ "success": false, "error": "..." }`.

### List supported commands

```http
GET /api/commands?type=nut
```

Returns the commands supported for the given device type (`nut` or `apcupsd`;
defaults to `nut`):

```json
[
  { "name": "beeper.mute", "supported": true },
  { "name": "test.battery.start", "supported": true }
]
```

> NUT supports control commands (beeper, battery test, calibration, load,
> shutdown). apcupsd command execution is not implemented and reports
> `"supported": false`.

### Execute a command

```http
POST /api/command
Content-Type: application/json
```

```json
{ "id": 1, "device": "office_ups", "command": "beeper.mute" }
```

Response:

```json
{ "id": 1, "success": true, "message": "<human-readable result>" }
```

`device` and `command` are required; missing fields or a failed command return
`400`.

## WebSocket

Connect to `ws://<host>:8080/ws` for real-time updates.

On connect the server immediately sends the most recent status snapshot, then
broadcasts the full status array on a fixed interval (half the poll interval).
The broadcast message format matches `GET /api/status`:

```json
[
  { "device_name": "office_ups", "type": "apcupsd", "attributes": { "status": "OL" } }
]
```

### Sending commands over the WebSocket

The socket is bidirectional. Send a command frame and receive a result frame:

```json
// client -> server
{ "id": 1, "device": "office_ups", "command": "beeper.mute" }

// server -> client
{ "id": 1, "success": true, "message": "<human-readable result>" }
```

The connection uses periodic ping/pong keepalives; idle sockets are closed after
the read timeout.

### Common UPS status codes

| Status | Description |
|--------|-------------|
| `OL` | Online (utility power) |
| `OB` | On battery |
| `LB` | Low battery |
| `CHRG` | Charging |
| `DISCHRG` | Discharging |

## Network resilience

The add-on uses exponential backoff with jitter for unreachable devices:

- **Retry interval**: starts at the configured poll interval and grows up to a
  5-minute maximum.
- **Recovery**: reconnects automatically when a device becomes reachable again.
- **Log reduction**: the first 3 failures log at ERROR, failures 4–10 at WARN,
  and prolonged outages drop to DEBUG with a periodic WARN summary.
