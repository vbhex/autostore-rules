# Amazon Store ATVPDKIKX0DER — Sourcing Strategy

**Platform**: Amazon
**Store ID**: `ATVPDKIKX0DER`
**Store URL**: https://sellercentral.amazon.com/
**Sourcing method**: Manual seller sourcing (NOT automated 1688 discovery)
**Pipeline status**: **ACTIVE**
**Last updated**: 2026-03-28

---

## Sourcing Strategy: Manual Seller Selection

**Amazon does NOT use the automated 1688 search pipeline (Task 1).**

Unlike AliExpress/eBay/Etsy which use automated 1688 discovery across brand-safe categories,
Amazon products come from **manually identified trusted 1688 sellers**.

### How it works

1. User finds a trusted 1688 seller (store URL or seller ID)
2. Products from that seller are scraped into the pipeline (starting at Task 2)
3. Products go through image check → brand verify → translate → flat file gen → upload
4. This gives full control over supplier quality and brand safety

### Why manual sourcing for Amazon?

- Amazon has **stricter enforcement** — violations lead to faster suspensions
- Amazon customers have **higher quality expectations** — need reliable suppliers
- Manual vetting of sellers ensures consistent quality across all products
- The user can verify supplier reliability, shipping speed, and product quality before listing

---

## Eligible Categories

Amazon can list products from **any brand-safe category** (see `documents/BRAND_SAFE_CATEGORIES.md`).
The specific categories depend on what the manually-sourced sellers carry.

Current brand-safe categories eligible for Amazon:

| # | Category Group | Amazon | Notes |
|---|---|:---:|---|
| 1 | Phone cases / accessories | ✅ | |
| 2 | DIY / craft supplies | ✅ | |
| 3 | Jewelry findings / components | ✅ | |
| 4 | Handmade jewelry (no-brand) | ✅ | |
| 5 | Hair accessories | ✅ | |
| 6 | Shoe accessories | ✅ | |
| 7 | Sewing notions | ✅ | |
| 8 | Stickers / patches / iron-ons | ✅ | |
| 9 | Keychains / bag charms | ✅ | |
| 10 | Candles / home fragrance | ✅ | |
| 11 | Storage / organizers | ✅ | |
| 12 | Pet accessories | ✅ | |
| 13 | Fitness small accessories | ✅ | |
| 14 | Painting / art supplies | ✅ | |
| 15 | Custom / personalized gifts | ✅ | Strong Amazon fit |
| 16 | Eyewear accessories | ✅ | |
| 17 | Watch accessories | ✅ | |
| 18 | Scarves accessories | ✅ | |
| 19 | Disposable items | ✅ | |
| 20 | Stationery | ✅ | |

### ⛔ BANNED Categories — NEVER List on Amazon

| Category | Reason | Since |
|----------|--------|-------|
| **Earphones / Headphones** | High brand violation risk — 52 policy violations received (2026-03-28). Permanently banned. | 2026-03-28 |
| **Power Banks / Battery products** | Global ban across all platforms. | 2026-03-22 |

> **Earphones/headphones includes:** wired earphones, wireless earbuds, TWS earphones, over-ear headphones, on-ear headphones, gaming headsets, noise-cancelling headphones, bone conduction headphones, and any audio listening device worn on/in the ear. This ban applies regardless of brand status — even unbranded/generic earphones are banned from Amazon.

Additionally, **legacy 3C categories** remain in the system from the previous strategy (earphones/headphones REMOVED — now banned):
smart watches, action cameras, portable projector, vr glasses, power station,
ip camera, smart doorbell, soundbar, solar panel, gimbal stabilizer, lavalier microphone, smart ring.

---

## Pricing

**Markup**: 2.5x wholesale CNY price, minimum $15.99 USD
**Pricing formula**: `priceUSD = priceCNY * 2.5 / 7.2`, floor at $15.99
**MSRP (list_price)**: `standardPrice * 1.3` (strikethrough price)

---

## Amazon-Specific Technical Rules

### Flat File Upload
- **One file per product type** — mixing types causes error 90057
- TemplateSignature = `base64(PRODUCT_TYPE_NAME)` (e.g., HEADPHONES → `SEVBRFBIT05FUw==`)
- Settings: `attributeRow=3&dataRow=4`
- Output: `output/amazon-export-YYYY-MM-DD-{product-type}.txt`

### Required Fields (All Product Types)
- `age_range_description` = `Adult`
- `model_name` = derived from title (first 3 words)
- `country_of_origin` = `China`
- `warranty_description` = `1 Year Manufacturer Warranty`
- `gtin_exemption_reason` = `generic_brand`
- `list_price` = `standard_price * 1.3`
- `condition_type` = `New`

### Variation Rules
- Parent rows: `parent_child=Parent`, `variation_theme=Color|Size|ColorSize`, no price/qty
- Child rows: `parent_child=Child`, MUST have `variation_theme` (same as parent), `parent_sku`, price, qty

---

## How to Add Products from a New Seller

1. User provides the 1688 seller store URL
2. Scrape all products from that store (Task 2 — detail scraping)
3. Run Task 3 (image check) → Task 8 (brand verify) → Task 4 (translate)
4. If the product type doesn't exist in `AMAZON_PRODUCT_TYPES`, add it to `amazon/src/models/product.ts`
5. Add column definitions to `amazon/src/excel/column-mapping.ts` if needed
6. Run flat file generation: `node dist/tasks/task1-excel-gen.js`
7. Upload to Seller Central: `npm run task:upload`

---

## Update Log

- **2026-03-20**: File created with 13 3C sub-categories (old strategy).
- **2026-03-22**: Store ID set to `ATVPDKIKX0DER`. Pipeline reactivated.
- **2026-03-23**: **Strategy changed to manual seller sourcing.** 3C categories retained as legacy. All brand-safe categories now eligible. Automated 1688 discovery no longer used for Amazon.
- **2026-03-28**: **Earphones/headphones permanently banned from Amazon.** 52 policy violations received — all earphone/headphone products removed. Category added to BANNED list. Legacy 3C list updated.
