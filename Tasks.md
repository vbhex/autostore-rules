# AutoStore Pipeline — All Tasks

> **Project:** `1688_scrapper/`
> **Run on:** China MacBook (unless noted)
> **Build:** `./node_modules/.bin/tsc` before running any task
> **Pipeline order:** Task 1 → 1B → 2 → 3 → 8B → 4 → 5 (core flow, runs continuously via `run-alternating-pipeline.sh`)

---

## Core Sourcing Pipeline

---

### Task 1 — Product Discovery (`task1-discover.ts`, 411 lines)

Runs **two discovery modes** every execution:

**Mode A — Blue-ocean keyword search** (AliExpress / eBay / Etsy)
- Loads enabled search terms from `src/data/blue-ocean-search-terms.json` (335 categories, 108 tagged `brand_safe`)
- Searches 1688.com for each term, applies `isBannedBrand()` + price range + MOQ filters
- Inserts new products with `source_type = 'brand_safe_discovery'` or `'auto_discovery'`
- Brand-safe categories sorted first so they flow through the pipeline faster

**Mode B — Verified provider store scraping** (Amazon + any platform in `providers.target_platforms`)
- Queries `providers WHERE trust_level = 'verified' AND (last_scraped_at IS NULL OR last_scraped_at < N days ago)`
- Navigates to each `shop_url/shop/offerlist.htm`, paginates through all pages
- `isBannedBrand()` filter still applied to every product title
- Inserts with `source_type = 'manual_seller'`, `provider_id = providers.id`
- Updates `providers.last_scraped_at` after each store is scraped
- New verified providers added by `3c-outreach` skill (Tasks 10 → 11 → 12)

CLI:
- `--all-blue-ocean` — run all 335 blue-ocean categories (Mode A)
- `--batch N` — process only N categories then exit (for alternating pipeline)
- `--resume` — skip categories that already have ≥ limit products in DB
- `--limit N` — max products per category (default: 20)
- `--l1 "Watches"` — filter blue-ocean categories by L1 group
- `--provider-rescrape-days N` — re-scrape verified stores after N days (default: 7)
- `--headless` — run browser headless

---

### Task 1B — Brand Pre-Filter (`task1b-brand-prefilter.ts`, 206 lines)

Lightweight brand check between Task 1 and Task 2. No browser needed — pure DB work.

- Re-runs `isBannedBrand()` on `title_zh` — catches brands added to `brand_list` after Task 1 ran
- Checks for suspicious Chinese title patterns: `同款`, `联名`, `合作款`, `授权`, `正品`
- Checks if the product's 1688 ID appears in any prior violation record
- Checks if any blacklisted provider's shop name appears in the product URL
- Products that fail → `status = 'skipped'` with a `skip_reason`; clean products stay `'discovered'`

CLI: `--limit N`, `--dry-run`

---

### Task 2 — Scrape Details (`task2-scrape-details.ts`, 419 lines)

Visits each `discovered` product's 1688 page and scrapes full details into normalized tables.

- Scrapes title, description, specifications, price, MOQ, seller info
- Scrapes all gallery + description + variant images
- Scrapes all variant dimensions (Size × Color, etc.) and SKU combinations
- Re-runs `isBannedBrand()` on title, description, specs, variant values, seller name
- Extracts extended seller info: Wangwang ID, shop URL, seller ID → upserts into `providers`
- Output tables: `products_raw`, `products_images_raw`, `products_variants_raw`
- Status: `discovered` → `scraped`

CLI: `--limit N`, `--headless`

---

### Task 2B — Re-scrape Variants (`task2b-rescrape-variants.ts`, 199 lines)

Targets already-processed products that are missing variant data. Does not modify other product fields.

- Queries `exported` or `translated` products with no rows in `products_variants_raw`
- Re-visits the 1688 page just to extract variant/SKU data
- Useful for products scraped before variant normalization was added

CLI: `--limit N`, `--headless`

---

### Task 3 — Image Check (`task3-image-check.ts`, 127 lines)

OCR scan of all gallery images to reject Chinese text and watermarks.

- Provider auto-detection: `GOOGLE_CLOUD_API_KEY` set → Google Vision API; otherwise → Tesseract.js (local, free)
- Each gallery image is analyzed via URL download
- Product advances if it has **≥ 3 passing gallery images** (`passed = true`)
- Products with too few clean images → `status = 'skipped'`
- Output table: `products_images_ok`
- Status: `scraped` → `images_checked` or `skipped`

CLI: `--limit N`

---

### Task 8B — Automated Brand Safety Verification (`task8b-auto-verify.ts`, 469 lines)

Multi-layer automated brand check for products at `images_checked` that have no seller response yet.

**Phase 1 fast-track** (current — brand-safe categories):
- Products with `source_type = 'brand_safe_discovery'` → **instant pass**, no checks run
- These categories are structurally brand-impossible and passed Task 1B already

