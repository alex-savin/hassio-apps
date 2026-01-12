# MySubaru Websocket Add-on

Home Assistant add-on that exposes MySubaru vehicle state over a websocket for integration with Home Assistant.

## Features

- Real-time vehicle status via websocket
- Configurable polling intervals for vehicle and location data
- AppArmor security profile for hardened container security
- s6-overlay service management

## Run locally

```bash
WS_LISTEN_ADDR=":8080" \
POLL_INTERVAL_SECONDS=300 \
LOCATION_POLL_INTERVAL_SECONDS=300 \
go run ./cmd/ws-server
```

The server expects credentials to be provided at runtime via the Home Assistant integration (or a manual POST for local testing):

```bash
curl -X POST http://localhost:8080/auth/config \
	-H "Content-Type: application/json" \
	-d '{
		"username":"user",
		"password":"pass",
		"pin":"1234",
		"device_id":"device-id",
		"device_name":"device-name",
		"region":"USA"
	}'
```

Hit `http://localhost:8080/health` for readiness and `ws://localhost:8080/ws` for the stream.

- Check auth state: `GET /auth/status`
- Send verification code: `POST /auth/send_code`
- Verify code: `POST /auth/verify?code=123456`

## Project Structure

```
├── apparmor.txt                 # AppArmor security profile
├── build.sh                     # Build helper script
├── config.yaml                  # Add-on configuration
├── Dockerfile                   # Multi-stage build
├── cmd/ws-server/main.go        # WebSocket server entry point
└── rootfs/
    └── etc/
        ├── cont-init.d/
        │   ├── 00-set-env           # Environment setup
        │   └── 10-check-config.sh   # Configuration validation
        └── services.d/ws-server/
            ├── run                  # Service start script
            └── finish               # Service cleanup script
```

## Add-on notes

- Add-on options:
  - `log_level`: Log verbosity (debug, info, warn, error). Default: `info`
  - `poll_interval_seconds`: How often to poll vehicle status. Default: `300` (5 minutes)
  - `location_poll_interval_seconds`: How often to force GPS location update. Default: `300` (5 minutes). Set to `0` to disable.
- Credentials come from the integration during setup.
- The app always listens on internal `:8080`; choose any host port by mapping `<host_port> -> 8080` in Supervisor, then set the integration `ws_url` to `ws://<ha-host>:<host_port>/ws`.
- `init: false` is set in `config.yaml` to ensure s6 `/init` runs as PID 1 (avoids the `s6-overlay-suexec` error).

## Building the binary (uses Docker)

```bash
# Default (linux/amd64) -> ws-server (requires bash)
./build.sh

# Override arch/output
GOARCH=arm64 OUTPUT=/tmp/ws-server ./build.sh

# Drop the binary into add-on data for local runs: /data/ws-server
```

## Building the add-on image

```bash
# Build the image via the helper (example for amd64; uses arch-specific base)
BUILD_IMAGE=1 TARGETARCH=amd64 IMAGE_TAG=local/mysubaru-ws-addon-amd64 ./build.sh

# Or build manually (run from repo root)
docker build \
	--build-arg BUILD_FROM=ghcr.io/home-assistant/amd64-base:latest \
	--build-arg TARGETARCH=amd64 \
	-f Dockerfile \
	-t local/mysubaru-ws-addon-amd64 \
	.
```

Adjust `GOARCH`/`TARGETARCH` to `arm64` for aarch64 hosts (and `BUILD_FROM` to `ghcr.io/home-assistant/arm64-base:latest` if building manually). Tag images with the arch suffix (e.g., `local/mysubaru-ws-addon-arm64`) to match `image: local/mysubaru-ws-addon-{arch}`. If GHCR access is restricted, set `BUILD_FROM` to an accessible base image.

## Running the container locally (optional)

When running outside Home Assistant Supervisor, keep the s6 `/init` entrypoint (do **not** override it) to avoid `s6-overlay-suexec: fatal: can only run as pid 1`.

Example:

```bash
docker run --rm -p 8080:8080 \
	-e LOG_LEVEL=info \
	-e POLL_INTERVAL_SECONDS=300 \
	local/mysubaru-ws-addon-amd64
```

Provide credentials via the integration (or POST to `/auth/config`), same as in HA.

## Security

This add-on includes an AppArmor profile (`apparmor.txt`) that restricts container capabilities:

- Allows network binding for the websocket server
- Grants read/write access to `/config` for configuration
- Restricts access to sensitive `/proc` paths
- Permits s6-overlay service management
