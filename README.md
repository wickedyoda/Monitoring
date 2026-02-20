# Prometheus + InfluxDB + Grafana Stack

This repository now runs a focused monitoring stack for a Debian Arm64 host:

- Prometheus (metrics collection and query)
- Node Exporter (host metrics source for Prometheus)
- InfluxDB (separate time-series store)
- Grafana (dashboarding with both data sources)

## Important Architecture Note

Prometheus does not use InfluxDB as its primary TSDB. Prometheus always stores locally in `/prometheus`.
InfluxDB runs alongside it, and Grafana can query both systems.

## Startup

1. Ensure these directories exist on the host:
   - `/home/traver/docker/monitoring/prometheus`
   - `/home/traver/docker/monitoring/influxdb`
   - `/home/traver/docker/monitoring/grafana`
2. Update credentials in `env.env` (especially InfluxDB and Grafana admin password).
3. Start stack:

```bash
docker compose up -d
```

## Access

- Prometheus: `http://localhost:9090`
- Grafana: `http://localhost:3000`
- InfluxDB: `http://localhost:8086`
- Node Exporter: `http://localhost:9100/metrics`

## Key Config Files

- `docker-compose.yml`
- `prometheus/prometheus.yml`
- `grafana/provisioning/datasources/datasources.yml`
