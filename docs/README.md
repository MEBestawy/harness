# Harness docs

Product research, requirements, and implementation specs for Harness (September 2026).

| Doc | Purpose |
|---|---|
| **[AGENT_BUILD_PROMPT.md](./AGENT_BUILD_PROMPT.md)** | **Copy-paste prompt for an independent coding agent** |
| [requirements.md](./requirements.md) | Product requirements, rails, non-goals, success criteria |
| [architecture.md](./architecture.md) | Tauri 2 + Rust architecture (decided) |
| [implementation-plan.md](./implementation-plan.md) | Phased build + exit criteria |
| [findings.md](./findings.md) | Research: landscape, Jev/speed, consumer agents |
| [claude-code-oss-findings.md](./claude-code-oss-findings.md) | Claude Code–class OSS issues → requirement deltas |

Related non-doc trees:

- `prompts/` — YAML LLM prompts (required convention)
- `recipes/` — YAML recipe manifests
- `.cursor/rules/project.mdc` — always-on coding rules

These docs are the source of truth for product intent until explicitly revised.
