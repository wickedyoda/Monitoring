# WickedYoda Homelab Monitoring Stack

Unified network, metrics, and log monitoring for the homelab.

## Documentation

- **[STACK.md](docs/STACK.md)** — Architecture, data flow, service topology
- **[GETTING_STARTED.md](docs/GETTING_STARTED.md)** — Step-by-step setup guide
- **[ALERTING.md](docs/ALERTING.md)** — Alert rules, receivers, and notification setup

## What This Stack Does

- Discovers and polls network devices via SNMP
- Collects host, container, and application metrics
- Runs HTTP/TCP/ICMP probes for uptime and latency
- Aggregates logs from syslog and Docker containers
- Visualizes everything in Grafana
- Alerts through Alertmanager
- Provides a status page and notifications via Uptime Kuma

## Quick Start

```bash
git clone https://github.com/wickedyoda/Monitoring.git
cd Monitoring
cp env.env.example env.env
# Edit env.env with your settings
docker compose up -d
```

## Stack Overview

| Category | Services |
|----------|----------|
| Core | LibreNMS, MariaDB, Redis, msmtpd |
| Metrics | Prometheus, Alertmanager, Grafana, InfluxDB |
| Exporters | Node, cAdvisor, Blackbox, Redis, SNMP, Ping, Process, SpeedTest, Tailscale |
| Logging | Loki, Promtail |
| Operations | Uptime Kuma, Grafana Image Renderer |

## Volumes

- `db_data` — MariaDB
- `librenms_data` — LibreNMS data
- `prometheus_data` — Prometheus TSDB
- `alertmanager_data` — Alertmanager state
- `influxdb_data` — InfluxDB storage
- `grafana_data` — Grafana dashboards/settings
- `loki_data` — Loki chunks/index
- `uptime_kuma_data` — Uptime Kuma data

## Branch Policy

- `main` — canonical branch
- `master` — mirrors main
- All changes via PR with required review
- Default branch: `main`

## License

GPLv3
