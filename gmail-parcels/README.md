# Gmail Parcels Tracker - v3 (Hybrid Architecture)

A Home Assistant addon that monitors Gmail for shipping notifications and tracks parcel deliveries. This version uses a **hybrid architecture**: deterministic regex + checksum validation for tracking numbers, with optional ONNX-based NER for soft data extraction.

## Features

### Parcel Tracking
- **Multi-carrier support**: UPS, FedEx, USPS, DHL, Amazon Logistics, OnTrac, LaserShip
- **Checksum validation**: Validates tracking numbers using carrier-specific algorithms (Mod10, Mod7, S10)
- **Amazon integration**: Extracts both TBA tracking numbers and Amazon Order IDs
- **Status tracking**: Tracks parcel lifecycle (label created → in transit → out for delivery → delivered)

### Data Extraction
- **Tracking numbers**: Regex patterns with checksum validation (high confidence)
- **Carrier detection**: Brand keyword matching + tracking number format inference
- **Delivery dates**: Extracts estimated delivery from carrier emails
- **Sender information**: Extracts shipper/merchant names
- **Delivery proof**: Downloads and stores delivery photos locally
- **Left-at location**: Extracts where package was left (Front Door, Mailbox, etc.)

### Persistence & History
- **JSON file storage**: Persists to `/data/parcels.json`
- **Status history**: Tracks all status changes with timestamps
- **Auto-archive**: Archives delivered parcels after 7 days
- **Retention policy**: Removes archived parcels after 30 days
- **Photo storage**: Saves delivery proof images to `/data/photos/`

### Gmail Integration
- **OAuth2 authentication**: Supports multiple Gmail accounts
- **Push notifications**: Uses Google Cloud Pub/Sub for real-time email notifications
- **Per-account filtering**: Prevents cross-account message fetching
- **Retry logic**: Handles 404 errors with exponential backoff

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Email Processing                         │
├─────────────────────────────────────────────────────────────────┤
│  1. Gmail Push Notification (Pub/Sub)                           │
│  2. Fetch message content                                        │
│  3. HARD DATA: Tracking detector (regex + checksum)             │
│  4. SOFT DATA: NER extraction (status, date, sender, photos)    │
│  5. Store to persistent JSON                                     │
│  6. Notify Home Assistant                                        │
└─────────────────────────────────────────────────────────────────┘
```

### Hybrid Detection Strategy

| Data Type | Method | Confidence |
|-----------|--------|------------|
| Tracking Number | Regex + Checksum | High (validated) |
| Carrier | Brand keywords + TN format | High |
| Status | Keyword matching | Medium |
| Delivery Date | Regex patterns | Medium |
| Sender | Regex patterns | Medium |
| Photo/LeftAt | Regex patterns | Medium |

## Configuration

```yaml
# config.yaml
accounts:
  - email: "user@gmail.com"
    credentials_file: "/data/credentials.json"
    token_file: "/data/token.json"

pubsub:
  project_id: "your-gcp-project"
  subscription: "gmail-parcels-sub"

server:
  port: 8080
```

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/healthz` | GET | Health check |
| `/parcels` | GET | List active parcels |
| `/parcels/archived` | GET | List archived parcels |
| `/push` | POST | Receive Pub/Sub notifications |

## Building

### Local Development
```bash
# Build without ONNX (regex-only)
go build -o app ./cmd/...

# Build with ONNX support
go build -tags onnx -o app ./cmd/...
```

### Docker (Home Assistant Addon)
```bash
# Build the addon image
./build.sh

# Or manually
docker build -f Dockerfile.addon -t gmail-parcels-v3 .
```

## Supported Carriers

| Carrier | Tracking Format | Checksum |
|---------|-----------------|----------|
| UPS | 1Z + 16 alnum | Mod10 |
| FedEx | 12, 15, 20, 22 digits | Mod10 |
| USPS | 20-22 digits (9xxx) | Mod10 |
| USPS International | 2 letters + 9 digits + US | S10 |
| DHL Express | 10 digits | Mod7 |
| Amazon | TBA + 12-15 digits | None |
| OnTrac | C + 14 digits | None |
| LaserShip | 1LS + digits | None |

## Parcel Data Model

```json
{
  "tracking_number": "1Z999AA10123456784",
  "carrier": "UPS",
  "status": "delivered",
  "estimated_delivery": "2026-01-15",
  "sender": "Amazon.com",
  "created_at": "2026-01-10T10:00:00Z",
  "last_updated": "2026-01-15T14:30:00Z",
  "delivered_at": "2026-01-15T14:30:00Z",
  "status_history": [
    {"status": "label created", "timestamp": "2026-01-10T10:00:00Z"},
    {"status": "in transit", "timestamp": "2026-01-12T08:00:00Z"},
    {"status": "out for delivery", "timestamp": "2026-01-15T06:00:00Z"},
    {"status": "delivered", "timestamp": "2026-01-15T14:30:00Z"}
  ],
  "photo_proof": "/data/photos/1Z999AA10123456784.jpg",
  "left_at": "Front Door",
  "amazon_order_id": "123-4567890-1234567"
}
```

## ONNX Model (Optional)

The addon can optionally use an ONNX-exported DistilBERT model for NER extraction. This is used for soft data (status, sender) when regex patterns don't match.

### Artifacts
- `onnx-out/model.onnx` - The ONNX model
- `onnx-out/vocab.txt` - WordPiece vocabulary
- `onnx-out/labels.txt` - BIO label mapping

### Export Recipe
```bash
python scripts/export_onnx.py \
  --model ./model_dir \
  --output ./onnx-out \
  --opset 17 \
  --max-seq-length 256 \
  --validate
```

## Development

### Project Structure
```
go-gmail-parcels-v3-pure/
├── cmd/                    # Main application
├── internal/
│   ├── api/               # HTTP server & endpoints
│   ├── gmail/             # Gmail API client
│   ├── ner/               # NER extraction (regex + ONNX)
│   ├── photo/             # Delivery photo downloader
│   ├── state/             # Parcel store & persistence
│   └── tracking/          # Tracking number detector
├── config/                # Configuration types
├── rootfs/                # S6 overlay scripts
├── onnx-out/              # ONNX model artifacts
└── scripts/               # Build & export scripts
```

### Running Tests
```bash
# All tests
go test ./...

# With ONNX
go test -tags onnx ./...

# Specific package
go test -v ./internal/tracking/...
go test -v ./internal/ner/...
```

## License

MIT
