# Go Gmail Parcels (v2) - Home Assistant Add-on

[![Add add-on to Home Assistant](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Falex-savin%2Fhassio-apps)
[![GitHub Release](https://img.shields.io/github/v/release/alex-savin/go-gmail-parcels)](https://github.com/alex-savin/go-gmail-parcels/releases)
[![License](https://img.shields.io/github/license/alex-savin/go-gmail-parcels)](LICENSE)

A hybrid Go/Python Home Assistant Add-on that monitors Gmail for parcel delivery notifications using a local NER (Named Entity Recognition) model.

## Features

- **Real-time Updates**: Uses Google Cloud Pub/Sub and Gmail Push Notifications (no polling).
- **Local Inference**: Runs a fine-tuned DistilBERT NER model locally (Python) to extract tracking numbers and carriers.
- **Privacy Focused**: Email content is processed locally; only metadata interacts with Google Cloud.
- **High Performance**: Go handling networking/management, Python optimized for PyTorch inference.

## Architecture

This add-on uses a "sidecar" pattern:
1.  **Go Binary**: Handles Gmail OAuth, Pub/Sub processing, logic, and Home Assistant communication.
2.  **Python Script**: Runs a lightweight HTTP server exposing the NER model.

## Prerequisites: Google Cloud Setup

To use this add-on, you must configure a Google Cloud Project with Pub/Sub and Gmail API access.

### 1. Project & APIs
1.  Go to [Google Cloud Console](https://console.cloud.google.com/).
2.  Create a new project.
3.  Enable **Gmail API** and **Cloud Pub/Sub API**.

### 2. Pub/Sub Setup
1.  **Create a Topic**:
    *   Navigate to Pub/Sub > Topics.
    *   Create a topic (e.g., `gmail-parcels`).
2.  **Grant Publish Rights to Gmail**:
    *   Select the topic -> "Show Info Panel" -> "Add Principal".
    *   Add `gmail-api-push@system.gserviceaccount.com`.
    *   Role: **Pub/Sub Publisher**.
3.  **Create a Subscription**:
    *   Create a subscription for that topic (e.g., `gmail-parcels-sub`).
    *   Delivery type: **Pull**.

### 3. Service Account (for Pub/Sub)
1.  Go to IAM & Admin > Service Accounts.
2.  Create a Service Account.
3.  Grant it the role: **Pub/Sub Subscriber** (on the subscription or project).
4.  Create a JSON Key for this account. Save it as `service-account.json`.

### 4. OAuth Credentials (for Gmail)
1.  Go to APIs & Services > Credentials.
2.  Create **OAuth Client ID** (Desktop App).
3.  Download JSON as `credentials.json`.
4.  (One-time) You must generate a user `token.json` locally using these credentials. You can use a local helper script to perform the OAuth dance and save the `token.json`.

## Configuration

The add-on requires a `config/config.yaml` file mounted or present in the container.

### `config.yaml`

```yaml
model_path: "/app/model" # Path inside container
port: 8080

accounts:
  - email: "your.email@gmail.com"
    credentials_file: "/config/gmail/credentials.json"
    token_file: "/config/gmail/token.json"
    # Whitelist carriers (optional). If empty, all carriers allowed.
    carriers: 
      - "usps"
      - "ups"
      - "fedex"
    
    # Pub/Sub Configuration
    project_id: "your-gcp-project-id"
    topic_name: "gmail-parcels"        # Just the name, not full path
    subscription_id: "gmail-parcels-sub" # Just the ID
```

## Installation (Home Assistant)

1.  Copy the `go-gmail-parcels-v2` directory to your local add-ons folder.
2.  Ensure your `config` directory in Home Assistant contains the necessary JSON keys:
    *   `/config/gmail/credentials.json` (OAuth Client)
    *   `/config/gmail/token.json` (Generated Token)
    *   (Optional) If using standard GCP auth for Pub/Sub, ensure `GOOGLE_APPLICATION_CREDENTIALS` matches the path in the Dockerfile or environment variables.
3.  Install the add-on via the Home Assistant Store (Local Add-ons).
4.  Start the add-on.

## Local Development

### Requirements
- Go 1.25+
- Python 3.9+
- `pip install -r requirements.txt`

### Running
1.  Start the Python Inference server (optional, the Go app handles this, but for debugging logic only):
    ```bash
    python3 inference.py
    ```
2.  Run the Go application:
    ```bash
    export GOOGLE_APPLICATION_CREDENTIALS="/path/to/service-account.json"
    go run cmd/main.go
    ```

## Token Generation
(If you need a helper script to generate `token.json`)
Run a simple script locally on your PC with `credentials.json` to authorize the app and get the token.
