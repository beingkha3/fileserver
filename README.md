# Nextcloud AIO + Cloudflare Tunnel (Windows, Docker Desktop)

This workspace deploys Nextcloud All‑in‑One (AIO) and a Cloudflare Tunnel client using Docker Compose on Windows. It follows the official AIO guide and uses Cloudflare Tunnel in token mode.

## What’s included
- `docker-compose.yml` – runs the AIO mastercontainer and the cloudflared client
- `.env` – holds your Cloudflare `TUNNEL_TOKEN`

## Prerequisites
- Windows with Docker Desktop (Compose v2)
- A domain on Cloudflare: `kazdrive.kazrotech.org`
- A Cloudflare Zero Trust Tunnel (token-based)

## Configure
1) Put your tunnel token into `.env`:
   - `TUNNEL_TOKEN=...` (already set)

2) Cloudflare Zero Trust → Tunnels → select your tunnel → Public Hostnames:
   - Hostname: `your-domain.com`
   - Service: `http://nextcloud-aio-apache:11000`
   - Preserve host header: on; No TLS verify: off
   - Zone SSL/TLS mode: Full (not Flexible)
   - Disable Rocket Loader; temporarily disable “Always Use HTTPS”/HSTS if blocking setup

Note: The compose connects `cloudflared` to the external `nextcloud-aio` Docker network that the AIO stack uses, so it can route to `nextcloud-aio-apache:11000` directly.

## Start
- Open PowerShell in this folder and run:
  - `docker compose up -d`
- Open the AIO interface locally and complete setup:
  - `https://127.0.0.1:8080` (accept self‑signed cert)
  - Enter your domain `kazdrive.kazrotech.org`
  - Click Start containers and wait until all are Running

## Access
- Public: `https://kazdrive.kazrotech.org`
- AIO interface (internal): `https://127.0.0.1:8080`

## Customizations (optional)
- The AIO Apache listens on port `11000` inside Docker. In compose, we set:
  - `APACHE_PORT=11000`
  - `APACHE_IP_BINDING=0.0.0.0` (required so `cloudflared` can reach it over the Docker network)
- To change Nextcloud’s datadir or other AIO options later, follow the AIO docs and add the respective environment variables to the mastercontainer if needed.

## Useful commands
- Show status: `docker compose ps`
- View logs:
  - Mastercontainer: `docker logs nextcloud-aio-mastercontainer`
  - Apache: `docker logs nextcloud-aio-apache`
  - Nextcloud: `docker logs nextcloud-aio-nextcloud`
  - Tunnel: `docker logs cloudflared`
- Stop all (example):
  - `docker stop cloudflared nextcloud-aio-mastercontainer nextcloud-aio-apache nextcloud-aio-notify-push nextcloud-aio-nextcloud nextcloud-aio-imaginary nextcloud-aio-redis nextcloud-aio-database`

## Troubleshooting
- Redirects to domain but nothing loads:
  - Ensure Tunnel status is Healthy and Public Hostname maps to `http://nextcloud-aio-apache:11000`
  - Verify compose uses `APACHE_IP_BINDING=0.0.0.0`
- 502/522:
  - Check Cloudflare dashboard logs, tunnel routes, and SSL/TLS mode is Full
- Upload limits/timeouts (Cloudflare):
  - Free plan proxy limits uploads to 100MB (unchunked) and 100s request timeout
  - For large uploads, disable proxy for the DNS record or use another method
- Talk TURN on 3478 will not work through Cloudflare Tunnel; use an external TURN server

## Security notes
- Keep your `TUNNEL_TOKEN` secret. `.env` is read by Docker Compose automatically.
- `11000` is not published by compose; only the tunnel should expose your instance.
