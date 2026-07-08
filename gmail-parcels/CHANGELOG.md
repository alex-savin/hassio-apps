# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.6.0] - 2026-07-07

Reliability, correctness, and security hardening from a full code review,
prompted by an OAuth-token-expiry incident that took the add-on down silently.

### Fixed

- A follow-up email that couldn't extract the **sender/recipient** (or status)
  no longer overwrites the values captured from an earlier email with blanks.
- **USPS 22-digit** tracking numbers (`9400…`) are now detected as USPS instead
  of being misread as FedEx.
- The add-on no longer **crash-loops** when the Gmail OAuth token is expired or
  revoked; it backs off and reports the condition instead of restarting in a
  tight loop.
- Duplicate or replayed emails no longer **resurrect archived parcels** or reset
  their delivery date.
- Assorted detection fixes: non-Amazon senders (e.g. `order-update@` at other
  domains) are no longer skipped as Amazon; phone numbers and dates are no
  longer accepted as tracking numbers; NER tokenizer and FedEx door-tag handling
  corrected.

### Added

- **`/healthz` now reports real per-account state** (`listening` / `error` /
  `auth_error`) so an expired OAuth grant is visible instead of a silent stall.
- Delivery-proof **photos are deleted when a parcel is purged** by retention, so
  `/data/photos` no longer grows without bound.
- Documentation: the Google setup now covers publishing the OAuth consent screen
  to **Production** to avoid the 7-day refresh-token expiry.

### Changed

- WebSocket delivery reworked (per-connection writer, deadlines, keepalive) so a
  single stalled client can no longer block updates to everyone else.
- The OAuth token file and the carrier-settings file are now written
  **atomically**, preventing loss on a crash mid-write.

### Security

- The delivery-photo downloader re-checks the carrier-domain allowlist across
  **redirects** (closing an SSRF path) and caps the download size.
- The in-UI carrier-credentials endpoint now requires the Home Assistant ingress
  gateway, not just a client-supplied header.

## [1.5.1] - 2026-06-24

### Changed

- Web UI: each carrier block in the ⚙ Settings panel is now foldable
  (collapsed by default) for a more compact layout.

## [1.5.0] - 2026-06-24

### Added

- **Edit carrier API credentials from the web UI.** A new **⚙ Settings** panel
  lets you set/change FedEx/UPS/USPS/DHL keys and poll intervals from the
  dashboard, applied **live** (no add-on restart). It's reachable only through
  the authenticated Home Assistant UI (ingress), masks saved secrets on display,
  and stores overrides in `/data` (layered over the add-on Configuration values;
  "Clear" reverts a carrier to the add-on config).

## [1.4.0] - 2026-06-24

### Added

- **USPS and DHL live tracking (optional).** Carrier enrichment now covers all
  four carriers. Set `usps_client_id`/`usps_client_secret` (USPS v3 OAuth2) and/or
  `dhl_api_key` (DHL Unified Tracking, API key) to poll live status/ETA/delivery
  for active USPS/DHL parcels, the same way FedEx and UPS already work. Off by
  default; see the docs for creating each carrier's keys. DHL calls are spaced to
  respect its free-tier rate limit.

## [1.3.0] - 2026-06-24

### Added

- **UPS Tracking API enrichment (optional).** Like the FedEx one, but for UPS:
  set `ups_client_id` / `ups_client_secret` and the add-on polls the UPS
  Tracking API for active UPS parcels (`ups_poll_minutes`, default 45) and
  merges live status, ETA, delivery time, and delivery location back in. Off by
  default; see the docs for creating UPS API keys. FedEx and UPS are independent.

### Changed

- Carrier live-tracking is now a shared mechanism (one poller drives both FedEx
  and UPS), so future carriers are easy to add.

## [1.2.0] - 2026-06-24

### Added

- **FedEx Track API enrichment (optional).** When FedEx developer API keys are
  configured (`fedex_api_key` / `fedex_secret_key`), the add-on polls the FedEx
  Track API for every active FedEx parcel on an interval (`fedex_poll_minutes`,
  default 45) and merges live **status, ETA, delivery time, and delivery
  location** back in — so parcels stay current between emails and are updated
  even if an email is missed. Batches up to 30 numbers per call, caches the
  OAuth token, and keeps the Secret Key out of the log. Off by default; see the
  docs for how to create the keys. Sender/recipient stay from the email parser
  (the public Track API usually redacts them); FedEx-only for now.

## [1.1.2] - 2026-06-24

### Fixed

