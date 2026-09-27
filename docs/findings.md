# Research findings

**Date:** 26 September 2026  
**Scope:** Competitive landscape for desktop LLM clients; what fast / System One models unlock; consumer and prosumer agent products and failure modes.

Primary research was conducted across three tracks: existing desktop/BYOK apps and feature inventory; fast-inference and Jev; consumer/prosumer agent products and UX. Claims below summarize that work. Vendor-published numbers and single-source items are called out where relevant.

---

## 1. Thesis that survived research

Harness is a **desktop harness for agentic work**: give non-experts (starting with prosumers who already pay for AI) the ability to delegate multi-step tasks without becoming developers.

The open gap is **not** “another feature-dense chat client” and **not** “another coding agent workspace.” Mid-2026, serious players stampeded toward developer agent UIs (terminals, `AGENTS.md`, shell approval, MCP setup that needs Node/`npx`). LM Studio’s flagship became *Bionic*; Open WebUI shipped embedded terminals; LibreChat shipped coding agents. Users who wanted a polished, trustworthy *delegation* surface were left behind — while OpenAI and Google each **killed** their flagship consumer agent products (ChatGPT agent removed Aug 2026; Project Mariner shut May 2026).

The binding constraint for consumer agents is **cost predictability, verifiability, and reversibility** — not raw model capability.

---

## 2. Naming note

