# Getting Started

## Prerequisites

- Docker Engine 20.10+ on docker2 (ARM64)
- Docker Compose v2+
- Tailscale installed on docker2 (for tailscale_exporter)
- SNMP community string configured on all network devices

## Step 1: Clone the Repository

```bash
git clone https://github.com/wickedyoda/Monitoring.git
cd Monitoring
```

## Step 2: Configure Environment

```bash
# Copy the example env file
cp env.env.example env.env

# Edit with your settings
nano env.env
```

### Required Changes

1. **Database passwords**
   - `MARIADB_ROOT_PASSWORD`
   - `MARIADB_PASSWORD`
   - `GRAFANA_PASSWORD`
   - `GF_SECURITY_ADMIN_PASSWORD`

2. **InfluxDB credentials**
   - `DOCKER_INFLUXDB_INIT_PASSWORD`
   - `DOCKER_INFLUXDB_INIT_ADMIN_TOKEN`

3. **SNMP community**
   - `LIBRENMS_SNMP_COMMUNITY`

4. **SMTP relay** (optional)
   - `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASSWORD`

5. **Tailscale exporter** (optional)
   - `TAILSCALE_API_KEY` — generate at https://login.tailscale.com/admin/settings/keys
   - `TAILSCALE_TAILNET` — your tailnet name (e.g., `your-name.ts.net`)

## Step 3: Configure SNMP Devices

LibreNMS will auto-discover devices on the network, but you can pre-configure SNMP community strings on your devices:

### OpenWrt (Outdoor-AP, problem-child)
```bash
uci set snmpd.@snmpd[0].community='your_community_string'
uci commit snmpd
/etc/init.d/snmpd restart
```

### Dell N3048P
```bash
configure
snmp-server community your_community_string ro
end
write memory
```

### Flint4
```bash
uci set snmpd.@snmpd[0].community='your_community_string'
uci commit snmpd
/etc/init.d/snmpd restart
```

## Step 4: Deploy the Stack

```bash
# Start all services
docker compose up -d

# Check status
docker compose ps

# View logs
docker compose logs -f
```

## Step 5: Initialize LibreNMS

1. Visit `http://docker2:8000` in your browser
2. Complete the web installer:
   - Database: use `db` as host, `librenms` as database name
   - Create admin user
   - Add your SNMP community string
3. After install, add devices:
   - Auto-discovery: Settings → Discovery → Discover
   - Manual add: Devices → Add Device

## Step 6: Configure Grafana

1. Visit `http://docker2:3000`
2. Login with admin / your `GF_SECURITY_ADMIN_PASSWORD`
3. Add data sources:
   - **Prometheus**: URL `http://prometheus:9090`
   - **InfluxDB**: URL `http://influxdb:8086`, org `monitoring`, your token
   - **Loki**: URL `http://loki:3100`

## Step 7: Configure Uptime Kuma

1. Visit `http://docker2:3001`
2. Create admin account
3. Add monitors for your services:
   - HTTP: `http://librenms:8000`
   - TCP: `db:3306`
   - Ping: all hosts

## Step 8: Verify Exporters

```bash
# Check all exporters are responding
curl http://docker2:9100/metrics | head -5  # Node
curl http://docker2:8080/metrics | head -5  # cAdvisor
curl http://docker2:9121/metrics | head -5  # Redis
curl http://docker2:9116/metrics | head -5  # SNMP
curl http://docker2:9273/metrics | head -5  # Ping
curl http://docker2:9256/metrics | head -5  # Process
curl http://docker2:9430/metrics | head -5  # SpeedTest
curl http://docker2:9184/metrics | head -5  # Tailscale
```

## Step 9: Configure Alerting

1. Edit `alertmanager/alertmanager.yml`
2. Add your notification receivers:
   - Email
   - Discord webhook
   - Gotify push URL
3. Reload Alertmanager:
   ```bash
   docker compose exec alertmanager kill -HUP 1
   ```

## Step 10: Import Grafana Dashboards

Recommended dashboards:
- **Node Exporter Full**: ID `1860`
- **cAdvisor**: ID `893`
- **Redis**: ID `763`
- **Prometheus Stats**: ID `2`
- **Loki Logs**: ID `13757`

Import via Grafana → Dashboards → Import → Enter ID.

## Updating the Stack

```bash
cd Monitoring
git pull origin main
docker compose pull
docker compose up -d
```

## Backup

```bash
# Backup all volumes
docker compose down
tar czf monitoring-backup.tar.gz \
  /var/lib/docker/volumes/monitoring_db_data \
  /var/lib/docker/volumes/monitoring_librenms_data \
  /var/lib/docker/volumes/monitoring_prometheus_data \
  /var/lib/docker/volumes/monitoring_alertmanager_data \
  /var/lib/docker/volumes/monitoring_influxdb_data \
  /var/lib/docker/volumes/monitoring_grafana_data \
  /var/lib/docker/volumes/monitoring_loki_data \
  /var/lib/docker/volumes/monitoring_uptime_kuma_data
```

## Next Steps

- Configure SNMP on all network devices
- Set up alert rules in `prometheus/rules/`
- Customize Grafana dashboards
- Add notification channels in Uptime Kuma
- Review and tune alert thresholds
