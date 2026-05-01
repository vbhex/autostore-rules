# AutoStore Knowledge Distillation Strategy

**Status:** Active. Adopted 2026-05-01.
**One-line summary:** Use Claude (offline) to encode every ecommerce workflow into deterministic macro tools, so weak models (qwen-plus, glm-4-flash, deepseek-chat) can run all of AutoStore by doing nothing more than intent classification.

---

## The Theorem

> Anything Claude can do step-by-step on a known platform can be **encoded** as a deterministic sequence of tool calls.
> Once encoded, the weak model only needs to do **intent matching** — a one-shot classification problem that even the cheapest models handle reliably.

The hard part for weak models is **multi-step reasoning** (plan 10 steps ahead, recover from errors, parse unstructured pages). The easy part is **routing** ("user wants X" → "call macro Y"). We push all the hard work into design-time (Claude's job, once per workflow) and leave the easy part for runtime (the user's chosen model, every request).

---

## Why this matters

AutoStore's target users are in mainland China and have no access to Claude. They use Chinese LLMs:

| Model | Tool-call quality | Reasoning | Suitable for AutoStore? |
|---|---|---|---|
| Claude Sonnet/Opus | Excellent | Excellent | ✅ Reference standard |
| GPT-4o | Excellent | Excellent | ✅ Excellent |
| qwen-max | Good | Good | ✅ Works with current macros |
| deepseek-chat | Good | OK | ✅ Works with current macros |
| qwen-plus | OK | Weak | ⚠️ Needs more macros |
| qwen3-* (thinking) | OK | Drifts into prose | ⚠️ Bail-out detector helps |
| glm-4-flash | Weak | Weak | ❌ Without macros: useless |
| qwen-turbo | Weak | Weak | ❌ Without macros: useless |

**Goal:** With a complete macro library, every model in this table reaches the same capability level. The user's choice becomes about cost and speed, never capability.

---

## The architecture

### Offline (design-time, with Claude)

```
Workflow recording loop:
  1. Claude (in a session) solves a real task end-to-end
  2. Claude reviews the tool sequence used
  3. Claude generalizes it into a parameterized macro
  4. Claude writes:
       a) New static method in PlatformKnowledge.swift
       b) New tool definition in LLM_TOOLS (LocalLLMService.swift)
       c) New case branch in ChatView.swift's tool dispatcher
       d) New rules in the LLM_PROMPT_<PLATFORM> system prompt section
  5. Claude commits + pushes
  6. Next AutoStore release ships the new macro
```

### Online (runtime, with weak model)

```
User: "polish my Amazon listings for low-conversion ASINs"
   ↓
Weak model:
  - Identifies platform: amazon
  - Identifies intent: polish listings
  - Calls polish_amazon_listings(filter="low_conversion")
   ↓
AutoStore:
  - Reads the recorded sequence
  - Executes deterministically: login check → filter listings → fetch each → score → rewrite → submit
  - Returns structured summary
   ↓
Weak model:
  - Reports back: "polished 14 listings, 3 needed manual review"
```

The weak model's **only** decisions: which macro + which arguments. That's it.

---

## What ships today (the current distilled assets)

| Layer | What's encoded | Where it lives |
|---|---|---|
| **URL registry** | Every platform's URL for orders/listings/upload/messages/violations/etc. | `PlatformKnowledge.swift` → `PlatformURL` |
| **Page extractors** | JS that parses each platform's order tables, listing tables, message inboxes | `PlatformKnowledge.swift` → `PlatformExtractor` |
| **Login macros** | Multi-step login per platform with `.env` credentials | `PlatformKnowledge.swift` → `sellerLogin()` + `seller_login` tool |
| **Navigation macro** | Navigate + extract + annotated screenshot, atomic | `PlatformKnowledge.swift` → `run()` + `platform_go` tool |
| **Image injection** | Amazon shadow-DOM file upload via DataTransfer | `PlatformKnowledge.swift` → `amazonInjectImageJS()` |
| **Platform rules** | Error codes (Amazon 90008/90057/99006), banned phrases (Etsy Xinjiang), forbidden categories (Amazon earphones, all-platform power banks) | `LocalLLMService.swift` → `LLM_PROMPT_<PLATFORM>` sections |
| **Bail-out detector** | Patterns that indicate the model is asking instead of acting | `LocalLLMService.swift` → `looksLikeBailout` |
| **Annotated screenshots** | Numbered overlay on every interactive element + index of (label, x, y) | `in-chrome/extension/service-worker.js` |
| **Page action menu** | Structured list of clickable elements + form fields, returned with every `browser_go` | `ChatView.swift` → `extractActionMenu()` |

---

## What's missing — the macro roadmap

### Tier 1 — Daily ecommerce (target: ship in 2 weeks)

| Macro | Replaces | Platform |
|---|---|---|
| `bulk_revise_ebay_prices(file)` | upload CSV → handle errors → confirm batch | eBay |
| `mark_ebay_dispatched(orderIds)` | per-order ship + tracking entry | eBay |
| `polish_amazon_listing(asin)` | fetch → analyze → rewrite title/bullets → submit | Amazon |
| `audit_suppressed_amazon_listings()` | iterate suppressed list → identify cause → queue fixes | Amazon |
| `optimize_etsy_seo(listingId)` | fetch listing → keyword analysis → rewrite title/tags → save draft | Etsy |
| `create_etsy_draft(productId)` | pull from `etsy_autostore` → API createDraft → upload images | Etsy |
| `send_1688_brand_inquiry(supplierId, brand)` | open chat → send templated msg → log outreach | 1688 |
| `find_supplier_brand_authorization(brand)` | search for sellers → filter authorized → return list | 1688 |
| `check_brand_safety(text)` | run isBannedBrand against title/desc/specs | 1688/all |
| `respond_to_amazon_violation(notificationId)` | open notice → identify type → queue draft response | Amazon |

