# n8n + Cloudflare Tunnel (Docker Compose)

This package adds `cloudflared` to the supplied n8n Compose setup and configures n8n to generate public HTTPS webhook URLs.

## Before starting

1. In Cloudflare Zero Trust, create or open a **named tunnel** and copy its token.
2. Add a **Published application route** / public hostname to that tunnel:
   - Hostname: the same hostname you put in `N8N_HOST` (example: `n8n.example.com`)
   - Service type: `HTTP`
   - Service URL: `http://n8n:5678`
3. Copy `.env.example` to `.env` in this directory.
4. Edit `.env`:
   - Set `N8N_HOST` to your actual public hostname, without `https://`.
   - Set `N8N_ENCRYPTION_KEY` to the **existing** encryption key used by your current n8n installation.
   - Set `CLOUDFLARE_TUNNEL_TOKEN` to the tunnel token from Cloudflare.

Do not share `.env` or commit it to GitHub; it contains secrets.

## Start

Run PowerShell in this directory:

```powershell
Copy-Item .env.example .env
notepad .env
# After editing and saving .env:
docker compose config
docker compose up -d
docker compose ps
docker compose logs -f cloudflared
```

Open `https://YOUR_HOSTNAME` after both containers are running.

## Important: preserve existing n8n data

- This Compose file keeps the volume name `n8n_data`, but Docker volume names are scoped to the Compose project. Run this from the **same project directory / Compose project** as your current installation if you need to reuse the existing volume.
- Before changing the setup, back up your n8n data and keep a copy of your current Compose file and `.env`.
- Do not change `N8N_ENCRYPTION_KEY` for an existing instance. A different key can make stored credentials unreadable.
- If your current installation uses a differently named Docker volume or a custom Compose project name, identify and reuse that volume/project instead of accidentally creating a new empty n8n instance.

## Local access and networking

The mapping `127.0.0.1:5678:5678` is retained so n8n is reachable from the host computer only. The `cloudflared` container reaches n8n over the Compose network using `http://n8n:5678`; do not use `localhost:5678` as the Cloudflare service URL when the tunnel runs in Docker.

## Troubleshooting

```powershell
docker compose ps
docker compose logs --tail 100 n8n
docker compose logs --tail 100 cloudflared
```

- If Cloudflare shows a 502, confirm the tunnel's service URL is `http://n8n:5678` and n8n is healthy.
- If the public URL is wrong in webhook nodes, verify `N8N_HOST` and `N8N_WEBHOOK_URL`, then recreate the n8n container with `docker compose up -d`.
- Your computer and Docker must remain running and connected to the internet for this self-hosted instance to be reachable.
