# Claude Code–class OSS harness research

**Date:** 26–27 September 2026  
**Purpose:** Extract praise, complaints, and issue patterns from open-source Claude Code–style agent harnesses to refine Harness requirements.  
**Companion docs:** [findings.md](./findings.md), [requirements.md](./requirements.md)

Harness is **not** a coding IDE. Transfer lessons about **agent loops, cost, permissions, context, and session durability** — not terminals, diffs-as-primary-UX, or repo maps.

---

## 1. Landscape (projects that matter)

| Project | Role | License / posture | Why it matters for Harness |
|---|---|---|---|
| **[Claude Code](https://code.claude.com/)** (closed) | Reference product OSS clones imitate | Anthropic subscription / API | Sets expectations for plan/act, permissions, compaction, subagents, MCP |
| **[OpenCode](https://github.com/anomalyco/opencode)** | Closest OSS “Claude Code in a TUI/desktop” | MIT, BYOK + Zen/Go credits | Richest public issue corpus on loops, compaction, cost mismatch |
| **[Cline](https://github.com/cline/cline)** | IDE agent with Plan/Act + approvals | Apache 2.0 | Plan/Act and approval UX praise; stall/timeout bugs |
| **Roo → Kilo lineage** | Cline fork → rebuild | Evolving | Token bloat, crash at large chats, mode complexity |
| **[Aider](https://github.com/Aider-AI/aider)** | Git-native terminal pair programmer | Apache-style OSS | Best **undo = git** story; tight context; users want Claude Code *exploration* without Claude Code *spend* |
| **[Continue](https://github.com/continuedev/continue)** | IDE assistant (less agentic) | OSS → Cursor acqui-hire (~Jun 2026), wound down | Stability issues historically; **abandonment lesson** |
| **[DeepSeek Harness (`dsh`)](https://github.com/deepseek-ai/deepseek-harness)** | Plugin-architecture agent harness + desktop | MIT, preview | Session-log corruption / fail-closed load is a cautionary tale |
| **[OpenHuman](https://github.com/tinyhumansai/openhuman)** | Rust/Tauri personal agent orchestrator | GNU (verify) | Middleware stack (cost budget, repeated-tool breaker, approvals) as architecture reference |
| **[goose](https://github.com/aaif-goose/goose)** (Block → AAIF) | General desktop/CLI agent + MCP recipes | Apache 2.0 | **Closest OSS peer to Harness’s category** (not a coding IDE) |
| **[OpenClaw](https://github.com/openclaw/openclaw)** | Messaging gateway + agent runtime | OSS foundation | Always-on channels praise; **ClawHub skill malware = anti-pattern** for open recipe stores |
| **Local Claude Code reimpls / patches** (claw-code, Wraith, OpenClaude, Ollama patches) | Provider-unlock forks | Mixed | Prove **harness≠model**: local models often fail tool contracts silently |
| **Educational** ([learn-claude-code](https://github.com/shareAI-lab/learn-claude-code)) | Nano harness teaching material | — | “Bash + tools loop” minimal core |
| **Adjacent** | Codex CLI, Amp, Pi, Hermes | Mixed | Durable remote runs, approval fatigue, sandbox-escape risk |

Star counts in this category are noisy (SEO, mirrors, plugin marketplaces, alleged gaming). Prefer issue threads and user reports over leaderboards.

---

## 2. What users praise (patterns to copy)

### 2.1 Agentic exploration over static context packing

Claude Code’s felt advantage vs Aider/Cline-style “tell me which files” is **driving tools** (glob/grep/read/bash) to discover the world, re-read when files change, and keep going across steps ([Aider #3362](https://github.com/Aider-AI/aider/issues/3362) discussion). Users describe it as “working with someone,” not one-shot completion.

**Harness transfer:** recipes should explore within a **scoped workspace** (folder / knowledge set), not require the user to pre-attach every file — *with* cost and loop guards.

### 2.2 Plan / Act separation

Cline’s Plan mode (read-only / propose) vs Act (mutate) is repeatedly cited as the reason to trust autonomous steps. OpenCode’s Build vs Plan agents and custom slim agents echo this.

**Harness transfer:** plan-then-confirm is already required; make Plan the **default for recipes**, Act only after approval (or a large safe auto tier).

### 2.3 Reversible edits as first-class

Aider’s auto-commit / `git revert` undo is the clearest “I can try this without fear” story in OSS coding agents.

**Harness transfer:** filesystem checkpoints + undo (already P0); for non-git user content use snapshot/worktree-style checkpoints (Kilo moved snapshots out of the repo for similar reasons).

### 2.4 Permission model with remember + patterns

OpenCode’s `allow` / `ask` / `deny` with wildcards, `once` / `always` / `reject`, `external_directory`, and `doom_loop` is a concrete schema ([docs](https://opencode.ai/docs/en/permissions/)). Claude Code’s allowlists + Auto mode + sandbox are the closed-source parallel ([best practices](https://code.claude.com/docs/en/best-practices.md)).

**Harness transfer:** ship an explicit permission vocabulary for recipes (read / write / network / send / spend), not a single “approve all tools” toggle.

### 2.5 Subagents as context firewalls

Anthropic documents subagents as **separate context windows** that return only a summary ([session value blog](https://claude.com/blog/maximizing-the-value-of-your-claude-code-sessions)). Users praise this for keeping main sessions usable.

**Harness transfer:** hide multi-agent UI, but **implement** isolated child runs that cannot pollute parent context — with parent-visible abort.

### 2.6 Session hygiene commands

`/clear` between tasks and proactive `/compact` mid-task are official advice and Reddit consensus; quality degrades long before hard context limits ([r/ClaudeAI on 1M context](https://www.reddit.com/r/ClaudeAI/comments/1s3bcit/your_claude_code_limits_didnt_shrink_i_think_the/)).

**Harness transfer:** recipe boundaries should **auto-clear or fork sessions**; compact mid-run; never encourage one immortal chat for all work.

### 2.7 On-demand tools / Skills

Claude Code Tool Search reportedly cut MCP context ~47% by deferring schemas. Open WebUI / Claude patterns: show name+description, load full schema on use.

**Harness transfer:** already required; OSS pain makes it **P0**, not polish.

---

## 3. Ranked pain points (evidence)

### P1 — Unbounded agent loops / identical tool spam

OpenCode [#45442](https://github.com/anomalyco/opencode/issues/45442): subagent issued **364 identical greps over ~50 minutes**, ~184k input + ~142M cache-read tokens; parent showed `running`; manual DB + HTTP interrupt required. Related: [#43673](https://github.com/anomalyco/opencode/issues/43673), [#43800](https://github.com/anomalyco/opencode/issues/43800), [#43603](https://github.com/anomalyco/opencode/issues/43603). OpenCode’s `doom_loop` (3 identical calls → ask) **exists** but is bypassed when models interleave narration, when set to ask under `--auto`, or when “successful” repeats look legitimate ([thread discussion](https://github.com/anomalyco/opencode/issues/45442)).

**Requirement implication:** loop detection must cover identical args **and** same resource/same result with no progress; apply to **subagents**; expose last-activity; parent can abort children.

### P2 — Compaction that never terminates

OpenCode [#27924](https://github.com/anomalyco/opencode/issues/27924): overflow → compact → still overflow → infinite API burn. Repros include **empty user message** when MCP tool catalog + system prompt alone exceed usable window. Users disable auto-compaction to stop credit burn. Multiple related issues (#15533, #27594, #30805, #48827…).

**Requirement implication:** max compaction attempts + minimum reduction ratio + terminal `compaction_failed` / `blocked_context_overflow` with recovery actions; **preflight irreducible overhead** (system + tools + recipe) before starting a run.

### P3 — Cost UI that lies (or diverges from billing)

OpenCode [#8945](https://github.com/anomalyco/opencode/issues/8945): UI ~$4.70 vs spending limit $14 hit. [#44224](https://github.com/anomalyco/opencode/issues/44224): wrong tier pricing below real context thresholds. [#45249](https://github.com/anomalyco/opencode/issues/45249): compaction never fired → monthly Copilot quota exhausted; local cost (~$102) ≠ provider impact. Claude Code Reddit: “best tool for the **45 minutes a day** I can use it”; MCP alone tens of thousands of tokens before first prompt ([MorphLLM summary of Reddit](https://www.morphllm.com/claude-code-reddit)).

**Requirement implication:** label estimates vs settled; reconcile when providers expose usage; **hard pause** still binds even if estimate is wrong; warn when fixed overhead is large.

### P4 — Context / prompt bloat as default

OpenCode users cut **~90%+ tokens** by swapping default Build agent (full tool set / long system prompt) for a minimal custom agent ([video report](https://www.youtube.com/watch?v=FX7jcd3GYtI)). Roo users: MCP-heavy default prompts **50–60k characters**; disabling MCP halves cost ([comparison threads](https://www.reddit.com/r/ChatGPTCoding/comments/1jn36e1/roocode_vs_cline_updated_march_29/)). Claude Code: 30–40k+ tokens at session start with tools/memory; four MCP servers → ~67k tokens before typing (community reports).

**Requirement implication:** **minimal default tool surface per recipe**; MCP off unless recipe needs it; slim system prompts in YAML; measure cold-start tokens as a KPI.

### P5 — Model–harness mismatch (local / weak tool calling)

Local Claude Code / Ollama threads: models **talk about** files without emitting `Read`/`Glob`; hallucinate contents; reasoning models “think” 10–20 minutes per turn on CPU ([r/LocalLLaMA](https://www.reddit.com/r/LocalLLaMA/comments/1rwwl05/local_claude_code_totally_unusable/), [codingProtection writeup](https://www.reddit.com/r/codingProtection/comments/1u146tb/i_tried_to_run_claude_code_100_locally_gemma_4/)). Aider’s plain-text search/replace is more robust for weak models than JSON tool contracts.

**Requirement implication:** **capability gates** — don’t run tool-heavy recipes on models that fail a tool-call probe; for local ambient use constrained / non-agent paths; prefer structured tools only when reliability is proven.

### P6 — Approval / stall UX bugs

Cline [#14065](https://github.com/cline/cline/issues/14065): approval await with **no timeout**; auto-proceed never fires. [#13763](https://github.com/cline/cline/issues/13763): first tool approval IPC unavailable in one-shot sessions. Classic approval fatigue remains (Anthropic Auto Mode ~17% FNR — see prior findings).

**Requirement implication:** approval waits need timeouts + recoverable states; never hang forever as “running.”

### P7 — Session durability / fail-closed load

DeepSeek Harness discussions: missing `message.id` on injected notices → **entire session unopenable** ([#4819](https://github.com/deepseek-ai/deepseek-harness/discussions/4819)); reused provider tool-call ids break history ([#5296](https://github.com/deepseek-ai/deepseek-harness/discussions/5296)); one illegal abort-cause field → total loss ([#6236](https://github.com/deepseek-ai/deepseek-harness/discussions/6236)); seq-gap corrupt logs ([troubleshooting](https://dshdocs.com/troubleshooting/corrupt-session-log-seq-gap/)).

**Requirement implication:** validate **on write**; tolerate/skip corrupt events on read with repair; never make one bad event brick the run history; mint **session-unique** tool-call ids even if providers reuse message-local ids.

### P8 — Long-session quality collapse / memory

Roo [#2700](https://github.com/RooCodeInc/Roo-Code/issues/2700): extension crash at 100MB+ chats; new chats jumping to 85MB; **$1k burned in ~7 hours** when context crossed expensive tiers. Claude users: accuracy drops ~300–400k into a 1M window.

**Requirement implication:** virtualize UI (already); **bound agent context** independently of UI history; force recipe-scoped sessions; detect provider tier cliffs in cost preflight.

### P9 — Subagent observability gap

Parents cannot see child progress/stalls; children don’t inherit parent doom_loop / permission scheme reliably (OpenCode #45442 thread).

**Requirement implication:** unified progress events for parent+children; inheritance rules for permissions and loop guards documented and tested.

### P10 — Open skill / plugin marketplaces as supply chain

OpenClaw’s ClawHub repeatedly cited for malware and prompt-injection-as-skill (community audits / #1-downloaded-skill reports). goose has had recipe prompt-injection history even with a more curated posture.

**Requirement implication:** first-party curated recipes only in v1; imports quarantined; **no popularity-ranked skill store**.

### P11 — Soft-lock when tools “finish” but the harness never notices

Cline cluster: terminal command completes; agent never sees completion → infinite spinner and retries ([#4737](https://github.com/cline/cline/issues/4737), [#6603](https://github.com/cline/cline/issues/6603), [#5990](https://github.com/cline/cline/issues/5990)). Users toggle Plan/Act or paste output as workaround.

**Requirement implication:** prefer **structured tool results** over shell-integration heuristics; timeouts on every tool wait; “kill and keep receipts” exit.

### P12 — False undo / sandbox theater

Claude Code `/rewind` does not reverse bash/`rm`/migrations ([r/ClaudeAI](https://www.reddit.com/r/ClaudeAI/comments/1u8cgpc/claude_codes_rewind_isnt_an_undo_button_it_doesnt/)). Codex CLI sandbox escapes are a recurring class ([advisory](https://github.com/openai/codex/security/advisories/GHSA-w5fx-fh39-j5rw)).

**Requirement implication:** side-effect taxonomy in undo copy; never market unified rewind; don’t ship “dangerously bypass approvals” as a default.

### P13 — Abandonment / fork churn

Continue acqui-hire wind-down; Roo sunset (May 2026) → ZooCode/Cline migration pain.

**Requirement implication:** product durability and clear ownership matter as much as open-source optics for trust.

---

## 4. What transfers vs what does not

| Transfers to Harness (general desktop agent) | Does **not** transfer as product surface |
|---|---|
| Tool-driven exploration inside a sandbox | Terminal-as-primary UI |
| Plan/Act, permissions, doom loops | Repo map / LSP-centric coding |
| Compaction circuit breakers | Git commit-every-edit as the only undo (optional for code recipes) |
| Subagent context isolation | Exposing agent rosters to users |
| Minimal prompts / lazy MCP | Competing with OpenCode/Cline as a coding tool |
| Cost pause + honest metering | Assuming Claude-grade tool calling from every model |
| Session hygiene (/clear between jobs) | Immortal multi-day sessions as the happy path |

---

## 5. Architecture references worth stealing

**OpenHuman middleware stack** (documented in their harness architecture): approval/security gating, tool policy, malformed-arg recovery, **cost budget pre-check**, **repeated-tool-failure circuit breaker**, context trim/compress, stop hooks — one loop for chat, channel, and subagents ([OpenHuman agent harness](https://tinyhumans.gitbook.io/openhuman/developing/architecture/agent-harness)).

**OpenCode permission keys:** `read` / `edit` / `bash` / `task` / `skill` / `webfetch` / `websearch` / `external_directory` / `doom_loop` / `question` ([permissions docs](https://opencode.ai/docs/en/permissions/)).

**Claude Code product ops:** `/cost`, `/compact`, `/clear`, `/model`, `/permissions`, `/rewind` preferred over compact after failed paths ([docs](https://code.claude.com/docs/en/best-practices.md), [help center](https://support.claude.com/en/articles/14552983-models-usage-and-limits-in-claude-code)).

---

## 6. Unverified / caution

- Exact “67k tokens from four MCP servers” and “46.9% Tool Search savings” come from secondary aggregations; treat as directional.
- OpenCode / dsh / OpenClaw star counts and “fastest growing OSS” claims are marketing-adjacent or contested; issue quality is the signal.
- Local Claude Code “totally unusable” threads mix LM Studio bugs, attribution headers, network blocks, and true tool-call failure — diagnose carefully.
- DeepSeek Harness is in developer preview with breaking changes; session bugs may improve rapidly.
- Claude Code “hidden retry classifier burns Max quotas” reverse-engineering posts are **single-source / unverified**.
- ForgeCode / ClawCode “parity” and benchmark marketing claims need independent verification.
- Anthropic subscription reuse via third-party harnesses is **policy-sensitive** and should not be a Harness feature bet.

---

## 7. Bottom line for Harness

Coding-agent OSS proves the **same failure modes** we already prioritized for consumer agents — cost, loops, fake progress, context bloat — with sharper engineering detail:

1. **Assume the model will loop.** The harness must stop it.
2. **Assume compaction can fail.** Bound it and fail loudly with recovery.
3. **Assume cost UI is wrong.** Hard caps still bind; reconcile when possible.
4. **Assume MCP/tools will eat the window.** Lazy-load and preflight overhead.
5. **Assume weak models break tool contracts.** Gate recipes by capability.
6. **Assume session logs get weird.** Validate on write; degrade gracefully on read.
7. **Assume open skill stores get poisoned.** Curate; never rank by downloads.
8. **Treat goose as the peer category** and **OpenClaw as the anti-pattern** for always-on + marketplace blast radius.
9. **Feature accretion is a risk** — later harness versions often burn more tokens without quality gains (harness evaluation discourse on HN).

Concrete requirement deltas are listed in [requirements.md §17](./requirements.md#17-refinements-from-claude-codeclass-oss-research).

*Incorporates parallel synthesis from deeper multi-source research (Sep 2026), including goose/OpenClaw/Continue/Codex threads.*
