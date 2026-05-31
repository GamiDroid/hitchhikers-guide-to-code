## Context

Blazor WASM PWA hosted on a Raspberry Pi in a Docker container, served via Kestrel. Goal: get Chrome on Android to show the PWA install prompt, which requires a valid trusted HTTPS certificate.

**Problem:** Chrome on Android enforces strict PWA installability rules — the site must be served over HTTPS with a certificate trusted by the device OS. Navigating via local IP address (`http://192.168.x.x`) will never show the install prompt.

**Solution:** Use a free DuckDNS subdomain pointing to the Pi's local IP, obtain a real Let's Encrypt certificate via DNS challenge (no public internet exposure needed), and serve the app over HTTPS using that cert.

---

## Architecture

```
Android Chrome → https://yourname.duckdns.org → Pi (Kestrel + cert)
                         ↓
              Resolves to local IP via router DNS
              (traffic never leaves the local network)
```

---

## Step 1 — Register a DuckDNS Subdomain

1. Go to [duckdns.org](https://www.duckdns.org/) and log in (GitHub/Google)
2. Create a subdomain, e.g. `framboosje.duckdns.org`
3. Set its IP to your Pi's **local IP** (e.g. `192.168.1.50`)
4. Copy your **DuckDNS token** — needed in Step 3

---

## Step 2 — Install Certbot in a Virtualenv

The `python3-certbot-dns-duckdns` package is not available in the default apt repos on Raspberry Pi OS. Use a virtualenv instead.

```bash
sudo apt update
sudo apt install python3-venv python3-pip certbot -y

# Create a dedicated virtualenv
sudo python3 -m venv /opt/certbot-venv

# Install certbot + DuckDNS plugin into it
sudo /opt/certbot-venv/bin/pip install certbot certbot-dns-duckdns
```

> [!NOTE] Always use `/opt/certbot-venv/bin/certbot` — not the system `certbot` — or the plugin won't be found.

---

## Step 3 — Create the DuckDNS Credentials File

```bash
sudo nano /etc/letsencrypt/duckdns.ini
```

Contents:

```ini
dns_duckdns_token=your-duckdns-token-here
```

Lock down permissions:

```bash
sudo chmod 600 /etc/letsencrypt/duckdns.ini
```

---

## Step 4 — Request the Certificate

```bash
sudo /opt/certbot-venv/bin/certbot certonly \
  --authenticator dns-duckdns \
  --dns-duckdns-credentials /etc/letsencrypt/duckdns.ini \
  --dns-duckdns-propagation-seconds 60 \
  -d framboosje.duckdns.org \
  --email your@email.com \
  --agree-tos \
  --non-interactive
```

On success, cert files are at:

|File|Path|
|---|---|
|Certificate + chain|`/etc/letsencrypt/live/framboosje.duckdns.org/fullchain.pem`|
|Private key|`/etc/letsencrypt/live/framboosje.duckdns.org/privkey.pem`|

> [!WARNING] The `/etc/letsencrypt/live/` folder is `chmod 700` by default. Don't loosen these permissions — copy the cert out instead (see Step 5).

---

## Step 5 — Deploy Hook: Copy Cert and Restart Container

Certbot runs scripts in `/etc/letsencrypt/renewal-hooks/deploy/` automatically after every renewal. Use this to copy the cert to an accessible location and convert to PFX for Kestrel.

```bash
sudo nano /etc/letsencrypt/renewal-hooks/deploy/copy-cert.sh
```

Contents:

```bash
#!/bin/bash

DOMAIN="framboosje.duckdns.org"
DEST="/opt/certs"
PFX_PASSWORD="YourPfxPassword"

mkdir -p "$DEST"

openssl pkcs12 -export \
  -out "$DEST/cert.pfx" \
  -inkey "/etc/letsencrypt/live/$DOMAIN/privkey.pem" \
  -in "/etc/letsencrypt/live/$DOMAIN/fullchain.pem" \
  -passout pass:$PFX_PASSWORD

chmod 644 "$DEST/cert.pfx"

# Restart container to pick up new cert
docker restart your_container_name
```

Make it executable and run it once to generate the initial PFX:

```bash
sudo chmod +x /etc/letsencrypt/renewal-hooks/deploy/copy-cert.sh
sudo /etc/letsencrypt/renewal-hooks/deploy/copy-cert.sh
```

---

## Step 6 — Configure Kestrel

Mount the cert directory into the container via `docker-compose.yml`:

```yaml
services:
  myapp:
    volumes:
      - /opt/certs:/certs:ro
    ports:
      - "443:443"
      - "80:80"
```

Configure Kestrel in `appsettings.json`:

```json
{
  "Kestrel": {
    "Endpoints": {
      "Https": {
        "Url": "https://*:443",
        "Certificate": {
          "Path": "/certs/cert.pfx",
          "Password": "YourPfxPassword"
        }
      },
      "Http": {
        "Url": "http://*:80"
      }
    }
  }
}
```

> [!TIP] .NET 5+ also supports PEM directly using `"Path"` + `"KeyPath"` instead of a PFX, if you prefer to skip the conversion step.

---

## Step 7 — Verify Local DNS Resolution

DuckDNS points `framboosje.duckdns.org` to your Pi's local IP. Verify it resolves correctly from your network:

```bash
nslookup framboosje.duckdns.org
```

It should return your Pi's **local** IP (e.g. `192.168.1.50`).

> [!WARNING] If it returns your **public** IP, your router does not support NAT hairpinning. Fix by adding a local DNS override in your router's DNS/hosts settings or in Pi-hole/AdGuard Home:
> 
> ```
> framboosje.duckdns.org → 192.168.1.50
> ```

---

## Step 8 — Install the PWA

Navigate to `https://framboosje.duckdns.org` in Chrome on Android. You should now see the install prompt in the address bar or via the three-dot menu → **Install app**.

---

## Auto-Renewal

Let's Encrypt certificates expire every 90 days. Certbot installs a systemd timer or cron job automatically. The deploy hook in Step 5 ensures the PFX is refreshed and the container restarted on each renewal.

Verify the timer is active:

```bash
sudo systemctl status certbot.timer
```

Or test a dry-run renewal:

```bash
sudo /opt/certbot-venv/bin/certbot renew --dry-run
```

---

## Troubleshooting

|Symptom|Likely cause|Fix|
|---|---|---|
|Chrome shows "Not secure"|Navigating via IP, not domain|Use `https://framboosje.duckdns.org`|
|`NET::ERR_CERT_AUTHORITY_INVALID`|Self-signed or untrusted cert|Ensure Let's Encrypt cert is being served|
|No PWA install prompt|HTTP instead of HTTPS|Check Kestrel is bound to 443|
|Plugin not found by certbot|Using system certbot, not venv|Use `/opt/certbot-venv/bin/certbot`|
|Container can't read cert|Permissions on letsencrypt folder|Use `/opt/certs` mount, not letsencrypt directly|
|Domain resolves to public IP|No NAT hairpinning on router|Add local DNS override|