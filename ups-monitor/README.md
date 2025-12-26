# UPS Monitor Home Assistant Add-on

This add-on packages the `go-ups` server for Home Assistant.

## Features
- Starts empty and accepts UPS registrations via the integration (`/api/device`).
- Exposes WebSocket `/ws`, health `/health`, and status `/api/status` endpoints.
- Persists config to `/data/config.yml` inside the add-on container.

## Configuration (add-on options)
```json
{
   "config_path": "/data/config.yml"
}
```

## Ports
- 8080/tcp: go-ups API (must be exposed/mapped if accessed outside Supervisor network).

## Build Notes
- Dockerfile supports multi-arch; Home Assistant build system supplies `BUILD_FROM` and `TARGETARCH`.

## Usage
1. Add this repository as a local add-on repo or build locally: `docker build -t local/ups-monitor -f hassio-addon-ups/Dockerfile .`
2. Start the add-on; it listens on port 8080 with an empty config.
3. In Home Assistant, add the "UPS Monitor" integration and register each UPS (type, name, host, port, credentials, attributes). The integration calls `/api/device`, which validates, persists, and starts polling.
