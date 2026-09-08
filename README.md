# Ekam Foundation — website

Static site for **ekamfdn.org**. One page, self-contained.

```
site/
  index.html      the page
  emblem.webp     the sun logo (referenced by index.html)
docker-compose.yml               TrueNAS app: git-sync + static server, behind Nginx Proxy Manager
docker-compose.cloudflared.yml   TrueNAS app: git-sync + static server + Cloudflare Tunnel (CGNAT, no NPM)
Caddyfile                        only used by the cloudflared variant / standalone Caddy
```

## How updates reach the live site

1. A change is committed to `main` in this repo.
2. On TrueNAS the **git-sync** container pulls `main` every 60 seconds.
3. The **web** container (Caddy in file-server mode) serves the checked-out `site/`.
   No restart, no rebuild.

Push to `main` -> live within a minute.

## TrueNAS setup (with your existing Nginx Proxy Manager)

Dataset in use: **`Raid HDD / ekam-web`** -> `/mnt/Raid HDD/ekam-web`

1. **Make the git folder** - TrueNAS shell:
   ```
   mkdir -p "/mnt/Raid HDD/ekam-web/git"
   ```
2. **Install the app** - Apps -> Discover Apps -> Custom App -> *Install via YAML* ->
   paste `docker-compose.yml`. It starts `ekam-gitsync` and `ekam-web`
   (serving on host port **8088**).
3. **Nginx Proxy Manager -> Hosts -> Proxy Hosts -> Add Proxy Host**
   - Domain Names: `ekamfdn.org`
   - Scheme: `http`   Forward Hostname/IP: `<your TrueNAS LAN IP>`   Forward Port: `8088`
   - Block Common Exploits: on
   - **SSL** tab: Request a new SSL Certificate (Let's Encrypt), enter e-mail, agree,
     then enable **Force SSL** and **HTTP/2 Support**.
4. **NPM -> Hosts -> Redirection Hosts -> Add**
   - Domain Names: `www.ekamfdn.org`   Forward to: `https://ekamfdn.org`   Code: `301`
   - SSL tab: request a certificate for `www.ekamfdn.org` too, Force SSL.
5. **DNS** at the registrar:
   `A  ekamfdn.org -> <public IP>` and `A  www.ekamfdn.org -> <public IP>`
   (port-forwarding of 80/443 to the NAS is already done if NPM serves your other domains)
6. Open `https://ekamfdn.org`.

If the site is at `192.168.x.x:8088` on the LAN but NPM can't reach it, put NPM and this
app on the same Docker network, or use the NAS host IP as the forward target (above).

## No public IP (CGNAT)

Use `docker-compose.cloudflared.yml` instead (Cloudflare Tunnel, no NPM, no port-forward).
See the comments in that file.

## Editing locally

`site/index.html` is plain HTML with an inline `<style>`. Open it in a browser to preview.
Replace `site/emblem.webp` to change the logo (keep the filename).
