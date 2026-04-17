# Clothing & Apparel — Category Strategy

**Effective**: 2026-03-20 (Red Ocean pivot — Women's/Men's Clothing L1 fully retired)
**Previous version**: 2026-03-10 (opportunity category pivot — now superseded)
**Replaces**: 3C / Consumer Electronics (retired — see `PLATFORM_PIVOT_3C_TO_CLOTHING.md`)
**Platform**: AliExpress (only active platform)

> ⛔ **RED OCEAN RULING (2026-03-20)**: ALL sub-categories under Women's Clothing and Men's Clothing L1 are permanently Red Ocean for AliExpress store 2087779. Do NOT scrape any of them. New targets are Watches, Apparel Accessories, Jewelry, Luggage & Bags, and other non-clothing L1 categories. See `aliexpress-2087779-blue-ocean-categories.md` for the current approved list.

---

## Why We Pivoted (2026-03-10)

Most products uploaded in the first batch were rejected with:

> "The products you have published already have many similar products on the platform. It is not recommended to publish them for the time being. You are recommended to publish the opportunity category."

**Root cause**: Our original 12 categories all routed to 3 AliExpress template sheets:
- `TShirts` — tens of millions of sellers
- `HoodiesSweatshirts` — hundreds of thousands of sellers
- `CasualPants` — heavily saturated

AliExpress blocks new stores from entering already-crowded sheets. Even niche 1688 search terms (印花, oversize, Y2K) did not help, because all products still land in the *same* AE template sheet.

**Solution**: Target entirely different template sheets where the seller population is much lower.

---

## ~~Active Categories~~ — RETIRED (all Women's/Men's Clothing = Red Ocean)

The 15 categories below are permanently retired as of 2026-03-20. They are listed here for historical reference only — do not re-activate.

## Retired Categories (15 total — Red Ocean as of 2026-03-20)

### Women's — Opportunity (8)

| CLI Category | 1688 Search Term | AliExpress Sheet | Why Less Saturated |
|---|---|---|---|
| `womens skirts` | 女士半身裙 | `Skirts` | Specific garment type; sub-types vary |
| `womens jumpsuits` | 女士连体裤 | `Jumpsuits` | Growing trend; distinct product |
| `womens blazers` | 女士西装外套 | `Blazers` | Professional niche; fewer mass-market sellers |
| `womens leggings` | 女士瑜伽裤 | `Leggings` | Active/yoga niche; quality-driven buyers |
| `womens sleepwear` | 女士睡衣套装 | `NightgownsSleepshirts` | Loungewear trend; low competition |
| `womens cardigan` | 女士针织开衫 | `Cardigan` | Seasonal knitwear; fewer year-round sellers |
| `womens dresses` | 连衣裙 | `Dresses` | Broad — sub-types (maxi, midi, wrap) are distinct |
| `womens jackets` | 女士外套 | `Jackets` | Outerwear is unique by design; natural variety |

### Women's — Supporting (1)

| CLI Category | 1688 Search Term | AliExpress Sheet | Notes |
|---|---|---|---|
| `womens sets` | 女士套装 | `PantSets` | Matching sets; acceptable competition level |

### Men's — Opportunity (4)

| CLI Category | 1688 Search Term | AliExpress Sheet | Why Less Saturated |
|---|---|---|---|
| `mens polo` | 男士Polo衫 | `PoloShirts` | Business casual; less generic than T-shirts |
| `mens shorts` | 男士休闲短裤 | `Shorts` | Spring/summer timing; seasonal reduction in year-round sellers |
| `mens suits` | 男士西装套装 | `Suits` | Professional; low volume reseller count |
| `mens cargo` | 男士工装裤 | `CargoPants` | Trending utility niche; specific item type |

### Men's — Supporting (1)

| CLI Category | 1688 Search Term | AliExpress Sheet | Notes |
|---|---|---|---|
| `mens shirts` | 男士花衬衫 | `Shirts` | Patterned/Hawaiian shirts; natural variety |

### Unisex (1)

| CLI Category | 1688 Search Term | AliExpress Sheet | Notes |
|---|---|---|---|
| `denim jackets` | 男女牛仔外套 | `DenimJacket` | Classic specific item; few mass-market competitors |

---

## Removed Categories (Saturated — Superseded by Red Ocean Ruling)

All categories below were already retired, and are now covered by the broader Red Ocean ruling.
No products from any of these sheets should be uploaded or scraped.

| Removed Category | Sheet | Reason |
|---|---|---|
| `womens tops` | `WomensTops` | Millions of sellers — platform blocks new entrants |
| `womens hoodies` | `HoodiesSweatshirts` | Hundreds of thousands of sellers |
| `mens tshirts` | `TShirts` | Millions of sellers |
| `mens hoodies` | `HoodiesSweatshirts` | Same sheet as womens hoodies — saturated |
| `mens pants` | `CasualPants` | Very broad, crowded |
| `streetwear` | `HoodiesSweatshirts` | Same saturated sheet |

