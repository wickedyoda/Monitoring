# Unified Monitoring Stack

This repository now runs a single monitoring system where infrastructure, services, and application endpoints are visible in one place.

## What Is Integrated

- LibreNMS for network discovery, SNMP polling, syslog, and traps
- Prometheus for metrics collection and alert evaluation
- Alertmanager for centralized alert routing
- Blackbox Exporter for HTTP/TCP synthetic checks
- Node Exporter and cAdvisor for host and container metrics
- InfluxDB for time-series workloads that need Flux queries
- Grafana for cross-system dashboards and alert views

## How Systems Share Information

- Prometheus scrapes exporters and probes all major services, so uptime and performance are centralized.
- Alert rules are evaluated once in Prometheus and routed through Alertmanager.
- Grafana is pre-provisioned with data sources for:
  - Prometheus metrics
  - InfluxDB metrics
  - Alertmanager alert state
  - LibreNMS MySQL data (inventory/events/metadata)
- LibreNMS, Prometheus, and InfluxDB can all be correlated inside Grafana dashboards.

## Startup

1. Update secrets and credentials in `env.env`.
2. Start everything:

```bash
docker compose up -d
```

3. Open interfaces:
   - LibreNMS: `http://localhost:8000`
   - Prometheus: `http://localhost:9090`
   - Alertmanager: `http://localhost:9093`
   - Grafana: `http://localhost:3000`
   - InfluxDB: `http://localhost:8086`

## Key Config Files

- `docker-compose.yml`
- `prometheus/prometheus.yml`
- `prometheus/rules/system-alerts.yml`
- `alertmanager/alertmanager.yml`
- `blackbox/blackbox.yml`
- `grafana/provisioning/datasources/datasources.yml`
