# AutoStore Local Model Strategy

**Status:** Active. Path A (Qwen 2.5 3B on existing CPU server) is the MVP.
**Authored:** 2026-05-06
**Predecessor docs:** `KNOWLEDGE_DISTILLATION_STRATEGY.md`, `COMPUTER_USE_STRATEGY.md`, `BUSINESS_MODEL.md`.

---

## Why a local model

The distillation strategy moved multi-step reasoning from runtime (model) to design-time (Claude → static macro library). This means **the runtime model only needs to do intent classification** — pick the right macro and fill in 1-3 parameters.

Intent classification is the easiest job in NLP. Even a 1-3B-parameter model handles it reliably. We do **not** need 70B+ frontier models at runtime.

The strategic implication: AutoStore can ship its own LLM endpoint, eliminating three sources of friction:

| Friction | Before local model | After local model |
|---|---|---|
| API key onboarding | User must register Aliyun/DeepSeek/etc., paste API key, hit quota errors | Zero — works out of box |
| Per-token cost to user | ¥1-30/month on cheap models | ¥0 |
| Privacy concerns (PIPL) | User data routes to third-party LLM provider | Stays in our infrastructure (HK / SG region) |
| Network reliability | Depends on user's connection to Aliyun/etc. | Depends only on user's connection to our backend |
| Onboarding completion rate | ~40% (API key cliff) | ~85% (estimated) |

---

## Architecture (Path A — current MVP)

**End users NEVER directly access the model server.** Mac client talks only to the AutoStore backend; the backend proxies LLM requests on the user's behalf.

```
┌──────────────────────────┐
│  AutoStore Mac Client    │   user device
│  (with user's JWT)       │
└──────────┬───────────────┘
           │ HTTPS + user JWT (existing AutoStore auth)
           ▼
┌──────────────────────────┐
│  AutoStore Backend       │   api.spriterock.com (existing)
│  - Validates user JWT    │
│  - Tracks quota & cost   │
│  - Holds AUTOSTORE_LLM_TOKEN in env vars
└──────────┬───────────────┘
           │ HTTPS + Bearer AUTOSTORE_LLM_TOKEN  (server-to-server only)
           ▼
┌──────────────────────────┐
│  Model Server            │   llm.spriterock.com  (this doc)
│  - nginx terminates TLS  │
│  - Validates bearer token│
│  - llama-server (loopback only)
│  - Qwen 2.5 3B Q4        │
│  - AWS SG restricts inbound to backend's IP
└──────────────────────────┘
```

**Why this layering:**
- Bearer token never leaves the backend → safe even if a Mac client is compromised
- Quota / per-user rate limits / Pro-tier gating live in the backend (where user identity exists)
- Model server is fungible — swap 3B for 7B or move to GPU without any client-side change
- AWS Security Group restricts inbound on port 443 to the backend's elastic IP — even with valid token, public callers are blocked at the network layer
- Failover (primary → backup) is centralized in the backend; Mac client doesn't know about model topology

**Runtime stack:**
- **Inference engine**: `llama.cpp` (latest release) compiled CPU-only with AVX2/AVX512 support
- **Model**: `Qwen2.5-3B-Instruct-Q4_K_M.gguf` (~2.0 GB)
- **Server**: `llama-server` binary — exposes OpenAI-compatible REST API at `/v1/chat/completions`
- **Reverse proxy**: nginx terminates TLS, validates `Authorization: Bearer <token>` header, proxies to `127.0.0.1:8080`
- **Process manager**: systemd unit (`autostore-llm.service`) — auto-restart on failure, auto-start on reboot
- **TLS**: existing Let's Encrypt cert via certbot; subdomain `llm.spriterock.com` (or path `/llm/v1` on existing domain — TBD at deploy time)

**Hardware reality on the existing server:**
- 2 vCPUs (Intel Xeon Platinum 8259CL @ 2.5 GHz)
- 7.6 GB RAM (already ~2.5 GB consumed by Ethereum node `gma`)
- No GPU
- 96 GB disk (4.4 GB used → plenty of room for model + cache)
- The model fits in remaining RAM with margin (~3 GB peak inference + 2.5 GB Ethereum + 2 GB system)

