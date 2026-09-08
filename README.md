# Ekam Foundation — website

Static site for **ekamfdn.org**. One page, self-contained.

```
site/
  index.html     the page
  emblem.webp     the sun logo (referenced by index.html)
  _redirects     www -> apex (used only if hosted on Cloudflare Pages)
  _headers       cache / security headers (Cloudflare Pages only)
Caddyfile        web-server config for the TrueNAS deployment
docker-compose.yml               TrueNAS app: git-sync + Caddy (needs a public IP)
docker-compose.cloudflared.yml   TrueNAS app: git-sync + Caddy + Cloudflare Tunnel (CGNAT)
```

## How updates reach the live site

1. A change is committed to `main` in this repo.
2. On the TrueNAS box, the **git-sync** container pulls `main` every 60 seconds.
3. **Caddy** serves the checked-out `site/` folder. No restart, no rebuild.

So: push to `main` → live within a minute.

## First-time TrueNAS setup

1. **Dataset** — create `<pool>/apps/ekam-web` with sub-folders `git/`, `data/`, `config/`.
2. **Caddyfile** — copy this repo's `Caddyfile` to `/mnt/<pool>/apps/ekam-web/Caddyfile`.
3. **Free the ports** — TrueNAS SCALE's UI uses 80/443. *System Settings → General → GUI* → set HTTP `81`, HTTPS `444`, save. The UI moves to `https://<nas-ip>:444`.
4. **Install the app** — *Apps → Discover Apps → Custom App → Install via YAML* → paste `docker-compose.yml` (edit the `/mnt/tank/...` paths to your pool).
5. **Router** — forward TCP **80** and **443** to the NAS IP.
6. **DNS** — at the registrar: `A  ekamfdn.org → <public IP>` and `A  www → <public IP>`.
7. Open `https://ekamfdn.org`. Caddy fetches the certificate automatically on first hit.

No public IP (CGNAT)? Use `docker-compose.cloudflared.yml` instead — see the comments in that file.

## Editing locally

`site/index.html` is plain HTML with an inline `<style>`. Open it in a browser to preview.
Replace `site/emblem.webp` to change the logo (keep the same filename).