**Full check layers** (for `auto_discovery`, `manual_seller`, and future Phase 2):
- Layer 1: Re-run `isBannedBrand()` on title
- Layer 3: Price ceiling per category (e.g. watches ¥150, sunglasses ¥120, hair accessories ¥40)
  - Products under `INSTANT_SAFE_PRICE_CNY = 15` → fast-track pass
  - Products in `INHERENTLY_SAFE_CATEGORIES` (phone cases, keychains, shoelaces…) → fast-track pass
- Specs brand field check: if spec shows a brand NOT in `brand_list` → flag for seller outreach
- Failed products stay at `images_checked` pending manual Task 8 review

**Platform routing** (fully data-driven):
- Reads `providers.target_platforms` via `provider_id` JOIN → uses that list
- Fallback if no provider: `brand_safe_discovery` / `auto_discovery` → `['aliexpress','ebay','etsy']`; `manual_seller` / `legacy_3c` → `['amazon']`
- Authorized products inserted into `authorized_products` with `confidence = 'auto_verified'`

> **Phase 2 note:** To re-enable full checks for brand-safe products, delete the `// ── PHASE 1 FAST-TRACK ──` block in `main()`. See `CLAUDE.md` → _TASK 8B PHASE SWITCH_.

CLI: `--limit N`, `--dry-run`, `--min-age-hours N`

---

### Task 4 — Translate (`task4-translate.ts`, 261 lines)

Translates all product data from Chinese to English. Runs only on `authorized` products — saves API fees.

- Provider auto-detection: Baidu Translate (China MacBook) or Google Translate (Main Computer)
- Batch translation: title + description combined in one API call; all variant values in one batch call (~10× faster than per-field calls)
- Translates: title, description, specifications, variant names + values
- Converts price: CNY → USD (live exchange rate with caching)
- Maps color variants to standard English color families
- Output tables: `products_en`, `products_variants_en`
- Status: `images_checked` → `translated`

CLI: `--limit N`

---

### Task 5 — AE Enrich (`task5-ae-enrich.ts`, 226 lines)

Final enrichment step before products are ready for platform import.

- Checks `products_images_ok` for any gallery images with Chinese text
- **No Chinese text** → advance directly to `ae_enriched` (use 1688 images as-is)
- **Has Chinese text** → search AliExpress for the same product:
  - Found on AliExpress → store AE images + English info → `ae_enriched`
  - Not found → `skipped` (cannot list with Chinese text images)
- Output table: `products_ae_match`
- Status: `translated` → `ae_enriched` or `skipped`

CLI: `--limit N`

---

## Brand Verification (Manual / Seller Outreach)

---

### Task 8 — Brand Verify Outreach (`task8-brand-verify.ts`, 296 lines)

Manual seller outreach via Wangwang for products that Task 8B could not auto-verify.

- Targets `images_checked` products not yet in `authorized_products`
- Products from `trust_level = 'preferred'` providers → auto-authorized (no message needed)
- Products from `trust_level = 'blacklisted'` providers → auto-skipped
- Sends a single Wangwang message per seller asking: own brand? can authorize?
- Does NOT share company info in first message — only after seller agrees (Task 12 pattern)
- Records contact in `compliance_contacts` with `outreach_type = 'brand_verify'`

CLI: `--limit N`, `--dry-run`, `--headless`

---

### Task 9 — Brand Promote (`task9-brand-promote.ts`, 101 lines)

Gateway task — promotes products to `brand_verified` once they have an `authorized_products` row.

- Queries `ae_enriched` products that have `authorized_products.active = true`
- Advances them to `brand_verified` status (ready for Excel / CSV export)
- No browser needed — pure DB

CLI: `--limit N`

---

## Compliance (AliExpress / EU Market)

---

### Task 6 — Compliance Scan (`task6-compliance-scan.ts`, 189 lines)

Visits each `ae_enriched` product page to extract seller info and compliance certificates.

- Extracts: Wangwang ID, shop URL, seller ID → saved to `products_raw`
- Scans product description for cert mentions: OEKO-TEX, REACH, SGS, GOTS, ISO, CE, etc.
- Saves each cert to `compliance_certs` table
- Saves unique sellers to `compliance_contacts` (deduped) — flagged for Task 7 if no certs found

CLI: `--limit N`, `--headless`

---

### Task 7 — Wangwang Compliance Outreach (`task7-wangwang-outreach.ts`, 179 lines)

Contacts sellers in `compliance_contacts` with `status = 'pending'` via Wangwang IM.

- Sends a single message per seller requesting Testing Report + REACH / OEKO-TEX certificates
- Lists all product IDs for that seller in one message (one contact per seller, not per product)
- Records contact: `compliance_contacts.status` → `'contacted'`
- Does NOT share company info in initial message

CLI: `--limit N`, `--dry-run`, `--headless`

---

## Amazon 3C Supplier Pipeline

---

### Task 10 — 3C Supplier Discover (`task10-3c-supplier-discover.ts`, 183 lines)

Searches 1688 for 3C electronics factories/suppliers using company search (not product search).

- Searches using 13 3C keywords: earphones, smart watches, action cameras, portable projector, VR glasses, power station, soundbar, IP camera, smart doorbell, gimbal stabilizer, lavalier microphone, smart ring, solar panel
- Extracts: store name, seller ID, shop URL from company search results
- Deduplicates within results and against existing `providers` table
- Inserts new suppliers with `source = '3c_outreach'`, `trust_level = 'new'`, `target_platforms = ["amazon"]`

