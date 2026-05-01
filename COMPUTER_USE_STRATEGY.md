# AutoStore Computer Use Strategy

## The Core Problem

Chinese LLMs (Qwen, DeepSeek, Kimi, Zhipu) are weak at:
1. **Spatial reasoning** — guessing pixel coordinates from a screenshot
2. **Multi-step planning** — chaining 10+ tool calls without losing state
3. **Tool discipline** — calling tools as JSON, not printing them as text

Generic computer use (look at screenshot → estimate coords → click) fails with these models.

## The Strategy: Narrow the Task Space

AutoStore only needs ~15 ecommerce workflows on 4 platforms. This is not a general agent problem. The solution is to **reduce what the model has to figure out**, not to wait for better models.

---

## Implemented: Annotated Screenshots (2026-05-01)

**File:** `in-chrome/extension/service-worker.js` — `computer(action="screenshot")` case

**What it does:**
Every screenshot is now annotated before capture:
1. Query all visible interactive elements (buttons, links, inputs, selects, etc.)
2. Inject orange numbered badges (① ② ③ …) at each element's top-left corner
3. Capture the screenshot with labels rendered natively by the browser
4. Remove the overlay immediately after capture
5. Return the image **plus** a text index:
   ```
   Interactive Elements:
     [1]  button          (423,318)  登录
     [2]  a[href]         (520,318)  注册
     [3]  input[text]     (640,200)  (no label)
     ...
   To click: computer(action="left_click", coordinate=[x,y])
   ```

**Why this works:**
- The model reads the index (text) → picks the right number → uses the listed coordinate
- No coordinate estimation from the image is needed
- Works on any website without per-site coding
- The orange labels also help the user understand what the model is seeing

**System prompt rule added:**
```
ANNOTATED SCREENSHOTS: Every screenshot includes orange numbered badges and an
"Interactive Elements" index. ALWAYS use the coordinate from the index — never
guess by visually estimating position.
```

---

## Planned: High-Level Ecommerce Macro Tools

Instead of the model navigating eBay Seller Hub step by step, provide single-call tools:

| Tool (planned) | What it replaces |
|---|---|
| `platform_upload(platform, file)` | 15-step CSV/TSV upload flow |
| `platform_check_listings(platform)` | Screenshot → parse table |
| `platform_check_orders(platform)` | Screenshot → parse orders |
| `seller_login(platform)` | Multi-step login sequence |

These turn the model into an **intent router** ("upload to eBay") rather than a **step-by-step computer-use agent**. Implement when a workflow is run often enough to justify hardcoding.

---

## Planned: Page Action Menu

After `browser_go`, instead of returning raw page text, return a structured action list:
```
Page: eBay Seller Hub — Active Listings
Actions:
  [1] Button: "Create listing"
  [2] Button: "Download report"
  [3] Link: "Bulk upload" → /sell/import
  [4] Table: 47 active listings
```
The model picks `[3]` — no DOM parsing, no coordinate math.

---

## Key Principles

1. **Narrow specialisation beats general intelligence** — 15 known workflows > infinite generality
2. **Text beats images for weak models** — structured element index > visual coordinate estimation
3. **One tool, one outcome** — macro tools that do the whole flow atomically prevent half-finished states
4. **Error messages are diagnostic, not terminal** — system prompt rules (NEVER-GIVE-UP, LOOP PREVENTION) stop the model from bailing out on the first error
5. **Model choice matters for complex tasks** — Claude/GPT-4o for complex computer use, Qwen/DeepSeek for simple ecommerce tasks. The model warning system in Settings flags models with poor tool-use quality.

---

## Model Quality Tiers (for Computer Use)

| Tier | Models | Suitable for |
|------|--------|-------------|
| **Excellent** | Claude Sonnet, GPT-4o, Gemini 2.5 Pro | Any computer use task |
| **Good** | qwen-max, DeepSeek-chat | Simple ecommerce with annotated screenshots |
| **Poor** | qwen-plus, qwen-vl-plus, glm-4-flash, moonshot-v1-8k | Chat only; computer use will loop/hallucinate |

The Settings model picker shows a ⚠️ warning for Poor-tier models when computer use is involved.
