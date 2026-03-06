# Development Documentation

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

## Project Structure

```
├── apparmor.txt                 # AppArmor security profile
├── build.sh                     # Build helper script
├── config.yaml                  # App configuration
├── Dockerfile                   # Multi-stage build
├── cmd/ws-server/main.go        # WebSocket server entry point
├── docs/ARCHITECTURE.md         # Full API reference & architecture
└── rootfs/
    └── etc/
        └── s6-overlay/
            └── s6-rc.d/
                ├── ws-server/       # Service definition
                │   ├── run          # Service start script
                │   ├── finish       # Service cleanup script
                │   ├── type         # Service type (longrun)
                │   └── dependencies.d/
                └── user/
                    └── contents.d/
                        └── ws-server
```

## Technical Notes

- `init: false` is set in `config.yaml` to ensure s6 `/init` runs as PID 1 (avoids the `s6-overlay-suexec` error).
- Credentials come from the integration during setup.

## Building the binary (uses Docker)

```bash
# Default (linux/amd64) -> ws-server (requires bash)
./build.sh

# Override arch/output
GOARCH=arm64 OUTPUT=/tmp/ws-server ./build.sh

# Drop the binary into app data for local runs: /data/ws-server
```

## Building the app image

```bash
# Build the image via the helper (example for amd64; uses arch-specific base)
BUILD_IMAGE=1 TARGETARCH=amd64 IMAGE_TAG=local/mysubaru-ws-app-amd64 ./build.sh

# Or build manually (run from repo root)
docker build \
	--build-arg BUILD_FROM=ghcr.io/home-assistant/amd64-base:latest \
	--build-arg TARGETARCH=amd64 \
	-f Dockerfile \
	-t local/mysubaru-ws-app-amd64 \
	.
```

Adjust `GOARCH`/`TARGETARCH` to `arm64` for aarch64 hosts (and `BUILD_FROM` to `ghcr.io/home-assistant/arm64-base:latest` if building manually). Tag images with the arch suffix (e.g., `local/mysubaru-ws-app-arm64`) to match `image: local/mysubaru-ws-app-{arch}`. If GHCR access is restricted, set `BUILD_FROM` to an accessible base image.

## Running the container locally (optional)

When running outside Home Assistant Supervisor, keep the s6 `/init` entrypoint (do **not** override it) to avoid `s6-overlay-suexec: fatal: can only run as pid 1`.

Example:

```bash
docker run --rm -p 8080:8080 \
	-e LOG_LEVEL=info \
	-e POLL_INTERVAL_SECONDS=300 \
	local/mysubaru-ws-app-amd64
```

Provide credentials via the integration (or POST to `/auth/config`), same as in HA.

## Security

This app includes an AppArmor profile (`apparmor.txt`) that restricts container capabilities:

- Allows network binding for the websocket server
- Grants read/write access to `/config` for configuration
- Restricts access to sensitive `/proc` paths
- Permits s6-overlay service management

## Architecture

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for detailed architecture documentation.
