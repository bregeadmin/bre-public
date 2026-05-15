# Geo-link: apartments ↔ guide posts

This is the headline feature for linking the public site's apartments with the editorial guide.

## the core idea

Every **apartment** has `lat, lng`.
Every **guide post** has `lat, lng`.
Postgres function `distance(a, b)` computes haversine distance in km between two coordinates.
The site uses this to show nearby items both ways.

## what the user sees on the website

### on the homepage (already in v11)
Each apartment card shows a small link at the bottom:
```
↗ 5 spots nearby in the guide
```
Click → opens a page with a map.

### on the new `/guide/near?apt=<apt_id>` page
- Big interactive map (Mapbox) centered on the apartment
- Large orange pin = the apartment
- 5-10 smaller pins around = nearby guide places
- Below the map: cards of those places with photos, names, distance ("280m"), category badges
- Filter buttons on top: "all / eat & drink / wellness / move around"

### on each guide post page (future)
At the bottom: "stay nearby" block with 2-3 apartments closest to that post. Editor decides which apartments to feature (not pure auto by distance — see below).

### on the guide landing page `/guide` (future)
- One big map of Batumi with all guide posts as pins
- Color-coded pins by category
- Filter and search

## the data model

### apartments table (`objects`)
```sql
create table objects (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  slug text unique not null,
  description text,
  photos jsonb default '[]',
  price_per_night numeric,
  bedrooms int,
  area_m2 int,
  airbnb_id text,
  lat numeric not null,
  lng numeric not null,
  active boolean default true,
  created_at timestamptz default now()
);
```

### guide posts table (`guide_posts`)
```sql
create table guide_posts (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  slug text unique not null,
  body text not null,
  category_id uuid references categories(id),
  photo_url text,
  lat numeric,
  lng numeric,
  google_maps_url text,
  status text default 'draft', -- draft, published
  published_at timestamptz,
  created_at timestamptz default now()
);
```

### categories
```sql
create table categories (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  slug text unique not null,
  color text -- hex color for pin
);

-- seed
insert into categories (name, slug, color) values
  ('eat & drink', 'eat-drink', '#FF5E1A'),
  ('wellness', 'wellness', '#00B5B0'),
  ('move around', 'move-around', '#FFD60A'),
  ('stay & live', 'stay-live', '#E8797C'),
  ('nightlife', 'nightlife', '#7F77DD'),
  ('work & remote', 'work-remote', '#5DCAA5'),
  ('experiences', 'experiences', '#EF9F27'),
  ('explore batumi', 'explore', '#F0997B'),
  ('news', 'news', '#F09595');
```

### junction table (editor's manual links between posts and apartments)
```sql
create table post_apartments (
  post_id uuid references guide_posts(id) on delete cascade,
  apartment_id uuid references objects(id) on delete cascade,
  primary key (post_id, apartment_id)
);
```

### the distance function
```sql
create or replace function distance_km(lat1 numeric, lng1 numeric, lat2 numeric, lng2 numeric)
returns numeric
language sql immutable
as $$
  select 2 * 6371 * asin(
    sqrt(
      sin(radians((lat2 - lat1) / 2))^2 +
      cos(radians(lat1)) * cos(radians(lat2)) *
      sin(radians((lng2 - lng1) / 2))^2
    )
  );
$$;
```

### queries

Nearby posts for an apartment:
```sql
select
  p.*,
  c.name as category_name,
  c.color as category_color,
  distance_km(o.lat, o.lng, p.lat, p.lng) as distance_km
from guide_posts p
join categories c on c.id = p.category_id
join objects o on o.id = $1
where p.status = 'published'
  and p.lat is not null
order by distance_km asc
limit 5;
```

