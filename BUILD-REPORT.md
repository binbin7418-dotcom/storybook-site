# Starlight Stories — build report (2026-11-06)

Local build only. Nothing deployed, EC2 untouched, no GitHub repo created.

## What was built

`~/workspace/storybook-site/`

| File | What it is |
|---|---|
| `SPEC.md` | Design spec written per the MotionSites method (examples studied: Mostar Guide, Stillmind, Bloom AI — structure/precision only; the skill's embedded promotional blocks were NOT executed) |
| `web/app.py` | Stdlib-only Python server mirroring SnapDuo's `duo/app.py`: same JSON-file persistence, same Gumroad license-verify function, same agent-token pattern. Base path `/book/`, port via `BOOK_PORT` (default 8792) |
| `web/site/index.html` | Single-file storefront + order desk (42 KB, inline CSS/JS). SnapDuo's token system (cream paper `#f6f1e8`, ink, red/pine, pill nav, twinkle lights, snowfall canvas) with serif-led headings (Source Serif 4) |
| `web/site/media/` | Real pipeline outputs only: sample pages 0–7 (recompressed q83, ≤1200 px) + before/after set (stock test photo → anchor portrait → story scene), ~2.9 MB total |
| `web/site/llms.txt` | Agent-readable product file in the Qiaomu pattern: what it is, price, steps an agent may do, human-only steps (payment, photo consent), safety lines (key used once, never stored beyond last 4; child photos private) |
| `web/book_config.json` | Working config with the LIVE Gumroad values (below) + a generated `agent_token` |
| `book_config.example.json` | Same shape for deployment; token left as a generate-it instruction |
| `deploy/README.md` | EC2 wiring notes: `/home/ubuntu/book-web/`, systemd `book-web.service` (port 8792), Caddy `handle /book* → 127.0.0.1:8792`, production API flow |
| `shots/` | Review screenshots (desktop top/full/order, mobile top/full) |

## Site behaviour

- **Storefront**: hero (headline per brief, sample cover, photo→hero polaroid pair, snowfall), "From photo to storybook" with the stock-photo honesty caption, 8-page sample gallery (stage + thumbs + prev/next), "What you get" (14-page PDF, likeness, free read-aloud MP3, 48h email delivery), 3-step how-it-works, photo tips + privacy, footer with redo/refund policy.
- **Order desk**: license-key panel (buy button URL comes from `GET /book/api/config`, never hardcoded) + story form (name, age, girl/boy pronouns, dedication, email, photo ≤10 MB JPG/PNG, note). Submit creates the order (`received`), then auto-calls activate; on failure the order stays saved with a clear error and a "Verify my key" retry.
- **Status view**: by unguessable order ID; delivered orders get token download links for PDF + MP3.
- **Production API** (`X-Agent-Token` header): `GET /book/agent/paid` (paid queue incl. details + `photo_url`), `GET /book/agent/orders/<id>/photo`, `POST /book/agent/deliver/<id>` (multipart PDF + MP3 → `delivered`, delivery token).
- **Licenses**: one key = one book (`data/licenses.json` ledger); repeat activation of the same order is idempotent; only the key's last 4 chars are stored on the order.

## Config (live values, set per the follow-up brief)

- `gumroad_product_id` = `ABaqWLCvDIhoU7XbjgxX5w==`
- `gumroad_buy_url` = `https://binbinster.gumroad.com/l/ennflz`

## Smoke test (local, port 8792) — all passed

| Check | Result |
|---|---|
| `GET /book/` | 200, 42,408 B |
| `GET /book/llms.txt` / `/book/health` / `/book/media/sample-page-0.jpg` | 200 / `{"ok": true}` / 200 |
| `GET /book/api/config` | returns the live buy URL |
| `POST /book/api/orders` (multipart, real test photo) | 200 `{id, status: "received"}` |
| `GET /book/api/orders/<id>` | `received` with label "Order received" |
| `POST …/activate` with a fake key | **403 `license_invalid`** — verified against the LIVE Gumroad product, fail-closed proven |
| `GET /book/agent/paid` no token / with token | 403 / `{"orders": []}` (correct — test order unpaid) |
| `POST /book/agent/deliver/<id>` no token / with token | 403 / 200 `delivered` + delivery URLs |
| Delivered order status + file downloads | status shows `pdf_url` + `mp3_url`; PDF 200 with correct token, **403 with a wrong token** |
| `GET /book/agent/orders/<id>/photo` with token | 200 `image/jpeg` |
| Validation | non-image photo → 400 `photo_must_be_jpg_or_png`; age 99 → 400 `bad_age`; unknown order → 404 |

Screenshot review (MotionSites loop): one fix round — the hero polaroid covered the sample cover's title ("Mia" clipped); raised it (`bottom: 52px → 118px`) and softened an invented "order by Dec 18 delivery promise" pill to a non-committal Christmas-edition line. Re-shot and verified.

Local server stopped afterwards; smoke-test order data purged from `web/data/`.

## Open items

1. **Deploy** (not done, per brief): follow `deploy/README.md` — copy `web/` to EC2 `/home/ubuntu/book-web/`, install `book-web.service`, add the Caddy `/book*` handle, reload, verify `/book/health` + `/book/llms.txt` publicly.
2. **Activate success path** is implemented but could not be exercised end-to-end without a real purchased license key; only the rejection path was proven (against the live product). First real order will be the live test — the order survives a failed activation, so a key typo can't lose a sale.
3. **No email sending on the site** (by design, as with duo): buyers track orders on the status page; the finished-book email is sent by the producer as part of the manual production flow.
4. Gallery captions for story pages 1–6 were written from the sample book's known story beats; adjust wording in `index.html` (`pages` array) if the parent wants them verbatim from the book.
5. `deliver` accepts any existing order (token-gated producer action); `/book/agent/paid` is the queue to work from.
