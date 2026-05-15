# bre-public

The public-facing website for **bre.ge** — apartments & the batumi guide.

Live dev URL: `<add after first vercel deploy>`  
Production: `bre.ge` (currently on Tilda, to be switched after dev sign-off)

---

## What's here

| Path | What it is |
|---|---|
| `/` | Home page — hero, edition cover, apartments preview, guide preview, doors, contact |
| `/guide` | Guide landing — 8 worlds, masthead, long reads, news, mini-map |
| `/guide/eat-drink` | First fully-built category page (template for the other 7) |
| `/_drafts/*` | Mobile audit + hero picker (dev helpers, not part of the site) |
| `/docs/*` | Context, geo-link feature spec, next-steps roadmap |

## Stack

- Vanilla HTML/CSS/JS, no build step
- Vercel hosting (free tier, auto-deploy on push)
- Future: Supabase (Postgres + storage) for apartments & guide-posts CRUD
- Future: Mapbox (free 50k loads/month) for the map blocks

## How to develop

1. Clone the repo
2. Open any `.html` directly in a browser — no server needed for now
3. For accurate mobile preview, open `_drafts/mobile-audit.html`

## How to deploy

Just push to `main`. Vercel auto-builds preview URLs for every branch, and updates the main URL on `main`.

```bash
git add .
git commit -m "describe what changed"
git push
```

## URL conventions

Clean URLs (no `.html` extensions) thanks to `vercel.json` `cleanUrls: true`. So:

- File `/index.html` → URL `/`
- File `/guide/index.html` → URL `/guide`
- File `/guide/eat-drink.html` → URL `/guide/eat-drink`

When linking between pages internally, **always use the clean URL** (`/guide/eat-drink`, not the .html).

## Next steps

See `/docs/02_NEXT_STEPS.md` for the full roadmap. Short version:

- [ ] `/stays` list page
- [ ] `/stays/[slug]` apartment detail page
- [ ] `/guide/[slug]` place / story / news detail page
- [ ] 7 remaining category pages (wellness, nightlife, etc — copy `eat-drink.html`, change `--cat-color`)
- [ ] `/own` and `/invest` landing pages
- [ ] `/about` page
- [ ] Admin panel (`/admin` — separate repo or subdomain)
- [ ] Hook up Supabase for live data instead of hardcoded counts
- [ ] Mapbox integration

## Brand cheatsheet

- Colors: orange `#FF5E1A` (accent), teal `#00B5B0`, yellow `#FFD60A`, pink `#E8797C`, purple `#7F77DD`, green `#5DCAA5`, coral `#F0997B`. Bg `#FBF8F1`, paper `#FDFBF6`, ink `#0a0908`.
- Fonts: Bricolage Grotesque (display) + JetBrains Mono (eyebrow/labels).
- All copy is **lowercase**.
- Italic words in headings are accent-orange.
- Voice is "we" — Artem & Olga. Never "I" or "you should".