“Harness” collides with [DeepSeek Harness (`dsh`)](https://github.com/deepseek-ai/deepseek-harness) (MIT, very large star count, agent harness) and with Perplexity marketing (“agent harness”). Product decision: **keep the name**; treat searchability as a marketing cost, not a rename blocker. Semantically the name fits the agentic intent.

---

## 3. What Jev is (and is not)

**Jev** (TypeSafe AI, early access ~15 Sep 2026) is a **System One decision model**, not a chat LLM.

| Property | Finding |
|---|---|
| I/O | Text state + typed questions in → calibrated typed answers out (yes/no, choice, score). **No text generation.** |
| Context | ~32K per question; ~64K combined (vendor/guide figures) |
| Price | ~$0.042/M input on OpenRouter; output not billed (as reported) |
| Latency | Vendor: 70–500 ms; independent: ~140 ms server-side / ~0.83 s end-to-end median (varies by gateway) |
| Strength | **Calibration** — at confidence ≥0.9, ~94–96% correct in tested tasks |
| Weakness | Not a quality judge of free-form answers; long option lists inflate cost; headline “193× / 444×” claims shrink sharply end-to-end |

**Implication:** Jev can never write a chat reply. It belongs in the **invisible layer**: routing, guardrails, tagging, relevance, gating agent steps. Open Apache-style System One clones appeared quickly after launch; for local/offline triage those matter as much as Jev’s API.

**Best evidenced use:** confidence cascade — Jev (or equivalent) routes easy/high-confidence turns to cheap models and escalates the rest. Measured pattern: matched strong-model accuracy at roughly **38–45% of that model’s cost** on tested hybrid setups (with methodology caveats across harnesses).

---

## 4. Speed: where it matters and where it does not

Adults read ~5–6 tokens/sec. **Above ~20 tok/s, faster streaming is essentially imperceptible** in a chat window.

Speed pays off in:

1. **Time to first token (TTFT)** — the wait users consciously feel (~1 s is Nielsen’s flow threshold).
2. **Invisible background work** — titling, tagging, memory, compaction, reranking (often fractions of a cent per call at Flash-Lite-class pricing).
3. **Multi-hop agent loops** — latency compounds; ~75% of tool turns issue a single tool call; each model turn often costs seconds. Cutting round trips beats chasing tok/s.

Supporting evidence: moving context compaction to a fast model (Augment Code / Mercury case) reported **~82% latency reduction and ~90% cost reduction** at maintained quality.

Also relevant:

- Fastest generative systems increasingly include **diffusion LLMs** (parallel denoise). Clients must support **replace-snapshot** streams, not only append-deltas.
- Local inference: **TTFT often matters more than decode speed**; llama.cpp vs MLX tradeoffs are machine-specific; naive speculative decoding can regress.
- Prompt caching can cut cost/TTFT sharply but breaks on silent prefix mutations.

---

## 5. Competitive landscape (condensed)

### 5.1 Categories

| Category | Role for Harness |
|---|---|
| Local-first desktops (LM Studio/Bionic, Jan, Ollama, …) | Local runtime, model UX, footprint benchmarks |
| Vendor apps (ChatGPT, Claude, Gemini, Perplexity, Copilot) | Polish and OS-integration bar |
| BYOK / aggregator clients (Cherry Studio, Msty, Chatbox, Big-AGI, TypingMind, Witsy, …) | Direct competitors for multi-provider chat |
| Consumer/prosumer agents (Manus, Genspark, Instinct, Town, Rabbit OS3, Zapier Agents→AI step, Lindy, …) | Delegation UX and failure modes |

### 5.2 Recurring gaps (table stakes vs opportunity)

**Wide open / poorly served**

- Live **dollar** cost per message/run + hard budget stop (Cursor removed $ from usage UX for some plans; few clients do this well).
- Transcript **virtualization** and memory discipline (long chats OOM / lag is endemic).
- Secrets in **OS keychain** (many “privacy” apps still use plaintext JSON/`localStorage`).
- Connector **health that proves tools are callable**, not merely “connected.”
- Local model fit that accounts for **KV cache at chosen context length**.
- Consumer-grade **realtime voice** in third-party clients (mostly absent; APIs exist).
- Sync without forcing cloud (LAN pairing exists in a few apps).

**Table stakes for a serious 2026 client**

- Drag/drop, clipboard paste, streaming markdown, KaTeX, Mermaid, branch/fork, folders, search, MCP (+ OAuth), per-chat params, reasoning-effort control, human-in-the-loop tool approval.

**Best-in-class exemplars to steal from (non-exhaustive)**

- ChatGPT Appshots; Gemini two-tier hotkey + Fn dictation; Claude on-demand tool schemas; Big-AGI Beam + AI Inspector; Perplexity Model Council; LibreChat cost traces; Open WebUI delta streaming; Jan keyring + composer micro-details; Lindy pause-not-overage; Microsoft agent workspace isolation.

### 5.3 Stack observation

In this category, **Tauri-class footprint (~100 MB)** vs **Electron (~350–400 MB+)** is a legible differentiator. Jan is the closest FOSS proof that Tauri + local inference + keyring works for a chat/agent-adjacent product. Transcript rendering quality (virtualization, incremental markdown) dominates perceived performance more than shell choice.

---

## 6. Consumer agents: evidence summary

### 6.1 Market

- Consumer AI spend grew sharply; **depth among payers**, not raw adoption, drives growth (Menlo 2026-style survey data).
- Agent use concentrates among **heavy payers / technical prosumers**, not “average joe” as a mass buyer yet.
- AI apps **churn faster** and refund more than non-AI apps while earning more per payer (RevenueCat-class industry reports).
- **48–60%** of consumers abandon an agent after one bad experience in shopping/AI-experience surveys; many others **downgrade autonomy** if a dial exists.
- Agents succeed often on **short** tasks and fail badly as duration grows (METR-style: near-perfect under ~4 minutes; very poor on multi-hour tasks).

### 6.2 Dominant failure modes (ranked)

1. Unpredictable / runaway cost (recursive calls, opaque credits, auto-reload surprises).
2. Silent wrong action without approval or clear trace (incl. prompt-injection / exfiltration cases).
3. User cannot tell whether the task succeeded (UI treats model narration as state).
4. Indefinite stalls (spinners keyed on heartbeats, not real progress).
5. Blank-page / “prompt gambling” before irreversible work.
6. Approval fatigue (dialogs rubber-stamped; material facts hidden).
7. Trust collapse after one bad run.

### 6.3 Authoring

Custom GPT–style no-code builders show **massive creation mortality** and weak retention evidence; OpenAI restricted new personal GPT creation (Aug 2026). Plan for **~99% consumption, ~1% authoring**. Prefer curated recipes + “save this run as a recipe,” not a visual agent builder.

### 6.4 Multi-agent as UX

For mainstream users, **multi-agent should be an implementation detail**. Exposing crews/rosters creates “who is talking?” confusion and multiplies failure attribution. The one legible exception: **“second opinion” multi-model** (Model Council shape) — single turn, one synthesized artifact, disagreement annotated.

### 6.5 UX patterns with evidence

| Pattern | Why it matters |
|---|---|
| Cost preflight + hard pause (no silent overage) | Addresses #1 failure mode; Lindy/Town-like |
| Product event log as source of truth | Fixes fake “success” and stall detection |
| Plan-then-confirm + large safe auto-approve tier | Trust + less fatigue |
| Mid-run multiple-choice clarifying questions | Recognition over recall; builds discursive trust |
| Chat → durable run-card promotion | Short tail vs long-running tasks |
| Action receipts + undo where possible | Verifiability / reversibility |
| Progressive disclosure of traces | Logs unreadably dense for both verbosity camps |

Controlled study worth keeping: *Why Johnny Can’t Use Agents* (CAIS ’26 / arXiv) — mental models, premature trust, collaboration mismatch, communication overload, weak metacognition.

---

## 7. Implications for Harness (bridge to requirements)

1. **Position:** trustworthy desktop delegation for prosumers first; mainstream later via out-of-box routing credits — not a coding IDE.
2. **Rails before breadth:** cost bounds, event-log verification, stall watchdog, approvals with material facts, undo on reversible actions.
3. **Recipes over blank agent:** few short, verifiable tasks first (documents before email/web money-adjacent actions).
4. **Jev / System One as router + gate**, never as the writer of answers.
5. **Three access modes:** first-party credits (Harness-held keys + router), BYOK, local ambient/privacy tier.
6. **Hide multi-agent machinery**; optionally expose multi-model second opinion.
7. **Do not compete on feature count** with Cherry Studio / Open WebUI; compete on trust rails and finishable jobs.

See [requirements.md](./requirements.md) for the product requirements derived from these findings.

For coding-agent harness issue archaeology (OpenCode, Cline, Aider, dsh, …) and how it sharpens the rails, see [claude-code-oss-findings.md](./claude-code-oss-findings.md).
