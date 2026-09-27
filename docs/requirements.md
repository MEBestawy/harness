# Product requirements

**Status:** implementation-ready (Tauri decided)  
**Derived from:** [findings.md](./findings.md), [claude-code-oss-findings.md](./claude-code-oss-findings.md), and product decisions through 27 Sep 2026  
**Product name:** Harness (kept despite adjacent naming collisions)  
**Stack:** Tauri 2 + Rust — see [architecture.md](./architecture.md)  
**Build order:** [implementation-plan.md](./implementation-plan.md)  
**Agent handoff prompt:** [AGENT_BUILD_PROMPT.md](./AGENT_BUILD_PROMPT.md)

---

## 1. Vision

Harness is a **general-purpose desktop app** that lets ordinary people **delegate multi-step work to LLMs** — via a clean chat front door that promotes into trustworthy, interruptible **runs**.

It must support models through:

1. **First-party access (default)** — Harness-held provider subscriptions / credits; a System One–class router (e.g. Jev) chooses lanes under the hood.
2. **Bring-your-own-key (BYOK)** — user API keys and/or router aggregators (e.g. OpenRouter).
3. **Local models** — on-device / local runtime for ambient intelligence, privacy-sensitive work, and offline degradation.

It should expose modern LLM capability (multimodal in/out, tools/MCP, reasoning modes, long context, agents) **without requiring the user to understand the machinery**.

**Primary audience (v1):** prosumers who already pay for AI and want stronger delegation than chat, without a coding agent IDE.  
**Secondary audience (later):** mainstream users reached via first-party credits so they never paste API keys.

**Non-audience (v1):** developers seeking terminals, coding harnesses, or `AGENTS.md` workspaces — crowded and wrong for the trust bar.

---

## 2. Product principles

1. **Reliability over reach** — a few tasks that finish in minutes beat a general agent that often fails on long work.
2. **Cost is a hard boundary** — never silent auto-top-up; pause and ask.
3. **Product events are truth** — UI state comes from a durable event log, not model narration.
4. **Trust is discursive** — ask clarifying questions; do not over-eagerly act without confirmation on ambiguous intent.
5. **Material facts in approvals** — show what will change, where, under which permission; no consent theater.
6. **Reversibility where promised** — undo file ops; refuse or explicitly warn on irreversible external actions.
7. **Multi-agent is under the hood** — user sees a task (and optionally a second opinion), not a roster.
8. **Prompts live in YAML** — never hardcode prompts in application code (project rule).
9. **Simplest approach that works** — narrow blast radius; appropriate design patterns when they fit.
10. **Assume the model will misbehave** — loops, bad tool JSON, failed compaction, and lying “success” are harness bugs if the product does not stop them (lesson from Claude Code–class OSS).

---

## 3. Access modes and routing

### 3.1 Modes

| Mode | Who pays providers | Router | User sees |
|---|---|---|---|
| **Credits / included** | Harness | Required (System One / Jev-class) | Lane + cost/credits; override escalate |
| **BYOK** | User | Optional (same router) | Full model control + optional auto-route |
| **Local** | N/A (device) | Local classifier / System One clone when available | “Stayed on device” indicator |

### 3.2 Router requirements (Jev / System One)

The router **must not generate user-facing text**. It classifies and gates.

Minimum typed decisions per turn / step (keep option sets small):

- Task type (e.g. chat / research / extract / transform / tool-heavy)
- Difficulty / escalate?
- Needs vision / tools / long context?
- Safe to auto-approve next tool step? (agent path)
- Confidence on the above

Rules:

- High confidence + easy → cheap/fast lane.
- Low confidence or hard → frontier / stronger lane.
- Prefer **visible routing**: “Answered via \<lane\> in \<time\> — [Retry with stronger model]”.
- Do not use the decision model as a free-form answer quality judge.
- Cloud router outage → degrade to default lane or local triage; never block the app entirely.
- Validate provider ToS / multi-tenant billing before shipping multi-user credits on shared keys.

