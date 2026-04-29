# ADR: GIR Hub-and-Spoke Modularization

**Date**: 2026-03-03
**Status**: Implemented

## Decision

Split GIR from monolithic CLAUDE.md into a hub-and-spoke plugin architecture:
- `gir-core`: mandatory hub — agents, core skills, commands
- `gir-web`, `gir-atc`, `gir-automation`, `gir-ai`, `gir-database`, `gir-qa`: optional domain spokes

## Rationale

Single CLAUDE.md loaded all context regardless of project type. Token overhead was unacceptable for solo founder use. Spoke isolation reduces per-session cost to only installed modules.

## Implementation

6 phases completed 2026-03-03. Self-registering SessionStart hooks. `gir-module.json` manifests per plugin. `.gir/` memory bank for persistent session state. All phases shipped.

## Known Issue at Completion

`core-practices/SKILL.md` still contained spoke MCP tool docs (Gemini-CLI, v0, Figma, Vercel, Supabase, n8n, CodeRabbit, Jules). Resolved in v2.1.0.
