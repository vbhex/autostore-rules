# AliExpress Store Order Diagnosis

**Date**: 2026-03-19
**Scope**: Live seller-center inspection on Main Computer + codebase fixes in `aliexpress/`

## Executive Summary

The store is not truly "no orders" historically, but the visible orders are old **3C / electronics** orders.
The current **clothing** strategy has two main problems:

1. **Category saturation**: many products are being rejected because AliExpress says there are already too many similar products and recommends opportunity categories instead.
1. **Listing quality gaps on online clothing products**: sampled live listings still have empty attributes and weak description content.

A third structural issue remains:

1. **Catalog size is too small**: only `182` products are currently online, which is too thin for a store-level conversion strategy.

## Live Seller-Center Findings

### Product counts

- `Online`: `182`
- `Draft`: `2`
- `Pending approval`: `0`
- `Review failed`: `134`
- `Offline`: `52`

### Review-failed root cause

The review-failed page shows repeated warnings of this form:

> The products you have published already have many similar products on the platform. It is not recommended to publish them for the time being. You are recommend to publish the opportunity category...

This confirms that **red-ocean / over-saturated categories are a real current blocker**.

### Order page findings

The order page still contains multiple historical orders, but they are **legacy electronics products** such as:

- power banks
- wireless charging power banks
- cameras

This means the current clothing assortment is not the source of those past orders, and the present clothing catalog still has not proven demand.

### Sampled live clothing listing findings

One sampled live clothing product (`Pant Sets`) showed:

- swatch / variant images present
- many attribute fields still empty (`Select` / `Please Input or select option` placeholders)
- PC description present but effectively **text-only**
- no clear evidence of description images being uploaded
- no `Product Instruction Manual` detected in the page snapshot

So the listing-quality problem is real, but it is **not only swatches**. The larger gaps are:

- missing attributes
- weak description media
- missing manuals / compliance-supporting content

## Code Fixes Completed

The following fixes were completed locally in `aliexpress/`:

- `src/excel/attribute-mapper.ts`
  - added safe optional defaults for active clothing sheets so exported listings are less bare
- `src/utils/generate-manuals.ts`
  - added clothing manual generation for active clothing categories
  - regenerated local `manuals/*.pdf`
- `src/tasks/polish-products.ts`
  - added clothing category detection
  - changed attribute verification from electronics-only labels to generic select-field checks
  - expanded prioritized attribute filling to include clothing fields

TypeScript build passes after these changes.

## Remaining Live Blocker

The current mass-polish automation still cannot repair all live listings yet because the seller-center product-management page now renders inconsistently for fresh automation sessions:

- the page shows the correct top-level counts
- but the product table often fails to expose row data to Puppeteer immediately
- `polish-products.js` therefore detects `0` products even while the UI shows `182`

This is now an **automation selector/rendering issue**, not a strategic diagnosis issue.

## Conclusions

### 1. Category choice

Your instinct is correct: **some current clothing categories are too red-ocean**.
This is confirmed by `Review failed (134)` and the live AliExpress warning about too many similar products.

### 2. Listing polish

Your instinct is also correct here.
Live clothing listings still need stronger polish, especially:

- attributes
- description images / richer description content
- manuals

### 3. Product count

Also correct.
`182` online products is too low. The store needs to move above `300` online products, but only after pruning saturated categories and improving listing quality.

## Recommended Next Steps

1. Replace the worst saturated clothing sheets first, instead of scaling them.
2. Fix the `polish-products.js` product-list detection so live repairs can run in bulk.
3. Re-polish current online clothing listings after that script fix.
4. Scale to `300+` online products only in validated lower-competition clothing categories.