### 3.3 Cost and budgets

**Required for all modes that spend money:**

- Preflight estimate before consequential runs.
- Live cost (or credit) on every run card and message where applicable.
- Per-task and daily/session caps.
- On hit: **pause and ask**; never charge past the cap; never silent auto-reload.
- Cached-token accounting when providers expose it.
- BYOK: same transparency (user’s money); Credits: same UX (Harness margin).

---

## 4. Core surfaces

### 4.1 Chat (front door)

- Clean, polished composer: attachments (drag/drop, paste), optional voice later.
- Streaming markdown with incremental parsing; KaTeX; Mermaid; code copy.
- Edit / regenerate / stop; branch/fork with sensible names and link back to parent.
- Drafts survive navigation and reload.
- Empty state: **goals and recipes**, not only a blank box.
- Optional: prompt improve hotkey; local ghost-text autocomplete (later).

### 4.2 Work / runs (durable delegation)

When a task becomes long-running (time, planned steps, pending tools), **promote** chat → **run card**:

- Stable run ID
- Plain-language goal
- Named states: pending / running / needs input / blocked / failed / completed / cancelled
- Milestone list (progressive disclosure to raw trace)
- Elapsed time and cost-so-far
- Pause / resume / cancel (first-class)
- Action receipts and undo hooks where available

**Notifications:** only on meaningful transitions (needs approval, blocked, failed, completed) — never per-token.

**Single persistent visibility affordance** outside the transcript (e.g. tray/badge with running count; distinct state when input needed).

### 4.3 Recipes

- Curated first-party recipe library (visibly maintained).
- “Save this run as a recipe” after a successful run (retroactive authoring).
- No general visual agent builder in v1.
- Imported Skills/folders treated as **untrusted** until reviewed.

---

## 5. Agent rails (non-negotiable)

These apply whenever Harness takes multi-step or tool-using actions.

| # | Requirement |
|---|---|
| R1 | Durable **product event log**; “completed” only when product invariants pass |
| R2 | **No-progress watchdog** keyed on last real progress (tool done / message persisted / model call finished) — force-abort and surface as interrupted |
| R3 | **Plan-then-confirm** for consequential runs; user can edit plan |
| R4 | **Risk-tiered auto-approval**: large safe tier silent; money / outbound message / credentials / out-of-scope writes stop; remember decisions |
| R5 | Approvals show **material facts** (paths, domains, effects, scopes) |
| R6 | Mid-run **multiple-choice clarifying questions** (recognition over recall); answers persist across navigation |
| R7 | **Action receipts** for consequential actions |
| R8 | **Undo/checkpoint** for filesystem (and similar) ops; do not imply undo for irreversible external acts |
| R9 | One-click **“I’ll do this myself”** exit that carries context |
| R10 | Failure states: **completed / blocked / uncertain** + plain-language next action |
| R11 | Parent-surface primacy: sub-agents never become primary UI; fold into parent milestones |
| R12 | Optional **second opinion**: multi-model fan-out + synthesis with disagreement annotation (not a crew roster) |
| R13 | **Doom / progress loop guard** on parent *and* child runs (see §17) |
| R14 | **Compaction circuit breaker** with measured reduction (see §17) |
| R15 | **Irreducible context preflight** before a run starts (see §17) |
| R16 | **Model capability gate** before tool-heavy recipes (see §17) |
| R17 | **Session log integrity**: validate on write; degrade/repair on read (see §17) |

---

## 6. Ambient / local intelligence

Resident local (or free on-device) capability should power:

- Auto-title, tag, language detect, memory candidate extraction
- Continuous context compaction
- PII / sensitive-data screening before cloud send (where feasible)
- Semantic index over local chat history
- Optional local triage if cloud router unavailable

Rules:

