# Prompt library

All model-facing prompts live here as YAML. Application code must **load by key**, never embed prompt bodies in Rust/TS/JS.

## Conventions

- One file per concern (e.g. `router.yaml`, `recipe_document_wrangle.yaml`, `compaction.yaml`).
- Each entry: `id`, optional `description`, `template` (string), optional `variables` list.
- Use `{{variable}}` placeholders; render in Rust/TS with a small safe templater.
- Keep prompts single-purpose (OSS finding: stacked constraints degrade small/fast models).

## Starter files

Implementers should create at least:

- `chat_system.yaml` — default chat system prompt (minimal)
- `recipe_document_wrangle.yaml` — P0 plan + act prompts
- `ask_user.yaml` — instructions for when to ask clarifying MC questions
- `compaction.yaml` — summarization prompt (phase 5)
- `router.yaml` — only if using an LLM fallback classifier (prefer System One / Jev for routing)

Until those exist, do not hardcode equivalents in source.
