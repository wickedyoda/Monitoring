# Alerting Configuration

## Alertmanager Setup

Alertmanager handles alerts from Prometheus and routes them to notification channels.

### Configuration File

`alertmanager/alertmanager.yml`

### Receivers

#### Email (via msmtpd)
```yaml
- name: 'email'
  email_configs:
    - to: 'alerts@example.com'
      from: 'monitoring@example.com'
      smarthost: 'msmtpd:25'
      require_tls: false
```

#### Discord
```yaml
- name: 'discord'
  webhook_configs:
    - url: 'https://discord.com/api/webhooks/YOUR_WEBHOOK_URL'
      send_resolved: true
```

#### Gotify
```yaml
- name: 'gotify'
  webhook_configs:
    - url: 'http://gotify:8080/message?token=YOUR_TOKEN'
      send_resolved: true
```

### Routing

```yaml
route:
  receiver: 'email'
  group_by: ['alertname', 'host']
  group_wait: 10s
  group_interval: 5m
  repeat_interval: 1h
  
  routes:
    - match:
        severity: critical
      receiver: 'email'
    - match:
        severity: warning
      receiver: 'discord'
```

## Prometheus Alert Rules

### System Alerts

File: `prometheus/rules/system-alerts.yml`

#### Host Down
```yaml
- alert: HostDown
  expr: up == 0
  for: 1m
  labels:
    severity: critical
  annotations:
    summary: "Host {{ $labels.instance }} is down"
    description: "{{ $labels.instance }} has been unreachable for more than 1 minute."
```

#### High CPU Usage
```yaml
- alert: HighCPU
  expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 90
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "High CPU on {{ $labels.instance }}"
    description: "CPU usage is {{ $value }}% for more than 5 minutes."
```

#### Low Disk Space
```yaml
- alert: LowDiskSpace
  expr: (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100 < 20
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Low disk space on {{ $labels.instance }}"
    description: "Disk space is {{ $value }}% free on {{ $labels.instance }}."
```

#### High Memory Usage
```yaml
- alert: HighMemory
  expr: (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100 < 20
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Low memory on {{ $labels.instance }}"
    description: "Only {{ $value }}% memory available on {{ $labels.instance }}."
```

### Container Alerts

```yaml
- alert: ContainerDown
  expr: absent(container_last_seen{name=~".+"})
  for: 2m
  labels:
    severity: critical
  annotations:
    summary: "Container {{ $labels.name }} is down"
    description: "Container {{ $labels.name }} has not been seen for 2 minutes."
```

### Network Alerts

```yaml
- alert: NetworkDeviceDown
  expr: probe_success{job="ping_exporter"} == 0
  for: 1m
  labels:
    severity: critical
  annotations:
    summary: "Network device {{ $labels.name }} unreachable"
    description: "{{ $labels.name }} ({{ $labels.host }}) is unreachable via ICMP."

- alert: HighLatency
  expr: probe_duration_seconds{job="ping_exporter"} > 0.5
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "High latency to {{ $labels.name }}"
    description: "Ping latency is {{ $value }}s to {{ $labels.name }}."
```

### Application Alerts

```yaml
- alert: LibreNMSDown
  expr: probe_success{job="http_probes", instance="http://librenms:8000"} == 0
  for: 1m
  labels:
    severity: critical
  annotations:
    summary: "LibreNMS is down"
    description: "LibreNMS web UI is unreachable."

- alert: GrafanaDown
  expr: probe_success{job="http_probes", instance="http://grafana:3000/login"} == 0
  for: 1m
  labels:
    severity: critical
  annotations:
    summary: "Grafana is down"
    description: "Grafana is unreachable."

- alert: InfluxDBDown
  expr: probe_success{job="http_probes", instance="http://influxdb:8086/health"} == 0
  for: 1m
  labels:
    severity: critical
  annotations:
    summary: "InfluxDB is down"
    description: "InfluxDB health check failed."
```

## Alert Severities

| Severity | Description | Example |
|----------|-------------|---------|
| **Critical** | Service down, data loss risk | Host down, database unreachable |
| **Warning** | Degraded performance | High CPU, low disk, high latency |
| **Info** | Informational | Deployment completed, backup finished |

## Testing Alerts

```bash
# Trigger a test alert
curl -X POST http://prometheus:9090/api/v1/alerts \
  -H "Content-Type: application/json" \
  -d '{
    "alerts": [
      {
        "labels": {"alertname": "TestAlert", "severity": "info"},
        "annotations": {"summary": "Test alert from Prometheus"},
        "status": "firing"
      }
    ]
  }'

# Silence an alert
curl -X POST http://alertmanager:9093/api/v1/silences \
  -H "Content-Type: application/json" \
  -d '{
    "matchers": [{"name": "alertname", "value": "HostDown", "isRegex": false}],
    "duration": "1h",
    "comment": "Maintenance window"
  }'
```

## Notification Channels

### Discord Webhook

1. Create webhook in Discord server settings
2. Add to Alertmanager config:
```yaml
webhook_configs:
  - url: 'https://discord.com/api/webhooks/CHANNEL_ID/TOKEN'
    send_resolved: true
```

### Gotify Push

1. Create app in Gotify
2. Add token to Alertmanager config:
```yaml
webhook_configs:
  - url: 'http://gotify:8080/message?token=YOUR_APP_TOKEN'
    send_resolved: true
```

### Email (via msmtpd)

msmtpd is already in the stack. Configure SMTP settings in `env.env`:
```bash
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_TLS=on
SMTP_USER=your_email@example.com
SMTP_PASSWORD=your_password
SMTP_FROM=monitoring@example.com
```

## Uptime Kuma Alerts

Uptime Kuma provides additional notification channels beyond Alertmanager:

1. Go to `http://docker2:3001`
2. Settings → Notification Settings
3. Add notification providers:
   - Discord
   - Gotify
   - Telegram
   - Email
   - Slack
   - Webhook

4. Create monitors for critical services:
   - LibreNMS: `http://librenms:8000`
   - Grafana: `http://grafana:3000`
   - InfluxDB: `http://influxdb:8086/health`
   - Prometheus: `http://prometheus:9090/-/ready`
   - All network devices: ping monitors

## Maintenance Windows

To suppress alerts during maintenance:

```bash
# Silence all alerts for 2 hours
curl -X POST http://alertmanager:9093/api/v1/silences \
  -H "Content-Type: application/json" \
  -d '{
    "matchers": [],
    "duration": "2h",
    "comment": "Scheduled maintenance"
  }'

# Silence specific alert for 30 minutes
curl -X POST http://alertmanager:9093/api/v1/silences \
  -H "Content-Type: application/json" \
  -d '{
    "matchers": [{"name": "alertname", "value": "HighCPU", "isRegex": false}],
    "duration": "30m",
    "comment": "Load testing in progress"
  }'
```

## Best Practices

1. **Start with broad alerts** — refine thresholds after observing baseline behavior
2. **Group related alerts** — use `group_by` to reduce notification noise
3. **Set appropriate intervals** — critical: 1m, warning: 5m, info: 15m
4. **Use labels consistently** — `host`, `service`, `severity`, `team`
5. **Test alerts regularly** — verify notification channels work
6. **Document runbooks** — link to remediation steps in alert annotations
7. **Avoid alert storms** — use `repeat_interval` and `group_interval` wisely
