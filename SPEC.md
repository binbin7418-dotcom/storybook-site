# Starlight Stories · Personalized Christmas Storybook storefront + order desk

## 0. Brief
Build a single-page storefront + order desk (`index.html`, inline CSS/JS, no build step) for **Starlight Stories** — a $19.99 personalized Christmas storybook: a child's photo becomes a 14-page watercolor PDF where the child is the hero, with a free read-aloud MP3. Audience: parents/grandparents buying a Christmas gift.
Aesthetic: warm cream paper, storybook serif, candlelight red & pine.
Signature idea: **the two polaroids** — in the hero and again in section 1, a real photo (polaroid card, rotated) visibly *becomes* the watercolor hero (framed illustration). Real media carries the page; nothing is a stock decoration.
Fidelity: sibling of SnapDuo (same token system, pill nav, numbered sheet sections, snowfall) so the two shops feel like one studio; headings switch to serif-led for storybook warmth. Add nothing not listed here.

## 1. Stack & files
Single `index.html` with inline `<style>`/`<script>`; served by `web/app.py` under `/book/`. Icons: inline SVG only (star, arrow, book, speaker, mail, shield). No emoji as icons (one 🎄 in a pill note, matching the sibling site's voice, is copy not iconography).

## 2. Fonts
Google Fonts (same URL family as SnapDuo):
`https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600&family=Source+Serif+4:ital,opsz,wght@0,8..60,500;0,8..60,600;1,8..60,400;1,8..60,500&family=IBM+Plex+Mono:wght@500&display=swap`
| Role | Family | Used on |
|---|---|---|
| Display | Source Serif 4, 500/600, italic accents | H1, H2, book title, prices |
| Body/UI | Poppins 400/500/600 | nav, body, buttons, form |
| Labels | IBM Plex Mono 500, uppercase, ls .16–.22em | eyebrows, sec tags, proof line, footers |

## 3. Colour tokens (identical to SnapDuo + one gold)
```css
--bg:#f6f1e8; --ink:#211a13; --ink60:rgba(33,26,19,.6); --ink40:rgba(33,26,19,.4);
--paper:#fffdf9; --line:rgba(33,26,19,.1); --accent:#b3372a; --accent-deep:#93301f;
--pine:#1f6f4a; --gold:#c9922e;
--shadow:0 4px 14px rgba(33,26,19,.08), 0 18px 44px rgba(33,26,19,.08);
--ease:cubic-bezier(.22,1,.36,1);
```
First CSS rule: `html,body{background:var(--bg)}` (no white flash). Grain overlay: same SVG noise data-URI as SnapDuo, opacity .05, z 60, pointer-events none.

## 4. Assets (all local, under /book/media/)
| Slot | File | Size | Fit |
|---|---|---|---|
| Hero book cover | sample-page-0.jpg (1200×1087) | composed page, cream margins | shown whole, radius 26px, no crop |
| Hero polaroid (before) | before-photo.jpg (800×1546) | real photo of a boy, green background | object-fit cover, 4/4.6 |
| Hero card (after) | after-portrait.jpg (900×1721) | watercolor portrait, same boy | object-fit cover, top-aligned |
| Process cards | before-photo.jpg → after-portrait.jpg → after-scene.jpg (900×1721, snowy scene) | three cards, arrow chips between | cover |
| Gallery | sample-page-0..7.jpg (1200×~1100) | the 8 sample pages | stage + thumbs |
Art direction: cream paper tones dominate; every image is a real output of the pipeline. Section 1 carries the honest caption: sample made from a stock photo.

## 5. Layout & sections (reading order)
- **Nav** (pill, as SnapDuo): wordmark `Starlight✶Stories` (serif italic accent), links The book / Sample / How it works, CTA pill `Order — $19.99` → #order. Then the twinkling lights strip (SnapDuo's radial-gradient lights, 16px, `margin-top:74px` is replaced by 26px — the nav here is not fixed).
- **Hero**: grid 1.02fr/.98fr, max 1240px, padding 44/32/30. Left: eyebrow `★ Personalized Christmas storybook · One child, one story`; H1 serif clamp(42px,5.8vw,72px), lh 1.02, three rising lines: "Your child is / the *hero* of this / Christmas story." (em = serif italic, accent red). Sub 17px/1.55 ink60 max 29rem. CTAs: solid `Start your book — $19.99 →` (#order), ghost `Read the sample` (#sample). Proof mono line: `14 pages · PDF + free read-aloud MP3 · By email within 48 hours`. Right stage: cover image (radius 26, shadow), polaroid before-photo rotated −7° at left −26px/top 44px width 30%, after-portrait card rotated 5° at right −18px/bottom 56px width 27% (white frame padding 7px), chip bottom-left `✶ Made from one photo — see how, below` (dot pine).
- **Sheet** (paper, radius 40 top, as SnapDuo) containing numbered sections:
  1. **From photo to storybook** — 3 cards grid (photo / portrait / scene) with `→` arrow chips absolutely placed between on desktop; step captions `1 · You send a photo`, `2 · We paint your hero`, `3 · The adventure begins`; honest caption paragraph below.
  2. **Read the sample book** (#sample) — stage image (radius 24, shadow, aspect ~1200/1100) with fade swap 240ms; thumb strip 8 × 84px, active thumb accent ring; prev/next round buttons 46px; caption line updates: `Cover — Mia and the Night Before Christmas`, `Page 1 …` … `Dedication page`; below, the note `A complete sample book, made by our studio from an illustrated hero. Yours starts from your child's photo.`
  3. **What you get** — 4 cards grid (2×2 mobile 1): Book icon `14 pages` (Cover + 12 story pages + a dedication page with your words); Portrait icon `Their face, every page` (one anchor portrait, painted once, carried through all 12 scenes); Speaker icon `Read-aloud MP3, free` (a warm narrated version of the whole book, included — not an upsell); Mail icon `In your inbox in 48 hours` (email delivery + a private download link that stays yours). Then **How it works** 3 steps in numbered rows: `1 Buy the book on Gumroad` → receipt email carries your license key; `2 Come back here` → enter the key, upload one photo, tell us their name; `3 We craft it by hand` → a person checks every page, then the PDF + MP3 land in your inbox.
  4. **Photo tips & privacy** — two columns: tips (front-facing, eyes open, no sunglasses/hat shadow, one child per photo, phone photos are fine) / privacy (photo used only to paint this book; never sold, never used to train anything; deleted from our working files within 7 days of delivery; the book is yours to print and share).
  5. **Order your book** (#order) — the order desk, two stacked panels:
     - Panel A `Your license key`: mono label, key input (uppercase, ls .06em), button `Get your license key — $19.99` (ink, href filled from /book/api/config buy_url), fine print `Already bought? The key is in your Gumroad receipt email.`
     - Panel B `Your child's story`: fields grid 2col — Child's first name (text), Age (number 1–12), Pronouns (select: She / her · girl / He / him · boy), Your email (email); Dedication line (text, optional, placeholder `For Mia — may your Christmas always be full of wonder. Love, Grandma`); Photo dropzone (dashed, preview thumb, `One clear photo, JPG or PNG, up to 10 MB`); Note (text, optional, `Anything we should know? (optional)`).
     - Submit `Send my order →` (solid). Status line under. On success: success panel replaces button area: `Order received — order #ID`, status link `/book/#status` prefilled, and if activation failed: clear warning + `Verify my key` retry button.
     - Below: **Check an order** box: order-id input + `Check status` → renders status card (status pill, child name, timeline text, download buttons when delivered: `Download the book (PDF)`, `Download the read-aloud (MP3)`).
- **Footer** (paper, 4 cols as SnapDuo): The book / Privacy / Redos & refunds (`If a page doesn't look like your child, tell us within 7 days — we repaint it free. If we can't get it right, we refund you.`) / Contact `binbin7418@gmail.com · We reply within one business day.`

## 6. Motion
Default ease `cubic-bezier(.22,1,.36,1)`. Hero lines rise (translateY 112%→0, .12s stagger, 1s). fadeUp .8–1.1s staggered as SnapDuo. `.reveal` scroll-in (IO threshold .05, rootMargin 500px, 3s fallback). Snowfall canvas: SnapDuo's script verbatim (≤90 flakes, pauses when hidden, skipped under reduced motion). Gallery swap: opacity 0→1 240ms. Hovers: buttons scale 1.035, cards translateY −4px. Reduced motion: kill all animation/transition, reveals visible.

## 7. Interaction & state (JS)
State: `buyUrl` (from GET /book/api/config on load), `photoFile`, `orderId`, `licenseKey`.
Order submit algorithm: validate (key non-empty → else focus key panel with message; name/email/photo present; photo type jpg/png, ≤10MB) → build FormData (fields + photo) → POST /book/api/orders → on 200 store order id → POST /book/api/orders/<id>/activate {license_key} → if activated: success state `Paid — in production`; if 403: warning state with the server error + retry button calling activate again; if create failed: error message, form stays editable.
Status check: GET /book/api/orders/<id> → render pill (Received / In production / Delivered) + copy per status + delivery buttons from returned URLs. Read `?order=` / `#status` hash prefill on load.
Gallery: `pages[8]` array with captions; thumb click / prev / next set index, swap stage src with fade, update caption + active ring.

## 8. Responsive
≤900px: hero 1 col, stage below copy; process cards 1 col (arrows rotate 90°/hidden); form grid 1 col. ≤640px: nav links hidden; thumbs 64px. No horizontal scroll at 375px (polaroid offsets shrink like SnapDuo).

## 9. Constraints & acceptance
Do not: invent testimonials, ratings or sales numbers; promise instant/AI-in-a-minute delivery (this is hand-checked, 48h); hide the stock-photo honesty caption.
Must: English copy throughout; real sample media only; form works without any framework; buy button URL comes from server config, never hardcoded.
Acceptance (first viewport): cream paper, pill nav, twinkle lights, serif headline with red italic "hero", cover standing right with the photo polaroid bottom-left and portrait card, snow falling gently.
