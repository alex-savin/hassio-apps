# RTLAMR WebSocket Add-on

This add-on reads utility meters (water, gas, electric) using an RTL-SDR dongle and broadcasts readings over WebSocket for integration with Home Assistant.

## How it Works

The add-on uses three components working together:

1. **rtl_tcp** - Provides network access to your RTL-SDR dongle
2. **rtlamr** - Decodes AMR (Automatic Meter Reading) radio signals
3. **rtlamr-ws** - Broadcasts decoded readings via WebSocket

## Requirements

- RTL-SDR USB dongle (RTL2832U-based)
- Utility meters using AMR technology
- Home Assistant OS or Supervised installation

## Configuration

### General Settings

| Option | Default | Description |
|--------|---------|-------------|
| `sleep_for` | `60` | Seconds between reading cycles |
| `verbosity` | `info` | Log level: `debug`, `info`, `warning`, `critical`, `none` |
| `device_id` | `"0"` | RTL-SDR device index |
| `rtltcp_host` | `""` | External rtl_tcp server (optional) |

### Custom Parameters

| Option | Description |
|--------|-------------|
| `rtltcp` | Additional rtl_tcp command-line arguments |
| `rtlamr` | Additional rtlamr command-line arguments |

### Meter Configuration

Each meter requires:

| Option | Required | Description |
|--------|----------|-------------|
| `id` | Yes | Meter ID number |
| `protocol` | Yes | Protocol type (see below) |
| `name` | Yes | Friendly name |
| `format` | No | Reading format (e.g., `"######.###"`) |
| `unit_of_measurement` | No | Unit: `gal`, `kWh`, `CCF`, etc. |
| `device_class` | No | HA class: `water`, `energy`, `gas` |
| `state_class` | No | HA state: `total_increasing` |

### Supported Protocols

| Protocol | Description |
|----------|-------------|
| `idm` | Interval Data Message |
| `netidm` | Network IDM |
| `r900` | Badger Orion (water meters) |
| `r900bcd` | R900 with BCD encoding |
| `scm` | Standard Consumption Message |
| `scm+` | Enhanced SCM |

## Example Configuration

```yaml
general:
  sleep_for: 60
  verbosity: info
  device_id: "0"

custom_parameters:
  rtlamr: "-unique=true"

meters:
  - id: 12345678
    protocol: r900
    name: water
    format: "######.###"
    unit_of_measurement: "gal"
    device_class: water
    state_class: total_increasing
```

## Finding Your Meter ID

To discover meters in range, check the add-on logs after starting. The rtlamr tool will output all detected meters. Look for your meter's ID (usually printed on the meter itself) and note the protocol type.

## API Endpoints

| Endpoint | Description |
|----------|-------------|
| `/ws` | WebSocket for real-time readings |
| `/health` | Health check (returns 200 OK) |
| `/status` | Service status information |

## WebSocket Messages

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

## Troubleshooting

### No Readings
- Verify RTL-SDR is connected and visible in Hardware settings
- Check meter ID and protocol are correct
- Ensure you're within range of the meter (100-500m typical)
- Try adjusting gain: `custom_parameters.rtlamr: "-gainbyindex=29"`

### USB Device Not Found
- Go to **Settings** → **System** → **Hardware**
- Verify the RTL-SDR device is listed
- Restart the add-on after plugging in the device

### Permission Issues
The add-on requires USB, UART, and udev access which are configured automatically.

## Support

- [Documentation](https://github.com/alex-savin/hassio-addon-rtlamr-ws/tree/main/docs)
- [Report Issues](https://github.com/alex-savin/hassio-addon-rtlamr-ws/issues)
- [Discussions](https://github.com/alex-savin/hassio-addon-rtlamr-ws/discussions)
