
---

### `configs/3xui/3xuiTemplate.txt`

This JSON file is a configuration template for the **3x-ui proxy service**. It's designed to intelligently route and filter internet traffic. Key features include:
*   **WARP Tunneling:** Routes traffic for Russian government domains and a specific list of Google AI services (like Gemini, Bard) through a Cloudflare WARP (WireGuard) tunnel to bypass regional restrictions.
*   **Ad & Spyware Blocking:** Blocks various ad networks, Windows spyware, and BitTorrent traffic using a "blackhole" outbound.
*   **Local API & Metrics:** Exposes a local API for service management and collects system/user statistics.
*   **Logging:** Logs DNS events but disables access logs.

---

### `configs/3xui/Readme.md`

This Markdown file provides a detailed explanation of the `3xuiTemplate.txt` configuration. It describes how the setup allows access to **Google AI services (Gemini, Bard, Deepmind)** from a server located in Russia (where they are normally restricted). This is achieved by securely routing specific Google AI traffic through a **Cloudflare WARP tunnel**. Simultaneously, it leverages the server's Russian location to maintain **ad-free YouTube** and blocks general ads, spyware, and BitTorrent traffic, detailing the logging, API, routing rules, and practical outcomes.

---

### `configs/3xui/docker-compose.yml`

This Docker Compose file defines the deployment of the **3x-ui proxy service** as a Docker container. It specifies:
*   **Image:** Uses `ghcr.io/mhsanaei/3x-ui:latest`.
*   **Port Mappings:** Exposes port `10501` for client connections and `127.0.0.1:2053` for local web administration.
*   **Volume Mounts:** Persists configuration data (`db/`) and certificates (`cert/`).
*   **Environment Variables:** Configures `XRAY_VMESS_AEAD_FORCED` and enables `FAIL2BAN`.

---

### `configs/monitoring/promtail-config.yml`

This YAML file configures **Promtail**, a log collection agent for Loki. It's set up to:
*   **Scrape Logs:** Collects logs from both host system files (`/var/log/*.log`) and Docker containers (auto-discovering them via `docker.sock`).
*   **Send to Loki:** Forwards the collected log data to a Loki instance at `http://loki:3100/loki/api/v1/push`.

---

### `configs/monitoring/loki-config.yml`

This YAML file defines the configuration for **Loki**, the log aggregation system. It specifies:
*   **Server:** Listens on port 3100 for incoming logs.
*   **Storage:** Uses `boltdb-shipper` for indexing and the filesystem for storing log chunks.
*   **Retention:** Configures a log retention period of 168 hours (7 days).

---

### `configs/monitoring/prometheus.yml`

This YAML file configures **Prometheus**, the monitoring system. It primarily defines:
*   **Global Settings:** Specifies a global scrape and evaluation interval of 15 seconds.
*   **Scrape Jobs:** Configures Prometheus to scrape its own metrics (`localhost:9090`) and metrics from `node-exporter` instances (`node-exporter:9100` and `100.123.56.70:9100`).
*   *Note:* The file also contains an unexpected, duplicate section that resembles a Loki configuration, including settings for `ingester`, `schema_config`, `storage_config`, `limits_config`, and `table_manager`.

---

### `configs/monitoring/docker-compose.yml`

This Docker Compose file orchestrates a comprehensive **monitoring stack** using several services:
*   **Prometheus:** For collecting and storing metrics.
*   **Grafana:** For data visualization and dashboards, exposed via Traefik with HTTPS and basic authentication.
*   **Loki:** For log aggregation.
*   **Promtail:** For collecting logs from the host and Docker containers and sending them to Loki.
*   **Node-Exporter:** For collecting host system metrics.
It defines networks, persistent volumes, and integrates Grafana with Traefik for external access and security.

---

### `configs/n8n/docker-compose.yml`

This Docker Compose file defines the deployment of **n8n**, a workflow automation platform, as a Docker container. Key aspects include:
*   **Image:** Uses a specific version of `docker.n8n.io/n8nio/n8n`.
*   **Persistent Data:** Mounts a volume (`n8n_data`) for persistent data and a local directory (`local-files`).
*   **Traefik Integration:** Configures Traefik labels to expose n8n at `n8n.teplyshko.ru`, ensuring HTTPS redirection and proper routing.
*   **Environment Variables:** Sets various n8n-specific environment variables for host, port, protocol, and timezone.

---

### `configs/traefik/traefik.yml`

This is the primary YAML configuration file for **Traefik**, an edge router. It defines:
*   **API & Dashboard:** Enables the Traefik dashboard and debug mode.
*   **Entry Points:** Configures HTTP (port 80) to redirect to HTTPS (port 443).
*   **Providers:** Uses Docker for dynamic service discovery (only exposing services explicitly labeled) and a file provider for static configurations.
*   **SSL Certificates:** Sets up ACME (Let's Encrypt) certificate resolution using the **Cloudflare DNS challenge**, storing certificates in `acme.json`.
*   **Logging:** Configures logging for Traefik itself and access logs.
*   *Note:* Includes commented-out sections for TCP routing, indicating potential VPN service integration (e.g., `3x-ui`) via `tls: passthrough`.

---

### `configs/traefik/docker-compose.yml`

This Docker Compose file defines how to deploy **Traefik** as a Docker container. It:
*   **Exposes Ports:** Maps host ports 80 and 443 to the container.
*   **Cloudflare Integration:** Provides Cloudflare API credentials as environment variables for the ACME DNS challenge.
*   **Volume Mounts:** Mounts the main Traefik configuration (`traefik.yml`), ACME certificate storage (`acme.json`), additional configuration (`config.yml`), and log directory.
*   **Self-Configuration:** Uses Traefik labels to configure its own dashboard (accessible via HTTPS with a domain pattern `teplyshko.ru`) and manage SSL certificates for `teplyshko.ru` and its subdomains using the 'cloudflare' certificate resolver.
