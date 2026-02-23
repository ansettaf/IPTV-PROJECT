# Restreaming Architecture & Traffic Flow

## Overview

This IPTV lab functions as a highly efficient reverse proxy for media
streams, known as "Restreaming". It bridges external upstream providers
and end-users, managing authentication, buffering, and obfuscation of
the original source.

------------------------------------------------------------------------

## 7.1 Restreaming Lifecycle

The stream delivery process follows a strict, three-phase lifecycle
executed by decoupled services.

### Phase 1: Client Ingress & Authentication

1.  **Request:** End-user client (e.g., Smart IPTV, VLC) sends an HTTP
    GET request to the Nginx Edge gateway on port `8880` with
    credentials and channel ID (e.g.,
    `/username/password/channel_id.ts`).
2.  **Validation:** Nginx forwards authentication parameters to PHP-FPM
    backend.
3.  **State Check:** PHP queries Redis cache to verify account status
    and concurrency limits. If valid, an internal token is generated.

### Phase 2: Upstream Ingestion (Source Fetching)

1.  **Source Lookup:** System queries MariaDB `streams` table to find
    associated `stream_source` (JSON array with primary/backup URLs).
2.  **Ingestion:** Nginx-RTMP/FFmpeg connects to upstream provider
    (external `.xyz` source).
3.  **Buffering:** Incoming MPEG-TS packets are stored in RAM (14GB) to
    absorb upstream network jitter.

### Phase 3: Egress & Delivery

1.  **Proxying:** Buffered stream is repackaged and sent to end-user
    over port `8880`.
2.  **Source Obfuscation:** End-user communicates only with VPS IP.
    Original upstream DNS/IP remains hidden.

------------------------------------------------------------------------

## 7.2 Sequence Diagram of Stream Delivery

``` mermaid
sequenceDiagram
    participant Client as End-User Client
    participant Edge as Nginx-RTMP (Port 8880)
    participant Backend as PHP-FPM / Redis
    participant DB as MariaDB (Config)
    participant Upstream as External Source

    Client->>Edge: GET /user/pass/stream.ts
    Edge->>Backend: Validate Credentials & Max Conns
    Backend-->>Edge: Auth Approved
    Edge->>DB: Fetch JSON Stream Source URL
    DB-->>Edge: Return Upstream URL
    Edge->>Upstream: Open Persistent Connection
    Upstream-->>Edge: Raw Video Packets (Ingress)
    Note over Edge: Buffer payload in RAM to mitigate jitter
    Edge-->>Client: Proxied Video Stream (Egress)
```

------------------------------------------------------------------------

## 7.3 Directory & Service Tree Representation

    XUI.one-IPTV-Lab/
    ├─ nginx-rtmp/                  # Edge Gateway for streaming
    │  ├─ conf.d/
    │  │  └─ default.conf           # RTMP & HTTP config
    │  └─ logs/
    ├─ php-fpm/                     # Application logic
    │  ├─ www.conf                  # Pool config
    │  └─ workers/
    │     ├─ watchdog.php           # Stream health monitoring
    │     ├─ signals.php            # Real-time admin commands
    │     └─ queue.php              # Async task processing
    ├─ redis/                       # Session & stream metadata caching
    ├─ mariadb/                      # Persistence layer
    │  └─ streams/                  # Stream source storage (JSON)
    ├─ ffmpeg/                       # Transcoding engine (optional)
    └─ scripts/
       └─ install.sh                # Automated deployment script

This tree diagram represents all the major services and their roles
within the restreaming architecture.
