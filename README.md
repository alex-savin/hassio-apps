# MySubaru Websocket Home Assistant Add-on Repository

This repository exposes the MySubaru websocket add-on for Home Assistant. It relies on a Docker image that contains the websocket server; you can build and tag the image locally or publish it to a registry, then point the add-on to that image via `config.yaml`.

## Using in Home Assistant
1. In Home Assistant, go to **Settings → Add-ons → Add-on Store → ⋮ → Repositories** and add this repository URL.
2. Install the **MySubaru Websocket** add-on.
3. Configure options (`log_level`, `poll_interval_seconds`) and map the host port as desired; credentials are provided through the integration, not the add-on.
	- `init: false` is set in `config.yaml` so s6 `/init` remains PID 1 and avoids `s6-overlay-suexec` errors.

## Building the image
- Build and tag the image locally (example): `BUILD_IMAGE=1 TARGETARCH=amd64 IMAGE_TAG=local/mysubaru-ws-addon-amd64 ./hassio-addon-mysubaru/build.sh` (run from the main source repo that contains the Go code). For arm64, use `TARGETARCH=arm64` and tag as `local/mysubaru-ws-addon-arm64`.
- `config.yaml` uses `image: local/mysubaru-ws-addon-{arch}`; ensure your tags include the arch suffix so Supervisor resolves correctly.
- Update `config.yaml` `image:` to match your tag/registry if you push elsewhere.

## Notes
- Container listens on 8080; host port is user-configurable.
- Credentials are set via the Home Assistant integration which POSTs to `/auth/config`.
