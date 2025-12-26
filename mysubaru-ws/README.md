# MySubaru Websocket Add-on

Exposes MySubaru vehicle state over websocket for the Home Assistant integration.

## Usage
- Configure `log_level` and `poll_interval_seconds` in the add-on options.
- Map the host port as needed; container listens on 8080.
- Credentials are supplied by the Home Assistant integration, which POSTs to `/auth/config`.

## Image
- Default image tag: `local/mysubaru-ws-addon:latest` (built from the main MySubaru source repo).
- If you push to a registry or use a different tag, update `config.yaml` `image:` accordingly.

## Build (from source repo)
Run in the main MySubaru source repository that contains the Go code:

```bash
BUILD_IMAGE=1 TARGETARCH=amd64 IMAGE_TAG=local/mysubaru-ws-addon:latest ./hassio-addon-mysubaru/build.sh
```

For arm64 hosts, set `TARGETARCH=arm64` and use the corresponding base image if overriding `BUILD_FROM`.