Nearby apartments for a post (mix of editor-selected + auto by distance):
```sql
-- explicitly linked apartments
select o.*, distance_km(p.lat, p.lng, o.lat, o.lng) as distance_km
from objects o
join post_apartments pa on pa.apartment_id = o.id
join guide_posts p on p.id = pa.post_id
where p.id = $1 and o.active = true;

-- or pure auto-by-distance if no explicit links
select o.*, distance_km(p.lat, p.lng, o.lat, o.lng) as distance_km
from objects o
join guide_posts p on p.id = $1
where o.active = true and p.lat is not null
order by distance_km asc
limit 3;
```

## the guide editor in admin (new tab in bre.ge/finance)

When creating or editing a post, the editor sees:

### standard fields
- Title (text)
- Body (textarea)
- Category (dropdown)
- Photo (upload to Supabase Storage)

### NEW: map picker (left half)
- Embedded Mapbox map of Batumi
- Draggable pin
- OR paste Google Maps URL → backend parses coordinates and drops the pin
- Shows current lat/lng readout

### NEW: apartments nearby (right half)
- As soon as pin is set, list auto-updates
- Sorted by distance to pin
- Nearest highlighted in orange tint ("0.3 km")
- Further ones muted ("1.4 km")
- Checkbox next to each — checked apartments will be linked in `post_apartments`
- Help text: "checked apartments will show 'stay nearby' on this post page"

### the principle
Distance is a **suggestion**, editor decides. Some apartments may be 0.3km away on the map but separated by a busy road / not in walking culture. Editorial control beats pure automation.

## the three input modes for adding a place (long-term)

1. **Screenshot from Instagram** — drag/drop an image into admin
   - Claude API (vision) reads the screenshot
   - Recognizes place name, location, type
   - Generates a post in bre's voice
   - Drops the pin if location is recognizable
   - Suggests category
   - Editor reviews and publishes

2. **Voice dictation** — using the existing voice assistant in admin
   - "new post: pure bistro, oyster weekend, 8-10 may, wine included, address 47 chavchavadze"
   - AI parses, fills fields, drops pin via geocoding

3. **Manual** — just fill the form

All three write to `guide_posts` with same schema.

## map technical stack

- **Mapbox GL JS** loaded from CDN
- Free tier: 50k map loads/month (enough)
- Custom marker styling:
  - Apartments: big orange (#FF5E1A) circles with `bre.` mini-logo
  - Guide posts: smaller colored dots, color = category
- Map style: Mapbox `streets-v12` or custom minimalist style
- Center: Batumi `[41.6168, 41.6367]`
- Default zoom: 13

## phased rollout

### phase 1: schema + manual data (1-2 days)
- Apply the SQL above in Supabase
- Hand-fill `lat, lng` for existing 10 apartments
- Pick 5 real guide posts from Tilda, manually add to `guide_posts` with coordinates
- Test the distance query

### phase 2: the website map page (2-3 days)
- Add `/guide/near.html` page
- Mapbox map, Supabase fetch, render pins
- Update v11 apartment cards: replace static `5 spots nearby` with real link

### phase 3: admin guide editor (3-5 days)
- New tab `guide` in bre.ge/finance
- Form with Mapbox picker
- Nearby apartments panel with checkboxes
- Save to Supabase

### phase 4: AI input modes (1-2 weeks)
- Screenshot → vision API → structured post
- Voice → existing assistant + form-fill
- Migration: bulk-import 100+ posts from Tilda

### phase 5: full /guide landing (1 week)
- Big map of all Batumi with all posts
- Category filters
- Search

## mapbox setup checklist

1. Register at mapbox.com (free)
2. Settings → Access Tokens → copy default public token
3. Store in `.env` (or for static HTML: as a JS variable, but limit by URL referrer in Mapbox dashboard)
4. In HTML: `<script src="https://api.mapbox.com/mapbox-gl-js/v3.0.0/mapbox-gl.js"></script>`
   `<link href="https://api.mapbox.com/mapbox-gl-js/v3.0.0/mapbox-gl.css" rel="stylesheet">`
5. Init: `mapboxgl.accessToken = '<token>'; const map = new mapboxgl.Map({ container, style, center, zoom });`
