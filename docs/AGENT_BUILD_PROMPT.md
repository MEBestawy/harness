# Agent build prompt (copy everything below the line to an independent coding agent)

---

## Mission

You are implementing **Harness**, a general-purpose **desktop** app (not a coding IDE) that lets non-experts and prosumers **delegate multi-step work to LLMs** with strong trust rails: cost caps, event-log truth, plan-then-confirm, loop guards, approvals with material facts, and reversible file ops.

**Stack (decided):** Tauri 2 + Rust core, web frontend, **macOS first**.

Your job is to **build the product in phases** until the P0 document-wrangling recipe meets the success criteria. Do not invent a competing vision. Do not turn this into OpenCode/Cline/Claude Code.

---

## Mandatory reading (do this first)

Read these files in the repo **before writing code**. They are the source of truth. If code and docs conflict, **stop and ask the user** before changing product intent; you may fix docs for factual implementation details (paths, crate names) after asking.

| Priority | Path | Why |
|---|---|---|
| P0 | `docs/requirements.md` | Product requirements, rails R1–R17, non-goals, success criteria |
| P0 | `docs/architecture.md` | Tauri layout, crates, event log, security |
| P0 | `docs/implementation-plan.md` | Phased delivery + exit criteria |
| P0 | `.cursor/rules/project.mdc` | Coding rules (simplicity, blast radius, YAML prompts, no commit without consent) |
| P1 | `docs/findings.md` | Why the product exists (landscape, Jev, consumer agents) |
| P1 | `docs/claude-code-oss-findings.md` | What failed in Claude Code–class OSS (loops, compaction, cost lies, MCP bloat) |
| P1 | `prompts/README.md` + `prompts/*.yaml` | Prompt loading conventions + starters |
| P1 | `recipes/document_wrangle.yaml` | P0 recipe manifest |

Also skim `docs/README.md` for the doc index.

---

## Hard constraints

1. **Prompts:** Never hardcode LLM prompt bodies in Rust/TS. Load from `prompts/*.yaml` by id.
2. **Recipes:** Declarative YAML in `recipes/`. No open skill marketplace.
3. **Event log is truth:** Run UI state is projected from durable validated events, not model “done” narration.
4. **Cost:** Pause at budget; never silent auto-top-up or unbounded loops.
5. **Assume model misbehavior:** Identical-tool loops and no-progress success loops must abort with a clear recovery message.
6. **Secrets:** OS keychain only; no plaintext API keys on disk.
7. **Simplest approach:** Prefer working thin slices over abstraction. Latest stable deps.
8. **Git:** Do **not** commit or push unless the user explicitly asks.
9. **Scope:** Phase 1 is **BYOK only**. Stub the router. First-party credits later (seams OK, billing not required).
10. **Non-goals:** No terminal coding IDE, no AGENTS.md workspace product, no ClawHub-style store, no “bypass all approvals” default, no subscription-OAuth reuse hacks.

---

## What to build (order)

Follow `docs/implementation-plan.md` exactly:

0. Scaffold Tauri 2 + Rust workspace + frontend + SQLite + keyring + YAML prompt/recipe loading  
1. Rails: event log, run card, loop guard, stall watchdog, budget pause, streaming BYOK chat  
2. Permissions + scoped filesystem tools + checkpoints/undo + approval UX  
3. P0 recipe `document_wrangle` end-to-end meeting `requirements.md` §14  
4. Router stub + visible lane / escalate; optional Jev behind a flag  
5. Hardening (compaction breaker, session hygiene, export) — only after P0 works  

**Do not** jump to voice, MCP marketplace, multi-agent UI, or credits billing before Phase 3 exit.

---

## Success criteria (Phase 3 / MVP bar)

A non-technical person can run **document wrangling** and:

1. See and approve a plan before execution  
2. See live cost and hit a pause at a low cap without surprise charges  
3. Answer ≤3 clarifying questions mid-run without losing state  
4. Pause/cancel cleanly, including stuck child steps  
5. Get a spot-checkable deliverable plus plain-language steps + cost  
6. Know completed vs blocked vs failed without reading tool JSON  
7. If the model loops or compaction fails, the run **stops** with a recovery action (no silent credit burn)  

Prove with fixtures under something like `tests/fixtures/documents/` and automated tests for loop guard + budget pause.

---

## UI feel

**Instant, quiet, honest** (`requirements.md` §11):

- Optimize time-to-first-token; don’t chase vanity tok/s  
- Sparse notifications; ambient failures silent  
- Show model lane, cost, and whether data stayed local or went to a provider  
- Virtualize transcripts from day one  
- Chat front door → promote long work to **run cards**  
- Empty state: recipe picker  

---

## Suggested technical defaults (implementer may adjust if justified in README)

- Tauri 2, Rust 2021 edition, SQLite via `sqlx` or `rusqlite`  
- Frontend: React or Svelte + TypeScript (pick one; latest stable)  
- Streaming: SSE/HTTP for providers; support append deltas **and** full-replace snapshots  
- Providers: OpenRouter and/or Anthropic + OpenAI-compatible  
- Logging: structured; never log secrets or full prompt bodies in production builds  

---

## Working style

- Read docs → propose a short phase plan in chat → implement phase → show how exit criteria were met → proceed  
- Ask briefly if something material is ambiguous (e.g. React vs Svelte is yours to choose; changing success criteria is not)  
- When referencing requirements, prefer section numbers (e.g. §17.1 loop guards)  
- Update `docs/architecture.md` or `implementation-plan.md` only for factual mapping of what you built; do not silently rewrite product vision  
- Keep PRs/commits out unless asked  

---

## Deliverables checklist

- [ ] App runs on macOS via documented commands in root `README.md`  
- [ ] Phases 0–3 complete per `implementation-plan.md`  
- [ ] Keys in keychain; prompts/recipes from YAML  
- [ ] Tests for loop guard + budget pause  
- [ ] P0 recipe demo path documented  
- [ ] Known limitations listed (no credits yet, stub router, macOS-first, etc.)  

---

## One-sentence north star

**Harness is the desktop app that makes agentic work trustworthy for normal people — by refusing to silently waste money, lie about “done,” or act without a plan — not by shipping another coding agent.**
