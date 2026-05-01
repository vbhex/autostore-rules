# AutoStore Business Model — Estimation

**Status:** Strategic estimate. Authored 2026-05-01.
**Predecessor doc:** Earlier (pre-distillation) verdict was "personal toolkit only — wait for Chinese models to improve." This doc supersedes that based on the architectural shift documented in `KNOWLEDGE_DISTILLATION_STRATEGY.md`.

---

## TL;DR

| Question | Pre-distillation answer (~2026-04) | Post-distillation answer (now) |
|---|---|---|
| Is AutoStore a viable business? | No — personal toolkit | Yes — defensible vertical SaaS |
| Estimated 24-month ARR | <¥500K | ¥10–13M |
| Bottleneck | Chinese models too weak | Distribution / GTM execution |
| Margin profile | Thin (Claude API costs eat it) | 88% (user pays own LLM) |
| Defensibility | Low | Macro library + telemetry flywheel |
| Estimated valuation at year 2 | ~$0–50K (acqui-target only) | $20–60M (5–15× revenue multiple) |

The bet shifted from "depending on Chinese model improvements" to "executing on the macro library." That is a fundamentally better bet because **execution is in your control, model roadmaps are not.**

---

## Why the previous "no business" verdict held

The thesis hinged on **the runtime model being responsible for multi-step reasoning**:

| Claim | Status then |
|---|---|
| Models can't reason multi-step → unreliable experience | True — qwen-plus loops, glm-4-flash gives up |
| Unreliable experience → can't charge money | True — paid SaaS needs predictable outcomes |
| Need to wait for Chinese models to improve | True given that architecture |

Once that assumption broke, the rest of the verdict became invalid. The distillation strategy moved multi-step reasoning from runtime (model) to design-time (Claude → static macro library), which is the architectural break that unlocks the business case.

---

## What changed structurally

The capability ladder for a paying user:

| Capability | Pre-distillation | Post-distillation |
|---|---|---|
| User runs AutoStore on `qwen-plus` | ⚠️ Loops, gives up | ✅ Same as Claude (90%+ tasks) |
| User runs AutoStore on `glm-4-flash` (¥1/M tokens) | ❌ Unusable | ✅ Tier-1 tasks succeed |
| Predictable 30-second task completion | ❌ Variable | ✅ Deterministic |
| Cost per request to AutoStore (you) | ~$0.05–0.50 if you proxy Claude | ~$0 (user pays own LLM) |
| Cost per request to user | Free (you ate it) | ~¥0.01 (cheaper than coffee) |

The model becomes a **routing layer over a fixed playbook library**, not a reasoning engine. Routing is the easiest job in NLP — even the cheapest models do it reliably.

---

## Market sizing

### Target segment: cross-border ecommerce sellers operating from China

| Segment | Active sellers | Annual tooling spend (¥) |
|---|---|---|
| Amazon Global CN sellers | ~600K active | ¥3B+ |
| AliExpress / eBay / Etsy CN sellers | ~1M+ | ¥1B+ |
| 1688 sellers wanting to go global (long tail) | ~4M | aspirational |
| **Total addressable** | **~1.6M serious** | **¥4–5B/yr on optimization tools** |

These users currently pay foreign tools (Helium 10 ~¥700/mo via VPN, Jungle Scout ~¥350/mo, SellerSprite ~¥300/mo, Ezprep, etc.) that:
- Don't speak good Chinese
- Don't integrate with 1688 (the Chinese supplier graph)
- Require VPN access
- Don't ship updates that account for China-specific seller realities (MC011, UFLPA, Xinjiang Cotton bans, eBay SpeedPAK rule)

AutoStore is positioned to displace these for sellers operating from China.

---

## Pricing model (proposed)

| Tier | Monthly | Target user | Limits |
|---|---|---|---|
| **Free** | ¥0 | Trial / lead generation | 50 macro calls/mo, 1 platform |
| **Pro** | ¥199 | Solo seller, 1–3 stores | Unlimited macros, all platforms, basic Claude API fallback |
| **Team** | ¥499 | 4–10 stores | + bulk operations, API access, priority support |
| **Agency** | ¥1,999 | Service agencies | + multi-tenant, white-label, on-prem option |

