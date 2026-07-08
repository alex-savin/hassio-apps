# Gmail Parcels

Watches one or more Gmail mailboxes for carrier shipping notifications,
extracts parcel data (carrier, tracking number, status, estimated delivery,
sender, delivery-proof photo), and exposes it over a WebSocket/REST API for the
companion
[hassio-integration-gmail-parcels](https://github.com/alex-savin/hassio-integration-gmail-parcels)
component.

Detection is two-stage: deterministic regex + carrier checksums for the hard
data (carrier, tracking number), and an internal Go NER model for the soft
fields (status, dates, sender). The NER model is downloaded at runtime into
`/data/onnx-out` (cached across restarts); if the download fails the service
falls back to a regex-only extractor.

## Prerequisites

Mail arrives via **Gmail push notifications over Google Cloud Pub/Sub**, so a
one-time Google Cloud setup per mailbox is required:

1. A Google Cloud project with the **Gmail API** and **Pub/Sub API** enabled.
2. A Pub/Sub **topic** (e.g. `gmail-parcels`) with the publisher role
   `roles/pubsub.publisher` granted to
   `gmail-api-push@system.gserviceaccount.com` (this is what lets Gmail publish
   to the topic — without it `watch` succeeds but no notifications ever arrive).
3. A Pub/Sub **pull subscription** (e.g. `gmail-parcels-sub`) attached to that
   topic. Note: subscriptions with an idle expiration policy are auto-deleted
   after the TTL; if the add-on logs `NotFound (resource=...sub)`, recreate it.
4. An **OAuth client** (Desktop type) credentials JSON, and a **token** JSON
   containing a refresh token, authorized for the `gmail.readonly` and
   `pubsub` scopes.

> The add-on only ever **reads** Gmail (`gmail.readonly`). It does not send,
> modify, or delete mail.

> **Publish the OAuth consent screen to "Production."** While it stays in
> **Testing** (the default), Google expires the refresh token after **7 days**,
> after which the add-on stops with `invalid_grant: Token has been expired or
> revoked`. Publishing to Production gives a non-expiring refresh token — no
> verification is needed for personal use (click through the "unverified app"
> warning at consent). Generate the token **after** publishing, since tokens
> minted in Testing keep the 7-day limit.

## Credential files

Place the OAuth credential and token JSON files in the Home Assistant
**`/share`** folder (the add-on is granted `share:rw` so it can read them and
persist refreshed tokens):

```
/share/credentials-<account>.json
/share/token-<account>.json
```

Reference them in the options by **bare filename** (the run script looks them up
under `/data`, `/config`, then `/share`).

## Configuration

```yaml
log_level: info
accounts:
  - email: you@gmail.com
    credentials_file: credentials-you-gmail-com.json
    token_file: token-you-gmail-com.json
    project_id: my-gcp-project
    topic_name: gmail-parcels
    subscription_id: gmail-parcels-sub
    carriers:
      - UPS
      - FedEx
      - DHL
```

| Option | Description |
|--------|-------------|
| `log_level` | Log verbosity: `debug`, `info`, `warn`, or `error`. |
| `accounts` | One or more mailboxes to watch (each block below). |
| `accounts[].email` | The Gmail address. Push notifications for other addresses are ignored. |
| `accounts[].credentials_file` | OAuth client credentials JSON filename (in `/share`). |
| `accounts[].token_file` | OAuth token JSON filename with a refresh token (in `/share`). |
| `accounts[].project_id` | Google Cloud project id that owns the topic/subscription. |
| `accounts[].topic_name` | Pub/Sub topic Gmail publishes change notifications to. |
| `accounts[].subscription_id` | Pub/Sub pull subscription the add-on consumes. |
| `accounts[].carriers` | Whitelist of carriers to keep (e.g. `UPS`, `FedEx`, `DHL`). Empty = accept all. Amazon-originated mail is always ignored (handled by a separate project). |

## Carrier live tracking (optional)

The add-on can pull **live status/ETA straight from the carrier** (not just from
emails), so parcels stay current between notifications and update even if an
email is missed. It's off until you add API keys. **FedEx, UPS, USPS, and DHL**
are supported; each is independent — enable any combination.

It refreshes status, ETA, delivery time, and delivery location for active
(non-delivered) parcels every `*_poll_minutes` (default 45). Sender/recipient
stay from the email parser — the public APIs redact them.

> **Two ways to set the keys:** the add-on **Configuration** tab (below), or the
> **⚙ Settings** panel in the web UI — which applies changes **live** (no
> restart). The web-UI editor is reachable only through the authenticated Home
> Assistant UI, masks saved secrets, and stores overrides in `/data`.

### FedEx

1. Sign up at **<https://developer.fedex.com>** → **Create API Project** → add the
   **Track API**.
2. Copy the **API Key** and **Secret Key** (use the **Production** pair for real
   parcels; **Test** keys only return FedEx mock numbers).

| Option | Value |
|--------|-------|
| `fedex_api_key` | FedEx **API Key** |
| `fedex_secret_key` | FedEx **Secret Key** (masked) |
| `fedex_poll_minutes` | refresh interval, default **45** (5–1440) |
| `fedex_sandbox` | `true` only with **Test** keys; otherwise `false` |

### UPS

1. Have a **UPS account number** (free at <https://www.ups.com>).
2. At **<https://developer.ups.com>** → **Add Apps**, associate your account
   number and add the **Tracking API**. Copy the **Client ID** and **Client
   Secret**.

| Option | Value |
|--------|-------|
| `ups_client_id` | UPS **Client ID** |
| `ups_client_secret` | UPS **Client Secret** (masked) |
| `ups_poll_minutes` | refresh interval, default **45** (5–1440) |
| `ups_sandbox` | `true` only for the UPS CIE sandbox; otherwise `false` |

### USPS

USPS replaced Web Tools with **OAuth2 v3 APIs**.

1. Register at **<https://developer.usps.com>** → **Add App** → add the
   **Tracking** API. Copy the **Consumer Key** and **Consumer Secret**.

| Option | Value |
|--------|-------|
| `usps_client_id` | USPS **Consumer Key** |
| `usps_client_secret` | USPS **Consumer Secret** (masked) |
| `usps_poll_minutes` | refresh interval, default **45** (5–1440) |
| `usps_sandbox` | `true` only for the TEM test host; otherwise `false` |

### DHL

DHL uses a single **API key** (no OAuth).

1. At **<https://developer.dhl.com>** → **Create App** → add **Shipment Tracking
   — Unified**. Copy the **API Key**.

| Option | Value |
|--------|-------|
| `dhl_api_key` | DHL **API Key** (masked) |
| `dhl_poll_minutes` | refresh interval, default **45** (5–1440) |

> DHL's free tier is ~250 calls/day, 1 call / 5 s — the add-on spaces its DHL
> calls accordingly.

Save + restart after entering keys.

## How it works

- On start the add-on calls Gmail `users.watch` (renewed every 24 h, since
  watches expire after 7 days) and listens on the Pub/Sub subscription.
- Each push triggers a Gmail history sync; new messages are run through the
  detector + NER, and matching parcels are stored in a SQLite DB at
  `/data/parcels.db` (persistent across restarts and updates).
- Delivered parcels are archived 7 days after delivery and purged 30 days after
  that.
- Detection is push-only — it processes mail that arrives **after** the add-on
  is running; it does not backfill mail received while it was stopped.

## Web UI

The add-on ships a built-in dashboard, served through Home Assistant **Ingress**
(HA proxies and authenticates it; no host port is exposed). Open it from the
**Parcels** entry in the HA sidebar, or via **Open Web UI** on the add-on page.

It shows every parcel with carrier, status, ETA, and sender/recipient; tabs for
**Active / Delivered / All**; and a per-parcel detail view with the full field
set, the status-history timeline, the delivery-proof photo, and a **Track ↗**
link to the carrier's site. It updates live over WebSocket (a dot in the header
shows the connection state).

## API endpoints

Served on port `8080` (consumed by the companion integration and the web UI):

| Endpoint | Purpose |
|----------|---------|
| `GET /healthz` | Liveness check. |
| `GET /api/parcels` | Active (non-archived) parcels. |
| `GET /api/parcels/{tracking}` | A single parcel by tracking number. |
| `GET /api/archived` | Archived parcels. |
| `GET /api/history` | All parcels, active and archived. |
| `GET /photos/{tracking}` | Delivery-proof photo for a parcel. |
| `GET /ws` | WebSocket stream; pushes the full parcel list on every change. |

## Troubleshooting

- **No parcels detected.** Set `log_level: debug` and watch the log. Confirm
  the listener stays up (no `Listen loop exited`) and that pushes arrive
  (`Received push for <email>`). A `NotFound (resource=...sub)` means the
  Pub/Sub subscription is gone — recreate it. If pushes arrive but nothing is
  stored, confirm the sending carrier is in the `carriers` whitelist.
- **`watch` succeeds but no pushes ever arrive.** The topic is missing the
  `roles/pubsub.publisher` grant for `gmail-api-push@system.gserviceaccount.com`.
- **Credentials not found.** Ensure the files are in `/share` and referenced by
  bare filename.
</content>
