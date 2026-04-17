# Platform Pivot: Quit 3C → Switch to Clothing & Apparel

**Decision date**: 2026-03-05
**Status**: CONFIRMED — permanent strategic direction
**Platform**: AliExpress (default and only active platform)

---

## Decision

**Quit the 3C / Consumer Electronics main-category on AliExpress.**
**Join the Clothing & Apparel main-category on AliExpress.**

---

## Why We Are Leaving 3C

### The Compliance Wall

AliExpress requires sellers to provide qualification/certification documents before products can be made available to buyers in the US, EU, and UK. We cannot provide these documents because our 1688 budget suppliers do not hold the required certifications.

| Market | Required Certifications | Our Ability to Provide |
|--------|------------------------|------------------------|
| United States | FCC ID (wireless devices), FCC SDoC (wired digital devices) | ❌ Cannot provide |
| European Union | CE marking (EMC Directive, Low Voltage Directive, Radio Equipment Directive) | ❌ Cannot provide |
| United Kingdom | UKCA marking (post-Brexit equivalent of CE) | ❌ Cannot provide |

### Category-by-Category Breakdown

**Blocked in ALL THREE markets (US + EU + UK) — no certification available:**

| Category | Reason |
|----------|--------|
| earphones | Bluetooth → FCC ID + CE RED |
| smart watches | BT + WiFi → FCC ID + CE RED |
| speakers | Bluetooth → FCC ID + CE RED |
| wireless charger | Wireless power → FCC ID + CE |
| ip camera | WiFi → FCC ID + CE RED |
| smart doorbell | WiFi/BT → FCC ID + CE RED |
| gps tracker | Cellular/BT → FCC ID + CE RED |
| sim router | Cellular → FCC ID + CE RED |
| smart ring | Bluetooth → FCC ID + CE RED |
| translator | BT/WiFi → FCC ID + CE RED |
| vr glasses | Wireless → FCC ID + CE RED |
| soundbar | Bluetooth → FCC ID + CE RED |
| action cameras | WiFi → FCC ID + CE RED |
| gimbal stabilizer | BT app control → FCC ID + CE RED |
| power station | UL/ETL electrical safety |
| portable projector | WiFi/BT variants → FCC ID + CE RED |

**Wired/passive categories — US-safe but still blocked in EU + UK without CE/UKCA:**

| Category | US | EU | UK |
|----------|----|----|-----|
| lavalier microphone (wired) | ✅ | ❌ Needs CE | ❌ Needs UKCA |
| gaming mouse (wired) | ✅ | ❌ Needs CE | ❌ Needs UKCA |
| mechanical keyboard (wired) | ✅ | ❌ Needs CE | ❌ Needs UKCA |
| webcam (wired) | ✅ | ❌ Needs CE | ❌ Needs UKCA |
| usb hub (wired) | ✅ | ❌ Needs CE | ❌ Needs UKCA |
| phone cooler (wired) | ✅ | ❌ Needs CE | ❌ Needs UKCA |
| solar panel (passive) | ✅ | ❌ Needs CE | ❌ Needs UKCA |

**Net result: 0 out of 23 categories freely available in all three major markets.**

### The Problem is Structural

This is not fixable by finding better suppliers or negotiating differently. The FCC/CE/UKCA certification process:
- Must be completed by the product manufacturer (not the seller)
- Costs $1,000–$10,000+ per product model
- Takes weeks to months per product
- Is completely impractical for a dropship/bulk-upload AliExpress model sourcing from budget 1688 suppliers

---

## Why Clothing & Apparel

| Factor | 3C Electronics | Clothing & Apparel |
|--------|---------------|-------------------|
| US market access | ❌ Blocked (FCC) | ✅ Open |
| EU market access | ❌ Blocked (CE) | ✅ Open (basic garments) |
| UK market access | ❌ Blocked (UKCA) | ✅ Open (basic garments) |
| Qualification docs required | Yes — FCC, CE, UKCA | No — basic apparel is unregulated |
| AliExpress category strength | Strong | **#1 category on AliExpress globally** |
| 1688 supply quality | Good | Excellent — 1688 dominates global clothing wholesale |
| Listing volume potential | Limited by compliance | Virtually unlimited |
| Infrastructure rebuild needed | — | Yes (one-time) |

---

## What Needs to Be Built (New Pipeline)

### 1688 Scraper — New Categories
The existing `1688_scrapper` task flow (discover → scrape → image check → translate → image translate) can be reused. New target categories on 1688 need to be defined (e.g. women's tops, men's casual, streetwear, dresses, etc.).

### AliExpress Excel Templates
Clothing templates are different from electronics. Key new fields:
- Size (S / M / L / XL / XXL etc.)
- Color
- Material / Fabric
- Gender (Men / Women / Unisex)
- Style / Occasion
- Season

New `CATEGORY_SHEET_MAP` entries required in `aliexpress/src/models/product.ts`.

### Translation
Product titles and descriptions translate the same way. Size/material terms may need a controlled vocabulary mapping (CN size → international size).

### Image Handling
Clothing images frequently contain Chinese text (model shots with store branding). The existing image-check step (`task3`) already flags these — same logic applies.

---

## Permanent Rules (Effective 2026-03-05, updated 2026-03-06)

1. **Do not scrape, translate, or list any 3C / consumer electronics products going forward.**
2. **All new product work targets Clothing & Apparel on AliExpress.**
3. **Existing 3C listings on AliExpress are being deleted** (user deleting manually via seller center).
4. **The power bank ban and brand ban still apply** across all categories.
5. **Amazon, eBay, and Etsy are paused indefinitely (2026-03-06).** AliExpress is the only active platform. Do not plan or implement work for other platforms unless the user explicitly reactivates them.

---

## Update Log

- **2026-03-05**: Decision made. Confirmed by user. Document created.
- **2026-03-06**: Amazon, eBay, Etsy paused indefinitely. AliExpress is the only active platform. User deleting all 3C products manually.
