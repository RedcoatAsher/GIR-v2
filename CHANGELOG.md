# Changelog

All notable changes to the GIR plugin ecosystem. Format follows [Keep a Changelog](https://keepachangelog.com/); versions are lock-stepped across all plugins except `gir-migrate`.

## [2.6.0] — 2026-07-22

Orchestrator AI-practices release. ADR: [`docs/decisions/2026-07-22-orchestrator-ai-practices.md`](docs/decisions/2026-07-22-orchestrator-ai-practices.md).

### Added
- **Model routing** — routing-stub table (haiku → mechanical, sonnet → default, inherit → open-ended); `parallel-orchestrator` pinned to `sonnet` (`autonomous-executor` deliberately stays unpinned)
- **Structured agent returns** — parallel agents end with a fenced `agent_result` YAML block (status, files, concerns, `escalation_hit`, `retry_safe`); malformed results degrade to manual diff verification, never silent merge
- **Failure handling for parallel dispatch** — bounded retry/fallback ladder: 1 retry per stream (never after an escalation hit), sequential absorb on second failure, abandon parallelism when >50% of streams fail
- **Context checkpointing** — autonomous runs rewrite `.gir/CLAUDE-activeContext.md` at every phase boundary; file state is authoritative on resume
- **`/gir:doctor`** — deterministic ecosystem health check (manifest JSON validity, version lock-step, GIR.modules registry, memory-bank presence)
- **Context Economy rule** — always-loaded skills and hook prompts are cache-prefix material: stable within a release, no volatile state (OPERATING-MODEL.md)
- **Memory-first lookup** — check `.gir/` troubleshooting/patterns/decisions before re-deriving (core-practices)
- **External first-pass code review** — models configured in `.gir/ai-integrations.md` (Gemini-CLI, Codex, ...) are enabled at will and tackle code review as the first pass; the `code-reviewer` agent keeps the final DOD verdict
- **Observability & Cost docs** — README section on `/cost`, `/context`, OpenTelemetry export; dispatch logs now carry agent count, models, duration, retries
- Root `CHANGELOG.md`; Workflow Index and new Rule Authority Map rows in OPERATING-MODEL.md

### Fixed
- gir-core SessionStart hook's `GIR.modules` entry drifted from `gir-module.json` — now lists `parallel-orchestrator`, `autonomous-executor`, on-demand skills `parallel-agents`/`autonomous-mode`, and commands `parallel`/`run`/`doctor`

## [2.5.1] — 2026-04-29

Ecosystem overhaul release.

### Breaking
- Core plugin renamed `gir` → `gir-core` — reinstall with `claude plugin install gir-core`; all spokes now require `gir-core`
- `gir-tools` renamed → `gir-atc` (Agentic Traffic Control); deprecated `subtask-manager` and `agenthub` agents removed

### Fixed
- Sequential-thinking MCP server renamed to `gir-sequential-thinking` — no longer conflicts with user-installed `sequential-thinking` servers
- Marketplace collection display name now shows as `gir` in the Claude Code TUI

### Added
- `gir-ai` first-run integration detection — prompts to configure AI tools if none are set up
- `gir-ai/templates/ai-integrations.md` — template for wiring Gemini CLI, Codex, Anthropic API

### Versioning
- Spokes `2.0.0` → `2.5.1`; core `2.2.0` → `2.5.1`; `gir-migrate` `1.0.0` → `1.1.0`