- Background work has its own budget, rate limit, and cancellation scope.
- Debounce and batch; never starve the foreground turn.
- Fail silent for ambient tasks (no error toasts for work the user didn’t request).
- Show **where data went**: on-device vs provider.

---

## 7. Connectors, tools, MCP

- MCP client (stdio + HTTP) with OAuth where needed.
- Lazy load tool schemas; avoid dumping all schemas into every context.
- **Connector health panel**: prove tools are callable, not just connected.
- Human-in-the-loop for tools with memory of decisions.
- One-click / deep-link install paths preferred over Node scavenger hunts for mainstream recipes.
- Treat tool results and untrusted documents as hostile (prompt injection).

---

## 8. Privacy, security, permissions

- API keys and secrets in **OS keychain / safe storage** from day one; refuse silent plaintext fallback.
- Hardened webview: never give untrusted HTML (search, MCP, model HTML) native API access.
- macOS/Windows permissions requested **on demand** per feature; app remains useful if denied; handle relaunch-after-grant explicitly.
- Start filesystem scope narrow (one folder); expand deliberately.
- For egress allowlists, bind credentials to **session**, not attacker-supplied keys in content.
- Telemetry off means off.

---

## 9. Multimodal and “full LLM capability”

**v1 must support (chat + recipes as applicable):**

- Image and document input; sensible parsing for PDFs/Office where recipe needs it
- Tool calling / structured extraction
- Reasoning / thinking UI that does not leak into final text and remains scrollable mid-stream
- Citations / provenance on research-style outputs

**Explicitly later (not v1 blockers):**

- Full duplex realtime voice (ship push-to-talk first if voice is prioritized)
- Computer use / authenticated web agents
- Video generation
- Broad OS automation

---

## 10. First recipes (scope boundary)

Order by: short duration, reversible, verifiable by a non-expert, minimal credentials.

| Priority | Recipe | Why |
|---|---|---|
| **P0** | Document / data wrangling (PDFs/docs → structured table or summary pack) | Minutes-scale; new artifact; spot-checkable |
| **P1** | Multi-source research → deliverable **with per-claim provenance** | High value; needs provenance to be verifiable |
| **P2** | Local file organize/cleanup with checkpoint/undo | Reversible; lower perceived value |

**Defer:** email/calendar send, purchases, authenticated browsing, money movement.

---

## 11. UI feel

Three words: **instant, quiet, honest.**

- **Instant:** optimize TTFT; resident local model for ambient; phased warm-up status.
- **Quiet:** chrome disappears; ambient failures silent; reasoning collapsed by default; notifications sparse.
- **Honest:** show lane, cost, and data destination; actionable errors (e.g. context overflow → “Increase context”); confirm before actions that bust prompt cache / waste spend.

Micro-details that are requirements, not polish:

- ↑ recalls last composer message
- Queue outbound messages while generating (editable pending chips)
- Draft persistence
- Local timezone timestamps
- Platform window/tab close conventions
- Auto-name forks; link to parent

---

## 12. Technical constraints (from research)

- Virtualize transcripts from the first implementation; do not retain rendered HTML for off-screen turns.
- Stream **deltas**; support diffusion-style **full replace** streams.
- Persist in-flight replies so reload resumes.
- Provider quirk normalization layer (schema strictness, reasoning fields, effort vocabularies).
- Prefer a small, fast desktop shell with native keyring access (Tauri 2 + Rust is the leading candidate; final stack TBD).
- Instrument separately: TTFT, time-to-first-answer (incl. thinking), inter-token latency, end-to-end agent latency.

---

## 13. Non-goals (v1)

- Competing on feature count with Cherry Studio / Open WebUI
- Shipping a coding agent / embedded terminal IDE
- Exposing multi-agent org charts to end users
- Building a general no-code agent canvas
- Speculative prefetch of next user turns as a core bet
- Relying on Jev (or any single vendor) as the only path for generation or offline use

---

