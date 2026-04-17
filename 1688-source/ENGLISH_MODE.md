# 1688 English Mode — How It Works

**Discovered:** 2026-03-25
**Status:** Active — all scraper runs produce English content

---

## The Mechanism

The Puppeteer browser in `1688_scrapper` is launched with the flag:

```
--lang=en-US
```

This is set in `1688_scrapper/src/scrapers/1688Scraper.ts` inside the Puppeteer `launch()` call (under `args`). 1688.com respects the browser language header and serves the entire page — product titles, descriptions, specifications, variant names, category filters — in English.

**This is NOT:**
- A cookie setting
- An account/login language preference
- A URL parameter (`?lang=en_US` does not work)
- Specific to `global.1688.com` (still uses `s.1688.com/selloffer/...`)

**It IS:**
- The standard Puppeteer `--lang=en-US` Chrome flag, already present in the scraper launch config
- Inherited automatically by every scraper run on the China MacBook

---

## Impact on the Pipeline

### What gets scraped in English
All content scraped by Tasks 1 & 2 is already in English when it lands in the DB:

| DB Field | Stored Value |
|----------|-------------|
| `products.title_zh` | English title (field name is misleading) |
| `products_raw.title_zh` | English title |
| `products_raw.description_zh` | English description |
| `products_raw.specifications_zh` | JSON array with English keys & values |
| `products_variants_raw.option_name` | English variant dimension (e.g. "Color", "Size") |
| `products_variants_raw.option_value` | English variant value (e.g. "Red", "XL") |

> **Note:** The `_zh` suffix on field names is a historical artifact from when the scraper ran in Chinese. The field names are NOT being renamed — too many queries depend on them. Just know the content is English.

### Task 4 — Free pass for new products

Task 4 (`task4-translate.ts`) includes `isAlreadyEnglish()` detection added 2026-03-25:

```typescript
function isAlreadyEnglish(text): boolean {
  // < 10% CJK characters = already English
  const cjkCount = (text.match(/[\u4e00-\u9fff...]/g) || []).length;
  return cjkCount / totalChars < 0.1;
}
```

If `isAlreadyEnglish(title) && isAlreadyEnglish(description)` → **skips Baidu/Google API entirely**, copies `_zh` fields directly to `_en` fields, and only runs CNY→USD price conversion.

**Result: Zero translation API cost for all products scraped from the China MacBook.**

### Fallback still works
If for any reason a product has Chinese content (e.g., old legacy products scraped before this was set up), Task 4 falls through to the Baidu/Google API as before.

---

## Prices — Still CNY

Even though titles are in English, **prices are shown in CNY (¥) as the primary currency** on the search/detail pages. The HK$ shown alongside is just a secondary display.

- Task 4 still reads `price_cny` and calls `convertPrice()` → `price_usd`
- No change needed to price handling

---

## Confirming the Flag Is Set

To verify the `--lang=en-US` flag is present in the scraper:

```bash
grep -n "lang=en" ~/projects/autostore/1688_scrapper/src/scrapers/1688Scraper.ts
```

To confirm live — check running Chrome processes:

```bash
ps aux | grep "chrome.*lang=en" | grep -v grep
```

Should show `--lang=en-US` in the Puppeteer Chrome args.

---

## History

| Date | Event |
|------|-------|
| Pre-2026 | Scraper ran in Chinese; Task 4 called Baidu/Google for all products |
| 2026-03-25 | Discovered `--lang=en-US` flag already present in Puppeteer launch |
| 2026-03-25 | Added `isAlreadyEnglish()` bypass to Task 4 — eliminates API cost for new products |
| 2026-03-25 | Baidu API account balance: ¥0 (`54004: Please recharge`) — confirms bypass is essential |
