# Brand-Safe Categories — Phase 1 (2026-03-24)

**Rule:** For AliExpress, eBay, and Etsy, we ONLY scrape and list brand-safe categories.
These are product types that are **structurally impossible** to have brand/IP issues.

Amazon uses a different strategy: manual seller sourcing (see ROOT_CLAUDE.md).

## Why Brand-Safe First?

- Even one violation can lead to store suspension
- The cost of losing a store far exceeds the benefit of extra product volume
- Brand-safe categories flow through the pipeline 10x faster (skip seller verification)
- Phase 2 (wider categories) only after 3+ months of zero violations

## Brand-Safe Category Groups

| # | Category Group | AliExpress | Amazon | eBay | Etsy | 1688 Search Terms |
|---|---|:---:|:---:|:---:|:---:|---|
| 1 | **Phone cases / accessories** | ✅ | ✅ | ✅ | ✅ | 手机壳, 手机支架, 手机挂绳 |
| 2 | **DIY / craft supplies** (beads, buttons, lace, ribbon) | ✅ | ✅ | ✅ | ✅ | 串珠, 纽扣, 蕾丝辅料, 丝带 |
| 3 | **Jewelry findings / components** (clasps, hooks, jump rings) | ✅ | ✅ | ✅ | ✅ | 首饰配件, 龙虾扣, DIY饰品材料 |
| 4 | **Handmade jewelry** (no-brand artisan, generic) | ✅ | ✅ | ✅ | ✅ | 手工耳环, 手工项链 |
| 5 | **Hair accessories** (clips, ties, scrunchies, claws) | ✅ | ✅ | ✅ | ✅ | 发夹, 头绳, 大肠发圈, 鲨鱼夹 |
| 6 | **Shoe accessories** (insoles, laces, decorations) | ✅ | ✅ | ✅ | ✅ | 鞋垫, 鞋带, 洞洞鞋装饰 |
| 7 | **Sewing notions** (zippers, snaps, elastic, thread) | ✅ | ✅ | ✅ | ✅ | 拉链, 四合扣, 松紧带 |
| 8 | **Stickers / patches / iron-ons** | ✅ | ✅ | ✅ | ✅ | 布贴, 刺绣贴, 烫画 |
| 9 | **Keychains / bag charms** | ✅ | ✅ | ✅ | ✅ | 钥匙扣, 包包挂件 |
| 10 | **Candles / home fragrance** | ✅ | ✅ | ✅ | ✅ | 蜡烛, 香薰 |
| 11 | **Storage / organizers** | ✅ | ✅ | ✅ | ✅ | 收纳盒, 桌面整理 |
| 12 | **Pet accessories** (collars, toys, bows) | ✅ | ✅ | ✅ | ✅ | 宠物项圈, 宠物玩具 |
| 13 | **Fitness small accessories** (bands, grips, ropes) | ✅ | ✅ | ✅ | ❌ | 弹力带, 握力器, 跳绳 |
| 14 | **Painting / art supplies** | ✅ | ✅ | ✅ | ✅ | 画笔, 颜料, 画布 |
| 15 | **Custom / personalized gifts** | ❌ | ✅ | ✅ | ✅ | 定制礼品 |
| 16 | **Eyewear accessories** (cases, chains, nose pads, lens cloths) | ✅ | ✅ | ✅ | ✅ | 眼镜盒, 眼镜链, 眼镜鼻托 |
| 17 | **Watch accessories** (bands, boxes, tools) | ✅ | ✅ | ✅ | ✅ | 表带, 手表盒, 修表工具 |
| 18 | **Scarves accessories** (scarf rings, clips) | ✅ | ✅ | ✅ | ✅ | 丝巾扣, 围巾配件 |
| 19 | **Disposable items** (masks, slippers, underwear) | ✅ | ✅ | ✅ | ✅ | 一次性口罩, 一次性拖鞋 |
| 20 | **Stationery** (stickers, washi tape, pen holders) | ✅ | ✅ | ✅ | ✅ | 手账贴纸, 和纸胶带 |

## Why These Categories Are Safe

1. **No brand premium** — brands don't compete in these spaces at wholesale prices
2. **Raw materials** — beads, fabric, findings are commodities, not branded products
3. **Generic by nature** — hair clips, insoles, keychains are inherently unbranded
4. **Customizable** — blank items for personalization have no pre-existing brand
5. **Disposable** — single-use items (masks, disposable slippers) are never branded at wholesale
6. **Components** — jewelry findings, sewing notions are parts, not finished branded goods

## AliExpress Template Sheet Mapping

108 of the 625 AliExpress template sheets are tagged `brand_safe: true` in:
`1688_scrapper/src/data/blue-ocean-search-terms.json`

These 108 sheets are the ONLY ones Task 1 discovers during Phase 1.

## Phase 2 Expansion (FUTURE — requires 3+ months zero violations)

After sustained zero violations, gradually expand to:
- Watches (generic quartz, digital)
- Bags & wallets (unbranded fashion)
- Shoes (generic casual, slippers)
- Finished jewelry (generic fashion rings, earrings, necklaces)
- Sunglasses (unbranded fashion)

Phase 2 categories require Task 8 seller verification — NOT auto-authorized by Task 8B.

## How Brand-Safe Categories Flow Through Pipeline

```
Task 1 (discover, brand_safe=true only)
  → Task 1B (brand pre-filter — double check)
  → Task 2 (scrape details)
  → Task 3 (image check)
  → Task 8B (auto-verify — instant pass for brand-safe categories)  ← SKIP seller outreach
  → Task 4 (translate)
  → Task 5 (enrich)
  → Ready for all platforms
```

Brand-safe products skip Task 8 (seller outreach) entirely — Task 8B auto-authorizes them
because `isInherentlySafeCategory()` returns true. This saves days of waiting for seller replies.
