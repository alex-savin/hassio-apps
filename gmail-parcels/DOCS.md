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
