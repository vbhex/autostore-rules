# Multi-Platform Category Strategy

> **AS OF 2026-05-13: 3 active platforms (Amazon / eBay / Etsy).**
> eBay and Etsy use **automated 1688 discovery** for 20 brand-safe category groups.
> Amazon uses **automated 3C discovery + manual seller sourcing** — Task 1 runs for 12 Amazon-3C categories (see autostore root CLAUDE.md), and the user also identifies trusted seller stores manually.
> See `BRAND_SAFE_CATEGORIES.md` for the full category-platform matrix.
>
> **AliExpress store `$ALIEXPRESS_STORE_ID` is PERMANENTLY CLOSED (2026-03-25).** Do not run scraping, translation, upload, or listing tasks for AliExpress. The `aliexpress/` and `ae_scrapper/` code is archived.

---

## Platform Overview

| Platform | Status | Sourcing | Categories |
|----------|--------|----------|------------|
| ~~AliExpress~~ | **CLOSED (2026-03-25)** | — | — |
| **Amazon** | **ACTIVE** | Automated (3C, 12 categories) + manual seller sourcing | 3C / Consumer Electronics |
| **eBay** | **ACTIVE** | Automated 1688 pipeline | 20 brand-safe groups (see BRAND_SAFE_CATEGORIES.md) |
| **Etsy** | **ACTIVE** | Automated 1688 pipeline | 19 brand-safe groups (no fitness accessories) |

---

## Two Sourcing Strategies

### Strategy 1: Automated Discovery (AliExpress, eBay, Etsy)

Products are sourced via the shared 1688 scraper pipeline:
```
Task 1 (discover) → 1B (brand filter) → 2 (scrape) → 3 (image check)
→ 8B (auto-authorize brand-safe) → 4 (translate) → 5 (enrich)
→ Platform-specific import → Excel/CSV gen → Upload
```

All 3 platforms share the same source product pool. A single product can be listed on
AliExpress + eBay + Etsy simultaneously.

### Strategy 2: Manual Seller Sourcing (Amazon only)

Products come from trusted 1688 sellers the user identifies manually:
```
User provides seller store URL → Scrape seller's products (Task 2)
→ 3 (image check) → 8 (brand verify) → 4 (translate)
→ Flat file generation → Seller Central upload
```

Amazon does NOT use Task 1 automated discovery.

---

## Brand-Safe Categories (20 groups)

These categories are **structurally impossible** to have brand/IP issues.
Phase 1 focuses exclusively on these. Phase 2 (wider categories) only after 3+ months of zero violations.

| # | Category Group | AliExpress | Amazon | eBay | Etsy |
|---|---|:---:|:---:|:---:|:---:|
| 1 | Phone cases / accessories | ✅ | ✅ | ✅ | ✅ |
| 2 | DIY / craft supplies | ✅ | ✅ | ✅ | ✅ |
| 3 | Jewelry findings / components | ✅ | ✅ | ✅ | ✅ |
| 4 | Handmade jewelry (no-brand) | ✅ | ✅ | ✅ | ✅ |
| 5 | Hair accessories | ✅ | ✅ | ✅ | ✅ |
| 6 | Shoe accessories | ✅ | ✅ | ✅ | ✅ |
| 7 | Sewing notions | ✅ | ✅ | ✅ | ✅ |
| 8 | Stickers / patches / iron-ons | ✅ | ✅ | ✅ | ✅ |
| 9 | Keychains / bag charms | ✅ | ✅ | ✅ | ✅ |
| 10 | Candles / home fragrance | ✅ | ✅ | ✅ | ✅ |
| 11 | Storage / organizers | ✅ | ✅ | ✅ | ✅ |
| 12 | Pet accessories | ✅ | ✅ | ✅ | ✅ |
| 13 | Fitness small accessories | ✅ | ✅ | ✅ | ❌ |
| 14 | Painting / art supplies | ✅ | ✅ | ✅ | ✅ |
| 15 | Custom / personalized gifts | ❌ | ✅ | ✅ | ✅ |
| 16 | Eyewear accessories | ✅ | ✅ | ✅ | ✅ |
| 17 | Watch accessories | ✅ | ✅ | ✅ | ✅ |
| 18 | Scarves accessories | ✅ | ✅ | ✅ | ✅ |
| 19 | Disposable items | ✅ | ✅ | ✅ | ✅ |
| 20 | Stationery | ✅ | ✅ | ✅ | ✅ |

Full reference with 1688 search terms: `BRAND_SAFE_CATEGORIES.md`

---

## Legacy: 3C / Consumer Electronics (historical)

The original strategy (pre-2026-03-05) sourced 3C electronics. This was retired because:
- FCC (US) + CE (EU) + UKCA (UK) certifications blocked most electronics categories
- Budget 1688 suppliers cannot provide certification docs — structural problem
- Multiple brand violations from electronics products

Some 3C products remain in the database from legacy runs. They can still be exported to
Amazon via flat files if needed (47 products were generated on 2026-03-23).

---

## Update Log

- **2026-02-24**: Initial document. AliExpress active (24 3C cats). Amazon + Etsy categories defined.
- **2026-02-28**: Amazon category split from AliExpress (13 vs 11 categories).
- **2026-03-04**: Power bank BANNED from all platforms.
- **2026-03-05**: AliExpress pivoted from 3C to Clothing & Apparel.
- **2026-03-06**: Amazon, eBay, Etsy paused. Only AliExpress active.
- **2026-03-22**: Amazon reactivated.
- **2026-03-23**: **Major strategy change.** All 4 platforms active. Brand-safe categories replace 3C/clothing. Amazon switches to manual seller sourcing. eBay + Etsy activated.