CLI: `--category <name>`, `--limit N`, `--dry-run`, `--headless`

---

### Task 11 — 3C Supplier Outreach (`task11-3c-supplier-outreach.ts`, 197 lines)

Contacts 3C suppliers discovered by Task 10 via Wangwang IM.

- Gets providers where `source = '3c_outreach'` and not yet contacted
- Sends a Chinese B2B message asking if they have their own brand and can provide authorization (品牌授权书) for Amazon
- Does **NOT** share company info in the initial message — that's Task 12
- Records each contact in `compliance_contacts` with `outreach_type = '3c_amazon_outreach'`

CLI: `--limit N`, `--dry-run`, `--headless`

---

### Task 12 — 3C Supplier Follow-up (`task12-3c-supplier-followup.ts`, 252 lines)

Semi-manual task for handling seller responses and completing Amazon brand authorization.

Actions:
- `list` — show all contacted 3C suppliers + current status
- `share-company-info --seller-id X` — send HK company info via Wangwang (only after seller agrees)
- `mark-authorized --seller-id X [--doc-url URL]` — mark seller as `trust_level = 'verified'` + store authorization doc URL
- `mark-no-response --seller-id X` — mark seller as non-responsive
- `stats` — pipeline summary counts

Once a seller is marked `trust_level = 'verified'`, **Task 1 automatically scrapes their store** on the next run and inserts all products with `source_type = 'manual_seller'` and `provider_id` pointing to their row. No code changes needed.

HK company info: stored in `1688_source.company_info` — use `getDefaultCompanyInfo()`

CLI: `--action <action>`, `--seller-id X`, `--doc-url URL`

---

## Utility Tasks

---

### Task Brand Import (`task-brand-import.ts`, 304 lines)

CLI tool to proactively expand the `brand_list` database (574+ brands).

Sources:
- `--source manual --brand "Nike" [--zh "耐克"] --category fashion_luxury`
- `--source file --path brands.csv` — bulk import from CSV
- `--source violation --brand "NewBrand" --notes "AE violation #123"`

Also: `--list`, `--stats`, `--search "keyword"` for inspection.

> Brands added here are immediately active in `isBannedBrand()` across Tasks 1, 1B, 2, and 8B.

---

## Pipeline Flow Diagram

```
                ┌──────────────────────────────────────────────────┐
                │            SHARED SOURCE PIPELINE                 │
                │            (platform-agnostic)                    │
                └──────────────────────────────────────────────────┘

  1688 blue-ocean search              Verified provider stores
  (AliExpress / eBay / Etsy)          (providers.trust_level = 'verified')
          │                                       │
          └─────────────────┬─────────────────────┘
                            │
                       Task 1: Discover
                            │
                       Task 1B: Brand pre-filter
                            │
                       Task 2: Scrape details
                            │
                       Task 3: Image check (OCR)
                            │
                       Task 8B: Auto brand verify ──► fail → Task 8 (manual Wangwang)
                            │
                       Task 4: Translate (authorized only)
                            │
                       Task 5: AE Enrich
                            │
               ┌────────────┴──────────────────┐
               │       ae_enriched status        │
               │   authorized_products row       │
               └────────────┬──────────────────┘
                            │
         ┌──────────────────┼────────────────┬──────────────────┐
         ▼                  ▼                ▼                   ▼
    AliExpress            eBay             Etsy               Amazon
    (Excel bulk         (CSV gen        (Etsy API          (flat file TSV
     upload)             + upload)       listing)            + Seller Central)
    (store-specific IDs live in documents/ per-platform folders)


  Amazon 3C supplier pipeline (parallel, runs independently):
  Task 10 (discover suppliers)
    → Task 11 (Wangwang outreach)
    → Task 12 (follow-up + mark-authorized)
    → trust_level = 'verified'
    → Task 1 auto-scrapes their store next run
```

---

## Quick Reference — Status Flow

| Status | Set by | Next task |
|---|---|---|
| `discovered` | Task 1 | Task 1B → Task 2 |
| `scraped` | Task 2 | Task 3 |
| `images_checked` | Task 3 | Task 8B |
| `translated` | Task 4 | Task 5 |
| `ae_enriched` | Task 5 | Platform import |
| `skipped` | Task 1B / 3 / 5 / 8B | — (manual review) |
| `ae_exported` | AliExpress Excel export | — |
| `amazon_exported` | Amazon TSV export | — |

---

## Quick Reference — source_type Values

| source_type | Set by | Authorized platforms |
|---|---|---|
| `brand_safe_discovery` | Task 1 (blue-ocean, `brand_safe: true`) | aliexpress, ebay, etsy |
| `auto_discovery` | Task 1 (blue-ocean, `brand_safe: false`) | aliexpress, ebay, etsy |
| `manual_seller` | Task 1 (verified provider store) | from `providers.target_platforms` |
| `legacy_3c` | Migration (old 3C products) | amazon |
