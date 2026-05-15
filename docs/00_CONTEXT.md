# bre.ge — handover for the next Claude chat

## who I am

- **Артём Гринберг** — founder, real estate operator
- **bre.ge** — Batumi (Georgia) real estate & lifestyle brand
- 10+ apartments managed, $1.5M property transaction volume in a year
- Co-founder Olga
- Telegram for guests: @bre_ge · Channel: t.me/batumirealflats
- Personal site (for reference, conservative style): grinbergart.com

## current stack

- **Public site:** still on Tilda (will be replaced — see plan below)
- **Admin/PMS:** **already built** — PWA at bre.ge/finance
  - Stack: HTML/JS, hosted on **GitHub Pages**, data in **Supabase** (cloud Postgres)
  - 9 sections: bookings, expenses, owners, calendar, reports, P&L, monthly, object, settings
  - Has integrated Claude AI voice assistant
  - Dark mode, installable as PWA
  - Brand color in admin: **dusty pink #e8797c**
- **Guide:** ~100+ posts on Tilda Потоки (Feeds), 9 categories: Eat & Drink, Wellness, Move Around, Stay & Live, Nightlife, Work & Remote, Experiences, Explore Batumi, News
- **I built admin myself using Claude** (no other tools, no developer)

## what we just designed in the previous chat

Public homepage v11 — full HTML/CSS/JS file. Open `bre-home-v11.html` to see.

### v11 sections (top to bottom)
1. **Fixed top bar** with mix-blend-difference text (inverts on every background) — left: stays/guide/own with us/invest, center: `bre.` logo with orange dot, right: live Batumi clock
2. **Hero**: 3-panel grid (big main photo + 2 side cards), with `find a flat, fall for the city` headline. Right top side card is styled as **"issue №042" magazine cover** (this is the guide entry point in hero). Right bottom is a guide post (Bolt Taxi).
3. **Ask form** below hero: simple dropdown with apartment choices + "ask in telegram" button. Submits to `t.me/bre_ge` with prefilled message.
4. **Ticker** with district names (boulevard · old town · chavchavadze · gonio cape · etc.) as marquee.
5. **Apartments** section: 6 cards in 3 columns on desktop, 1 column on mobile. Each card has photo + name + price + rating + "X spots nearby in the guide" link (this is what links to map of guide places near this apt — see geo-link plan below).
6. **Full-bleed editorial spread**: full-width photo, giant headline "the batumi we *actually* recommend", "issue №042" stamp, button "open the guide".
7. **Three guide categories preview**: Eat & Drink / Move Around / Wellness — each with one featured post + 2 small posts.
8. **"Right now in batumi" strip**: dark band with date/temp/sunset/latest guide post.
9. **Pull quote**: "we do the *flat*. you do the *city*." with orange underline accent.
10. **Doors block**: 2 cards — "hand us the keys" (for owners, 30% commission) and "buy a flat that pays" (for investors, 11% yield).
11. **Contact section**: huge "say hi." + channel rows (telegram, whatsapp, phone, channel, office).
12. **Footer guide strip**: dark band with "last week we wrote ↓" and 3 latest posts.
13. **Bottom footer**: copyright + back to top.
14. **Floating logo `bre.`** bottom-left, also with mix-blend-difference.
15. **Custom cursor** (desktop only): black dot, grows to colored circle on hover with text label:
    - **Orange #FF5E1A** on regular links/buttons
    - **Teal #00B5B0** on apartment cards, label "ask →"
    - **Yellow #FFD60A** on guide posts/cards, label "read →"
    - **Pink #E8797C** on contact rows, label "say hi →"

## brand & design system

### colors
- **Public site accent (primary):** `#FF5E1A` — electric orange. Used for accents, hovers, prices, italic emphasis words, the logo dot
- **Public site bg:** `#FBF8F1` — warm near-white cream
- **Public site bg-2:** `#F1ECDF` — slightly deeper cream for cards
- **Ink (text):** `#0A0908` — almost black with brown undertone
- **Ink-soft:** `#4D544E` — muted gray-green for secondary text
- **Cursor secondary colors:** teal `#00B5B0`, yellow `#FFD60A`, pink `#E8797C`
- **Admin keeps its color:** dusty pink `#E8797C` (do not change admin)

### typography
- **Bricolage Grotesque** (Google Fonts) — variable font, weights 200-900, opsz 10-48, wdth 75-100. Used for headlines, body, prices.
- **JetBrains Mono** (Google Fonts) — for labels, dates, metadata, button text, kicker text
- **Everything lowercase** — no Title Case, no ALL CAPS in body
- Italic accent words colored in orange (e.g. "fall *for* the city")

### voice / tone
- Lowercase, casual, confident, slight insider attitude
- "we do the flat. you do the city." is the brand line
- Stats used as proof: "30% commission", "11% yield", "120+ flats managed", "$1.5M transaction volume"
- "we don't scrape from tripadvisor" — guide is written by us
- No marketing fluff, no "transform your stay", no "unforgettable experience"

## architecture: how public site connects to admin

### current state
- `bre.ge` → Tilda (will be replaced)
- `bre.ge/finance` → my custom PMS on GitHub Pages + Supabase
- Both should eventually share one Supabase database

