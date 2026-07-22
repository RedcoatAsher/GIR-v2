# ADR: Orchestrator AI-Practices Update (v2.6.0)

**Date**: 2026-07-22
**Status**: Implemented

## Decision

Modernize the orchestration layer (routing-stub → parallel-agents → parallel-orchestrator, autonomous-mode → autonomous-executor) with current AI-engineering practices, placed point-of-use in existing files. Zero new skills, zero executable scripts, zero new `.gir/` templates.

## Practice dispositions

| Practice | Disposition |
|----------|-------------|
| Token efficiency | Already covered — modular install, lazy skill loading, Graphify gate, `CLAUDE-resources.md` on-demand fetch. No new text. |
| Human-in-the-loop | Already covered — 3-tier `ESCALATION.md`, decomposition confirmation, unattended mode never suppresses escalation/DOD. No new text. |
| Batching | Already covered — single-message multi-agent dispatch, "invoke independent ops simultaneously". One line added to gir-ai delegation. |
| Workflow-first design | Already covered — commands are workflows. Workflow Index added to OPERATING-MODEL for discoverability. |
| Prompt caching | Adapted — caching is automatic in Claude Code; the practice is prefix stability. "Context Economy" rule in OPERATING-MODEL. |
| Model routing | Added — Model Routing table in routing-stub; `model: sonnet` pin on parallel-orchestrator. |
| Retries / fallbacks | Added — Failure Handling ladder in parallel-agents. |
| Structured outputs | Added — fenced `agent_result` YAML schema in parallel-agents. |
| Context management | Added — Context Checkpointing in autonomous-mode. |
| Observability / FinOps | Adapted — observable facts in REVIEW-LOG dispatch entries; real numbers via native `/cost`, `/context`, OTEL export (README). |
| Hybrid AI + rules | Adapted — `/gir:doctor`: deterministic checklist executed by the model, no shipped scripts. |
| Semantic caching | Adapted — memory-first lookup rule in core-practices (check troubleshooting/patterns/decisions before re-deriving). |
| Guardrails / governance | Added — this ADR, root `CHANGELOG.md`, new Rule Authority Map rows. |
| Agent specialization | Sharpened — orchestrator pin below; executor deliberately unpinned. |

## Contentious calls

**`parallel-orchestrator` pinned to `sonnet`; `autonomous-executor` deliberately unpinned.** The orchestrator coordinates, parses results, and gates merges — it never does final implementation (its own Phase 4 pushes fixes to the main session). Pinning caps the cost of the most-spawned orchestration path, matching team-lead's existing pin. The executor performs final implementation across arbitrary phases in the highest-risk (unattended) mode — down-pinning trades quality where errors are most expensive; users who run opus should get opus there. `tools` stays inherited on both: merge/state handling needs Edit/Bash, and premature scoping is where frontmatter breakage actually happens.

**Fenced `agent_result` YAML over the loose return format.** The Failure Handling ladder needs machine-readable `escalation_hit` and `retry_safe` flags; `autonomous_run` in MISSION.md is the existing YAML-block precedent. Malformed/missing blocks degrade to `DONE_WITH_CONCERNS` + manual diff verification — never to silent merge.

**Zero new skills.** Dispatch-time rules live in parallel-agents (already loaded at the decision moment), run-time rules in autonomous-mode, contributor rules in OPERATING-MODEL. A grab-bag "ai-practices" skill would either be absent when needed or become a fourth always-loaded file. The Model Routing table is the one deliberate always-loaded addition (~80 words in routing-stub): routing decisions happen before any on-demand skill loads, so a lazy home guarantees the rule is missing at decision time.

**REVIEW-LOG enrichment over a new `.gir/METRICS.md`.** A prompt cannot measure token spend honestly — prompt-estimated counts are fabrications. Dispatch entries carry observable facts only (agent count, models, duration, retries); real cost data comes from `/cost`, `/context`, and OTEL export.

**Prompt-driven `/gir:doctor` over a script-type hook.** A checklist of exact `jq`/`grep` commands executed by the model gets ~90% of a rules engine with zero shipped scripts, preserving GIR's prompt-only distribution. Revisit criteria: if doctor checks are being run manually more than once per release, promote to a command-type hook.

## Flagged for 3.0 (not this release)

Version strings hard-coded in `hooks.json` prompt text (`### gir-core (vX.Y.Z)`) are a fourth mechanical touch-class per release and mild cache-prefix churn. Dropping the version from the `GIR.modules` entry format would simplify releases but changes a format existing users' files already contain — defer to a 3.0 discussion.
