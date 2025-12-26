  listen 443 ssl http2;
  server_name staging.dubixo.com;

  ssl_certificate     /etc/letsencrypt/live/staging.dubixo.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/staging.dubixo.com/privkey.pem;
  include             /etc/letsencrypt/options-ssl-nginx.conf;
  ssl_dhparam         /etc/letsencrypt/ssl-dhparams.pem;

  # API
  location ^~ /api/ {
    proxy_pass http://127.0.0.1:8000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    proxy_read_timeout 120s;
    proxy_send_timeout 120s;
  }

  # Admin redirects
  location = /admin  { return 302 /admin/login; }
  location = /admin/ { return 302 /admin/login; }

  # Admin assets
  location ^~ /admin/_next/ {
    proxy_pass http://127.0.0.1:3001/admin/_next/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    proxy_read_timeout 120s;
    proxy_send_timeout 120s;
  }

  # Admin pages
  location ^~ /admin/ {
    proxy_pass http://127.0.0.1:3001;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    proxy_read_timeout 120s;
    proxy_send_timeout 120s;
  }
# Web assets
  location ^~ /_next/ {
    proxy_pass http://127.0.0.1:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    proxy_read_timeout 120s;
    proxy_send_timeout 120s;
  }

  # Favicon from web
  location = /favicon.ico {
    proxy_pass http://127.0.0.1:3000/favicon.ico;
    proxy_set_header Host $host;
  }

  # Web pages
  location / {
    proxy_pass http://127.0.0.1:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    proxy_read_timeout 120s;
    proxy_send_timeout 120s;
  }
}

server {
  listen 80;
  server_name staging.dubixo.com;
  return 301 https://$host$request_uri;
}
