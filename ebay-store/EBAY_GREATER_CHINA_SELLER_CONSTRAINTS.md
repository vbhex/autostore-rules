# eBay Greater China Seller Constraints — Operational Rules

**Applies to**: All AutoStore eBay seller accounts registered as Greater China (China mainland, Hong Kong, Taiwan, Macau) WITHOUT a dedicated eBay account manager.

**Status**: ACTIVE — confirmed 2026-04-17 on a live Greater China seller account. These constraints apply universally to any Greater China seller without account manager enrollment.

---

## Rule 1 — SpeedPAK-Only Shipping (CRITICAL)

Greater China seller accounts can **ONLY** use eBay SpeedPAK-branded shipping services. Any listing or policy update that tries to use a non-SpeedPAK service is blocked by eBay.

### Allowed shipping services

| Service | Delivery | Use case |
|---------|----------|----------|
| **eBay SpeedPAK Standard** | 5–12 business days | Faster option, US + worldwide |
| **eBay SpeedPAK Economy** | 6–15 business days | Cheaper alternative, US + worldwide |

### Blocked shipping services

| Service | Reason |
|---------|--------|
| Economy Shipping from Greater China to worldwide | Not SpeedPAK |
| Standard Shipping from Greater China to worldwide | Not SpeedPAK |
| Third-party direct (DHL, FedEx, UPS) | Not SpeedPAK |

### What eBay shows when blocked

At listing submission:
> *"According to our Direct shipping service standard policy, you are only allowed to use SpeedPAK-related shipping options. To continue, select the appropriate shipping method for your listing."*

In bulk policy update activity log (misleading generic error):
> *"The item cannot be listed or modified. The title and/or description may contain improper words, or the listing or seller may be in violation of eBay policy."*

**Important**: This "improper words" message is NOT a content moderation issue. It is the generic bulk-edit error when the new policy contains a disallowed shipping service. Always check the service name first.

### Recommended shipping policy config

**Domestic (US):**
- Service: eBay SpeedPAK Standard, 5–9 business days
- Cost: $0 (Offer free shipping enabled)

**International:**
- Service: eBay SpeedPAK Economy (or SpeedPAK Standard)
- Cost type: Flat, same cost to all buyers
- First item: $5.00 USD
- Each additional: $2.00 USD
- Ships to: Worldwide
- Account-level excludes: Russia, Ukraine (cannot be removed)

---

## Rule 2 — High-Risk Category Blocks

Certain eBay categories are blocked for Greater China sellers without an account manager. Error shown:

> *"You're not allowed to list items in this high-risk category. This restriction is in place because you currently do not have a dedicated account manager service."*

### Confirmed BLOCKED categories

| Category ID | Name | Context |
|-------------|------|---------|
| 45220 | Hair Accessories | all sub-types blocked |
| 155189 | Unisex Sunglasses | |
| 79720 | Men's Sunglasses | all sunglass sub-categories appear blocked |

### Confirmed WORKING categories

| Category ID | Name | Path |
|-------------|------|------|
| 155101 | Fashion Necklaces & Pendants | Jewelry & Watches > Fashion Jewelry |
| 167902 | Handkerchiefs & Pocket Squares | Clothing, Shoes & Accessories > Men > Men's Accessories |
| 261986 | Body Jewelry | Jewelry & Watches > Fashion Jewelry |
| 50647 | Fashion Earrings | Jewelry & Watches > Fashion Jewelry |
| 261993 | Necklaces & Pendants (Fine Jewelry) | Jewelry & Watches > Fine Jewelry |
| 15662 | Men's Ties | Clothing, Shoes & Accessories > Men > Men's Accessories |

### How to handle a blocked category

1. Test-list ONE product before filling out an entire listing form.
2. If blocked, skip the product; do not attempt workarounds.
3. Add the blocked category ID to this doc so other users know.

---

## Rule 3 — Visual Brand Inspection (Image Check)

1688 supplier titles hide famous-brand counterfeits behind vague descriptors. The `isBannedBrand()` title filter is **not sufficient**. Always inspect product images before listing.

### Title phrases that hint at a brand copy

- "European and American [Trendy/Big/Famous] Brand"
- "High-End Version of [vague name]"
- "Phantom" / "Original Business" / "Authentic Style"
- "Empress / Empress Dowager" (sometimes = Vivienne Westwood copy)
- "Gracia" / "Lola" / "Agete" (Japanese jewelry brand copies)

### Concrete example (2026-04-17)

Product 78: "European and American Trendy Brand Silk Tie, Narrow British Tie, Formal Tie, Wedding and Work Tie, Original Business Tie"

- Title brand check: PASSED (no famous brand in title)
- Image check: Main image had Gucci GG monogram + Gucci green packaging background
- Action: **Aborted listing**. Gucci is on the must-remove famous brand list.

### Workflow

1. After selecting a product, query its `products_images_ok.image_url` rows.
2. Before filling the listing form, visually inspect the main + secondary images.
3. Watch for: monogram patterns, brand logos, trademark colors/shapes.
4. If ANY famous-brand element is visible, skip and pick a different product.

### Famous brands to detect by sight

Nike • Gucci • Louis Vuitton • Burberry • Chanel • Hermès • Dior • Tiffany & Co • Apple • Samsung • Sony • Rolex • Ferrari • Lamborghini

---

## Rule 4 — Default Excluded Countries

All Greater China seller listings automatically exclude:
- Russian Federation
- Ukraine

These are account-level defaults set by eBay. You cannot remove them from individual listings.

---

## Rule 5 — Bulk Policy Update Fallback Pattern

When a shipping policy is edited while bound to active listings, the following can happen:

1. The policy entity saves successfully (updated version exists).
2. eBay tries to apply the new policy to all bound listings.
3. If ANY listing can't accept the new policy (e.g., disallowed service), ALL of those listings get reassigned to an auto-generated **"Copy" of the policy with PREVIOUS settings**.
4. The original policy becomes empty (0 listings) or shows "Errors found".
5. Listings continue working on the old (now-copied) policy.

### How to recover

1. Fix the root cause in the original policy (e.g., switch back to allowed SpeedPAK service).
2. Go to Seller Hub → Listings → Active listings → bulk select all.
3. Edit → Bulk edit → Shipping policy → "Change to" → pick the fixed original policy.
4. Submit. The bulk revise should now succeed.

### Positive confirmation signal for international listings

When a listing is configured to ship to EU/UK, the eBay listing form adds an **"ITEM DISCLOSURES"** section requiring product safety/quality info. Seeing this section = worldwide shipping is active.

---

## Rule 6 — Account Manager Enrollment Path

To lift all the above restrictions (high-risk categories + non-SpeedPAK shipping + related constraints), a Greater China seller must enroll in the **eBay Greater China Enterprise Platform**:

🔗 https://emp.ebay.com.hk/

**Decision criteria**: only worth the overhead when the store reaches significant volume. For new seller accounts, operate within SpeedPAK + unblocked-categories + brand-safe mode.

---

## References

- `ebay/CLAUDE.md` — project-level Critical Rules (short form)
- `documents/ebay-store/EBAY_SOURCING_STRATEGY.md` — demand-first sourcing playbook
- Operational memory (session-local, per Claude instance):
  - `feedback_ebay_speedpak_only.md`
  - `feedback_ebay_high_risk_categories.md`
  - `feedback_ebay_inspect_images_for_brands.md`

**This document is canonical.** Update here first when discovering new constraints; then reference it in project CLAUDE.md and per-session memory.
