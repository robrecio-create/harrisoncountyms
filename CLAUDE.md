# Harrison County MS — Claude Code Context

## Project Overview
Business directory website for Harrison County, Mississippi (harrisoncountyms.com).
Lists local businesses, events, deals/coupons across six communities on the Gulf Coast.

## Tech Stack
- **Framework:** Astro (static site generator)
- **Deployment:** Vercel (auto-deploys on git push to GitHub)
- **Email:** Resend API (`RESEND_API_KEY` env var on Vercel)
- **Repo:** github.com/robrecio-create/harrisoncountyms
- **Node:** v24.14.0 via NVM

## Towns
Gulfport, Biloxi, Long Beach, D'Iberville, Pass Christian, Saucier

## Key Data Files
- `src/data/businesses.json` — All business listings (source of truth for counts)
- `src/data/events.json` — Community events
- `src/data/constants.ts` — Town slugs, category slugs, nav links (NOTE: town `businesses` and category `count` fields here are STALE/legacy — pages compute counts dynamically from businesses.json)

## Dynamic Counts Pattern
All business counts are computed at build time from `businesses.json`, NOT from hardcoded values in constants.ts. This pattern is used across multiple pages:

### Per-town counts
```js
const townsWithCounts = towns.map(town => ({
  ...town,
  businesses: businesses.filter(b => b.location?.includes(town.name)).length,
}));
```
Used in: `index.astro`, `locations/index.astro`

### Per-category counts
```js
const categoriesWithCounts = categories.map(cat => ({
  ...cat,
  count: businesses.filter(b => b.categories?.includes(cat.name)).length,
}));
```
Used in: `categories/index.astro`

### Total + coupons counts
```js
const businesses = businessData as any[];
const deals = businesses.filter(b => b.deal_title && b.deal_title.trim());
// Then in template: {businesses.length} Businesses, {deals.length} Coupons
```
Used in: `index.astro` hero stats section

**IMPORTANT:** Never hardcode business/category/coupon counts. Always compute from businesses.json so they auto-update when listings change.

## Key Pages
- `src/pages/index.astro` — Homepage with hero, stats, deals, events, towns
- `src/pages/locations/[slug].astro` — Individual town pages (already fully dynamic)
- `src/pages/locations/index.astro` — All towns overview
- `src/pages/categories/[slug].astro` — Individual category pages (already fully dynamic)
- `src/pages/categories/index.astro` — All categories overview
- `src/pages/submit-event/index.astro` — Event submission form
- `src/pages/api/contact.ts` — Email endpoint via Resend

## Business JSON Key Fields
- `title` — Business name
- `slug` — URL slug
- `location` — City name (used for town filtering)
- `categories` — Array of category names (used for category filtering)
- `deal_title` — If non-empty, business has an active coupon/deal
- `featured` — Boolean, featured listings sort first

## Development
```bash
export PATH="/Users/robrecio/.nvm/versions/node/v24.14.0/bin:/usr/bin:/bin:/usr/sbin:/sbin:$PATH"

npm run dev     # Local dev server on port 4321
npm run build   # Production build
git push        # Triggers Vercel auto-deploy
```

## Do Not
- Hardcode business, category, town, or coupon counts — always compute from businesses.json
- Commit API keys or secrets — use Vercel env vars
- Worry about sharp/Icon ENOENT errors on local build — Vercel builds fine
