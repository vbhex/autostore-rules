# AutoStore Rules

Shared, public knowledge for the AutoStore platform — read by autonomous Claude sessions and contributors. Updated continuously as macros land, platforms change, and the strategy evolves.

## What lives here

| Doc | What it covers |
|---|---|
| `KNOWLEDGE_DISTILLATION_STRATEGY.md` | The core thesis: encode every workflow as a deterministic macro so weak quantized models only need intent matching. Master plan + macro roadmap. |
| `CONTRIBUTING_PLATFORM_KNOWLEDGE.md` | Where each kind of knowledge belongs in the Mac source tree (URLs / extractors / system-prompt rules / multi-step workflows). |
| `COMPUTER_USE_STRATEGY.md` | Why generic computer-use fails for quantized models and how annotated screenshots + macro tools fix it. |
| `LOCAL_MODEL_STRATEGY.md` | On-device inference architecture: Qwen 2.5 7B Instruct Q4_K_M via llama.cpp + Metal GPU on the user's Mac. |
| `BUSINESS_MODEL.md` | Revenue projections + market sizing under the distilled architecture. |
| `MULTI_PLATFORM_CATEGORY_STRATEGY.md` | Which categories ship to which platforms — 3 active (Amazon / eBay / Etsy), AliExpress closed 2026-03-25. |
| `BRAND_SAFE_CATEGORIES.md` | The Phase-1 brand-safe matrix (20 category groups, structurally impossible to have IP issues). |
| `CLIENT_SETUP_ARCHITECTURE.md` | Server-driven first-launch setup: manifest endpoint, dependency checks, schema initialization. |
| `REFERENCE_DATA.md` | Central tables served via `/api/reference/*` — AE categories, brand DB, blue-ocean search terms. |
| `Tasks.md` | Full reference for all 15 pipeline tasks (Task 1–12 + utilities) including the Mac-client downstream tasks. |
| `Pre-Training-Sessions.md`, `training.txt` | Distilled session learnings + raw training data for future fine-tuning. |
| `1688-source/` | 1688-specific rules (blue-ocean discovery, English-mode scraping). |
| `aliexpress-store/` | AliExpress historical rules — store closed 2026-03-25. Kept for traceability. |
| `amazon-store/` | Amazon strategy (3C automated discovery + manual seller sourcing). |
| `ebay-store/` | eBay strategy, seller constraints, shipping-fee strategy. |
| `etsy-store/` | Etsy strategy. |
| `claude-skills/` | Skills that ship to client-side autonomous Claude sessions. |

## Current state (2026-05-13)

- **Active platforms**: Amazon, eBay, Etsy
- **Sourcing platform**: 1688 (China firewall — only the China MacBook accesses it)
- **Closed**: AliExpress store `$ALIEXPRESS_STORE_ID` (since 2026-03-25)
- **On-device model**: Qwen 2.5 7B Instruct Q4_K_M (~4.4 GB, Metal GPU)
- **Macro library**: 30+ deterministic `PlatformTask` cases + 3 `.staticAnswer` macros
- **Bench coverage**: 150+ regression cases in the in-app `BenchView` harness (Cmd+B)

## How to contribute

The autostore-mac repo has a `CHANGELOG.md` per release. Use it. When you encode a new macro:
1. Add the URL in `mac/AutoStore/Sources/Services/PlatformKnowledge.swift` → `PlatformURL`.
2. Add a `PlatformTask` enum case if it's a new shape.
3. Wire `PlatformURL.url(platform:task:)`.
4. Add a router rule in `mac/AutoStore/Sources/Services/IntentRouter.swift`.
5. Add 1–3 bench cases in `BenchView.swift`.
6. Build, commit, push, notarize, ship.

See `CONTRIBUTING_PLATFORM_KNOWLEDGE.md` for the full protocol.
