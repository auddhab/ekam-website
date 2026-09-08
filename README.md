# Ekam Foundation — website

Static site for **ekamfdn.org**. One page, self-contained.

```
site/
  index.html      the page
  emblem.webp     the sun logo (referenced by index.html)
docker-compose.yml               single container: clone + pull every 60s + serve (recommended)
docker-compose.two-services.yml  same job as two containers (git-sync + Caddy)
docker-compose.cloudflared.yml   two containers + Cloudflare Tunnel, for CGNAT (no NPM)
Caddyfile                        only used by the cloudflared / standalone-Caddy variant
```

## How updates reach the live site

Push to `main` → the container `git pull`s it within 60 s → it's served immediately.
No restart, no rebuild, nothing to do on the NAS.

---

## Install on TrueNAS SCALE — behind your existing Nginx Proxy Manager

Use **Apps** (not *Containers* — that's experimental LXC). Apps → Discover Apps →
**Custom App**.

First, in a TrueNAS shell:
```
mkdir -p "/mnt/Raid HDD/ekam-web/data"
```

### Option A — "Install via YAML" (if the button is there, top-right of the page)

Paste `docker-compose.yml` as-is. Done.

### Option B — the Custom App form

Fill only these fields:

| Section | Field | Value |
|---|---|---|
| Application name | Application Name | `ekam-web` |
| Image Configuration | Repository | `caddy` |
| Image Configuration | Tag | `2-alpine` |
| Container Configuration | Entrypoint / Command | `/bin/sh` |
| Container Configuration | Args (add three, in order) | `-c` |
| | | `apk add --no-cache git >/dev/null 2>&1; [ -d /site/.git ] \|\| git clone --depth 1 https://github.com/auddhab/ekam-website /site; ( while true; do sleep 60; git -C /site pull -q 2>/dev/null \|\| true; done ) & exec caddy file-server --root /site/site --listen :80 --browse=false` |
| Network Configuration | Add Port → Host Port | `8088` |
| Network Configuration | Container Port | `80` |
| Storage Configuration | Add → Host Path | `/mnt/Raid HDD/ekam-web/data` |
| Storage Configuration | Mount Path | `/site` |

(If the form has a single "Command" list instead of Entrypoint+Args, put the items
in this order: `/bin/sh`, `-c`, then the long string.)

Install. Check `http://<NAS-IP>:8088` on your LAN — the site should load.

### Then, in Nginx Proxy Manager

1. **Proxy Hosts → Add Proxy Host**
   - Domain Names: `ekamfdn.org`
   - Scheme `http` · Forward Hostname/IP `<TrueNAS LAN IP>` · Forward Port `8088`
   - Block Common Exploits: on
   - **SSL** tab → Request a new SSL Certificate (Let's Encrypt) → email + agree →
     **Force SSL** + **HTTP/2 Support**
2. **Redirection Hosts → Add**
   - `www.ekamfdn.org` → `https://ekamfdn.org` · 301 · request a cert for it too
3. **DNS** at the registrar: `A  ekamfdn.org → <public IP>`, `A  www.ekamfdn.org → <public IP>`

Open `https://ekamfdn.org`.

## No public IP (CGNAT)

Use `docker-compose.cloudflared.yml` (Cloudflare Tunnel, no NPM, no port-forward).

## Editing locally

`site/index.html` is plain HTML with an inline `<style>`. Open it in a browser to preview.
Replace `site/emblem.webp` to change the logo (keep the filename).
