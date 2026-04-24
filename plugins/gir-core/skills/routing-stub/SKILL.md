---
name: routing-stub
description: GIR orchestration core — routing rules, cost discipline, Graphify integration, escalation trigger, and lazy-load instructions. Always loaded.
---

# GIR — Orchestration Core

Route correctly. Load cheaply. Escalate cleanly.

## Session Start Protocol

Run silently on every session start:
1. Read `.gir/MISSION.md` if exists → load active priorities
2. Read `.gir/CLAUDE-activeContext.md` if exists → restore session state
3. Read `.gir/ESCALATION.md` if exists → hold must-escalate conditions in context
4. Structural/architectural/dependency task? → read `graphify-out/GRAPH_REPORT.md` before any raw file reads

## Graphify Rule

```
Structural query? → graphify-out/GRAPH_REPORT.md FIRST → then targeted file reads only
Path already known? → read directly, skip Graphify
Use Graphify for: impact analysis, dependency mapping, architecture queries, pre-task scoping
Not for: policy, decisions, session state, code correctness
Treat GRAPH_REPORT.md stale after 7 days or major merges → re-run /graphify
```

## Execution Routing

```
Default                                    → Single session
Narrow isolated result, only output matters → Subagent
≥3 interdependent parallel streams         → Agent Teams (log justification in .gir/REVIEW-LOG.md)
```

Agent Teams are never the default. Each member costs a full context window. Justify before spawning.

## Lazy Skill Loading

Load these only when the task requires them:
- New feature or scope change → `/gir-core:load-specgates` before proceeding
- Multi-step, delegated, or parallel work → `/gir-core:load-workflows`
- Iterative refinement or high uncertainty → `/gir-core:load-ralph`
- Complex agent routing → auto-delegation skill already loaded; use it

## Escalation Trigger

Before any risky action, check `.gir/ESCALATION.md`. When must-escalate condition met:
1. Stop immediately
2. Log full context to `.gir/ESCALATION-LOG.md` with timestamp
3. Write resume instructions to `.gir/CLAUDE-activeContext.md`
4. Notify human — do not proceed

## Completion Rule

Task is complete only when:
- All items in `.gir/DOD.md` pass
- CI / lint / typecheck / tests pass
- No unresolved must-escalate conditions

Never report success when these are not met. Check `.gir/DOD.md` before every completion claim.
