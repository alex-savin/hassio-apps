# hassio-app-gmail-parcels

Home Assistant add-on that packages the
**[go-gmail-parcels](https://github.com/alex-savin/go-gmail-parcels)** service:
it watches Gmail for shipping notifications, extracts parcel data, and exposes a
WebSocket/REST API that the
**[hassio-integration-gmail-parcels](https://github.com/alex-savin/hassio-integration-gmail-parcels)**
component consumes.

The Go service is **not** copied into this repo — it is included as the `src/`
git submodule and built by the Docker image.

## Repo layout

| Path | What |
|------|------|
| `Dockerfile` | Multi-stage build: ONNX Runtime → Go build from `src/` → HA add-on (s6) |
| `config.yaml` | HA add-on manifest (name, version, options schema, ports) |
| `rootfs/` | s6 service: generates the runtime config from add-on options, runs the binary |
| `build.sh` | Convenience wrapper around `docker build` |
| `src/` | **submodule** → go-gmail-parcels (the service source) |

## NER model (downloaded at runtime)

The NER model is **not** baked into the image. On first start the add-on
downloads `model.onnx`, `vocab.txt` and `labels.txt` from the public
[gmail-parcels-model](https://github.com/alex-savin/gmail-parcels-model)
release into `/data/onnx-out` (cached across restarts) and links it to
`/app/onnx-out`. If the download fails, the service falls back to a regex-only
extractor. Override the source with the `MODEL_BASE_URL` env var. This keeps the
image small (~60–80 MB instead of ~300 MB).

## Build prerequisites

Two gitignored artifacts must be present before building:

1. **`src/`** — initialize the submodule:
   ```bash
   git submodule update --init
   ```
2. **`onnxruntime-linux-x64-1.27.0.tgz`** — from
   [microsoft/onnxruntime releases](https://github.com/microsoft/onnxruntime/releases);
   the binary links against ONNX Runtime (the model itself is not needed to build).

## Build

```bash
./build.sh                       # amd64 -> local/gmail-parcels-addon-amd64
TARGETARCH=arm64 ./build.sh
IMAGE_TAG=ghcr.io/alex-savin/gmail-parcels-addon-amd64:0.0.2 ./build.sh
```

## Updating the service

Point the submodule at a newer go-gmail-parcels commit and rebuild:

```bash
cd src && git pull origin main && cd ..
git add src && git commit -m "Bump go-gmail-parcels"
```

## Configuration

Configured through the Home Assistant add-on options (see `config.yaml` schema):
per-account Gmail credentials/token file names, Pub/Sub project/topic/subscription,
optional carrier whitelist, and log level. The s6 `run` script renders these into
the service's `config.yaml` at startup.
