# Contributing Platform Knowledge to AutoStore

**Read this before adding anything.** AutoStore's design principle:
> The user's LLM only identifies intent. AutoStore's hardcoded knowledge handles the "how."

When you (a Claude session) discover something specific about a platform's UI, format, or behavior, that knowledge needs to flow back into AutoStore so the next user doesn't re-discover it. This file tells you where each kind of knowledge belongs.

---

## What counts as "platform knowledge"?

Anything that is **stable across users and time** about a specific platform:

| Knowledge type | Example |
|---|---|
| **Exact URLs** | "Amazon orders are at sellercentral.amazon.com/orders-v3" |
| **Page extractors** | "eBay listing table rows have class `listing-row`" |
| **Format rules** | "Amazon flat file requires `attributeRow=3&dataRow=4`" |
| **Required fields** | "HEADPHONES product type needs `headphones_form_factor`" |
| **Forbidden patterns** | "NEVER mention 'Xinjiang Cotton' on Etsy" |
| **Error code meanings** | "Amazon error 90057 = mixing product types in one file" |
| **Workflow sequences** | "eBay bulk upload: `/sh/lst/import` → upload CSV → wait for batch ID" |
| **Login/auth quirks** | "1688 must be accessed from China MacBook (foreign IP triggers captcha)" |

**Not platform knowledge:**
- One-off product fixes (those go to the relevant store DB)
- User preferences (those go to user memory)
- Code refactors (those go to the project README)

---

## Where each kind of knowledge goes

### 1. URLs → `mac/AutoStore/Sources/Services/PlatformKnowledge.swift` → `enum PlatformURL`

```swift
// Add a new constant in the right platform section:
static let amazonReturns = "https://sellercentral.amazon.com/returns/list"

// And register it in PlatformURL.url(platform:task:):
case (.amazon, .returns): return amazonReturns
```

If it's a brand-new task type that isn't in `enum PlatformTask`, add the case there too.

### 2. Page extractors → `PlatformKnowledge.swift` → `enum PlatformExtractor`

If the platform has a parseable order/listing/inventory table, write a JS snippet that returns structured data:

```swift
static let amazonReturns = """
(() => {
    const rows = document.querySelectorAll('.return-row');
    return { count: rows.length, returns: Array.from(rows).map(r => ({...})) };
})()
"""
```

Then route it in `PlatformExtractor.extractor(platform:task:)`.

### 3. Format rules / forbidden patterns → System prompt in `LocalLLMService.swift`

The big system prompt block has sections like `EBAY KNOWLEDGE`, `AMAZON KNOWLEDGE`, etc. Add **one-line rules** that always apply:

- ✅ "Amazon flat file: TemplateSignature = base64(PRODUCT_TYPE_NAME), settings need attributeRow=3&dataRow=4"
- ✅ "Etsy: NEVER mention 'Xinjiang Cotton' — triggers UFLPA removal, no appeal"
- ❌ "Sometimes Amazon shows error X" (too vague)

**Rules must be terse, actionable, and absolute.** Long explanations belong in `rules/{platform}-store/`.

### 4. Long-form documentation → `rules/{platform}-store/{TOPIC}.md`

Per-platform reference docs the *user* might read, or that the system prompt might cite by name:

```
rules/amazon-store/FLAT_FILE_FORMAT.md          ← detailed format guide
rules/etsy-store/LISTING_RULES.md               ← Etsy-specific dos and don'ts
rules/aliexpress-store/CSP_POLISH_WORKFLOW.md   ← step-by-step polish flow
```

### 5. Reusable workflows → `mac/AutoStore/Sources/Services/PlatformKnowledge.swift` → new helper

If a session has a multi-step workflow (e.g., "polish AliExpress listing"), and it's stable enough to hardcode, add it as a static method on `PlatformKnowledge`:

```swift
static func polishAliExpressListing(productId: String, ...) async -> String { ... }
```

Then expose it as a new tool in `LocalLLMService.swift`'s `LLM_TOOLS` array.

---

## Contribution checklist (paste this to other sessions)

When you ask a sibling session to contribute its knowledge, give them this checklist:

```
Document your platform knowledge into AutoStore so weak-model users benefit too.

Read: /Users/jameswalstonn/Documents/autostore/rules/CONTRIBUTING_PLATFORM_KNOWLEDGE.md

For each of the following you currently use or rely on, add it to AutoStore:

1. URLs you navigate to → PlatformKnowledge.swift (PlatformURL enum)
2. JS extractors you wrote for parsing pages → PlatformKnowledge.swift (PlatformExtractor)
3. Format rules / required fields / error codes → LocalLLMService.swift system prompt
4. Forbidden patterns (banned phrases, banned categories, etc.) → system prompt
5. Stable multi-step workflows → PlatformKnowledge.swift as a static func + new tool

Commit each addition. Don't ask the user to confirm individual entries — just add them
to all 5 files where applicable, then summarize what you added in your final reply.

Goal: a user with qwen-plus or glm-4-flash should still be able to do every task you
can do, because AutoStore now knows what you know.
```

---

## Anti-patterns

- ❌ **Documenting in chat instead of code** — if you only write it in a chat reply, the next session loses it
- ❌ **Mega-rules** — 200-word system prompt rules. Split into multiple one-liners
- ❌ **Conditional rules** — "if X then Y, but if Z then W". Pick the most common case and hardcode it; let the model handle edge cases via tools
- ❌ **Speculating** — only encode what you've actually verified. "I think eBay might..." → don't encode
- ❌ **Personal preferences** — those go to `~/.claude/projects/.../memory/`, not AutoStore

---

## Why this matters

AutoStore is sold to users in mainland China who have no access to Claude. They use qwen-plus, glm-4-flash, deepseek-chat. These models cannot reason their way through "find the eBay orders page" — but they *can* call `platform_go(platform="ebay", task="orders")`.

Every piece of knowledge you encode here lifts the floor of what those users can accomplish. The richer `PlatformKnowledge.swift` and the system prompt become, the less the runtime model matters.

**Target:** AutoStore should be model-agnostic. The user's LLM choice should affect speed and personality, not capability.
