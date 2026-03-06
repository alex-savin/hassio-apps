# Alex Savin Home Assistant Add-on Repository

A collection of Home Assistant add-ons for various integrations and services.

## Add-ons

| Add-on | Description | Architectures |
|--------|-------------|---------------|
| [Emporia Vue Websocket](emporia-ws/) | Exposes Emporia Vue energy monitor data over websocket for Home Assistant integration | amd64, aarch64 |
| [Gmail Parcels](gmail-parcels/) | Track parcels from Gmail emails using AI | amd64 |
| [MySubaru Websocket](mysubaru-ws/) | Exposes MySubaru vehicle state over websocket for Home Assistant integration | amd64, aarch64 |
| [RTL-SDR AMR Receiver](rtlamr-ws/) | RTL-SDR Automatic Meter Reading Bridge | amd64, aarch64 |
| [UPS Monitor](ups-monitor/) | Expose UPS statuses via go-ups for Home Assistant | amd64, aarch64 |
| [Zigbee2MQTT Instance 01](zigbee2mqtt-01/) | Use your ZigBee devices without the vendor's bridge or gateway (Instance 01) | amd64, aarch64 |
| [Zigbee2MQTT Instance 02](zigbee2mqtt-02/) | Use your ZigBee devices without the vendor's bridge or gateway (Instance 02) | amd64, aarch64 |

## Installation

1. In Home Assistant, go to **Settings → Add-ons → Add-on Store → ⋮ → Repositories**
2. Add this repository URL: `https://github.com/alex-savin/hassio-apps`
3. Find and install the desired add-on from the list
4. Configure the add-on options and start it

## Documentation

Each add-on has its own documentation:

- [Emporia Vue Websocket Docs](emporia-ws/DOCS.md)
- [MySubaru Websocket Docs](mysubaru-ws/DOCS.md)
- [RTL-SDR AMR Receiver Docs](rtlamr-ws/DOCS.md)
- [UPS Monitor Docs](ups-monitor/DOCS.md)
- [Zigbee2MQTT Instance 01 Docs](zigbee2mqtt-01/DOCS.md)
- [Zigbee2MQTT Instance 02 Docs](zigbee2mqtt-02/DOCS.md)

## Notes

- All add-ons listen on port 8080 by default; host port is user-configurable
- Add-ons use `init: false` so s6 `/init` remains PID 1 and avoids `s6-overlay-suexec` errors

## Support

For issues or feature requests, please open an issue on the [GitHub repository](https://github.com/alex-savin/hassio-apps).
