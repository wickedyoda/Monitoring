# WickedYoda Homelab Monitoring Stack

Unified monitoring with LibreNMS, Prometheus, Grafana, InfluxDB, and Loki.

## Stack

### Core
- **LibreNMS** — network device monitoring via SNMP (`:8000`)
- **MariaDB** — database for LibreNMS + Grafana
- **Redis** — cache/session for LibreNMS
- **msmtpd** — SMTP relay for alerts

### Metrics & Alerting
- **Prometheus** — time-series metrics (`:9090`)
- **Alertmanager** — alert routing (`:9093`)
- **Grafana** — visualization (`:3000`)
- **InfluxDB v2** — long-term metrics store (`:8086`)

### Exporters
- **Node Exporter** — host metrics (`:9100`)
- **cAdvisor** — container metrics (`:8080`)
- **Blackbox Exporter** — HTTP/TCP probes (`:9115`)
- **Redis Exporter** — Redis metrics (`:9121`)
- **SNMP Exporter** — network device metrics (`:9116`)
- **Ping Exporter** — ICMP latency/uptime (`:9273`)
- **Process Exporter** — process monitoring (`:9256`)
- **SpeedTest Exporter** — bandwidth tests (`:9430`)
- **Tailscale Exporter** — tailnet peer health (`:9184`)

### Logging
- **Loki** — log aggregation (`:3100`)
- **Promtail** — log collection agent

### Operations
- **Uptime Kuma** — status page + notifications (`:3001`)
- **Grafana Image Renderer** — PNG exports (`:8081`)

## Quick Start

```bash
cp env.env.example env.env
# Edit env.env with your settings, especially:
# - Grafana admin password
# - InfluxDB admin token
# - MariaDB passwords
# - Tailscale API key
docker compose up -d
```

## Volumes

- `db_data` — MariaDB
- `librenms_data` — LibreNMS data
- `prometheus_data` — Prometheus TSDB
- `alertmanager_data` — Alertmanager state
- `influxdb_data` — InfluxDB storage
- `grafana_data` — Grafana dashboards/settings
- `loki_data` — Loki chunks/index
- `uptime_kuma_data` — Uptime Kuma data

## Network Notes

- All exporters use `linux/arm64` platform for Raspberry Pi / ARM hosts
- SNMP Exporter config: `./snmp/snmp.yml`
- Ping Exporter config: `./ping/ping.yml`
- Process Exporter config: `./process/process.yml`
- Loki config: `./loki/loki.yml`
- Promtail config: `./promtail/promtail.yml`

## License

GPLv3