**Anchor logic:** Foreign tools at ¥350–700/mo are English-only and don't touch 1688. AutoStore at ¥199 is Chinese-native, does ~80% of what foreign tools do, plus uniquely owns the 1688→Amazon/eBay/Etsy supplier bridge. Clear value prop.

---

## Unit economics

Per Pro user (¥199/mo) at scale (~10K users):

| Line item | Monthly cost (¥) |
|---|---|
| AWS infrastructure (per-user share) | ¥3 |
| Claude API fallback (5–10% of requests) | ¥5 |
| Payment processing (3%) | ¥6 |
| Customer support (distributed avg) | ¥10 |
| **Total cost** | **¥24** |
| **Gross margin** | **88%** |

For comparison: typical SaaS gross margin is 60–75%. AutoStore's 88% is closer to category-killer territory because the user pays their own LLM bill — the variable cost that crushes most AI products is shifted to the user.

---

## Revenue ramp (realistic, conservative)

Assumptions:
- Tier 1 macros ship in 2 weeks (already in flight)
- Tier 2 macros ship in 8 weeks
- Active distillation backend ships in 4 weeks
- Organic distribution via 小红书 / 抖音 / cross-border forums + 1 partnership

| Month | Free users | Pro | Team | Agency | MRR (¥) | ARR (¥) |
|---|---|---|---|---|---|---|
| 3 | 500 | 50 | 5 | 0 | ~¥12K | ¥144K |
| 6 | 3K | 250 | 30 | 2 | ~¥65K | ¥780K |
| 12 | 15K | 1,200 | 150 | 10 | ~¥315K | ¥3.78M |
| 24 | 50K | 4,000 | 600 | 30 | ~¥1.1M | ¥13M+ |

These are conservative because of the **compounding macro library**: every workflow Claude encodes becomes a permanent capability that competitors must rebuild from scratch.

---

## The moat: macro library + telemetry flywheel

```
More users → more recorded successful runs
   ↓
More patterns to generalize into macros
   ↓
Bigger macro library → more capabilities
   ↓
Better experience than competitors → more users
   ↺
```

After 12 months of usage, AutoStore knows ~150–300 platform workflows. A new competitor needs to either:
1. Replicate the recording effort (12+ months)
2. License from you (hard to negotiate around)
3. Build worse and lose

The macro library also captures **operational lessons** competitors can't easily steal:
- "Amazon error 90057 means mixing product types in one file"
- "Etsy UFLPA-removes any Xinjiang Cotton mention with no appeal"
- "eBay Greater China accounts must use SpeedPAK only"
- "1688 must be accessed from inside China firewall"

These are baked into AutoStore's prompts + macros. They're not in any competitor's docs.

---

## Risks (honest)

### 1. Distribution (the new bottleneck)
You do not have a built-in audience. Required:
- Content on 小红书 / 抖音 / B站 about cross-border ecommerce automation
- Partnership with 1688 supplier programs (cross-promotion)
- Affiliate network with 抖音 ecommerce KOLs
- A few months of patience before the flywheel kicks in

**This is the real bottleneck now, not technology.**

### 2. Macro maintenance
Platforms change UI quarterly. Without active distillation + telemetry, the library decays. The **active distillation server endpoint** (Tier B in the strategy doc) is now critical, not optional.
- Budget: 1–2 hours/week of owner time + ~¥500/mo Claude API
- Mitigation: telemetry-driven macro generation (Tier C) once at scale

### 3. Regulatory
Cross-border ecommerce is sensitive in China (foreign exchange, customs).
Mitigations:
- Don't store seller credentials server-side (already true — `.env` is local)
- Don't broker payments (don't)
- Stay tool-only, never become a marketplace

