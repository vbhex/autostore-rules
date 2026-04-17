# eBay Shipping Fee Strategy

**Applies to**: All eBay listings created under AutoStore Greater China seller accounts.

**Status**: ACTIVE — adopted 2026-04-17.

---

## Rule 1 — Ship From China to US/Canada (Primary Markets)

All products ship from China (specifically Guangdong/Shenzhen) to our primary markets:

- **Primary destinations**: United States, Canada
- **Origin**: Guangdong, ShenzhenCity, China
- **Carrier**: eBay SpeedPAK Economy (worldwide) or SpeedPAK Standard (required per Greater China seller constraint — see `EBAY_GREATER_CHINA_SELLER_CONSTRAINTS.md`)

### What this means operationally

- The shipping policy ships worldwide, but **US and Canada are where we expect the majority of sales**. Price shipping with these two markets as the primary reference.
- Do NOT assume local delivery or domestic US shipping rates — every package travels China → destination via international route.
- Transit time: 6–15 business days via SpeedPAK Economy. Buyers in the US/Canada should expect ~2 weeks delivery.

---

## Rule 2 — Estimate Shipping Fees by Competitor Reference

For every product we list, the shipping fee set in the listing (or bundled into free-shipping price) must be based on **real market data**, not guesswork.

### Method

1. Before finalizing a listing, search eBay for a **similar weight/size product** shipping from China.
2. Filter by:
   - Same category or close substitute
   - Location: China (or "Ships from: China")
   - Destination: US (for weight-class reference)
3. Note the shipping fee those competing sellers charge.
4. Set our shipping fee at a **comparable or slightly lower** price to stay competitive.

### Why this matters

- Shipping fees are a ranking factor in eBay search (eBay weights cheap-or-free shipping favorably).
- Over-charging shipping kills conversion even when the item price is competitive.
- Under-charging shipping loses money on each order (especially for heavy items).

### Practical reference points (from 2026-04-17 competitor scan)

| Item weight class | Competitor ship fee to US (scan of 6 sold listings from China) |
|-------------------|------------------------------------------------------------------|
| Small jewelry / accessories (<50g) | **Free** (bundled into $10-14 price) |
| Medium jewelry / small bags (50–250g) | $7.58 – $18.94 (median ~$9) |
| Clothing / larger accessories (250g–1kg) | $10–$20 (estimated — rescan when listing) |
| Bulky items / multi-pack (1–2kg) | $15–$30 (estimated — rescan when listing) |

**Current AutoStore default** (as of 2026-04-17): **$3.00 first item / $1.00 each additional** on the SpeedPAK Free Shipping policy. This is a balanced middle-ground:
- For small jewelry: $3 is slightly above market (competitors offer free) but buyers accept $2-4 range without churn
- For body chains / medium items: $3 is well below market ($7-19 competitors) → strong competitive advantage

**Review this default if** the listing mix shifts heavily toward heavier items (>250g) — may need to raise or split policies.

**Previous default**: $5.00 / $2.00 (used 2026-04-17 morning through early afternoon). Lowered based on competitor scan showing small jewelry from China ships free.

---

## Workflow When Listing a New Product

1. **Pick product** from the 1688_source DB (must be `authorized` + `translated`).
2. **Check product weight/dimensions** — either from 1688 spec data or by visually assessing photos.
3. **Search eBay** for similar weight/size competitors:
   - Search query: relevant keywords + "from china"
   - Filter: Sold listings (to see what's actually selling)
4. **Note competitor shipping fees** for 3–5 similar listings.
5. **Set your shipping fee**:
   - If your product is small enough to bundle shipping into price → free shipping, price = item cost + markup + ~$5 postage
   - If the item is too heavy for free shipping → charge shipping explicitly, set to competitor median
6. **Apply the shipping policy** (SpeedPAK Free Shipping with worldwide SpeedPAK Economy at $5/$2) OR override per-listing if the weight class requires higher charges.

---

## Reference

- `documents/ebay-store/EBAY_GREATER_CHINA_SELLER_CONSTRAINTS.md` — account-level shipping restrictions (SpeedPAK-only, high-risk cat blocks, etc.)
- `documents/ebay-store/EBAY_SOURCING_STRATEGY.md` — demand-first sourcing (pick products that sell, then source)
- `ebay/CLAUDE.md` — project-level rules summary

**Update this doc when:**
- New destination markets are added (e.g., UK, Australia with local demand).
- Shipping fee defaults change due to carrier rate updates.
- New weight-class benchmarks are discovered through competitor analysis.
