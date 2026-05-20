# Decision: Liquid Theme over Hydrogen

**Date:** 2026-04-18
**Project:** Bodyline

## Decision
Stay with Shopify Liquid theme instead of pivoting to Hydrogen (React).

## Why
- Ron needs the drag-and-drop theme editor for day-to-day updates (sales banners, featured products, store hours)
- Hydrogen is headless — every change requires code edits and redeployment
- Ron is not a "log in and tweak the theme" power user, but he may want to make simple changes without calling Greg
- Liquid theme with Tailwind CSS still achieves high design fidelity

## What Was Considered
- Hydrogen would have allowed near-direct port of Figma Make React code
- Hydrogen uses Shopify Storefront API + Remix
- Would have been faster initial build but worse long-term maintainability for a non-technical owner

## Impact
- Design must be matched via CSS/Liquid rewrite, not React port
- Theme editor sections and blocks must be configured for merchandiser use
- All Shopify's built-in features (cart, checkout, POS) work natively