### target architecture
- **Public site (new v11)** lives in a new GitHub repo, deployed via **Vercel** (same way grinbergart.com is)
- **DNS migration plan:**
  - During development: test on `new.bre.ge` subdomain
  - When ready: switch `bre.ge` A-record from Tilda to Vercel
  - Old Tilda guide stays accessible at `guide.bre.ge` subdomain temporarily
- **Both public site and admin share one Supabase database** with shared tables:
  - `objects` — apartments (id, title, slug, photos, price, lat, lng, etc.)
  - `bookings` — reservations (id, object_id, dates, guest_info, status: pending/confirmed)
  - `guide_posts` — articles (id, title, slug, body, category_id, photo_url, lat, lng, status, published_at)
  - `categories` — guide categories
  - `post_apartments` — junction table linking posts to nearby/featured apartments
  - `owners` — property owners

### inquiry flow
- User clicks apartment on website → opens Telegram chat with prefilled question
- Later (next phase): website form writes `pending` row directly to `bookings` table → admin gets notification → I confirm

## the geo-link feature (last topic of previous chat)

This is the headline feature for guide ↔ apartments integration.

### concept
Each apartment has `lat, lng`. Each guide post has `lat, lng`. Postgres function `distance(a, b)` computes haversine distance in km.

### on the website
- **Apartment card** shows "↗ 5 spots nearby in the guide" → clicks lead to `/guide/near?apt=<apt_id>` — a page with a Mapbox/Leaflet map centered on apartment, pins for nearby guide places, post cards below
- **Guide post page** shows "stay nearby" block at the bottom — 2-3 apartments closest to that post
- **Guide landing page `/guide`** — big Mapbox map of all Batumi with all posts as pins, color-coded by category, filter buttons on top

### in the admin (new "guide editor" tab in bre.ge/finance)
When creating/editing a guide post:
1. Standard fields: title, body, category, photo
2. **Map picker** — drag a pin on Batumi map OR paste Google Maps URL → fills lat/lng
3. **Apartments nearby panel** — auto-sorts by distance to the pin, shows nearest first with distance label ("0.3 km" highlighted, ">1km" muted)
4. Editor checks which apartments to link to this post (saved in `post_apartments`)
5. Editorial decision: distance is suggestion, but editor decides which to show on public site

### AI-powered guide editor (next phase)
Long-term vision discussed in previous chat:
- Three input modes: screenshot from Instagram, voice dictation, manual form
- AI (using Claude API via existing voice assistant key) reads screenshot → recognizes place → generates post in bre voice → suggests category → user reviews & publishes
- Goal: turn "I saw a cool place on someone's Insta story" into a published guide post in 30 seconds

## map technical choice

- **Mapbox** preferred over Leaflet for the look (more like Airbnb)
- Free tier: 50,000 map loads/month — plenty for bre.ge
- Need: register on mapbox.com, get access token, store in env var
- Library: `mapbox-gl` from CDN
- Custom marker styling possible — orange pins for apartments, smaller colored dots for guide posts by category

## what's done vs what's pending

### done
- Full v11 homepage HTML/CSS/JS (`bre-home-v11.html`)
- Mobile preview wrapper (`bre-mobile-preview.html`)
- Brand system: colors, typography, voice, components
- Architecture diagrams (in chat — not exported)
- Admin PMS at bre.ge/finance — already shipping

### pending
1. **Stays page** `/stays.html` — full list of apartments (currently homepage shows only 6 of 26)
2. **Services page** `/services.html` — three blocks: manage (homestaging + rental mgmt), buy (acquisition help), renovate (turnkey)
3. **Guide page** `/guide.html` — initially just redirect to Tilda, eventually a Mapbox-powered map of all posts
4. **Apartment detail page** `/stays/<slug>.html` — gallery, info, "nearby in guide" map block
5. **Guide post page** `/guide/<slug>.html` — article, "stay nearby" block
6. **Supabase schema** for `objects`, `guide_posts`, `categories`, `post_apartments`, distance function
7. **Connect public site to Supabase** — fetch apartments, fetch posts, render dynamically
8. **Guide editor tab in admin** — map picker + apartment checker
9. **AI screenshot → post pipeline** in admin
10. **Migrate ~100 posts from Tilda Потоки to Supabase**
11. **Vercel deploy** of public site, DNS switch from Tilda
12. **Channel manager / iCal sync** with Airbnb for live availability (later phase)

## the v11 file — what to know

- Single HTML file, ~1000 lines including embedded CSS and JS
- Uses Google Fonts (Bricolage Grotesque + JetBrains Mono) loaded via `<link>`
- All apartment photos are Unsplash placeholders — need real Airbnb photos
- All guide posts in v11 are fabricated (Nova Coffee, Bolt Taxi, Pure Bistro, etc.) — need to replace with real bre.ge/guide posts
- Booking is currently a Telegram link (not a real booking system) — intentional, will be replaced when admin integration ready
- Real apartment IDs from Airbnb to use: 1460785484891517329, 1420129794643526163, 1179731395304757203, 1360921879373680229, 1417291548664785336

## tone for working with me

- I prefer **directness over politeness** — if I'm wrong, say so
- I work in Russian-English mix in chat — answer in the language I'm using
- I iterate fast — ship v9, v10, v11 of files rather than overthinking v1
- I trust modern web tech — Vercel, Supabase, GitHub, modern CSS, custom cursors
- I'm not afraid of code — I built bre.ge/finance myself, just talking to Claude
- Keep things lowercase, simple, no fluff in copy
- Use plain Russian for technical context, English for code/UI strings
