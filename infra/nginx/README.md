# Nginx — Dubixo (staging)

## What lives where

### Source-controlled (Git)
- `infra/nginx/sites-available/staging.dubixo.com.conf`
- `infra/nginx/includes/security_headers_baseline.conf`
- `infra/nginx/includes/csp_report_only_html.conf`

### Runtime on EC2 (what Nginx actually uses)
- Vhost:
  - `/etc/nginx/sites-available/staging.dubixo.com`
  - `/etc/nginx/sites-enabled/staging.dubixo.com` -> symlink to the file above
- Includes:
  - `/etc/nginx/includes/security_headers_baseline.conf`
  - `/etc/nginx/includes/csp_report_only_html.conf`

Nginx loads `sites-enabled` from `/etc/nginx/nginx.conf`, not from this repo checkout.

## Deploy (EC2)

Assuming the repo is checked out at `/srv/dubixo/dubixo-infra`:

```bash
INFRA="/srv/dubixo/dubixo-infra"

sudo mkdir -p /etc/nginx/includes

sudo cp "$INFRA/infra/nginx/includes/security_headers_baseline.conf" \
  /etc/nginx/includes/security_headers_baseline.conf

sudo cp "$INFRA/infra/nginx/includes/csp_report_only_html.conf" \
  /etc/nginx/includes/csp_report_only_html.conf

sudo cp "$INFRA/infra/nginx/sites-available/staging.dubixo.com.conf" \
  /etc/nginx/sites-available/staging.dubixo.com

sudo ln -sf /etc/nginx/sites-available/staging.dubixo.com \
  /etc/nginx/sites-enabled/staging.dubixo.com

sudo nginx -t && sudo systemctl reload nginx
