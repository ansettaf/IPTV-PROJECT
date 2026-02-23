# Hardware Allocation & Network Configuration

## Hardware Allocation

### Compute

Intel Xeon Processor (Cascadelake Architecture) --- 6 Sockets @ 2.0GHz

### Memory

14GB Physical RAM (Provisioned to prevent buffer underruns during peak
concurrent streaming)

### Storage

127GB Enterprise SSD (Low I/O wait times, optimized for database
transactions and rapid M3U generation)

### Operating System

RHEL-based Linux (Prime Intel NL-3 v.2)

------------------------------------------------------------------------

## Network Capabilities

### Port Speed

1 Gbps (1000 Mbps) uplink, ensuring sufficient bandwidth to prevent
bottlenecking during high-concurrency stream distribution.

### Bandwidth Allowance

Unmetered traffic. This is a critical architectural requirement for
operating video-on-demand (VOD) and continuous live IPTV services
without encountering ISP data caps or throttling.

------------------------------------------------------------------------

## Vendor-Specific Network Mitigation (IPv6)

During initial deployment, a routing instability was identified within
the IPv6 stack. Dual-stack networking introduced erratic socket binding
behavior for Nginx-RTMP streams and caused delayed upstream source
fetching.

### Resolution

IPv6 was explicitly disabled at the kernel level to enforce exclusive
IPv4 traffic routing, stabilizing stream handshakes and packet delivery.

``` bash
# Applied IPv6 Kernel Mitigation
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.lo.disable_ipv6=1
sudo sysctl -p
```
