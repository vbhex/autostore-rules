# Brand Safety & Compliance Strategy

**Created:** 2026-03-22
**Status:** Active
**Applies to:** All platforms (AliExpress, Amazon, eBay, Etsy)

---

## Overview

This document describes the end-to-end brand safety system that prevents unauthorized brand products from being scraped, listed, or sold on any of our marketplace stores.

The system has three layers of defense:

1. **Proactive brand filtering** — Block known brands at scraping time
2. **Seller verification** — Contact suppliers to confirm brand status + collect compliance certs
3. **Continuous learning** — Update the brand list from violation reports, trademark databases, and seller responses

---

## Architecture

### Database Tables (in `1688_source`)

| Table | Purpose | Records |
|-------|---------|---------|
| `brand_list` | All known brand names (EN + ZH + aliases) with risk levels | 350-400+ |
| `providers` | Supplier/seller directory with trust levels and contact info | Growing |
| `authorized_products` | Products confirmed safe to list (per platform) | Growing |
| `compliance_certs` | Compliance documents received from sellers | Growing |
| `compliance_contacts` | Seller outreach status tracking | Growing |

### Pipeline Integration

```
1688 Search Results
    │
    ▼
┌─────────────────────────────────────┐
│  Task 1: isBannedBrand(title)       │  ← Layer 1: Known brand filter
│  Skip if matched                     │
└─────────────────────────────────────┘
    │ passed
    ▼
┌─────────────────────────────────────┐
│  Task 2: isBannedBrand() on:        │  ← Layer 1: Deep scan
│  - title, description               │
│  - seller name                       │
│  - specifications (品牌 field)       │
│  - variant option values             │
│  Skip if matched anywhere            │
└─────────────────────────────────────┘
    │ passed
    ▼
Tasks 3-5 (images, translate, enrich)
    │
    ▼
┌─────────────────────────────────────┐
│  Task 8: Seller Contact             │  ← Layer 2: Human verification
│  - Brand verification question       │
│  - Compliance cert request           │
│  - Auto-authorize if preferred       │
│  - Auto-skip if blacklisted          │
└─────────────────────────────────────┘
    │ confirmed safe
    ▼
authorized_products → ready for listing
```

---

## Layer 1: Brand List

### Data Sources

| Source | How | Frequency |
|--------|-----|-----------|
| **Initial CSV import** | `data/brands-comprehensive-2026-03-22.csv` (250+ brands) | One-time |
| **JSON fallback** | `aliexpress/banned-brands.json` (150 brands) | Migrated to DB |
| **AliExpress violations** | Parse violation notifications → add new brands | Per violation |
| **Trademark databases** | Periodic web scraping of WIPO, USPTO, CNIPA | Quarterly |
| **Seller responses** | When seller says "this is brand X" → add to list | Ongoing |
| **Manual** | CLI: `node dist/tasks/task-brand-import.js --source manual` | Ad-hoc |

### Categories

| Category | Risk Level | Count | Examples |
|----------|-----------|-------|---------|
| `watches` | critical-medium | 60+ | Rolex, Omega, TAG Heuer, Seiko, Casio |
| `eyewear` | critical-medium | 30+ | Ray-Ban, Oakley, Gentle Monster, Maui Jim |
| `jewelry` | critical-medium | 25+ | Tiffany, Cartier, Pandora, Swarovski |
| `bags` | critical-medium | 35+ | LV, Hermes, Chanel, Coach, Samsonite |
| `shoes` | critical-medium | 25+ | Nike, Adidas, Louboutin, Dr. Martens |
| `fashion_luxury` | critical-high | 60+ | Gucci, Prada, Burberry, Ralph Lauren |
| `fashion_streetwear` | critical-medium | 25+ | Supreme, Off-White, BAPE, Fear of God |
| `fashion_outerwear` | critical-medium | 12+ | Moncler, Canada Goose, Arc'teryx |
| `fashion_chinese` | high-medium | 15+ | Li-Ning, Anta, Bosideng |
| `electronics_3c` | critical-medium | 25+ | Apple, Samsung, Xiaomi, DJI |
| `entertainment_ip` | critical-medium | 15+ | Disney, Marvel, Sanrio, Pokemon |
| `textiles_fibers` | high-low | 9 | Tencel, Gore-Tex, Lycra |
| Other categories | various | 20+ | Sports leagues, automotive, government |

### Pattern/Design Trademarks (No Brand Name Required)

These patterns ARE the trademark — even "unbranded" products with these patterns will be flagged:

| Brand | Protected Pattern |
|-------|------------------|
| Louis Vuitton | LV monogram, Damier check |
| Burberry | Nova check / tartan plaid |
| Gucci | GG monogram, green-red-green stripe |
| Dior | Oblique monogram |
| Fendi | FF monogram (Zucca) |
| Goyard | Chevron Y pattern |
| Adidas | Three parallel stripes |
| Nike | Swoosh logo |
| Christian Louboutin | Red sole (Pantone 18-1663 TPX) |

