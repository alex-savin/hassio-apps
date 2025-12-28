# RTLAMR2MQTT Go Add-on

[![GitHub Release][releases-shield]][releases]
[![License][license-shield]](LICENSE)

![Supports amd64 Architecture][amd64-shield]
![Supports aarch64 Architecture][aarch64-shield]

Read utility meters (water, gas, electricity) using an RTL-SDR USB device and send readings directly to Home Assistant via WebSocket.

## About

This add-on is a complete Go rewrite of the rtlamr-ws project. It manages `rtl_tcp` and `rtlamr` processes to decode AMR/ERT signals from utility meters and exposes readings via a WebSocket API on port 8070.

**Key Features:**
- 🚀 Native Go implementation - fast and efficient
- 📡 Direct WebSocket integration with Home Assistant (no MQTT needed)
- 🔄 Automatic process restart on failure
- 📊 Supports multiple meters simultaneously
- 🏷️ Multiple protocols: SCM, SCM+, IDM, NetIDM, R900, R900BCD
- 🔧 Custom parameters for rtl_tcp and rtlamr
- 💤 Configurable sleep intervals between readings
- 🔌 USB device selection for multiple RTL-SDR dongles

## Installation

1. Add this repository to your Home Assistant add-on store
2. Install the "RTLAMR2MQTT Go" add-on
3. Configure your meters (see Configuration section)
4. Start the add-on
5. Install the custom integration (see Integration Setup)

## Integration Setup

To receive readings in Home Assistant:

1. Copy `integration/` folder to `<ha_config>/custom_components/rtlamr_ws/`
2. Restart Home Assistant
3. Go to Settings → Devices & Services → Add Integration
4. Search for "RTLAMR WebSocket Bridge"
5. Configure:
   - Host: `rtlamr-ws` (or your addon container name)
   - Port: `8070`
   - Path: `/ws`

Entities will be created automatically as readings arrive.

## Configuration

### Example Configuration

```yaml
general:
  sleep_for: 60
  verbosity: info
  device_id: "001:010"  # Optional: specific USB device
  rtltcp_host: "127.0.0.1:1234"  # Or remote host

custom_parameters:
  rtltcp: "-s 2048000"
  rtlamr: "-unique=true"

meters:
  - id: 33333333
    protocol: scm+
    name: water_meter
    format: "######.###"
    unit_of_measurement: "m³"
    icon: mdi:water
    device_class: water
    state_class: total_increasing
    
  - id: 22222222
    protocol: r900
    name: electric_meter
    format: "#####.##"
    unit_of_measurement: "kWh"
    icon: mdi:flash
    device_class: energy
    state_class: total_increasing
```

### Options Reference

#### `general`
- `sleep_for` (int, optional): Seconds to sleep after reading all meters (0 = continuous)
- `verbosity` (string): Log level - `debug`, `info`, `warning`, `critical`, `none`
- `device_id` (string, optional): USB device ID in format `XXX:XXX` (from `lsusb`)
- `rtltcp_host` (string, optional): Remote rtl_tcp server `host:port`

#### `custom_parameters`
- `rtltcp` (string): Extra arguments for rtl_tcp
- `rtlamr` (string): Extra arguments for rtlamr

#### `meters` (array, required)
- `id` (int): Meter ID from device
- `protocol` (string): `scm`, `scm+`, `idm`, `netidm`, `r900`, `r900bcd`
- `name` (string): Friendly name
- `format` (string, optional): Number format using `#` for digits (e.g., `"###.##"`)
- `unit_of_measurement` (string): Unit like `m³`, `kWh`, `gal`
- `icon` (string): MDI icon like `mdi:water`
- `device_class` (string): `water`, `gas`, `energy`, `power`
- `state_class` (string): `measurement`, `total`, `total_increasing`

## Finding Your Meter ID

Run the add-on with a test meter entry and check logs. The raw rtlamr output will show all detected meter IDs.

## WebSocket API

The add-on exposes `ws://rtlamr-ws:8070/ws` with messages:

```json
{
  "meter_id": "33333333",
  "reading": 12345.678,
  "last_seen": "2025-12-27T12:00:00Z",
  "attributes": {
    "name": "water_meter",
    "protocol": "scm+",
    "unit_of_measurement": "m³",
    "device_class": "water"
  }
}
```

## Supported Meters

See the [rtlamr compatibility list](https://github.com/bemasher/rtlamr/blob/master/meters.csv)

## Support

- [GitHub Issues](https://github.com/alex-savin/rtlamr-ws/issues)
- [GitHub Discussions](https://github.com/alex-savin/rtlamr-ws/discussions)

[releases-shield]: https://img.shields.io/github/release/alex-savin/rtlamr-ws.svg
[releases]: https://github.com/alex-savin/rtlamr-ws/releases
[license-shield]: https://img.shields.io/github/license/alex-savin/rtlamr-ws.svg
[amd64-shield]: https://img.shields.io/badge/amd64-yes-green.svg
[aarch64-shield]: https://img.shields.io/badge/aarch64-yes-green.svg
