# next steps — priority order

## content model — decisions locked in

The guide has three content types. **Use these exact names in DB, admin, and UI** — no aliases.

| `type` (DB enum)  | UI label  | Has dates?      | Examples                                            |
|-------------------|-----------|-----------------|-----------------------------------------------------|
| `place`           | place     | no              | a café, a restaurant, a beach, a shop               |
| `story`           | story     | no              | longread, photo essay, interview, "mr. x recommends"|
| `news`            | news      | optional        | pop-up, festival, "new menu at X", "kava lova reopened" |

**Why `news` (not `event`):**
- `event` implies a hard required start/end date — too rigid.
- `news` accepts both dated items (pop-ups, festivals) *and* dateless items (a place opened, a menu changed). One mental model, broader scope.
- Same name in code = same name in UI = no mapping table to maintain.

```sql
CREATE TYPE guide_post_type AS ENUM ('place', 'story', 'news');

ALTER TABLE guide_posts
  ADD COLUMN type guide_post_type NOT NULL DEFAULT 'place',
  ADD COLUMN start_date DATE,           -- only news-type posts use these
  ADD COLUMN end_date DATE,             -- both nullable
  ADD COLUMN is_dated BOOLEAN GENERATED ALWAYS AS (start_date IS NOT NULL) STORED;

-- auto-hide expired news from public lists:
CREATE INDEX idx_news_active ON guide_posts (type, end_date)
  WHERE type = 'news' AND end_date IS NOT NULL;
```

Frontend filter for "active news":
```sql
SELECT * FROM guide_posts
WHERE type = 'news'
  AND (end_date IS NULL OR end_date >= CURRENT_DATE)
ORDER BY COALESCE(start_date, created_at) DESC;
```

## immediate (this week or next)

### 1. ship v11 as-is to a staging URL
- Create new GitHub repo `bre-public`
- Push `bre-home-v11.html` as `index.html`
- Connect repo to Vercel
- Set up `new.bre.ge` subdomain pointing to Vercel
- This is **1-2 hours of work** — nothing fancy, just deploy what we have

### 2. fill in the missing pages (static first)
Build these as separate HTML files, same design system as v11:
- **`stays.html`** — full list of all apartments with filters
- **`services.html`** — three blocks: manage / buy / renovate / homestaging
- **`guide.html`** — for now just visually styled landing that links to current Tilda `guide.bre.ge`

These can still be static HTML — no database needed yet. Just match v11's style.

## next 2-3 weeks

### 3. wire up Supabase for apartments
- Create `objects` table with the schema in `01_GEO_LINK_FEATURE.md`
- Add `lat, lng` columns
- Fill 10 real apartments manually (extract from Airbnb listings)
- Make v11 fetch this data via Supabase JS client
- Replace hardcoded apartment cards with dynamic ones

### 4. apartment detail pages
- Route: `/stays/<slug>.html` or `/stays.html?slug=...`
- Big photo gallery (from Airbnb-style images)
- Description, price, info
- "Ask in telegram about this apartment" button → opens t.me/bre_ge with prefilled message
- Mini-map showing apartment location + nearby places (geo-link!)

## next month

### 5. geo-link feature (full)
See `01_GEO_LINK_FEATURE.md` for details.
- Add `guide_posts`, `categories`, `post_apartments` tables
- Build `/guide/near?apt=<id>` page with Mapbox map
- Update apartment cards: "5 spots nearby" becomes a real link

### 6. guide editor in admin
Add new tab in `bre.ge/finance`:
- Form with Mapbox pin-picker
- Auto-list apartments nearby with checkboxes
- Save post to `guide_posts`

### 7. migrate guide posts from Tilda
- 100+ posts, 8 categories
- Could be done by bulk SQL insert
- Or manually 5-10 a day while the new admin form is being polished

## later (next 2-3 months)

### 8. AI-powered guide editor
- Screenshot → Claude vision → structured post draft
- Voice dictation → same
- 30-second flow from "I saw this on Insta" to "published post"

### 9. full `/guide` landing
- Big Batumi map with all posts as colored pins
- Category filters, search
- Editorial layout for featured posts

### 10. real bookings (replacing the Telegram link)
- Booking form on apartment page → writes to `bookings` table in Supabase
- Admin gets notification
- Channel manager integration with Airbnb (Hospitable / Smoobu / direct iCal sync)

### 11. DNS cutover
- Switch `bre.ge` from Tilda to Vercel
- Tilda guide moves to `guide.bre.ge` temporarily
- After full migration: shut down Tilda

## things to NOT do yet

- **Don't build real-time availability / booking calendar** — Telegram-link to inquiry is fine for now, until admin is ready
- **Don't migrate the admin to a different stack** — it works, leave it
- **Don't try to make Tilda and Vercel coexist perfectly** — the moment new site is good enough, switch DNS, move on
- **Don't over-engineer the database** — start with simple tables, add indexes/columns when needed
- **Don't agonize over mobile micro-details** — v11 mobile is "good enough" for launch, polish later

## what makes "good enough to launch"

- v11 homepage works on desktop and reads on mobile
- All 10 real apartments visible somewhere (even hardcoded)
- Three service pages exist with the homestaging / purchase / renovation copy from Tilda
- Guide accessible (even via subdomain redirect to old Tilda)
- Telegram inquiry flow works
- Domain `bre.ge` serves the new site (DNS switched)

That's it. Geo-link, AI editor, real bookings — all can come later.
