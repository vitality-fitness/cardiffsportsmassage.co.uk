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

Treatment prices, durations, and Acuity URLs live in `data/treatments.yaml`. Customer review text lives in `data/reviews.yaml`. Homepage feature blurbs live in `data/homefeatures.yaml`. The prices-page consultation notice is in `data/messages.yaml`. A price or booking link change starts in the treatments file, not in repeated HTML. Shortcodes read these files via `hugo.Data`.

Front matter on existing pages is `title`, `date`, `draft`, and `description`. Match that. Create pages with `hugo new content/<name>.md`, which uses `archetypes/default.md` and starts as `draft: true`. A draft is omitted from the production build. Set `draft: false` only when the page should go live.

The filename is the URL: `content/prices.md` is `/prices/`. Renaming a file changes the public URL.

| Label | File | URL |
|---|---|---|
| Home | `content/_index.md` | `/` |
| About Us | `content/about.md` | `/about/` |
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

`netlify.toml` is the Netlify build source of truth. Production runs `hugo --gc --minify` (no `-D`), so drafts stay out of the live site. Deploy previews and branch deploys override `baseURL` with `$DEPLOY_PRIME_URL`.

## Verify

After a content or layout change:

```bash
hugo --minify
hugo server -D --bind 127.0.0.1 --port 1313
```

Open the changed URL and one other page that shares the layout. `single.html` wraps every content page. For CSS, header, or footer changes, check a desktop width and a phone width.

One task, one layer. A price change stays in `data/treatments.yaml`. A nav change is `config.toml`. A footer change is `themes/default/layouts/partials/footer.html`.