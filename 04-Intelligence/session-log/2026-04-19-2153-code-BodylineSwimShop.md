---
date: 2026-04-19
time: 21:53
tool: code
project: BodylineSwimShop
duration_minutes: 90
---

## Summary
Rebuilt the shop-page layout so the category sidebar renders identically on `/pages/shop` and every `/collections/*` page via a shared snippet. Refactored `product-card.liquid` into a single source of truth consumed by both sections — restoring SALE badges, sold-out pills, and strikethrough compare-at prices using brand tokens. Committed Liquid-style + drag-and-drop conventions to project memory.

## Files touched
- `C:\Users\Greg\Documents\Claude\Projects\BodylineSwimShop\sections\shop-collections.liquid` (modified — replaced inline card with `{% render 'product-card' %}`, added `{% render 'shop-sidebar' %}`)
- `C:\Users\Greg\Documents\Claude\Projects\BodylineSwimShop\sections\collection-grid.liquid` (modified — full rewrite to match shop-collections shell; added shared sidebar render; brand-token breadcrumb; replaced inline card)
- `C:\Users\Greg\Documents\Claude\Projects\BodylineSwimShop\snippets\shop-sidebar.liquid` (created — shared sidebar with 11 hardcoded category links in 3 groups; derives active state from `collection.handle` / `page.handle`)
- `C:\Users\Greg\Documents\Claude\Projects\BodylineSwimShop\snippets\product-card.liquid` (modified — rewritten with brand tokens, aspect-square `object-cover`, `rounded-2xl`, SALE pill `bg-red-600`, sold-out pill `bg-ink/85`, strikethrough compare-at price)
- `C:\Users\Greg\Documents\Claude\Projects\BodylineSwimShop\assets\theme.css` (modified — rebuilt twice via `npm run build`, final 45,481 bytes minified)
- `C:\Users\Greg\.claude\projects\C--Users-Greg-Documents-Claude-Projects-BodylineSwimShop\memory\theme_conventions.md` (created — Liquid style rules, drag-and-drop principle, sidebar refactor path, CSS build pipeline)
- `C:\Users\Greg\.claude\projects\C--Users-Greg-Documents-Claude-Projects-BodylineSwimShop\memory\MEMORY.md` (modified — added `theme_conventions.md` bullet)

## Decisions made
- Shared sidebar as a **snippet** rather than a **section**: single source of truth, but deliberately NOT merchant-rearrangeable in the theme editor. Nav should not be drag-and-droppable.
- Product-card refactor via Option 3 (rewrite snippet with brand tokens; both sections render it) over inlining SALE/sold-out logic separately in each section. Prevents drift, single source of truth for card markup.
- Reverted `sm:` breakpoint classes back to `md:` after the search-bar experiment broke the layout. Root cause was pre-built Tailwind — new classes aren't in `assets/theme.css` until rebuilt.
- Abandoned `/prompt-check` audit of the Phase 2 Second Brain plan from Cowork early (user said "ignore"). Session scope stayed on theme work only.
- Sidebar categories kept hardcoded for now. Refactor path to `linklists['shop-sidebar'].links` documented in `theme_conventions.md` for when merchant-editable categories are wanted (~30 min effort).

## Open questions
- `snippets/price.liquid` is still used by `sections/product-main.liquid` (PDP), but `product-card.liquid` no longer renders it — minor drift risk if PDP pricing logic evolves separately from card.

## Next steps
- Sidebar → `linklists['shop-sidebar'].links` refactor when admin-editable categories are desired.

## Memory promotion candidates
- None outstanding. Tailwind build-pipeline lesson committed to `theme_conventions.md` at session end (approved via Staleness Prevention proposal).