**Performance expectations on this hardware:**
- Time-to-first-token: 2-4 seconds for typical prompts (~2K tokens system + tools + history)
- Generation speed: 6-10 tokens/sec
- Total latency for an 80-token tool call: **5-10 seconds**
- Concurrent requests: serialized (single-instance) — fine for early users; queue beyond ~5 concurrent

---

## API contract

The endpoint is **fully OpenAI-compatible**, so AutoStore backend just changes one base URL:

```
POST https://llm.spriterock.com/v1/chat/completions
Authorization: Bearer <AUTOSTORE_LLM_TOKEN>
Content-Type: application/json

{
  "model": "qwen2.5-3b",
  "messages": [...],
  "tools": [...],
  "stream": true,
  "temperature": 0.0
}
```

Response is a standard OpenAI streaming chunk format. Tool-call extraction works as-is — `llama.cpp`'s server supports the OpenAI tool-call format natively for Qwen.

**Backend integration** — this has TWO sides:

1. **`backend/src/llm/`** (NestJS, server-side) — add a new POST route `/api/llm/chat` that:
   - Validates the user's JWT (existing AutoStore auth)
   - Checks the user's quota and plan tier (Pro users get unlimited; Free tier limited to e.g. 50/month)
   - Forwards the request to `https://llm.spriterock.com/v1/chat/completions` with the server-side `AUTOSTORE_LLM_TOKEN` in the `Authorization` header
   - Streams the response back to the client
   - Logs usage for billing / abuse detection

2. **`mac/AutoStore/Sources/Services/LocalLLMService.swift`** (Mac app, client-side) — add a new `LLMProviderConfig`:

```swift
LLMProviderConfig(
  id: "autostore-cloud",
  name: "AutoStore Cloud (免费)",
  baseURL: "https://api.spriterock.com/api/llm",   // ← AutoStore backend, NOT the model server
  defaultModel: "qwen2.5-3b",
  models: ["qwen2.5-3b"],
  maxOutputTokens: 4096,
  maxIterations: 100,
  contextWindowSize: 32768
)
```

The Mac client authenticates with its **existing AutoStore JWT** (no new API key field). End users see `AutoStore Cloud (免费)` as a one-click option. The bearer token to the model server **stays exclusively on the backend** — Mac client never sees it, never holds it, never sends it.

**Inbound restriction**: AWS Security Group on the model server allows port 443 only from the AutoStore backend's elastic IP (e.g. `52.x.x.x/32`). Even a leaked bearer token is useless from any other source.

---

## Folder layout — `local-model/`

All deployment code lives under `/Users/jameswalstonn/Documents/autostore/local-model/`:

```
local-model/
  README.md                  — quick reference + run instructions
  server.txt                 — SSH info (existing, do not commit)
  scripts/
    01-install-deps.sh       — apt install build-essential cmake git curl python3-pip nginx
    02-build-llamacpp.sh     — git clone + cmake build of llama.cpp (CPU-only, AVX optimised)
    03-download-model.sh     — wget Qwen 2.5 3B Q4_K_M GGUF from HuggingFace mirror
    04-systemd-install.sh    — install autostore-llm.service, enable, start
    05-nginx-config.sh       — install nginx site config + reload
    06-tls-setup.sh          — certbot --nginx for llm.spriterock.com
    99-deploy.sh             — top-level orchestrator: runs 01→06 in order
  systemd/
    autostore-llm.service    — systemd unit definition (loopback :8080)
  nginx/
    autostore-llm.conf       — nginx site config (TLS + bearer auth + proxy)
  config/
    llm.env                  — LLAMA_MODEL_PATH, LLAMA_PORT, AUTOSTORE_LLM_TOKEN
    llm.env.example          — committable template (no secrets)
  test/
    smoke-test.sh            — curl test against the live endpoint
    bench.sh                 — measures TTFT and tok/s with sample prompts
```

**Conventions:**
- Every script idempotent (rerun-safe)
- Every script logs to stderr what step it is
- No script prompts for input — use environment variables or arguments
- Secrets (bearer token, model URL if private mirror) live in `config/llm.env`, gitignored
- The `99-deploy.sh` orchestrator can deploy the entire stack onto a fresh Ubuntu 24.04 box

---

## Operational model

