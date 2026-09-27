# Harness

Desktop app for trustworthy LLM delegation (chat → durable runs). **Stack:** Tauri 2 + Rust. **Status:** docs-complete; implementation not started.

## Docs (start here)

- **Hand an agent this:** [`docs/AGENT_BUILD_PROMPT.md`](docs/AGENT_BUILD_PROMPT.md)
- Requirements: [`docs/requirements.md`](docs/requirements.md)
- Architecture: [`docs/architecture.md`](docs/architecture.md)
- Implementation plan: [`docs/implementation-plan.md`](docs/implementation-plan.md)
- Research: [`docs/findings.md`](docs/findings.md), [`docs/claude-code-oss-findings.md`](docs/claude-code-oss-findings.md)

## Conventions

- Prompts live in [`prompts/`](prompts/) as YAML — never hardcode in app source.
- Recipes live in [`recipes/`](recipes/) as YAML.
- Do not commit/push unless explicitly requested.
