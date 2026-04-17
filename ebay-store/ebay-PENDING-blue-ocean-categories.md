# Blue Ocean Categories — eBay Store (PENDING Store ID)

**Platform**: eBay
**Store ID**: PENDING — check Seller Hub after first login
**Store URL**: https://www.ebay.com/sh/ovw
**Seller Hub Reports**: https://www.ebay.com/sh/reports
**Source scraper**: 1688_scrapper (shared with AliExpress)
**Last updated**: 2026-03-22

---

## Strategy: Cross-List from AliExpress Pipeline

eBay sources from the **same 1688 pipeline** as the AliExpress store. Products already in `authorized_products` are eligible for eBay cross-listing — no re-scraping or re-verification needed.

**Key differences from AliExpress:**
- eBay fees: ~13-15% (final value + payment processing) vs AliExpress ~8%
- eBay pricing: markup must be higher to compensate for fees
- eBay new seller limit: **10 items / $500** initially — grow gradually
- eBay VeRO program: aggressive IP enforcement — brand safety is even more critical
- Upload format: CSV via Seller Hub Reports (not Excel templates)
- Categories: eBay leaf category IDs (numeric), not template sheet names

---

## Red Ocean — AVOID These Categories

> **NEVER list products in these categories on eBay:**
> 1. **Women's Clothing** (15724) — oversaturated, high return rate, sizing issues
> 2. **Men's Clothing** (1059) — oversaturated, high return rate, sizing issues
> 3. **Electronics / 3C** — FCC/CE required, eBay very strict on compliance
> 4. **Shoes** — high return rate due to sizing, brand enforcement aggressive
> 5. **Any branded/designer products** — eBay VeRO program is more aggressive than AliExpress

---

## eBay Category Hierarchy (Relevant L1s)

| eBay L1 | Category ID | Our Strategy |
|---------|-------------|-------------|
| Jewelry & Watches | 281 | **PRIMARY** — watches, fashion jewelry |
| Clothing, Shoes & Accessories | 11450 | **Accessories only** — hats, scarves, sunglasses, belts, bags |
| Sporting Goods | 888 | **FUTURE** — fitness equipment/accessories |
| Health & Beauty | 26395 | **FUTURE** — beauty tools, grooming accessories |

---

## Category 1: Watches (PRIORITY — Start Here)

**eBay path**: Jewelry & Watches → Watches, Parts & Accessories → Watches
**eBay parent category ID**: 260325
**Strategy**: Unbranded fashion/quartz watches. Strong eBay watch market. Gift-driven demand. $15-80 price range.
**1688 source**: Same products as AliExpress `QuartzWristwatches`, `CoupleWatches`, `DigitalWristwatches`
**Status**: ACTIVE — first 10 listings

| CLI Category | 1688 Search Term | eBay Leaf Category | Competition | AE Cross-List |
|---|---|---|---|---|
| `quartz watches` | 石英表 简约 男女 跨境专供 | Wristwatches (31387) | Medium | QuartzWristwatches |
| `couple watches` | 情侣表 对表 跨境专供 | Wristwatches (31387) | Low | CoupleWatches |
| `digital watches` | 数字运动手表 跨境专供 | Wristwatches (31387) | Medium | DigitalWristwatches |

---

## Category 2: Fashion Jewelry

**eBay path**: Jewelry & Watches → Fashion Jewelry
**eBay parent category ID**: 10968
**Strategy**: Fashion/costume jewelry at 1688 wholesale CNY 3–50 with 4–5× markup. No certification needed. Large eBay market.
**Status**: ACTIVE — include in first 10 listings

| CLI Category | 1688 Search Term | eBay Leaf Category | Category ID | Competition | AE Cross-List |
|---|---|---|---|---|---|
| `fashion earrings` | 时尚耳环 女士 跨境 | Fashion Earrings | 50647 | Medium | Earrings |
| `fashion bracelets` | 时尚手链 跨境专供 | Fashion Bracelets & Charms | 261987 | Low | Bracelets |
| `fashion necklaces` | 时尚项链 女士 跨境 | Fashion Necklaces & Pendants | 155101 | Medium | Necklaces |
| `fashion rings` | 时尚戒指 跨境专供 | Fashion Rings | 67681 | Low | Rings |
| `fashion brooches` | 时尚胸针 跨境专供 | Fashion Brooches & Pins | 50677 | Low | — |

---

## Category 3: Sunglasses & Eyewear

**eBay path**: Clothing, Shoes & Accessories → Women's Accessories / Men's Accessories → Sunglasses
**Strategy**: High margins (3-4x), seasonal spikes in summer. No certification needed.
**Status**: NEXT BATCH — after seller limits increase

| CLI Category | 1688 Search Term | eBay Leaf Category | Competition | AE Cross-List |
|---|---|---|---|---|
| `polarized sunglasses` | 偏光太阳镜 男女 跨境 | Sunglasses & Sunglasses Accessories | Medium | Sunglasses |
| `sports sunglasses` | 运动太阳镜 骑行 跨境 | Sunglasses & Sunglasses Accessories | Low | SportsSunglasses |
| `blue light glasses` | 防蓝光眼镜 平光镜 跨境 | Eyeglasses | Low | BlueLightBlockingGlasses |
| `reading glasses` | 老花眼镜 时尚 跨境 | Reading Glasses | Low | ReadingGlasses |