### Tier 2 — Compliance & recovery (target: 4 weeks)

| Macro | Purpose |
|---|---|
| `appeal_etsy_takedown(listingId)` | full appeal flow with template |
| `upload_ebay_mc011_docs(file)` | doc upload + reply trigger |
| `request_authorization_doc(supplierId, brand)` | Wangwang outreach for 品牌授权书 |
| `diagnose_amazon_listing_issue(asin)` | check suppressed/inactive/policy state |
| `bulk_delist_aliexpress(productIds)` | atomic delist sequence (already handled by store closure but kept for reference) |
| `health_check_all_platforms()` | one call returns: # active listings, # unshipped orders, # new violations, # unread messages, per platform |

### Tier 3 — Discovery & analytics (target: 8 weeks)

| Macro | Purpose |
|---|---|
| `find_competitive_gap(category)` | compare our listings vs top sellers in category |
| `analyze_listing_conversion(asin)` | pull metrics → identify conversion blockers |
| `keyword_reverse_engineer(asin)` | scrape top 20 competitors → extract keyword density |
| `price_check(productId)` | cross-platform price comparison |

---

## How sibling sessions contribute (the recording protocol)

When a Claude session solves a workflow that doesn't have a macro yet, it MUST encode it before ending. The protocol:

### 1. Identify the gap
Did you spend more than 3 tool calls on something a single macro could have done? → encode it.

### 2. Pick the right home
Read `rules/CONTRIBUTING_PLATFORM_KNOWLEDGE.md` for the file/section mapping.

### 3. Write all four pieces
1. Static method on `PlatformKnowledge` — the actual sequence
2. Tool definition in `LLM_TOOLS` — the LLM-facing description
3. Case branch in `ChatView.swift` — the dispatcher
4. System prompt rule — when the model should call it

### 4. Test once, commit, push
Run the macro once to verify. Then `git push` from the project repo so the next AutoStore build includes it.

### 5. Self-document in your final reply
List exactly what you added so the human owner knows what shipped.

---

## Three ways to scale the recording phase

### A. Manual, by sibling sessions (active now)
Each session encodes what it learns. Slow but honest. **15 macros in 2 weeks** is the working target.

### B. Active distillation (planned)
When the weak model fails to route a request to any existing macro, AutoStore's backend pings the Claude API server-side. Claude generates a one-shot sequence; AutoStore caches it keyed by request hash. Future identical requests hit the cache.

- **Cost:** ~$5–10/month of Claude API across the user base
- **Convergence:** After ~3 months, the cache covers >95% of requests; Claude API spend drops to near-zero
- **Implementation:** new endpoint in `backend/`, new `cached_macros` table, fallback path in `LocalLLMService`

### C. Replay from observed runs (long-term)
Every successful tool sequence is logged. A nightly job:
1. Clusters similar sequences
2. Asks Claude to generalize each cluster into a parameterized macro
3. Auto-registers the macro after a human review step

End state: AutoStore self-extends its macro library from real usage telemetry.

---

## The honest limitation

This strategy works for **deterministic workflows**. It does NOT work for tasks needing genuine creativity:

- "Write a friendly response to this angry customer"
- "Suggest a price for a brand-new product category"
- "Decide which of these 50 products to delist"

For those tasks, the user's chosen model (or a Claude API fallback) does the actual work. But:

- Creative tasks are <10% of ecommerce
- Routing them to a paid API is acceptable (rare, low volume)
- Most "ecommerce" is repetitive and deterministic — that's exactly the territory this strategy owns

---

## How to measure success

| Metric | Target |
|---|---|
| % of user requests routed to a macro (not raw browser_*) | >80% |
| Average tool calls per user request | <3 |
| Bail-out detector activations per user session | 0 |
| Tasks executable on `glm-4-flash` (cheapest model) | 100% of Tier 1 |
| Claude API cost per active user per month | <$0.50 |

When all five hit target: **AutoStore is fully model-agnostic for ecommerce.**

---

## Adoption checklist for new sessions

Read this whenever you start a new Claude session in this workspace:

- [ ] You are a knowledge contributor, not just a task executor
- [ ] If you spend 3+ tool calls on something repeatable, encode it as a macro before ending
- [ ] Read `rules/CONTRIBUTING_PLATFORM_KNOWLEDGE.md` for where each kind of knowledge belongs
- [ ] Read `rules/COMPUTER_USE_STRATEGY.md` for why annotated screenshots and macros matter
- [ ] Commit + push your contributions to the relevant repo (mac/, in-chrome/, rules/)
- [ ] In your final reply, list what you encoded so the human owner can verify

---

## See also

- `rules/COMPUTER_USE_STRATEGY.md` — annotated screenshots, page action menu, why generic computer-use fails with weak models
- `rules/CONTRIBUTING_PLATFORM_KNOWLEDGE.md` — exact file/section mapping for each kind of knowledge
- `rules/Tasks.md` — the canonical pipeline reference (1688 source pipeline, Tasks 1–12)
- `mac/AutoStore/Sources/Services/PlatformKnowledge.swift` — the live macro registry
- `mac/AutoStore/Sources/Services/LocalLLMService.swift` — system prompt + tool definitions + bail-out detector