**Provisioning**: from main computer, run `./local-model/scripts/99-deploy.sh primary` — it SCPs the scripts to the server and runs them. Done.

**Updates**: when changing model or config:
- Re-run `03-download-model.sh` (downloads new GGUF)
- `systemctl restart autostore-llm`
- Test with `./local-model/test/smoke-test.sh`

**Monitoring**: log to `/var/log/autostore-llm.log` via systemd journal. Add a Cloudwatch / health-check ping later (Phase 2).

**Rolling failover**: if primary server `34.235.75.7` is down, AutoStore backend can fall back to backup `3.211.5.163` via DNS round-robin or in-app retry. Backup runs the same stack idle.

**Token rotation**: `AUTOSTORE_LLM_TOKEN` rotates quarterly. Update both the server's `config/llm.env` (bearer auth) and the AutoStore backend's env. Old tokens invalidated by `systemctl restart autostore-llm`.

---

## When to graduate to Path B / C

**Path B** = upgrade to Qwen 2.5 7B on the same CPU hardware
- Trigger: macro library has matured to handle 80% of tasks, and we're seeing the 20% edge cases that 3B fumbles
- Cost: zero (same hardware), but latency jumps to 30-60s per call → only makes sense if calls become async
- Likely never worth it; Path C is better

**Path C** = move to GPU instance (`g4dn.xlarge` ≈ ¥3,000/mo on-demand or ¥1,200/mo spot)
- Trigger: ≥50 paying users (¥10K+ MRR), where the cost amortizes to <5% of revenue
- Benefits: Qwen 2.5 7B at 40-50 tok/s → 2-3s per call, room for concurrent serving via vLLM
- Migration: same llama.cpp/vLLM API, just swap the systemd unit and nginx upstream

Document the decision and migration in `rules/LOCAL_MODEL_PATH_C.md` if/when we get there.

---

## Risks & mitigations

| Risk | Mitigation |
|---|---|
| Model server crashes / OOM | systemd auto-restart; nginx `proxy_next_upstream` to backup server |
| Token leaked publicly | Rotate token, restart server, audit access logs |
| HuggingFace download blocked from China region | Pre-download GGUF on US/HK and serve from our own CDN; users go through `llm.spriterock.com` so they don't need direct HF access |
| Concurrent users overload single instance | llama.cpp serializes — measure under load, add queue, eventually move to Path C |
| Quality regression on edge cases | Keep BYO-LLM as a fallback option in the model picker; users can always switch |
| RAM contention with Ethereum node | Monitor; if it bites, move LLM to backup server (which has same specs but lighter Ethereum load) — they're identical |

---

## Why we expose this as "AutoStore Cloud" (not "Qwen 3B")

Marketing positioning: users don't care about the model name. They care about:
- "免费"
- "无需配置 API Key"
- "数据不泄露给第三方"
- "中国可访问，不用 VPN"

The picker label should be **「AutoStore Cloud (免费)」** with a small subtitle "由 AutoStore 提供算力". This is honest (we host it), simple (one click), and brand-builds (every successful task = an AutoStore-branded experience, not a Qwen one).

---

## Compliance notes

This intersects with `rules/BUSINESS_MODEL.md` and the privacy policy (`frontend/src/app/privacy/page.tsx`):

- **Section 5 of the privacy policy** already covers "optional Claude API fallback"; we extend it to include "AutoStore Cloud (built on Qwen 2.5 3B)"
- User data sent to AutoStore Cloud stays on AWS HK / SG (one cross-border hop from Mainland China — covered by the existing PIPL disclosure)
- We do **NOT** train any model on user data — the model weights are the public Qwen2.5-3B-Instruct release, used as-is
- Bearer token is per-deployment (not per-user) — minimal PII

A short sentence will be added to `rules/COMPLIANCE_QUICKREF.md` (if it exists) summarizing the data flow.

---

## See also

- `rules/KNOWLEDGE_DISTILLATION_STRATEGY.md` — why intent routing is sufficient at runtime
- `rules/COMPUTER_USE_STRATEGY.md` — annotated screenshots + macro tools that lift weak-model floor
- `rules/BUSINESS_MODEL.md` — why eliminating API-key onboarding friction lifts conversion 2-3×
- `local-model/README.md` — operational instructions for deploying and maintaining the server