---

## Paused Categories

| Category | Reason | How to Re-enable |
|---|---|---|
| `kids clothing` | AliExpress requires Mother & Kids templates (not Clothing & Apparel). Listing under adult sheets triggers violation. | Download Mother & Kids templates from seller center, add sheet mappings to excel-gen. |

---

## AliExpress Opportunity Category Tool

AliExpress provides a built-in "Opportunity Category" dashboard that shows which categories
have high buyer demand but low seller supply. **Check this regularly** (monthly).

**Location**: AliExpress Seller Center → Products → Product Planning → Opportunity Category

This is the authoritative source for which categories to target next. The list above was derived
from manual analysis of the category tree (261 leaf paths captured 2026-03-06), but the AliExpress
tool will give real-time demand data.

---

## Template Coverage

All 15 active categories have confirmed template sheets in `templates/fresh-download/column-defs.json`
(624 sheets total, downloaded 2026-03-08):

| Sheet | Status |
|---|---|
| `Skirts` | ✓ |
| `Jumpsuits` | ✓ |
| `Blazers` | ✓ |
| `Leggings` | ✓ |
| `NightgownsSleepshirts` | ✓ |
| `Cardigan` | ✓ |
| `Dresses` | ✓ |
| `Jackets` | ✓ |
| `PantSets` | ✓ |
| `PoloShirts` | ✓ |
| `Shorts` | ✓ |
| `Suits` | ✓ |
| `CargoPants` | ✓ |
| `Shirts` | ✓ |
| `DenimJacket` | ✓ |

---

## Key Differences vs 3C Pipeline

### Variants
- 3C: typically Color only (or Color × Capacity)
- Clothing: **Size × Color** (potentially 20+ SKU combinations per product)
- Size values from 1688 use Chinese sizing (S/M/L/XL etc.) — map to international sizes

### Images
- Clothing on 1688 has more Chinese text in images than electronics
- Task 3 (image check) + Task 5 (AE enrichment) will reject or reroute more products
- Expect higher skip rates — normal

### Attributes
Required clothing attributes per sheet — configured in `src/excel/attribute-mapper.ts`:
- `Material`, `Style`, `Season`, `Craft of Weaving`, `Pattern Type`
- Sheet-specific: `Waist Type` (skirts), `Pant Style` (CargoPants), `Shirts Type` (Shirts), etc.

### Brand Bans (Clothing-Specific)
See `aliexpress/banned-brands.json` for the full list (100+ brands as of 2026-03-10).
Key additions after IP violations received: Maison Kitsune, mardi mercredi, MIU MIU, Thom Browne, aloyoga.

---

## Pipeline Run Commands

Run on China MacBook after `git pull && ./node_modules/.bin/tsc`:

```bash
# 1688 scrapper (discover + scrape)
cd ~/projects/autostore/1688_scrapper
node dist/tasks/task1-discover.js --category "womens skirts"    --limit 30
node dist/tasks/task1-discover.js --category "womens jumpsuits" --limit 30
node dist/tasks/task1-discover.js --category "womens blazers"   --limit 30
node dist/tasks/task1-discover.js --category "womens leggings"  --limit 30
node dist/tasks/task1-discover.js --category "womens sleepwear" --limit 30
node dist/tasks/task1-discover.js --category "womens cardigan"  --limit 25
node dist/tasks/task1-discover.js --category "womens dresses"   --limit 30
node dist/tasks/task1-discover.js --category "womens jackets"   --limit 25
node dist/tasks/task1-discover.js --category "womens sets"      --limit 25
node dist/tasks/task1-discover.js --category "mens polo"        --limit 30
node dist/tasks/task1-discover.js --category "mens shorts"      --limit 30
node dist/tasks/task1-discover.js --category "mens suits"       --limit 25
node dist/tasks/task1-discover.js --category "mens cargo"       --limit 30
node dist/tasks/task1-discover.js --category "mens shirts"      --limit 25
node dist/tasks/task1-discover.js --category "denim jackets"    --limit 25
node dist/tasks/task2-scrape-details.js --limit 500
node dist/tasks/task3-image-check.js    --limit 500
node dist/tasks/task4-translate.js      --limit 500
```

---

## Update Log

- **2026-03-20**: **RED OCEAN RULING** — All Women's Clothing and Men's Clothing L1 sub-categories retired permanently.
  New strategy: target Watches, Apparel Accessories, Jewelry, Luggage & Bags, Shoes, Underwear, World Apparel.
  See `aliexpress-2087779-blue-ocean-categories.md` for current approved targets.
- **2026-03-10**: Major pivot — replaced 7 saturated categories with 8 opportunity categories.
  Removed: WomensTops (TShirts), WomensHoodies, MensTShirts, MensHoodies, MensPants, Streetwear.
  Root cause: "too many similar products" rejection from AliExpress on saturated sheets.
- **2026-03-05**: Initial category list defined. 12 categories across Women's / Men's / Unisex.
