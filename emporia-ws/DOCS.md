# WebSocket API Reference

This document describes the WebSocket API for the Emporia Vue Add-on.

## Connection

Connect to `ws://<addon-ip>:8080/ws`

## HTTP Endpoints

| Endpoint | Description |
|----------|-------------|
| `/ws` | WebSocket connection |
| `/healthz` | Liveness check (returns `ok`) |
| `/metrics` | Plain-text metrics (connections, messages) |
| `/auth/status` | JSON authentication status |

## Message Format

All messages are JSON with a `type` field. Responses include `request_id` when provided in the request.

## Server → Client Messages

### hello

Sent immediately upon connection.

```json
{
  "type": "hello",
  "version": "1.0.0",
  "capabilities": ["usage", "control", "channels", "push"]
}
```

### auth_ok

Successful authentication response.

```json
{
  "type": "auth_ok",
  "request_id": "..."
}
```

### devices

Response to `get_devices` request.

```json
{
  "type": "devices",
  "request_id": "...",
  "devices": [
    {
      "deviceGid": 12345,
      "model": "Vue2",
      "firmware": "1.2.3",
      "channels": [...],
      "outlet": {...},
      "evCharger": {...}
    }
  ]
}
```

### usage

Response to `get_usage` request.

```json
{
  "type": "usage",
  "request_id": "...",
  "usage": {
    "devices": [...],
    "instant": 1234.56
  }
}
```

### snapshot

Push message broadcast to all authenticated clients at the configured poll interval.

```json
{
  "type": "snapshot",
  "timestamp": "2026-01-12T12:00:00Z",
  "devices": [...],
  "usage": {...}
}
```

### toggle_outlet_ack / toggle_evse_ack

Acknowledgement of control requests with resulting state.

```json
{
  "type": "toggle_outlet_ack",
  "request_id": "...",
  "device_gid": 12345,
  "on": true
}
```

### pong

Response to `ping` request.

```json
{
  "type": "pong",
  "ts": "2026-01-12T12:00:00Z",
  "request_id": "..."
}
```

### error

Error response.

```json
{
  "type": "error",
  "request_id": "...",
  "code": "unauthorized|bad_request|unsupported",
  "message": "Error description"
}
```

## Client → Server Messages

### ping

Keep-alive ping.

```json
{
  "type": "ping"
}
```

### authenticate

Authenticate with Emporia credentials.

```json
{
  "type": "authenticate",
  "request_id": "...",
  "username": "your-emporia-email",
  "password": "your-emporia-password"
}
```

### get_devices

Request device list.

```json
{
  "type": "get_devices",
  "request_id": "..."
}
```

### get_usage

Request usage data. Empty `device_gids` fetches all devices.

```json
{
  "type": "get_usage",
  "request_id": "...",
  "device_gids": [12345],
  "scale": "minute",
  "unit": "KilowattHours"
}
```

### toggle_outlet

Control a smart outlet.

```json
{
  "type": "toggle_outlet",
  "request_id": "...",
  "device_gid": 12345,
  "on": true
}
```

### toggle_evse

Control an EV charger.

```json
{
  "type": "toggle_evse",
  "request_id": "...",
  "device_gid": 12345,
  "on": true,
  "charging_rate": 32,
  "max_charging_rate": 40
}
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `ADDON_ADDR` | Listen address | `0.0.0.0` |
| `ADDON_PORT` | Listen port | `8080` |
| `POLL_INTERVAL_SECONDS` | Polling interval for Emporia API | `60` |
| `LOG_LEVEL` | Logging level (debug, info, warn, error) | `info` |
| `WS_AUTH_TOKEN` | Optional shared secret for WebSocket auth | (none) |
| `CREDENTIALS_FILE` | Path to Emporia API credentials | `/data/credentials.yaml` |

## Security

- **Credential persistence**: Credentials are stored in `/data/user_credentials.yaml` (mode 0600) and loaded on startup.
- **Optional WebSocket shared secret**: Set `WS_AUTH_TOKEN`; the server expects `Authorization: Bearer <token>` or `?token=` query parameter.
- **Emporia API tokens**: Cached in `EMPORIA_CREDENTIALS_FILE` (default `/data/credentials.yaml`).
