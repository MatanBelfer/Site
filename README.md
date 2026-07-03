# Personal site — Hugo starter

A minimal, dependency-free portfolio: no JavaScript, no external fonts or CDNs,
one fingerprinted stylesheet. Designed to run behind Caddy with a strict CSP.

## 1. Run locally

Install Hugo (extended not required):

```bash
# Debian/Ubuntu (check version is >= 0.120)
sudo apt install hugo
# or grab the latest release binary:
# https://github.com/gohugoio/hugo/releases

hugo server        # live-reload dev server at http://localhost:1313
hugo --minify      # production build into ./public/
```

## 2. Make it yours

| What                  | Where                                   |
|-----------------------|-----------------------------------------|
| Name, role, thesis    | `hugo.toml` → `[params]`                |
| Links index           | `hugo.toml` → `[[params.links]]` blocks |
| Intro paragraph       | `content/_index.md`                     |
| Projects              | `content/projects/*.md` (one file each) |
| Colors & type         | `assets/css/main.css` → `:root` tokens  |
| Favicon               | `static/favicon.svg`                    |

Add a project:

```bash
hugo new projects/my-thing.md
```

Front matter fields: `title`, `date`, `tech` (list), `repo`, `live`, `description`.

## 3. Deploy — Option A: GitHub Pages (free, recommended to start)

The repo includes `.github/workflows/hugo.yaml` (Hugo 0.163.3, official
Pages actions). One-time setup:

1. Create a **public** repo (e.g. `MatanBelfer/xaipen.net`) and push this
   directory to it (`main` branch).
2. Repo → **Settings → Pages** → Source: **GitHub Actions**.
3. Same screen → Custom domain: `xaipen.net`, then at your DNS provider:
   - Apex `A` records → `185.199.108.153`, `185.199.109.153`,
     `185.199.110.153`, `185.199.111.153`
   - `CNAME` record for `www` → `matanbelfer.github.io`
4. Tick **Enforce HTTPS** once the certificate is issued (can take a few
   minutes after DNS propagates).

From then on: `git push` = deployed. The workflow passes `--baseURL` from
Pages settings automatically, so it works before and after the custom
domain is attached.

## 3b. Deploy — Option B: your own VPS / LXC

Build locally, sync over SSH:

```bash
hugo --minify
rsync -avz --delete public/ deploy@your-server:/srv/www/example.com/
```

Tip: create a dedicated `deploy` user on the server that owns only
`/srv/www/example.com` — don't deploy as root.

## 4. Caddy config (automatic HTTPS + security headers)

`/etc/caddy/Caddyfile`:

```caddyfile
example.com {
    root * /srv/www/example.com
    file_server
    encode zstd gzip

    header {
        # Strict CSP works because the site has no JS and no inline styles.
        Content-Security-Policy "default-src 'none'; style-src 'self'; img-src 'self'; font-src 'self'; base-uri 'none'; form-action 'none'; frame-ancestors 'none'"
        Strict-Transport-Security "max-age=31536000; includeSubDomains"
        X-Content-Type-Options "nosniff"
        Referrer-Policy "strict-origin-when-cross-origin"
        Permissions-Policy "camera=(), microphone=(), geolocation=()"
        -Server
    }

    # Long cache for the fingerprinted CSS, short for HTML
    @static path /css/*
    header @static Cache-Control "public, max-age=31536000, immutable"
    header Cache-Control "public, max-age=300"
}

www.example.com {
    redir https://example.com{uri} permanent
}
```

Reload with `sudo systemctl reload caddy`. Certificates are automatic.

## 5. Notes

- If you later add JavaScript or inline styles, loosen the CSP accordingly
  (`script-src 'self'`, etc.) — Firefox devtools will tell you exactly
  what's blocked.
- `enableRobotsTXT = true` generates a permissive robots.txt; edit
  `layouts/robots.txt` if you want to restrict crawlers.
- RSS and taxonomies are disabled in `hugo.toml` (`disableKinds`) — remove
  entries from that list if you add a blog later.