## 14. Success criteria (for the first spike)

A non-technical person can run the **P0 document-wrangling recipe** end-to-end and:

1. Understand the plan before execution.
2. See live cost/credits and hit a pause at a low cap without surprise charges.
3. Answer ≤3 clarifying questions mid-run without losing state.
4. Pause/cancel cleanly — including when a child step is stuck.
5. Receive a result they can spot-check, plus a plain-language account of steps and cost.
6. Trust whether it finished, blocked, or failed — without reading raw tool JSON.
7. If the model loops or compaction fails, the run **stops with a clear recovery action** rather than burning credits silently.

If that fails, do not expand recipe surface area.

---

## 15. Open decisions

| Topic | Status | Notes |
|---|---|---|
| Stack | **Decided: Tauri 2 + Rust** | See [architecture.md](./architecture.md) |
| Platforms at launch | **macOS first** | Keep core portable; Windows later |
| Credit pricing | TBD | Phase 1 = BYOK only; design seams for credits |
| Default model roster | Small fixed set | Start with 1–2 BYOK models |
| Jev vs local System One for triage | Stub first | Cloud Jev + local fallback after rails |

---

## 16. Suggested build sequence

1. Event log + run card + stall watchdog + cost pause + **loop guard stub** (no recipes yet).
2. Router (System One / Jev) + 2–3 lanes behind Harness keys (dev) + visible override.
3. Compaction breaker + irreducible-overhead preflight + permission vocabulary.
4. P0 recipe with plan-confirm, approvals, receipts, undo/checkpoints, capability gate.
5. BYOK path reusing the same rails.
6. Local ambient tier (non-agent / constrained tools).
7. P1 research recipe with provenance; optional second-opinion.
8. Voice (push-to-talk), LAN sync, broader connectors — only after P0 feels trustworthy.

---

## 17. Refinements from Claude Code–class OSS research

Full evidence: [claude-code-oss-findings.md](./claude-code-oss-findings.md). These **add to or sharpen** earlier rails; they do not replace them.

### 17.1 Loop and stall protection (P0)

