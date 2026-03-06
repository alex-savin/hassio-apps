# Zigbee2MQTT Instance 02

This is **Instance 02** of the Zigbee2MQTT add-on, running alongside the stock add-on with a separate Zigbee coordinator. Each instance manages its own independent Zigbee network.

## Multi-Instance Setup

> [!IMPORTANT]
> Each Zigbee2MQTT instance **must** use:
> - A **separate Zigbee coordinator** (USB dongle or network adapter)
> - A **unique MQTT base topic** (e.g. `zigbee2mqtt-02`) — set in the Z2M frontend under Settings → MQTT, or in `configuration.yaml`
> - A **unique data path** (pre-configured to `/config/zigbee2mqtt-02`)

### Port Allocation

| Setting | Stock Instance | This Instance (02) |
|---------|---------------|-------------------|
| Slug | `zigbee2mqtt` | `zigbee2mqtt-02` |
| Data path | `/config/zigbee2mqtt` | `/config/zigbee2mqtt-02` |
| Socat host port | 8487 | 8487 |

## Pairing

By default the add-on has `permit_join` set to `false`. To allow devices to join you need to activate this after the add-on has started. Use the [built-in frontend](https://www.zigbee2mqtt.io/information/frontend.html) to achieve this.

## Enabling the Built-in Frontend

Enable `ingress` to have the frontend available in your UI: **Settings → Add-ons → Zigbee2MQTT Instance 02 → Show in sidebar**. See the [Zigbee2MQTT documentation](https://www.zigbee2mqtt.io/information/frontend.html) for more details.

## Configuration

### Onboarding

[Onboarding](https://www.zigbee2mqtt.io/guide/getting-started/#onboarding) allows you to set up Zigbee2MQTT without manually entering details. When starting with a brand new install, the frontend shows a quick setup page.

> [!NOTE]
> Successful detection of adapters may vary based on your setup. You may need to enter [details manually](https://www.zigbee2mqtt.io/guide/configuration/adapter-settings.html#basic-configuration).

> [!TIP]
> You can force onboarding to re-run using the toggle in the add-on configuration page (visible after checking `Show unused optional configuration options`).

### Manual

Configuration required for startup is available from the add-on configuration. Other options can be set via the frontend.

> [!CAUTION]
> Settings in the add-on configuration page take precedence over `configuration.yaml`. Remove them from the add-on config if you want full YAML control.

#### Examples

- **socat** (uses internal port 8487, mapped to host 8487)
  ```yaml
  enabled: false
  master: pty,raw,echo=0,link=/tmp/ttyZ2M,mode=777
  slave: tcp-listen:8487,keepalive,nodelay,reuseaddr,keepidle=1,keepintvl=1,keepcnt=5
  options: "-d -d"
  log: false
  ```
- **mqtt** — set a unique `base_topic` to avoid collisions with other instances
  ```yaml
  server: mqtt://localhost:1883
  user: my_user
  password: "my_password"
  base_topic: zigbee2mqtt-02
  ```
- **serial**
  ```yaml
  adapter: zstack
  port: /dev/serial/by-id/usb-Texas_Instruments_TI_CC2531_USB_CDC___<YOUR_ID>-if00
  ```

## Configuration Backup

The add-on creates a backup of `configuration.yaml` at `$DATA_PATH/configuration.yaml.bk` on startup if no previous backup exists.

## Watchdog

To automatically restart on soft failures (e.g. "adapter disconnected"):

```yaml
watchdog: default
```

Default retry delays: 1min, 5min, 15min, 30min, 60min. Custom delays also supported (e.g. `watchdog: 5,10,30`). See the [watchdog docs](https://www.zigbee2mqtt.io/guide/installation/15_watchdog.html).

## Custom Converters — Halo Smoke Detector

This instance ships with a custom build of `zigbee-herdsman-converters` from the [`feature/halo-smoke-detector`](https://github.com/alex-savin/zigbee-herdsman-converters/tree/feature/halo-smoke-detector) branch, which adds native support for Halo Smart Labs smoke detectors. No external converter files are needed — the support is baked into the Docker image.

## Adding Support for New Devices

See [How to support new devices](https://www.zigbee2mqtt.io/how_tos/how_to_support_new_devices.html).

## Notes

- The MQTT server config may need the port (e.g. `mqtt://core-mosquitto:1883`).
- To find serial ports: **Settings → System → Hardware → All Hardware**.

## Socat

Socat forwards a serial device over TCP. Options:

- `enabled` — true/false (default: false)
- `master` — first address in socat command (mandatory)
- `slave` — second address in socat command (mandatory)
- `options` — extra socat command line options (optional)
- `log` — true/false, logs to `$DATA_PATH/socat.log` (default: false)

> [!NOTE]
> The default socat slave listens on internal port `8487`, mapped to host port `8487`. Adjust the `master` and `slave` options per your needs. The Z2M serial port setting must be changed separately.
