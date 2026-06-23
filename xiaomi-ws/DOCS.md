# Development Documentation

## Run locally

```bash
WS_LISTEN_ADDR=":8080" \
POLL_INTERVAL_SECONDS=300 \
MAP_POLL_INTERVAL_SECONDS=5 \
go build -mod=vendor ./cmd/ws-server && ./ws-server
```

The server expects credentials at runtime via the Home Assistant integration (or
a manual POST for local testing):

```bash
curl -X POST http://localhost:8080/auth/config \
	-H "Content-Type: application/json" \
	-d '{
		"username":"you@example.com",
		"password":"secret",
		"region":"de",
		"device_id":"0123456789abcdef0123456789abcdef"
	}'
```

If the account needs verification you'll get `{"requires_2fa":true}`; then:

```bash
curl -X POST http://localhost:8080/auth/send_code        # -> {"sent":true,"method":"email"}
curl -X POST "http://localhost:8080/auth/verify?code=123456"
```

Hit `http://localhost:8080/healthz` for readiness and `ws://localhost:8080/ws`
for the stream. See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for the full
API.

## Dependency: go-xiaomi-home (private module)

The add-on depends on [`go-xiaomi-home`][lib], which is a **private** Go module.
It is pinned by version in `go.mod` and fetched with credentials — it is **not**
vendored into this repo (so its source is never committed here):

```
require github.com/alex-savin/go-xiaomi-home v0.0.0-20260617173019-6212969d63e2
```

### Local development — no token needed

A gitignored `go.work` points Go at the sibling checkout, so `go build` /
`go test` resolve the dependency from `../go-xiaomi-home` offline:

```
// go.work (not committed)
use (
	.
	../go-xiaomi-home
)
```

If you don't have the sibling checkout, configure git to reach the private repo
(e.g. `git config --global url."git@github.com:".insteadOf "https://github.com/"`)
and set `GOPRIVATE=github.com/alex-savin/*`; Go will then fetch the pinned
version directly.

### CI — SSH deploy key

CI authenticates to the private repo with a **read-only SSH deploy key** scoped
to `go-xiaomi-home`:

1. Generate a keypair: `ssh-keygen -t ed25519 -f deploy -N ""`.
2. Add `deploy.pub` as a **read-only deploy key** on `go-xiaomi-home`
   (Settings → Deploy keys → Add; leave "Allow write access" unchecked).
3. Add the private key (`deploy`) as the **`GO_XIAOMI_HOME_DEPLOY_KEY`** Actions
   secret on this repo.

Each Go lint/test job loads the key, points git at SSH (`insteadOf`), sets
`GOPRIVATE`, and runs directly against the module. The Docker build job
additionally runs `GOWORK=off go mod vendor` to produce a throwaway `vendor/`
tree in the build context — so the container compiles offline with `-mod=vendor`
and **no credential ever lands in a Docker layer**. `vendor/` is gitignored and
exists only for the duration of the CI run.

> Prefer a GitHub App token or a PAT instead? Replace the "Configure SSH" step
> with your token source — e.g. `actions/create-github-app-token` then
> `git config --global url."https://x-access-token:<token>@github.com/".insteadOf "https://github.com/"` —
> and keep `GOPRIVATE`.

### Bumping the pinned version

After `go-xiaomi-home` changes, update the pin (with private-repo access):

```bash
GOPRIVATE='github.com/alex-savin/*' GOWORK=off \
  go get github.com/alex-savin/go-xiaomi-home@main
go mod tidy
```

Or, once `go-xiaomi-home` is tagged, pin a released version:
`go get github.com/alex-savin/go-xiaomi-home@v1.2.3`.

## Project Structure

```
├── apparmor.txt                 # AppArmor security profile (profile: xiaomi_ws)
├── build.sh                     # Build helper script
├── config.yaml                  # Add-on configuration
├── Dockerfile                   # Multi-stage build (-mod=vendor; vendor produced by CI)
├── cmd/ws-server/               # The Go server
│   ├── main.go                  # Config, logging, poll loop, shutdown
│   ├── state.go                 # Auth/session state + serialized client access
│   ├── auth.go                  # /auth/* routes + 2FA flow
│   ├── devices.go               # Vacuum polling + status parsing + snapshots
│   ├── mapcache.go              # Map cache, live-stream manager, map.png
│   ├── commands.go              # /device/{did}/{action} routes
│   ├── hub.go                   # Websocket hub
│   └── retry.go                 # Exponential backoff
├── docs/ARCHITECTURE.md         # Full API reference & architecture
└── rootfs/                      # s6-overlay v3 service tree
```

> `vendor/` and `go.work` are gitignored. `vendor/` is produced on demand
> (CI, or `build.sh`); `go.work` is your local sibling-checkout workspace.

## Building

```bash
# Binary only (inside Docker):
./build.sh

# Full add-on image:
BUILD_IMAGE=1 TARGETARCH=amd64 IMAGE_TAG=local/xiaomi-ws-app-amd64 ./build.sh
```

`init: false` in `config.yaml` keeps s6 `/init` as PID 1. When running the
container outside Supervisor, do **not** override the `/init` entrypoint.

## Tests

```bash
go test -race ./...
```

## Security

The AppArmor profile (`apparmor.txt`, profile `xiaomi_ws`) restricts the
container: network binding for the websocket, `/data` read/write for credential
and session persistence, `/etc/ssl` read for TLS to the cloud, and s6-overlay
service management.

[lib]: https://github.com/alex-savin/go-xiaomi-home
