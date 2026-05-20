# SOP: Website Build (Shopify)

## Process

### Phase 1 — Prototype
1. Scrape existing site for product data, images, content
2. Build prototype in Figma Make (React/Tailwind)
3. Get owner approval on the prototype
4. Share via Figma Make Publish (.figma.site URL)

### Phase 2 — Platform Setup
1. Create Shopify account (free trial, no CC needed upfront)
2. Scaffold custom Liquid theme locally (Tailwind CSS)
3. Install Shopify CLI: `npm install -g @shopify/cli @shopify/theme`
4. Use `shopify theme dev` for live preview during development
5. Use `shopify theme push` to upload to store

### Phase 3 — Data
1. Build product import CSV (Shopify format)
2. Import via Products → Import in admin
3. Create automated collections (tag-based)
4. Verify collections populate correctly

### Phase 4 — Go Live
1. Push final theme
2. Owner switches DNS in GoDaddy (nameservers → Shopify)
3. Set up Shopify Payments
4. Configure Shopify POS for in-store
5. Cancel old platform (Squarespace, etc.)

## Key Lessons
- Prototype first, build second — gets buy-in before heavy investment
- Liquid theme > Hydrogen if the owner needs theme editor access
- CSV tags must match collection names exactly (case-sensitive)
- `shopify theme push` targets whatever theme you select — make sure it's the published one
- Development themes can't have pages assigned to their templates — must publish first
