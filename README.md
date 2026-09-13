# munimarga.in

Static site for Muni — three YouTube strands, a blog, and paid readings.
No framework, no dependencies. Posts are markdown; a small Node script turns them into the site.

## Layout

```
content/posts/*.md     the posts — this is the only place content lives
static/                the site itself (pages, assets, scripts)
admin/                 the writing interface (Sveltia CMS)
build.js               generates dist/ from the two folders above
```

`build.js` writes `dist/`, which is what gets served. It:

- parses each markdown file into the block format the post page renders
- writes one `content/<slug>.json` per post, so a post page loads only its own text
- generates `blog-index.js` (the Writing page's list), sorted newest first
- generates `sitemap.xml` and `robots.txt`
- skips any post with `draft: true`

Nothing in `dist/` should be edited or committed.

## Cloudflare Pages settings

| Setting | Value |
| --- | --- |
| Build command | `node build.js` |
| Build output directory | `dist` |
| Root directory | *(leave empty)* |

Every push to `main` rebuilds and deploys.

## Writing a post

Go to `munimarga.in/admin`, sign in with GitHub, click **New Post**. Publishing commits
the markdown file, which triggers a rebuild. Live in about a minute.

By hand, if you prefer: add a file to `content/posts/`.

```markdown
---
title: "The title"
published: "2026-10-04"
group: "astrology"
tag: "Astrology"
excerpt: "Two or three sentences, shown on the card and in search results."
cover: "/assets/uploads/something.jpg"
---

An opening paragraph.

## A section heading

### A sub-heading

![Caption for the picture](/assets/uploads/picture.jpg)

> A pulled quote.

```verse
तमसो मा ज्योतिर्गमय
```
```

The filename becomes the URL: `content/posts/my-post.md` → `/post.html?p=my-post`.
Renaming a published file breaks its links and loses its search ranking — don't.

`group` must be one of `astrology`, `philosophy`, `practice`; it sets the filter the post
appears under and its accent colour. `read` is optional and calculated if omitted.

## Setting up the CMS login (once)

The admin page needs a small OAuth relay so GitHub will let it commit.

1. GitHub → **Settings → Developer settings → OAuth Apps → New OAuth App**.
   Homepage `https://munimarga.in`; callback URL left blank for now. Note the
   **Client ID** and generate a **Client Secret**.
2. Deploy the relay: <https://github.com/sveltia/sveltia-cms-auth> — "Deploy to
   Cloudflare Workers". Set `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET` and
   `ALLOWED_DOMAINS=munimarga.in` as worker variables.
3. Copy the worker URL and set it as `base_url` in `admin/config.yml`.
4. Back in the OAuth app, set the callback URL to `<worker-url>/callback`.

Optionally add a second gate in Cloudflare **Zero Trust → Access** restricting
`munimarga.in/admin*` to your email address.

## Things that live outside this repo

- **Bookings** — Formspree form `xdeovrje`, emailed to you.
- **Newsletter** — Buttondown, `munimarga`.
- **Payments** — UPI deep links and QR codes generated in the page; PayPal.Me for
  international. No payment data touches the site.
- **Availability calendar** — the times offered on the readings page are stored in the
  browser of whoever set them, not on a server. Set them on one device and use that
  device. Reach the panel at `/readings.html?admin=muni`.
- **Post images from before the move** — still served from the old Wix CDN. New uploads
  land in `static/assets/uploads/` and belong to this repo.
