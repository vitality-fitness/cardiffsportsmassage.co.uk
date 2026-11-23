# Agent guide

Classic Hugo brochure site. Netlify builds and publishes it. There is no npm app and no Hugo modules.

## Source of truth

Edit only:

- `content/` — page copy (Markdown and shortcodes)
- `data/` — structured site data (e.g. `treatments.yaml` for prices and Acuity links)
- `layouts/` — project overrides, `shortcodes/`, (`banner.html`, `contact.html`, `messages.html`, `robots.txt`)
- `themes/default/` — site-wide chrome (header, footer, `single.html`, CSS)
- `static/` — images and other assets, except `static/open-iconic/`
- `archetypes/default.md`, `config.toml`, and `netlify.toml`

Never edit `public/` or `resources/`. Both are generated. Do not commit `public/`, `resources/_gen/`, `.hugo_build.lock`, or `hugo_stats.json`.

Layout lookup is project first, then theme. If `layouts/partials/banner.html` exists, change that file. Change `themes/default/` only when no project override exists.

## Content

Pages are Markdown plus shortcodes in `layouts/shortcodes/`. Do not put raw HTML in `content/` — `markup.goldmark.renderer.unsafe` is `false` in `config.toml`.

Treatment prices, durations, and Acuity URLs live in `data/treatments.yaml`. Customer review text lives in `data/reviews.yaml`. Homepage feature blurbs live in `data/homefeatures.yaml`. The prices-page consultation notice is in `data/messages.yaml`. FAQ copy lives in `data/faq.yaml` and renders via the `faq-list` shortcode; set `faqPage: true` on `content/faq.md` for FAQPage JSON-LD. Shortcodes read these files via `hugo.Data`.

Page `description` front matter should mention Cardiff and the service where it helps SEO (~150 characters). Do not add `AggregateRating` schema on the reviews page; Trustpilot and Google links are the source of truth for third-party ratings.

`params.htmlLang` sets the document language on layout shells. Inner pages use `layouts/partials/breadcrumbs.html` and `breadcrumb-schema.html` (not on home). Prices uses `pricesPage: true` for Service/Offer JSON-LD from `treatments.yaml`. Site-wide sitemap defaults sit in `config.toml` `[sitemap]`; home sets `sitemap.priority: 1`. `disableKinds` omits empty taxonomy URLs.

Front matter on existing pages is `title`, `date`, `draft`, and `description`. Match that. Create pages with `hugo new content/<name>.md`, which uses `archetypes/default.md` and starts as `draft: true`. A draft is omitted from the production build. Set `draft: false` only when the page should go live.

Confirmation and utility pages (contact received, order received/cancelled, GCR opt-in) use `seoNoIndex: true` and `sitemap: disable: true` so they are not indexed. Default share image and OG dimensions live under `[params]` in `config.toml`.

The filename is the URL: `content/prices.md` is `/prices/`. Renaming a file changes the public URL.

| Label | File | URL |
|---|---|---|
| Home | `content/_index.md` | `/` |
| About Us | `content/about.md` | `/about/` |
| Sports Massage | `content/about-sports-massage.md` | `/about-sports-massage/` (child under About in nav) |
| Vouchers | `content/gift-vouchers.md` | `/gift-vouchers/` |
| Prices | `content/prices.md` | `/prices/` |
| Reviews | `content/reviews.md` | `/reviews/` |
| FAQ | `content/faq.md` | `/faq/` |
| Contact | `content/contact.md` | `/contact/` |

Google Customer Reviews opt-in: `content/gcr.md` at `/gcr/`, layout `gcr`.

The header menu comes from `[[menu.main]]` in `config.toml`, rendered by `themes/default/layouts/partials/header.html`. A new page needs a menu entry only if it should appear in the nav.

Booking and voucher links (`vitfitcdf.as.me`, Acuity) are external. A price change in `data/treatments.yaml` does not change the price in Acuity.

## Do not touch

- `static/open-iconic/` — vendored icon set, not site content
- `baseURL` in `config.toml` — production URL
- `unsafe = false` — do not turn the Goldmark unsafe renderer on; use shortcodes for markup
- `HUGO_VERSION` in `netlify.toml` — stays `0.170.1`; match that version locally; do not change the pin, publish directory, or production build command unless asked
- A second build system or Hugo modules, unless asked

`netlify.toml` is the Netlify build source of truth. Production runs `hugo --gc --minify` (no `-D`), so drafts stay out of the live site. Deploy previews and branch deploys override `baseURL` with `$DEPLOY_PRIME_URL`. `content/covid-19.md` stays `draft: true`; Netlify redirects `/covid-19/` to `/faq/`.

## Verify

After a content or layout change:

```bash
hugo --minify
hugo server -D --bind 127.0.0.1 --port 1313
```

Open the changed URL and one other page that shares the layout. `single.html` wraps every content page. For CSS, header, or footer changes, check a desktop width and a phone width.

One task, one layer. A price change stays in `data/treatments.yaml`. A nav change is `config.toml`. A footer change is `themes/default/layouts/partials/footer.html`.

Performance: no Facebook or Twitter SDKs — footer share links use intent URLs only. Trustpilot loads in the footer next to its widget. Fonts use `display=swap`. Content photos use the `photo` shortcode with descriptive `alt`, `width`/`height`, lazy loading, and a matching `.webp` beside the JPEG in `static/img/`. When creating WebP from camera JPEGs, apply EXIF orientation first (e.g. Pillow `ImageOps.exif_transpose`) — `cwebp` alone can ignore rotation metadata.