- **False-positive parcels.** A marketing/listing email (e.g. an eBay deal)
  whose body contained a number that happened to pass a carrier checksum could
  be logged as a parcel. Parcel creation now requires a genuine shipping
  signal — a detected status, an ETA, or a carrier From-header — so non-shipment
  emails are dropped. ("on its way" / "has shipped" are also recognized as a
  status so real merchant shipments still register.)

## [1.1.1] - 2026-06-24

### Fixed

- **Recipient extraction.** FedEx Delivery Manager emails name the recipient
  only in a greeting line ("Hi, &lt;Name&gt;.") and a To-block, which neither the
  model nor the regex fallback caught — so parcels showed a sender but a blank
  recipient. Added a greeting pattern (and guarded against generic greetings
  like "Hi there"). Verified on real emails (recipient now populates).

## [1.1.0] - 2026-06-24

### Added

- **Web UI (Home Assistant Ingress).** The add-on now serves a built-in
  dashboard — open it from the **Parcels** sidebar panel (or the add-on's "Open
  Web UI"). It lists every parcel with carrier, status, ETA, sender/recipient
  and a live connection indicator; tabs for Active / Delivered / All; and a
  detail view per parcel with the full field set, the status-history timeline,
  the delivery-proof photo, and a "Track ↗" link to the carrier. Updates live
  over WebSocket. Served on the existing port; the add-on stays internal-only
  (HA proxies and authenticates ingress), no host port is exposed.

## [1.0.4] - 2026-06-24

### Added

- **Recipient is now stored and served.** The NER model already detected a
  `RECIPIENT`, but the parcel record had no field for it so it was dropped.
  Added `Parcel.Recipient`, a SQLite column with an idempotent migration for
  existing databases, and wired it through the API.

### Fixed

- **Estimated delivery date.** Relative wording ("scheduled for delivery
  tomorrow", "out for delivery today") is now resolved to an absolute ISO
  date against the email's `Date` header, and common absolute formats are
  normalized to `YYYY-MM-DD`. Garbage spans such as `tomorrow 516602` (a
  tracking-number fragment a greedy regex mistook for a date) are rejected.

### Changed

- Broadened sender extraction ("Sold by", "Sender:") and added conservative
  recipient extraction ("Ship to:", "Recipient:") that ignores delivery
  locations and street addresses. Note: FedEx Delivery Manager notifications
  still won't populate a sender — they don't name the merchant.

## [1.0.3] - 2026-06-24

### Fixed

- **Push-triggered emails were never detected** — the add-on stayed empty even
  with a healthy Gmail → Pub/Sub pipeline. `fetchChanges` listed Gmail history
  from the push notification's own `historyId`, but `Users.History.List` treats
  `startHistoryId` as **exclusive** (it returns only records *after* that id).
  Because the notification's id is the history record of the very message that
  triggered it, that message was always skipped and `History.List` returned
  zero records. The service now lists from the last `historyId` it actually
  processed (the Watch baseline on first run) and uses the push's id only as a
  wake signal, advancing the cursor afterward. Verified against a live mailbox:
  listing from a record's own id returns nothing; from `id - 1` returns the
  record.

## [1.0.2] - 2026-06-23

### Added

- `map: [share:rw]` so the add-on can read the Gmail OAuth credential/token
  files placed in `/share` and persist refreshed tokens back there.

## [1.0.1] - 2026-06-23

### Changed

- Updated ONNX Runtime to 1.27.0.

## [1.0.0] - 2026-06-23

### Added

- Initial release.
- Watches Gmail for shipping notifications via Gmail push (Cloud Pub/Sub) and
  extracts parcel data: carrier, tracking number, status, estimated delivery,
  sender, and delivery-proof photos.
- Deterministic carrier/tracking detection (regex + carrier checksums for UPS,
  FedEx, USPS, DHL, OnTrac, LaserShip) backed by an internal Go NER model for
  the soft fields (status, dates, sender).
- NER model downloaded at runtime into `/data/onnx-out` (cached across
  restarts); falls back to a regex-only extractor if the download fails, which
  keeps the image small (~60–80 MB).
- WebSocket + REST API (`/ws`, `/healthz`, `/api/parcels`, `/api/archived`,
  `/api/history`, `/photos/{tracking}`) consumed by the companion
  [hassio-integration-gmail-parcels](https://github.com/alex-savin/hassio-integration-gmail-parcels).
- Per-account configuration: Gmail credential/token file names, Cloud Pub/Sub
  project/topic/subscription, optional carrier whitelist, and log level.
</content>
</invoke>
