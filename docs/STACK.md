# Monitoring Stack Architecture

## Overview

The WickedYoda homelab monitoring stack is a unified observability platform running on docker2. It combines network monitoring, metrics collection, log aggregation, and visualization in a single Docker Compose deployment.

## Data Flow

```
Network Devices (SNMP/ICMP/HTTP probes)
    │
    ├─► LibreNMS ──► MariaDB
    │
    ├─► Prometheus ◄── Exporters (node, cAdvisor, redis, snmp, ping, process, speedtest, tailscale)
    │       │
    │       ├─► Alertmanager ──► Email/SMTP
    │       │
    │       └─► Grafana ◄── InfluxDB
    │               │
    │               └─► Loki ◄── Promtail
    │
    └─► Uptime Kuma ──► Notifications
```

## Service Dependencies

```
db ──► db-init ──► (creates grafana database)
  │
  ├─► librenms ──► dispatcher
  │       │
  │       └─► syslogng
  │       └─► snmptrapd
  │
  ├─► redis ──► librenms, dispatcher
  │       │
  │       └─► redis_exporter
  │
  ├─► prometheus ◄── all exporters
  │       │
  │       └─► alertmanager
  │       └─► grafana
  │
  ├─► influxdb ──► grafana
  │
  ├─► loki ──► grafana
  │       └─► promtail
  │
  └─► uptime-kuma
```

## Network Topology

```
Internet
    │
    ▼
Flint4 (10.0.84.1)
    │
    ├─► Dell N3048P (10.0.85.31)
    │       ├─► Outdoor-AP (10.0.85.17)
    │       ├─► NAS (10.0.85.10)
    │       ├─► rasp1 (10.0.85.11)
    │       └─► [ports 13-24: VLAN1 access]
    │
    ├─► docker2 (100.117.162.55) ◄── MONITORING STACK
    ├─► serv1 (100.76.68.53)
    ├─► serv2 (100.111.145.66)
    ├─► recipe (100.125.168.30)
    └─► problem-child (100.86.100.18)
```

## Storage Layout

| Volume | Container | Purpose | Retention |
|--------|-----------|---------|-----------|
| db_data | MariaDB | LibreNMS + Grafana DB | Persistent |
| librenms_data | LibreNMS | RRD, logs, configs | Persistent |
| prometheus_data | Prometheus | TSDB metrics | 15 days |
| alertmanager_data | Alertmanager | Alert state | Persistent |
| influxdb_data | InfluxDB | Time-series metrics | Persistent |
| grafana_data | Grafana | Dashboards, settings | Persistent |
| loki_data | Loki | Log chunks + index | 7 days |
| uptime_kuma_data | Uptime Kuma | Status pages, history | Persistent |

## Port Map

| Port | Protocol | Service | External Access |
|------|----------|---------|-----------------|
| 3000 | TCP | Grafana | Yes |
| 3001 | TCP | Uptime Kuma | Yes |
| 8000 | TCP | LibreNMS | Yes |
| 8080 | TCP | cAdvisor | No (internal) |
| 8081 | TCP | Grafana Image Renderer | No (internal) |
| 8086 | TCP | InfluxDB | Yes |
| 9090 | TCP | Prometheus | No (internal) |
| 9093 | TCP | Alertmanager | No (internal) |
| 9100 | TCP | Node Exporter | No (internal) |
| 9115 | TCP | Blackbox Exporter | No (internal) |
| 9116 | TCP | SNMP Exporter | No (internal) |
| 9121 | TCP | Redis Exporter | No (internal) |
| 9184 | TCP | Tailscale Exporter | No (internal) |
| 9256 | TCP | Process Exporter | No (internal) |
| 9273 | TCP | Ping Exporter | No (internal) |
| 9430 | TCP | SpeedTest Exporter | No (internal) |
| 162 | TCP/UDP | SNMP Trap Daemon | Yes |
| 514 | TCP/UDP | Syslog-NG | Yes |

## Monitoring Coverage

### Network Devices
- SNMP polling via LibreNMS + snmp_exporter
- ICMP ping via ping_exporter
- HTTP/TCP probes via blackbox_exporter
- Device types: routers, switches, APs, NAS, servers

### Hosts
- Node exporter: CPU, memory, disk, network
- cAdvisor: container metrics
- Process exporter: running services

### Applications
- LibreNMS: network health
- Redis: cache hit rate, memory, connections
- MariaDB: queries, connections, slow queries
- InfluxDB: write/query throughput, bucket sizes
- Grafana: dashboard load, API health

### Tailnet
- Peer latency
- Connection status
- Device count

### Logs
- System logs
- Docker container logs
- LibreNMS application logs
- Outdoor-AP syslog (via host mount)

## Alerting Flow

```
Event detected
    │
    ├─► Prometheus rule fires
    │       │
    │       └─► Alertmanager
    │               │
    │               ├─► Email (via msmtpd)
    │               ├─► Discord webhook
    │               └─► Gotify push
    │
    └─► Uptime Kuma
            │
            └─► Gotify / Discord / Telegram / Email
```

## Scaling Notes

- Prometheus retention: 15 days (configurable via `--storage.tsdb.retention.time`)
- Loki retention: 7 days (configurable in `loki/loki.yml`)
- InfluxDB: persistent, no built-in retention policy by default
- MariaDB: persistent, configure `innodb_file_per_table` for better space management

## Security

- No external auth on Prometheus/Alertmanager (internal only)
- Grafana: admin auth required, signup disabled
- InfluxDB: token-based auth
- SNMP: v2c community string in `env.env`
- All other exporters: no auth (internal Docker network)
