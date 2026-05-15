# Changelog

All notable changes to bre-public.

## [v0.1.0] — 2026-05-15

### Added
- Initial repo structure
- Home page (`/`) with hero, edition cover, apartments grid, guide preview, doors, contact section
- Guide landing page (`/guide`) with 8-world tiles, masthead cover story, long reads carousel, news block, map preview
- Eat & drink category page (`/guide/eat-drink`) — first category template
- Burger menu for mobile on all three pages
- Mobile audit preview at `/_drafts/mobile-audit`
- Hero photo picker at `/_drafts/hero-picker`
- Vercel config (`vercel.json`) with clean URLs
- 404 page in brand style
- README with stack, conventions, and roadmap
- Docs: context, geo-link spec, next-steps roadmap

### Notes
- Apartment count is hardcoded in JS (`CAT_COUNTS` in each guide page) — to be replaced with Supabase query
- All Unsplash photos are direct URLs — to be migrated to local `/assets/img` when finalised
- Telegram inquiry form removed from hero (will return later if needed)
- No build step. Pure HTML/CSS/JS, opens in any browser.
