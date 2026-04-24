# Mission State

_Updated: 2026-04-24_

## Active Mission
Make GIR the most reliable, token-efficient orchestration layer for Claude Code.

## Active Priorities (ordered)

1. **Merge auto-delegation into routing-stub** — delete `auto-delegation/SKILL.md`, fold the 3-tier routing matrix and key rules into `routing-stub/SKILL.md`. Single source of truth for routing decisions. Expected savings: −211 always-loaded lines.

2. **Trim core-practices further** — remove ~60 lines that duplicate lazy-loaded skills (Workflow Patterns section) and DOD.md (Pre-Completion Checklist). Pure subtraction.

3. **Wire feature-architect, debugger, spec-analyst agents to DOD/escalation** — add the same 3-line DOD/escalation check block already present in code-reviewer.md and team-lead.md. Closes the gap where these agents bypass GIR quality gates.

## In Progress
- [ ] Nothing currently in progress

## Blocked
<!-- none -->

## Completed This Cycle
- Routing-stub skill created — 2026-04-24 — commit ffec9e3
- Lazy-load strategy (load-workflows, load-specgates, load-ralph) — 2026-04-24 — commit ffec9e3
- core-practices spoke docs removed — 2026-04-24 — commit ffec9e3
- hooks.json reads MISSION.md + ESCALATION.md — 2026-04-24 — commit ffec9e3
- init-memory-bank scaffolds 10 files — 2026-04-24 — commit ffec9e3
- code-reviewer.md rewired to DOD/escalation/REVIEW-LOG — 2026-04-24 — commit 11fb595
- team-lead.md cost discipline + DOD wiring — 2026-04-24 — commit 11fb595

## Out of Scope (do not work on)
- marketplace.json / README version bump — opportunistic only, zero reliability impact
- drift-check MISSION.md enhancement — wait for evidence of real drift incidents
- Spoke agent DOD wiring — optional, core gates cover merge path
- Graphify usage guide — wait until GRAPH_REPORT.md format confirmed from live run
- skills_on_demand enforcement — comment in gir-module.json is sufficient

## Next Session: Start Here
Implement the three active priorities in order. Each is small and independent after Priority 1.
Start with Priority 1 (merge auto-delegation → routing-stub) — read both files in full before editing.