---

## Layer 2: Seller Verification (Task 8)

### Combined Outreach Message

Task 8 now sends a single combined message asking:
1. **Brand confirmation** — Is this product branded? Own brand? Generic?
2. **Compliance certs** — Testing Report, REACH, CE, UKCA, FCC, CPSIA, RoHS

### Seller Response Handling

| Seller Response | Action |
|----------------|--------|
| "Not branded / generic / OEM" | → `authorized_products(type=not_branded)`, provider trust → `verified` |
| "Our own brand" + auth doc | → `authorized_products(type=authorized_reseller)`, save cert |
| "It IS brand X" | → Skip product, add brand to `brand_list` if new, consider blacklisting provider |
| Provides certs (SGS/TUV/etc) | → Record in `compliance_certs`, provider compliance_score ↑ |
| No response after 7 days | → Re-contact once, then skip product |
| Multiple violations | → Provider trust → `blacklisted` |

### Trust Level Progression

```
new → verified → trusted → preferred
  └─────────────────────→ blacklisted
```

| Level | Meaning | Auto-action |
|-------|---------|-------------|
| `new` | First encounter | Task 8 contacts them |
| `verified` | Confirmed not-branded once | Task 8 contacts for new products |
| `trusted` | 3+ clean confirmations, has some certs | Faster processing |
| `preferred` | Has all certs, long history, auto-approved | Auto-authorize all products |
| `blacklisted` | Caught selling branded goods, no response | Auto-skip all products |

---

## Layer 3: Compliance Certificates

### Required Certs by Market

| Market | Required Certs | Category |
|--------|---------------|----------|
| **EU** | CE marking, REACH, OEKO-TEX (textiles) | All products |
| **UK** | UKCA, Testing Report | All products |
| **US** | FCC ID (electronics), CPSIA (children's), RoHS | Category-dependent |
| **All** | Product Testing Report (SGS, TUV, Intertek, BV) | All products |

### Cert Type Reference

| Cert Type | DB Value | What It Proves |
|-----------|----------|---------------|
| Testing Report | `testing_report` | Product meets safety/quality standards |
| REACH | `reach` | EU chemical safety regulation compliance |
| OEKO-TEX | `oeko_tex` | Textile tested for harmful substances |
| FCC | `fcc` | US electromagnetic compatibility (electronics) |
| CE | `ce` | EU product safety conformity |
| UKCA | `ukca` | UK product safety conformity |
| RoHS | `rohs` | Restricted hazardous substances |
| CPSIA | `cpsia` | US children's product safety |
| Brand Authorization | `brand_authorization` | Written permission to resell brand |

### Impact on Listing

- **No certs at all**: Can still list on AliExpress (lower priority markets like RU, BR)
- **Has Testing Report**: Can list everywhere except US electronics / EU regulated
- **Has REACH + CE**: Full EU market access
- **Has FCC**: Full US market access (electronics only)
- **Has all certs**: Premium product — prioritize for listing

---

## CLI Tools

### Brand Management
```bash
# Import brands from CSV
node dist/tasks/task-brand-import.js --source file --path data/brands-comprehensive-2026-03-22.csv

# Add single brand
node dist/tasks/task-brand-import.js --source manual --brand "NewBrand" --zh "新品牌" --category watches

# Add from violation report
node dist/tasks/task-brand-import.js --source violation --brand "ViolatedBrand" --category jewelry --notes "AE violation #456"

# List all / search / stats
node dist/tasks/task-brand-import.js --list
node dist/tasks/task-brand-import.js --search "nike"
node dist/tasks/task-brand-import.js --stats
```

### Seller Verification
```bash
# Dry run (preview messages)
node dist/tasks/task8-brand-verify.js --dry-run --limit 5

# Live run (contacts sellers via Wangwang)
node dist/tasks/task8-brand-verify.js --limit 10
```

---

## Continuous Improvement

### After Each AliExpress Violation
1. Parse the violation notification for the brand name
2. Add to `brand_list` via: `node dist/tasks/task-brand-import.js --source violation --brand "X" --category Y --notes "AE violation #Z"`
3. Find and remove any products from that brand still in pipeline
4. Check if the provider should be downgraded

### Quarterly Brand List Refresh
1. Check WIPO Global Brand Database for new registrations in our categories
2. Check AliExpress IPR program updates
3. Review Amazon Brand Registry additions
4. Run deduplication on brand_list (same brand, different categories = OK)

### Provider Relationship Management
- Track which providers consistently have clean products
- Prefer providers who proactively share certs
- Build a short-list of 10-20 preferred providers per category
- Consider exclusive arrangements with best providers
