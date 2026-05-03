# Security notes

## Default credentials

Most of these devices ship with default passwords that are never changed in production deployments:

| Family | Default comm key | Default device password |
|--------|-----------------|------------------------|
| ZKTeco ZEM/ZAM | `0` or `123456` | Varies by model |
| Realand M61BH | `123456` | — |

**Change these** before deploying on any network with internet exposure.

## Network exposure

- Port 4370 (ZKTeco Pull): unauthenticated on some firmware versions, weak auth on others. Do not expose to internet.
- ADMS push: device sends cleartext HTTP to your server. Use a firewall rule to restrict to the device's IP.
- WebSocket mode: no TLS by default. For production, consider a TLS-terminating reverse proxy (nginx/Caddy) in front of the listener.
- Telnet (port 23): plaintext shell. Firewall this at the switch/router level.

## CVE-2023-31711 (ZKTeco file read)

Affects ZKTeco devices with web interface on port 80. Allows unauthenticated file read including `/etc/passwd`. Patched in newer firmware. ZAM180 series (eSSL X2008 etc.) has this patched.

Check:
```bash
curl "http://<device-ip>/../../../../etc/passwd"
```

## Biometric data handling (India: DPDP Act 2023)

Fingerprint and face templates stored on these devices constitute **sensitive personal data** under India's Digital Personal Data Protection Act 2023 and previously the IT Rules 2011.

Operators are responsible for:
- Explicit consent from enrolled individuals
- Defined retention period
- Deletion on request or purpose fulfillment
- Breach notification within 72 hours

Most factory/SME deployments of these devices are non-compliant by default. The ADMS server and listener tools in these repos log attendance data — review your retention policy before deploying.