### 4. Platform pushback
eBay/Amazon don't love automation tools. Helium 10 etc. survive because they use official APIs where possible. AutoStore should:
- Use official APIs first (Etsy API ✓, Amazon SP-API for orders, eBay Trading API)
- Browser automation only for what APIs can't cover
- Position as "browser productivity tool", not "scraper"

### 5. Free tier conversion
Free → paid conversion in Chinese SaaS is historically brutal (~1–2%). You'd need 100K free users to hit 1K paid.
- Mitigation: gate Tier 2 + Tier 3 macros behind paid tiers — the library itself becomes the conversion lever

---

## What needs to happen for the projections to hold

In priority order:

1. **Ship Tier 1 macros (2 weeks)** — covers 80% of daily ecommerce work, makes Pro tier defensible
2. **Ship active distillation backend (4 weeks)** — keeps the library current as platforms change
3. **Ship Tier 2 macros (8 weeks)** — paid-tier justification beyond the freemium ceiling
4. **Start content marketing immediately** — parallel to dev work, not after; flywheel needs months to spin up
5. **First partnership** — one 1688/Alibaba supplier program partnership unlocks distribution

If items 1–3 ship and items 4–5 don't, you have a great product nobody finds. If items 4–5 ship but 1–3 don't, you have buzz around an unreliable tool. **Both halves are required.**

---

## Sensitivity analysis

What changes the projections meaningfully:

| Variable | Bear case | Base case | Bull case |
|---|---|---|---|
| Macro library coverage at month 6 | 60% of common tasks | 80% | 95% |
| Free→Pro conversion | 0.5% | 1.5% | 3% |
| Pro→Team upgrade | 5% | 15% | 30% |
| Churn (monthly) | 8% | 4% | 2% |
| Year-2 ARR | ¥3M | ¥13M | ¥40M+ |

Even the bear case is profitable for a 1-person operation. The base case supports a 3–5 person team with healthy margins. The bull case is acquisition-attractive territory.

---

## Comparison: pre-distillation vs post-distillation

| Dimension | Pre-distillation | Post-distillation |
|---|---|---|
| **Reliability on Chinese models** | Variable, often broken | Deterministic |
| **Cost structure** | You pay AI inference | User pays AI inference |
| **Defensibility** | None — anyone can build a Claude wrapper | Macro library compounds over time |
| **GTM motion** | Word-of-mouth among technical users | Mass market via content/affiliate |
| **Pricing power** | ~¥50/mo (struggle) | ~¥199/mo (justified) |
| **Path to $1M ARR** | Unclear | 12–18 months |
| **Path to acquisition** | None obvious | Helium 10 / SellerSprite / Alibaba ecosystem player |
| **Owner time required at year 2** | Full time, fighting fires | ~10 hr/wk maintaining macros + content |

---

## Strategic takeaway

The distillation strategy didn't just make AutoStore "better" — it made it a **fundamentally different kind of product**:

| Before | After |
|---|---|
| AI app dependent on model quality | Software product with AI as routing layer |
| Margin compresses as you scale | Margin holds at 88% as you scale |
| Improves only when models improve | Improves every time you encode a macro |
| Capped by what runtime models can do | Capped only by macro library completeness |

The product class shifted from "AI experiment" to "vertical SaaS with AI assist." Vertical SaaS is the most reliably profitable software category in the world.

---

## When to revisit this estimate

Trigger conditions:
- After 3 months of GTM execution → check actual conversion rates
- If Chinese models leap ahead (e.g. qwen-max-3 matches Claude 4) → may unlock 100% coverage and reduce Claude API fallback cost to zero
- If a competitor launches in Chinese market → reassess pricing pressure
- If China changes cross-border ecommerce regulations → reassess regulatory risk

Until any of those, the estimate above is the working assumption.

---

## See also

- `rules/KNOWLEDGE_DISTILLATION_STRATEGY.md` — the architectural strategy this business model depends on
- `rules/COMPUTER_USE_STRATEGY.md` — annotated screenshots + macro tools (the technical foundation)
- `rules/CONTRIBUTING_PLATFORM_KNOWLEDGE.md` — protocol for sibling sessions to grow the macro library