- Detect **N consecutive identical tool calls** (same tool + same input hash) → ask or deny (OpenCode `doom_loop` pattern).
- Also detect **no-progress success loops**: same tool/resource returning the same result repeatedly (even with model narration between calls) → abort after configurable threshold with a plain-language report.
- Apply to **subagents / child runs**; parent UI must show child state and **abort child**.
- Emit **last-real-progress** timestamp; if exceeded, force `interrupted` (not forever `running`).
- Approval waits must **time out** into a recoverable state (never hang like Cline #14065).
- Hard **max steps / max duration** per recipe run.

### 17.2 Compaction circuit breaker (P0)

- Cap consecutive overflow→compact cycles (e.g. 2–3).
- Require a **minimum token reduction ratio**; if not met, stop with `compaction_failed` / `blocked_context_overflow`.
- Persist before/after token counts on the event log.
- User recovery actions: clear/fork session, reduce tools, switch model, remove attachments — never silent credit burn.

### 17.3 Irreducible overhead preflight (P0)

Before starting a recipe run, measure (or estimate):

`system + recipe prompt + enabled tool schemas + pinned memory`

If that alone exceeds the usable context budget (context − reserved output), **refuse to start** with an actionable message (disable MCP, slim recipe, larger-context model). Do not enter a compaction loop that cannot remove fixed overhead (OpenCode MCP empty-session repro).

### 17.4 Minimal tool surface per recipe (P0)

- Each recipe declares an allowlist of tools/permissions in config (YAML).
- Default: **no MCP** unless the recipe enables named servers.
- Prefer Claude-style **lazy tool schema loading** (name/description first).
- Track **cold-start token count** as a KPI; regress if defaults balloon.

### 17.5 Honest cost metering (P0)

- UI must distinguish **estimate** vs **provider-settled** usage when possible.
- Hard budget pause binds even when the estimate is wrong.
- Warn on provider **context-tier price cliffs** when detectable.
- Never rely on local `session.cost` alone for credits that map to third-party quotas (OpenCode Copilot exhaustion case).
- At run start, show a **cost/context breakdown**: system + tools/MCP + memory vs user content (Claude Code “hidden overhead” complaint → product requirement).
- Prefer API-honest metering + pause-on-cap over subscription-illusion UX.

### 17.6 Permission vocabulary (P0)

Ship allow / ask / deny with pattern matching for at least:

`read`, `write` (edit), `network`, `send` (outbound messages), `spend`, `external_path`, `spawn_child`, `doom_loop`, `ask_user`

Approvals support **once / always-for-session / deny**, showing material facts. Child runs **inherit** parent policy unless a recipe explicitly narrows further (never silently widen).

### 17.7 Model capability gates (P0 for tool recipes)

- Before a tool-heavy recipe, run a cheap **tool-call probe** (or maintain a capability matrix per model/provider).
- If the model cannot reliably emit tools, **block the recipe** or fall back to a non-agent path — do not silently hallucinate file/document contents (local Claude Code failure mode).
- Reasoning/thinking models on slow local hardware: cap or disable thinking for agent loops to avoid multi-minute “empty” turns.

### 17.8 Session / event-log integrity (P0)

- Validate event shape **on append** (ids, roles, tool call/result pairing).
- Mint **run-unique tool-call ids** even when providers reuse message-local ids.
- On load: skip or quarantine corrupt events; offer repair/export; **never** brick the whole history for one bad event (DeepSeek Harness fail-closed lessons).
- Synthetic notices (auto-continue, system tips) must carry proper ids/roles.

### 17.9 Session hygiene as product behavior (P1)

- Starting a new recipe **forks or clears** working context by default (Claude `/clear` between tasks).
- Mid-recipe: auto-compact with breaker above; offer explicit “summarize and continue.”
- Prefer **rewind/checkpoint restore** after failed branches over stuffing failures into forever-context.

### 17.10 Exploration with a leash (P1)

- Allow tool-driven discovery inside the recipe’s scoped folder/knowledge set (Claude Code exploration praise).
- Do not require the user to pre-select every file — but keep scope narrow and cost-visible.
- Re-read when underlying files change (stale-context awareness).

### 17.11 Explicitly still non-goals

- Becoming OpenCode/Cline/Aider: no terminal-primary coding IDE, no competing on SWE-bench harness features.
- Trusting `doom_loop`-style guards alone without no-progress and subagent coverage.
- Shipping MCP marketplaces or popularity-ranked skill stores before overhead preflight and lazy loading exist (OpenClaw ClawHub anti-pattern).
- Discoverable “dangerously bypass all approvals” as a default.
- Shell-integration completion detection as the primary tool-finished signal — prefer structured tool results + timeouts (Cline soft-lock cluster).
- Anthropic (or any) **subscription OAuth reuse** as a growth hack (ToS/policy war across OpenCode/Pi).
- Feature accretion for leaderboard optics when it increases token burn without reliability gains.

### 17.12 Scope consent and undo honesty (P0)

- Treat **workspace folder**, **external paths**, and **network/egress** as separate consent objects — not one “approve tools” blob (OpenCode `external_directory` lesson).
- Undo copy must name **what is covered** (filesystem checkpoints) vs **what is not** (sent messages, purchases, irreversible bash). Never market a universal rewind.
- “Kill run / I’ll finish myself” must preserve **partial receipts** and any completed reversible artifacts.

### 17.13 Peer / anti-pattern positioning

- **Peer to study:** goose (general desktop agent + recipes + MCP), not Claude Code clones.
- **Anti-pattern:** OpenClaw-style always-on host shell + open skill marketplace.
- **Governance lesson:** Continue’s wind-down — users need a clear continuity story; stars don’t retain trust.
