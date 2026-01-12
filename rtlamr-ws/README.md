# RTLAMR WebSocket Add-on for Home Assistant

[![CI](https://github.com/alex-savin/hassio-addon-rtlamr-ws/actions/workflows/ci.yaml/badge.svg)](https://github.com/alex-savin/hassio-addon-rtlamr-ws/actions/workflows/ci.yaml)
[![GitHub Release](https://img.shields.io/github/v/release/alex-savin/hassio-addon-rtlamr-ws)](https://github.com/alex-savin/hassio-addon-rtlamr-ws/releases)
[![License](https://img.shields.io/github/license/alex-savin/hassio-addon-rtlamr-ws)](LICENSE)

A Home Assistant add-on that reads utility meters (water, gas, electric) using an RTL-SDR dongle and broadcasts readings over WebSocket. Built in Go for high performance and low resource usage.

## Features

- 📡 **RTL-SDR Integration** - Uses rtl_tcp and rtlamr to decode AMR (Automatic Meter Reading) signals
- 🔌 **WebSocket API** - Real-time meter readings broadcast to connected clients
- 🏠 **Home Assistant Native** - Designed as a Home Assistant add-on with full integration
- 📊 **Multiple Protocols** - Supports IDM, NetIDM, R900, R900BCD, SCM, and SCM+ meter protocols
- 💧 **Multi-Meter Support** - Configure multiple meters (water, gas, electric) simultaneously
- 🔄 **Auto-Reconnect** - Resilient process management with automatic restart on failure
- ❤️ **Health Monitoring** - Built-in health check endpoint for reliability
- 🏗️ **Multi-Architecture** - Supports amd64 and aarch64 (ARM64)

## Requirements

- Home Assistant OS or Supervised installation
- RTL-SDR USB dongle (e.g., RTL2832U-based)
- Compatible utility meters using AMR technolorgy

## Installation

### Add Repository

1. Navigate to **Settings** → **Add-ons** → **Add-on Store**
2. Click the three dots (⋮) → **Repositories**
3. Add: `https://github.com/alex-savin/hassio-apps`
4. Find "RTLAMR WebSocket" and click **Install**

### Manual Installation

```bash
# Clone the repository
git clone https://github.com/alex-savin/hassio-addon-rtlamr-ws.git

# Build locally
docker build -t rtlamr-ws .
```

## Configuration

Configure the add-on through the Home Assistant UI or edit `options.json`:

```yaml
general:
  sleep_for: 60          # Seconds between reading cycles
  verbosity: info        # Log level: debug|info|warning|critical|none
  device_id: "0"         # RTL-SDR device index
  rtltcp_host: ""        # Optional: external rtl_tcp server (host:port)

custom_parameters:
  rtltcp: "-s 2048000"   # Additional rtl_tcp parameters
  rtlamr: "-unique=true" # Additional rtlamr parameters

meters:
  - id: 12345678         # Meter ID (found on your meter or discovered via rtlamr)
    protocol: r900       # Protocol: idm|netidm|r900|r900bcd|scm|scm+
    name: water          # Friendly name
    format: "######.###" # Optional: format reading (e.g., add decimal point)
    unit_of_measurement: "gal"
    device_class: water  # none|current|energy|gas|power|water
    state_class: total_increasing
```

### Finding Your Meter ID

Run rtlamr in discovery mode to find nearby meters:

```bash
rtlamr -msgtype=all -unique=true
```

Look for your meter's ID in the output and note the protocol type.

## API Endpoints

| Endpoint | Description |
|----------|-------------|
| `GET /ws` | WebSocket connection for real-time readings |
| `GET /health` | Health check (returns 200 OK) |
| `GET /metrics` | Prometheus metrics (if enabled) |

### WebSocket Messages

Readings are broadcast as JSON:

```json
{
  "meter_id": "12345678",
  "value": "1234.567",
  "attributes": {
    "meter_name": "water",
    "protocol": "r900",
    "unit_of_measurement": "gal",
    "device_class": "water",
    "state_class": "total_increasing"
  }
}
```

## Project Structure

```
├── cmd/rtlamr-ws/        # Go WebSocket server
│   └── main.go
├── rootfs/               # Container filesystem
│   └── etc/
│       ├── cont-init.d/  # S6 initialization scripts
│       └── services.d/   # S6 service definitions
├── .github/
│   └── workflows/        # CI/CD pipelines
├── Dockerfile            # Multi-stage build
├── config.yaml           # Add-on configuration schema
├── go.mod                # Go module definition
└── build.sh              # Local build script
```

## Development

### Prerequisites

- Go 1.25+
- Docker
- RTL-SDR tools (for local testing)

### Building

```bash
# Build binary only
./build.sh

# Build Docker image
BUILD_IMAGE=1 ./build.sh

# Run tests
go test -v ./...

# Lint
golangci-lint run
```

### Local Testing

```bash
# Start rtl_tcp (in a separate terminal)
rtl_tcp -a 127.0.0.1

# Run the server
go run ./cmd/rtlamr-ws -config=test-config.yaml

# Connect via WebSocket
websocat ws://localhost:8080/ws
```

## Troubleshooting

### No Readings

1. Verify RTL-SDR is connected: `lsusb | grep RTL`
2. Check the correct meter ID and protocol
3. Ensure you're within range of the meter
4. Try different rtlamr parameters: `-gainbyindex=29`

### USB Device Not Found

Add the RTL-SDR device to Home Assistant:
1. **Settings** → **System** → **Hardware**
2. Find your RTL-SDR and note the device path
3. Ensure the add-on has USB access enabled

### Permission Denied

The add-on requires:
- USB access (`usb: true`)
- UART access (`uart: true`)
- udev access (`udev: true`)

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- [rtlamr](https://github.com/bemasher/rtlamr) - AMR packet decoder
- [rtl-sdr](https://osmocom.org/projects/rtl-sdr/wiki) - RTL-SDR library
- [Home Assistant](https://www.home-assistant.io/) - Home automation platform

## Support

- 🐛 [Report a Bug](https://github.com/alex-savin/hassio-addon-rtlamr-ws/issues/new?template=bug_report.md)
- 💡 [Request a Feature](https://github.com/alex-savin/hassio-addon-rtlamr-ws/issues/new?template=feature_request.md)
- 💬 [Discussions](https://github.com/alex-savin/hassio-addon-rtlamr-ws/discussions)
