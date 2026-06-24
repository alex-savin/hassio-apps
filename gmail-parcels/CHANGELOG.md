# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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
