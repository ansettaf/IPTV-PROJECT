# XUI.one IPTV Infrastructure & DevOps Lab

## Project Overview

This repository documents the deployment, architectural configuration,
and security hardening of a high-concurrency IPTV management system.
Operating as a lab environment, this infrastructure is designed to test
multi-protocol streaming, automated load balancing, and real-time
subscriber management under a decoupled service model.

------------------------------------------------------------------------

## 1. Installation Source and Deployment

The core system was deployed utilizing an automated installation script
from the `amidevous/xui.one` repository. This build implements the
XUI.one management panel alongside its required dependencies, optimized
for Ubuntu/RHEL-based distributions.

### Deployment Command Executed:

``` bash
sudo wget https://raw.githubusercontent.com/amidevous/xui.one/master/install.sh -O /root/install.sh && sudo bash /root/install.sh
```

Note: This specific build includes modifications/cracks tailored for
specific OS versions.

------------------------------------------------------------------------

## 2. System Architecture

The environment utilizes a multi-tier delivery model to separate the web
interface from the streaming engine and data persistence layer, ensuring
high availability.

-   **Edge Gateway (Nginx-RTMP):** A specialized Nginx build optimized
    for real-time media distribution and low-latency stream egress.

-   **Application Logic (PHP 7.4-FPM):** Executes the core backend
    logic, API requests, and background CLI workers.

-   **State Management (Redis):** An in-memory data store for high-speed
    session caching and stream metadata.

-   **Persistence Layer (MariaDB 10.11):** The relational database
    acting as the "Source of Truth" for subscriber credentials and
    stream source URLs (stored as JSON arrays for redundancy).

------------------------------------------------------------------------

## 3. Infrastructure Specifications

-   **Compute:** Intel Xeon Processor (Cascadelake Architecture) --- 6
    Sockets @ 2.0GHz

-   **Memory:** 14GB Physical RAM (High capacity to prevent buffer
    underruns during peak concurrent streaming)

-   **Storage:** 127GB Enterprise SSD (Low I/O wait times optimized for
    database transactions and fast M3U generation)

------------------------------------------------------------------------

## 4. Network and Security Hardening

A strict ingress filtering policy (via UFW/iptables) is enforced to
minimize the attack surface, isolating sensitive backend services from
the public internet.

  -------------------------------------------------------------------------------
  Service      Port     Protocol   Policy       Description
  ------------ -------- ---------- ------------ ---------------------------------
  HTTP/HTTPS   80/443   TCP        Public       Web Dashboard and Client API

  Streaming    8880     TCP        Public       Live Stream Delivery (M3U8/TS)

  Load         31210    TCP        Public       Internal Backend Communication
  Balancer                                      

  SSH          22       TCP        Restricted   Secure Remote Administration

  MariaDB      3306     TCP        Blocked      Isolated to Localhost (127.0.0.1)

  Redis        6379     TCP        Blocked      Isolated to Localhost (127.0.0.1)
  -------------------------------------------------------------------------------

### Background Orchestration

The system relies on dedicated systemd-managed PHP workers running under
the xui user context (preventing root filesystem exposure):

-   **watchdog.php:** Monitors stream health and auto-restarts failed
    RTMP processes.

-   **signals.php:** Processes real-time commands from the admin
    interface.

-   **queue.php:** Handles asynchronous tasks like bulk user updates.

------------------------------------------------------------------------

## 5. Domain Challenges and Operational Risks

Operating an IPTV infrastructure introduces significant technical and
industry-specific challenges.

### Technical Constraints

-   **Upstream Dependency:** High reliance on third-party source
    providers (e.g., external .xyz domains). Upstream downtime directly
    impacts local stream availability.

-   **Transcoding Overhead:** Real-time stream transcoding (via FFmpeg)
    is highly CPU-intensive. Lacking hardware acceleration
    (GPU/QuickSync), high-concurrency FHD streams risk thread
    exhaustion.

-   **Network Jitter:** Delivering stable bitrates across diverse ISP
    routes requires aggressive buffer tuning.

### Security & Industry Risks

-   **DDoS Vulnerability:** Public streaming ports (8880) are highly
    susceptible to volumetric Distributed Denial of Service attacks.

-   **Credential Stuffing:** Automated brute-force attacks targeting
    user accounts.

-   **Compliance & Integrity:** Utilizing cracked installation scripts
    introduces severe risks of embedded backdoors, botnet integration,
    and significant legal liability regarding copyright enforcement.

------------------------------------------------------------------------

## 6. Real-Time Observability and Diagnostics

To monitor system health, database connections, and process states, the
following commands are utilized.

### Network & Socket Mapping

``` bash
# View active listening ports and established streaming connections
ss -tulpn
netstat -anp | grep :8880 | grep ESTABLISHED | wc -l
```

### Database Thread Monitoring

``` bash
# Monitor MariaDB process list for active backend queries
mysqladmin -u root -p proc
```

### Live Log Tailing

``` bash
# Track real-time worker logs and service heartbeats
journalctl -u xuione.service -f
```
