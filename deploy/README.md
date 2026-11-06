# Starlight Stories — deploy notes (DEPLOYED 2026-11-06)

Target: the same EC2 box as SnapDuo (api.newkwlab.com), wired exactly
like duo-web: a systemd service running `web/app.py` on a loopback port,
with Caddy reverse-proxying the public path prefix to it.

## Layout on EC2

```
/home/ubuntu/book-web/
  app.py              <- from web/app.py
  book_config.json    <- create from book_config.example.json (see below)
  site/               <- from web/site/ (index.html, llms.txt, media/)
  data/               <- created at runtime (orders/, licenses.json)
```

## Config

```bash
cp book_config.example.json book_config.json
# generate the agent token once and paste it in:
python3 -c "import secrets; print(secrets.token_hex(24))"
```

- `gumroad_product_id`: `ABaqWLCvDIhoU7XbjgxX5w==` (live product,
  license keys enabled)
- `gumroad_buy_url`: `https://binbinster.gumroad.com/l/ennflz`
- `agent_token`: random hex, shared only with the production side
  (Muse's order-production step). Never commit the real token.

## systemd unit — /etc/systemd/system/book-web.service

```ini
[Unit]
Description=Starlight Stories web app
After=network.target

[Service]
WorkingDirectory=/home/ubuntu/book-web
Environment=BOOK_PORT=8792
ExecStart=/usr/bin/python3 /home/ubuntu/book-web/app.py
Restart=always
RestartSec=3
User=ubuntu

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now book-web
curl http://127.0.0.1:8792/book/health   # {"ok": true}
```

## Caddy (api.newkwlab.com site block)

Add next to the existing `/duo*` handle (keep the BingSiteAuth.xml
handle untouched):

```caddy
handle /book* {
    reverse_proxy 127.0.0.1:8792
}
```

```bash
sudo caddy validate --config /etc/caddy/Caddyfile
sudo systemctl reload caddy
curl https://api.newkwlab.com/book/health
curl https://api.newkwlab.com/book/llms.txt
```

## Production flow (for reference)

1. `GET /book/agent/paid` with header `X-Agent-Token: <agent_token>`
   → paid orders with child details + `photo_url`.
2. `GET /book/agent/orders/<id>/photo` (same header) → the photo.
3. Produce the book per the storybook-production pipeline.
4. `POST /book/agent/deliver/<id>` (same header), multipart form with
   `pdf` (the book PDF) and `mp3` (read-aloud) → order becomes
   `delivered`; buyers download from their order page links
   (`/book/files/<id>/<delivery_token>/book.pdf`).

## Data hygiene

- Unpaid (`received`) orders auto-delete after 7 days; all order data
  after 60 days (same pattern as duo, longer TTL for deliveries).
- Buyer photos live only inside `data/orders/<id>/` — the studio's
  working copy is deleted after delivery per the site privacy copy.
