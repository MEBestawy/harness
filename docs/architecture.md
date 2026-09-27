# Architecture (decided)

**Status:** decided for implementation  
**Stack:** **Tauri 2 + Rust** backend, web frontend (React or Svelte — implementer chooses latest stable, keep UI thin)  
**Launch platform:** **macOS first**; keep Windows/Linux feasible (no macOS-only APIs in the core loop)  
**Date:** 27 Sep 2026

This document is normative for structure. Product behavior remains in [requirements.md](./requirements.md). Research context: [findings.md](./findings.md), [claude-code-oss-findings.md](./claude-code-oss-findings.md).

---

## 1. High-level shape

```text
┌─────────────────────────────────────────────────────────┐
│  UI (Tauri webview)                                      │
│  Chat · Run cards · Approvals · Settings · Cost meter    │
└───────────────────────────┬─────────────────────────────┘
                            │ Tauri commands / events
┌───────────────────────────▼─────────────────────────────┐
│  Rust core (crates)                                      │
│  session · run_engine · permissions · cost · providers   │
│  tools · recipes · router · compaction · keyring         │
└───────┬─────────────────────┬───────────────────────────┘
        │                     │
   SQLite event log      LLM / router APIs
   checkpoints           (BYOK now; credits later)
```

**Invariant:** UI state for runs is projected from a **durable product event log**, never from model narration alone ([requirements §2.3, R1](./requirements.md)).

---

## 2. Recommended repo layout

```text
harness/
  docs/                 # product + architecture (source of truth)
  .cursor/rules/        # agent coding rules
  prompts/              # ALL LLM prompts as YAML (never inline in Rust/TS)
  recipes/              # recipe manifests (YAML) + assets
  src-tauri/            # Tauri + Rust workspace
    crates/
      harness-core/     # session, events, run engine, permissions, cost
      harness-providers/# OpenAI-compatible + Anthropic + OpenRouter clients
      harness-tools/    # filesystem tools, ask_user, etc.
      harness-router/   # System One / Jev / stub router
    src/                # Tauri shell (commands, plugins)
  src/                  # frontend
  tests/                # integration / fixture runs
```

Keep crates small; prefer clear module boundaries over premature abstraction.

---

## 3. Core domains

### 3.1 Event log (`harness-core`)

- Append-only events with monotonically increasing `seq`, unique `event_id`, `run_id`, timestamps.
- Validate **on write**. On read: skip/quarantine corrupt events; never brick the whole run ([§17.8](./requirements.md)).
- Mint **run-unique tool_call_id** even if providers reuse message-local ids.
- Project run state machine:  
  `pending → running → needs_input | blocked → completed | failed | cancelled | interrupted`

Suggested event kinds (extend carefully):  
`run_created`, `plan_proposed`, `plan_accepted`, `model_turn_started`, `model_turn_finished`, `tool_call`, `tool_result`, `approval_requested`, `approval_resolved`, `ask_user`, `cost_updated`, `budget_paused`, `compaction_started`, `compaction_finished`, `loop_guard_fired`, `run_completed`, `run_failed`, `run_cancelled`.

### 3.2 Run engine

- Single loop for chat turns and recipe runs (OpenHuman-style: one harness, multiple entry points).
- Middleware / ordered guards (conceptually):  
  budget check → permission → doom/progress loop → tool execute → cost account → emit events.
- Structured tool results only (no shell-completion heuristics as primary signal).
- Child runs inherit narrowed permissions; parent can abort children.

### 3.3 Permissions

Vocabulary from [§17.6](./requirements.md):  
`read`, `write`, `network`, `send`, `spend`, `external_path`, `spawn_child`, `doom_loop`, `ask_user`  
Actions: `allow` | `ask` | `deny`. Approvals: `once` | `always_session` | `deny` with material facts.

Workspace folder vs external path vs network are **separate consent objects** ([§17.12](./requirements.md)).

### 3.4 Cost

- Estimate vs settled when possible.
- Per-run and daily caps; **pause and ask** — never silent overage.
- Breakdown at start: system + tools + memory vs user content.
- Store usage on events for audit.

### 3.5 Providers

- Phase 1: BYOK via OpenRouter and/or direct OpenAI-compatible + Anthropic Messages.
- Normalize quirks (tool schemas, reasoning fields, streaming deltas **and** replace-snapshots for diffusion models).
- Keys in OS keyring via Tauri plugin / `keyring` crate — no plaintext JSON.

### 3.6 Router (`harness-router`)

- Phase 1: **stub** (fixed default model + optional manual override) is acceptable.
- Phase 2: Jev / System One typed classifier — **no text generation**; visible lane + escalate affordance.
- Never block the app if router is down.

### 3.7 Recipes

- Manifests in `recipes/*.yaml` (id, title, description, tool allowlist, permission defaults, budgets, prompts refs, max steps/duration).
- Prompts referenced by key into `prompts/*.yaml`.
- P0: document wrangling. No marketplace.

### 3.8 Tools (minimal P0)

- `list_dir`, `read_file`, `write_file` (scoped), `ask_user` (multiple choice + free text escape), `extract_tables` / PDF text as needed for P0.
- No MCP in P0 unless explicitly required and overhead-preflighted.
- Checkpoints before write batches; undo restores checkpoint.

### 3.9 Compaction

- Optional after P0 rails exist.
- Circuit breaker: max attempts + min reduction ratio ([§17.2](./requirements.md)).
- Irreducible overhead preflight before run start ([§17.3](./requirements.md)).

---

## 4. Frontend principles

- Instant / quiet / honest ([requirements §11](./requirements.md)).
- Virtualize long transcripts from day one.
- Stream deltas; support full-text replace streams.
- Surfaces: Chat (front door) + Work (run cards) + Settings (keys, budgets, workspace).
- Promotion: long-running work becomes a run card with pause/resume/cancel.
- Empty chat: recipe picker, not only blank box.

---

## 5. Security baseline

- Keychain storage; refuse silent plaintext fallback.
- Never load untrusted HTML with native API access (CVE class from Electron peers).
- Treat tool results and document content as hostile (prompt injection).
- No “dangerously bypass all approvals” default.

---

## 6. Testing expectations

- Unit tests for event validation, loop guard, budget pause, permission matching.
- Fixture integration: fake provider that loops identical tools → guard fires.
- Fake provider that overflows forever → compaction/preflight refuses or breaks cleanly.
- Manual script for P0 recipe success criteria ([requirements §14](./requirements.md)).

---

## 7. Explicit non-goals for the core architecture

- Coding IDE / terminal product surface
- Open skill marketplace
- First-party multi-tenant credits in phase 1 (design seams only)
- Always-on messaging gateway
