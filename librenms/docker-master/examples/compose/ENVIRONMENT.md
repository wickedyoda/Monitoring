# Environment Setup

Edit these files before production use:

- `.env` for database, Grafana, and InfluxDB credentials.
- `librenms.env` for LibreNMS app behavior and SNMP settings.

## SNMP Variables

Set these in `librenms.env`:

- `LIBRENMS_SNMP_COMMUNITY`: Default SNMP v2c community for device onboarding.
- `SNMP_USER`: SNMPv3 trap user for snmptrapd.
- `SNMP_AUTH`: SNMPv3 auth password.
- `SNMP_PRIV`: SNMPv3 privacy/encryption password.
- `SNMP_AUTH_PROTO`: `SHA` or `MD5`.
- `SNMP_PRIV_PROTO`: `AES` or `DES`.
- `SNMP_SECURITY_LEVEL`: `priv` or `noauth`.
- `SNMP_DISABLE_AUTHORIZATION`: set `no` to require trap auth.

## Apply Changes

Run from this folder:

```bash
docker compose -f compose.yml up -d
```

## Add Devices With SNMP Credentials

In LibreNMS Web UI (`http://localhost:8000`):

1. Go to `Devices` -> `Add Device`.
2. For SNMP v2c, enter the device community.
3. For SNMP v3, enter username/auth/privacy settings.
4. Save and test polling.
