This configuration for the **3xui service** is designed to address a specific limitation: because the server is recognized by Google as being located in Russia, there are no ads on YouTube, but Google Gemini AI services (both web and API) do not work natively from this region. To solve this, the config securely routes traffic to Gemini and other Google AI services through a Cloudflare WARP tunnel, bypassing regional restrictions while continuing to block ads.

## Configuration Purpose

The main objective is to enable access to **Google AI platforms** (like Gemini, Bard, Deepmind) from a Russian-based server, while still maintaining ad blocking for YouTube and other sites due to the regional server status. Traffic to restricted Google AI domains is routed via Cloudflare WARP for proper functionality.

## Functional Overview

### Logging and API

- **Logs:** Access logs are disabled, but DNS events are logged for debugging.
- **API interface:** The configuration exposes local API services listening on 127.0.0.1 for management and statistics.

### Inbounds and Outbounds

- **Inbound:** The API is exposed locally via the dokodemo-door protocol, allowing versatile control.
- **Outbounds:**
  - **Direct:** Handles standard outbound traffic with no routing alteration.
  - **Blocked:** Uses blackhole protocol to block specific traffic (ads, BitTorrent, spyware).
  - **WARP:** Utilizes WireGuard to tunnel selected traffic through Cloudflare WARP, spoofing server location for Google AI services.

### Routing Rules

- **Domain-Based Routing:** The core logic routes traffic to all Russian government domains and a curated list of Google AI domains via the WARP outbound, ensuring access despite geofencing.
- **API Routing:** Requests from the API inbound tag are routed to the API outbound.
- **Blocking:** Private IPs, BitTorrent protocol, global ad/tracking categories, and Russian-specific ad domains are forcefully blocked.

### Policies and Metrics

- **Statistics:** User and system traffic stats are enabled for analysis.
- **Metrics:** Service metrics are available locally for monitoring.

### DNS Handling

- **DNS:** Configured for IP-based query resolution with a dedicated DNS inbound, enabling reliable name resolution even under tunneling conditions.

## Key Feature Table

| Feature                 | Purpose                                                    |
|-------------------------|------------------------------------------------------------|
| **Cloudflare WARP**     | Enables access to geofenced Google AI services      |
| **Ad blocking**         | Blocks global ads & Russian ad domains              |
| **API access**          | Local API for management/statistics                 |
| **Routing logic**       | Selective domain/IP routing and blocking            |
| **Stats and metrics**   | System and user traffic statistics                  |

## Practical Outcomes

- **YouTube remains ad-free** due to Russian server region.
- **Google Gemini and related AI tools function properly** via WARP, circumventing Russian geoblocks.
- **Ad and spyware traffic** is systematically blocked, protecting users and reducing unwanted data.

