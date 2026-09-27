# Implementation plan

Phased delivery for Harness on **Tauri 2 + Rust**. Follow [requirements.md](./requirements.md) and [architecture.md](./architecture.md). Do not skip phase exit criteria.

---

## Phase 0 — Scaffold (1–2 days)

**Deliver**

- Tauri 2 app boots on macOS.
- Rust workspace + frontend shell (empty chat + settings placeholder).
- SQLite (or similar) wired; keyring plugin for a dummy secret round-trip.
- `prompts/` and `recipes/` directories with example YAML loaded by Rust (no hardcoded prompt strings).
- `.gitignore` covers `target/`, `node_modules/`, dist, `.env`, OS junk.

**Exit:** `cargo test` / `npm` (or bun/pnpm) scripts documented in README; app window opens.

---

## Phase 1 — Rails without recipes (trust spine)

**Deliver**

- Event log + run state projection.
- Run card UI: states, cost-so-far, pause/cancel, last-progress.
- Loop guard (identical tool + no-progress success loops).
- Stall watchdog → `interrupted`.
- Budget: preflight estimate stub + hard pause at cap.
- Approval wait timeout → recoverable state.
- BYOK settings: add OpenRouter or Anthropic/OpenAI key to keyring; simple chat completion streaming to prove provider path.

**Exit:** Automated tests for loop guard + budget pause; manual: chat streams; forced loop aborts; budget pause works.

---

## Phase 2 — Permissions + filesystem tools

**Deliver**

- Permission engine (`allow`/`ask`/`deny`, once/always/deny).
- Workspace folder picker; `external_path` separate.
- Tools: list/read/write with checkpoints + undo.
- Material-fact approval UI.
- Kill run keeps receipts.

**Exit:** User can approve a write, undo via checkpoint; denying external path works.

---

## Phase 3 — P0 recipe: document wrangling

**Deliver**

- Recipe manifest + prompts in YAML.
- Plan-then-confirm → Act.
- `ask_user` mid-run (≤3 questions), state survives navigation.
- Capability gate (skip or soft-fail if model can’t tool-call — document behavior).
- Overhead preflight (system+tools+recipe).
- Output: structured table or summary pack user can spot-check.
- Plain-language milestone list on run card.

**Exit:** Meets [requirements §14](./requirements.md) success criteria with a real PDF/DOCX or Markdown fixture set.

---

## Phase 4 — Router stub → System One

**Deliver**

- Visible model lane + “retry with stronger.”
- Stub router (rules or fixed map) first.
- Optional: Jev/System One integration behind feature flag; local fallback if unavailable.
- Cost breakdown panel (system/tools/memory vs user).

**Exit:** Routing override works; router outage does not brick chat.

---

## Phase 5 — Hardening + P1 prep

**Deliver**

- Compaction with circuit breaker (if not earlier).
- Session hygiene: new recipe forks/clears context.
- Export run as markdown/JSON.
- Basic connector health stub (even if MCP deferred).
- Windows build smoke (optional).

**Defer:** first-party credits, voice, LAN sync, P1 research recipe, MCP marketplace (never), coding IDE features.

---

## Definition of done (whole v1 MVP)

- Phases 0–3 complete with tests for rails.
- README: run instructions, architecture pointer, known limitations.
- No plaintext API keys on disk.
- No silent unbounded spend in happy or failure paths.
- Docs updated if behavior diverges (ask user before changing product intent).