---

## Category 4: Hats & Caps

**eBay path**: Clothing, Shoes & Accessories → Women's/Men's Accessories → Hats
**Strategy**: Low brand risk, seasonal demand, impulse buys.
**Status**: NEXT BATCH

| CLI Category | 1688 Search Term | eBay Leaf Category | Competition | AE Cross-List |
|---|---|---|---|---|
| `bucket hats` | 渔夫帽 男女 跨境专供 | Bucket Hats | Low | BucketHats |
| `baseball caps` | 棒球帽 时尚 跨境专供 | Baseball Caps | Medium | BaseballCaps |
| `beanies` | 针织帽 男女 冬季 跨境 | Beanies | Low | Hats |

---

## Category 5: Bags & Wallets

**eBay path**: Clothing, Shoes & Accessories → Women's Bags & Handbags / Men's Accessories
**eBay Women's Bags ID**: 169291
**Strategy**: Steady demand, no sizing issues. AVOID anything resembling designer bags.
**Status**: QUEUED

| CLI Category | 1688 Search Term | eBay Leaf Category | Competition | AE Cross-List |
|---|---|---|---|---|
| `waist packs` | 腰包 男女 跨境专供 | Waist Bags & Fanny Packs | Low | WaistPacks |
| `coin purses` | 零钱包 可爱 跨境 | Coin Purses & Pouches | Low | CoinPurses |
| `fashion backpacks` | 时尚双肩包 跨境专供 | Backpacks | Medium | Backpacks |

---

## Category 6: Hair Accessories

**eBay path**: Clothing, Shoes & Accessories → Women's Accessories → Hair Accessories
**Strategy**: Growing niche, very low competition, high margins.
**Status**: QUEUED

| CLI Category | 1688 Search Term | eBay Leaf Category | Competition | AE Cross-List |
|---|---|---|---|---|
| `hair claws` | 鲨鱼夹 发夹 跨境专供 | Hair Claws & Clips | Low | HairClaw |
| `hair pins` | 发夹 发卡 时尚 跨境 | Bobby Pins & Clips | Low | HairPin |

---

## Category 7: Scarves

**eBay path**: Clothing, Shoes & Accessories → Women's Accessories → Scarves & Wraps
**Strategy**: Seasonal demand (fall/winter), good margins.
**Status**: QUEUED

| CLI Category | 1688 Search Term | eBay Leaf Category | Competition | AE Cross-List |
|---|---|---|---|---|
| `silk scarves` | 真丝丝巾 女士 跨境专供 | Scarves & Wraps | Low | SilkScarves |
| `winter scarves` | 针织围巾 男女 跨境 | Scarves & Wraps | Medium | Scarves |

---

## Category 8: Fitness Accessories (FUTURE)

**eBay path**: Sporting Goods → Fitness, Running & Yoga → Fitness Equipment & Gear
**eBay category ID**: 28064
**Strategy**: Home fitness trend growing on eBay. Resistance bands, ab rollers, balance boards.
**Status**: QUEUED — add after seller limits increase substantially

---

## Pricing Strategy

eBay has higher fees than AliExpress (~13-15% vs ~8%), so pricing must compensate:

```
priceUsd = priceCny * markup / 7.2
markup = 3.0 (vs 2.5 for Amazon, 2-3 for AliExpress)
minRetailPriceUsd = $9.99
listPrice (strikethrough) = standardPrice * 1.4

eBay fee breakdown:
- Final value fee: ~12.9% (varies by category)
- Payment processing: ~2.7%
- Total: ~15.6%
```

---

## New Seller Growth Strategy

| Phase | Listing Limit | Action |
|-------|--------------|--------|
| Week 1-2 | 10 items / $500 | List top 10 products (watches + jewelry mix) |
| Week 3-4 | ~50 items | Request limit increase, add sunglasses + hats |
| Month 2 | ~200 items | Add bags, hair accessories, scarves |
| Month 3+ | 500+ items | Add fitness accessories, scale all categories |

---

## Pipeline Setup Checklist

- [ ] Confirm eBay store ID after first login
- [ ] Download Seller Hub Reports CSV templates for target categories
- [ ] Map eBay leaf category IDs (exact IDs require Seller Hub or Taxonomy API)
- [ ] Set up Business Policies (payment, return, shipping)
- [ ] Apply for selling limits increase after first 10 sales
- [ ] Configure international shipping or eBay Global Shipping Program

---

## How to Add a New Sub-Category

1. Confirm it is NOT in the Red Ocean list above
2. Look up the eBay leaf category ID via Seller Hub or https://www.ebay.com/n/all-categories
3. Confirm the product is already in `authorized_products` (1688_source DB)
4. Add the eBay category mapping to `ebay/src/models/product.ts`
5. Add to this file with status
6. Generate CSV and upload via Seller Hub Reports

---

## Update Log

- **2026-03-22**: File created. Initial category strategy: Watches + Fashion Jewelry (Priority 1), Sunglasses + Hats (Priority 2), Bags + Hair + Scarves (Priority 3), Fitness (Future).
