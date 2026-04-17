# AutoStore Reference Data — Central Database

> Last updated: 2026-04-11

## Overview

Shared reference data is stored in the `autostore_platform` database on the remote backend server. All Mac client users access this data via API — no local files needed.

This replaces the previous approach of bundling JSON/CSV files in each project repo.

## Tables

### `ref_ae_categories` — AliExpress Leaf Categories (623 entries)

Source: `documents/aliexpress-store/ae-categories-tiered.json`

| Column | Type | Description |
|--------|------|-------------|
| id | INT PK | Auto-increment |
| sheet_name | VARCHAR | Category sheet name (e.g., "QuartzWristwatches") |
| l1 | VARCHAR | Level 1 category (e.g., "Watches") |
| l2 | VARCHAR | Level 2 category (e.g., "Quartz Watches") |
| l3 | VARCHAR NULL | Level 3 category (optional) |
| display_path | VARCHAR | Human-readable path ("Watches > Quartz Watches") |
| template_file | VARCHAR NULL | Path to Excel template |
| has_size | BOOL | Whether category has size variants |
| match_source | VARCHAR | How category was matched: tree_match, manual, pattern, etc. |

**API**: `GET /api/reference/ae-categories?l1=Watches`

### `ref_brands` — Banned Brand List (357 entries)

Source: `1688_scrapper/src/data/brands-comprehensive-2026-03-22.csv`

| Column | Type | Description |
|--------|------|-------------|
| id | INT PK | Auto-increment |
| brand_name_en | VARCHAR | English brand name (e.g., "Rolex") |
| brand_name_zh | VARCHAR NULL | Chinese brand name (e.g., "劳力士") |
| category | VARCHAR | Product category (watches, jewelry, bags, etc.) |
| risk_level | VARCHAR | critical, high, medium, low |
| aliases | TEXT NULL | Alternative names, pipe-separated |
| notes | TEXT NULL | Context about the brand |
| active | BOOL | Whether brand is actively blocked |

**API**:
- `GET /api/reference/brands?category=watches` — list by category
- `GET /api/reference/brands/search?q=rolex` — search by name

### `ref_blue_ocean_categories` — Blue Ocean Categories (446 entries, 136 brand-safe)

Source: `1688_scrapper/src/data/blue-ocean-search-terms.json`

| Column | Type | Description |
|--------|------|-------------|
| id | INT PK | Auto-increment |
| name | VARCHAR UNIQUE | Category key (e.g., "HairClaws") |
| l1 | VARCHAR | Level 1 grouping (e.g., "Hair Accessories") |
| search_terms_zh | JSON | Chinese search terms for 1688 (e.g., ["鲨鱼夹"]) |
| enabled | BOOL | Whether category is active for scraping |
| disabled_reason | TEXT NULL | Why disabled (if applicable) |
| brand_safe | BOOL | Whether category is structurally brand-safe |
| target_platform | VARCHAR NULL | Platform-specific targeting |

**API**:
- `GET /api/reference/blue-ocean` — all categories
- `GET /api/reference/blue-ocean?brand_safe=true` — brand-safe only (136 entries)

## Stats Endpoint

`GET /api/reference/stats` returns:
```json
{
  "ae_categories": 623,
  "brands": 357,
  "blue_ocean": {
    "total": 446,
    "brand_safe": 136,
    "enabled": 117
  }
}
```

## Seeding

Reference data is seeded from local files using:
```bash
cd backend/
npx ts-node src/reference/seed-reference-data.ts
```

This reads from:
- `documents/aliexpress-store/ae-categories-tiered.json`
- `1688_scrapper/src/data/brands-comprehensive-2026-03-22.csv`
- `1688_scrapper/src/data/blue-ocean-search-terms.json`

Re-running the seed script **replaces** all existing data (truncate + insert).

Environment variables for the seed script:
- `API_URL` — backend URL (set to your deployed backend's `/api` path)
- `SEED_EMAIL` — login email (default: `test@autostore.com`)
- `SEED_PASSWORD` — login password (default: `test123456`)

## How Clients Use This Data

1. **Mac app setup wizard** → fetches blue ocean categories for the "类目管理" settings page
2. **Pipeline tasks** → will query brands and categories from the API instead of reading local JSON/CSV
3. **LLM chat** → system prompt can reference category counts and brand list dynamically

## Updating Reference Data

To add new brands or categories:
1. Edit the source files (CSV/JSON) locally
2. Re-run the seed script
3. All clients get the updated data immediately via API

No client app update needed — the backend is the single source of truth.
