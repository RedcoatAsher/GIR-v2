---
name: routing-stub
description: GIR orchestration core — routing rules, delegation tiers, cost discipline, Graphify integration, escalation trigger, and lazy-load instructions. Always loaded.
---

# GIR — Orchestration Core

Route correctly. Load cheaply. Escalate cleanly.

## Session Start Protocol

Run silently on every session start:
1. Read `.gir/MISSION.md` if exists → load active priorities
2. Read `.gir/CLAUDE-activeContext.md` if exists → restore session state
3. Read `.gir/ESCALATION.md` if exists → hold must-escalate conditions in context
4. Structural/cross-community/impact task? → check God Nodes + Community Hubs in `graphify-out/GRAPH_REPORT.md` before raw file reads. Skip if corpus fits in one context window and task touches ≤1 community.

## Graphify Rule

```
Corpus fits in one context window? (report warns: "fits in a single context window")
  → prefer inline reads unless task is cross-community or structural navigation

Consult graphify-out/GRAPH_REPORT.md when:
  - Touching a God Node (top 10 by edge count) → check impact before editing
  - Cross-community task → use Community Hubs + Surprising Connections to map blast radius
  - Unfamiliar area → use Surprising Connections to find undocumented links
  - Dependency/impact analysis → check Hyperedges for group participation patterns

Skip Graphify when:
  - Path or file already known → read directly
  - Task touches ≤1 community and corpus fits in context
  - Policy, decisions, session state, code correctness

Treat GRAPH_REPORT.md stale after 7 days or major merges → re-run /graphify
```

## Execution Routing

```
Default                                    → Single session
Narrow isolated result, only output matters → Subagent
≥3 interdependent parallel streams         → Agent Teams (log justification in .gir/REVIEW-LOG.md)
```

Agent Teams are never the default. Each member costs a full context window. Justify before spawning.

## Parallel Trigger Phrases

When the user message contains any of these patterns, automatically invoke `/gir:parallel` before responding:

- "parallelize [work / tasks / agents / this / these]"
- "dispatch agents"
- "run [X] in parallel" / "run these in parallel"
- "parallel agents" / "parallel tasks" / "parallel execution"
- "spawn agents" / "spawn [N] agents"
- "work on [X] simultaneously" / "do these simultaneously"
- "[X] and [Y] at the same time"
- "split [work / tasks] across agents"
- "concurrent [tasks / agents / execution]"

Do not wait for user to type `/gir:parallel` — detect and invoke automatically.

## Delegation Tiers

**Tier 1 — Always deploy:**
- `gir-sequential-thinking`: complex tasks (>3 steps), architecture, debugging, refactors (3+ files)
- `code-reviewer`: before every commit, after major tasks
- `Explore agent`: task involves >5 files, or "where is X" / "how does X work" questions

**Tier 2 — Usually deploy:**
- `parallel-orchestrator`: >2 independent file-modifying tasks — native worktree isolation, no external tools
- `Debugger agent`: no obvious cause, multi-system issue
- `Ralph loop`: clear completion criteria, iterative work (TDD, build fixes)

**Tier 3 — Never auto-delegate:**
- Final implementation, direct Q&A, architecture decisions, single-file edits (<50 lines), trivial fixes

## Model Routing

Pin models at the agent level; inherit only for open-ended work.

```
haiku   → mechanical + lookup: docs fetch, log scan, file inventory, boilerplate
sonnet  → default: implementation, review, debugging, orchestration
inherit → final implementation in autonomous runs, architecture, ambiguous scope
```

Routing above a task's tier requires one justification line in `.gir/REVIEW-LOG.md` — skip silently if `.gir/` isn't set up (memory bank is optional). External-CLI delegation thresholds live in the `ai-delegation` skill (gir-ai).

## Task Priority Order

```
CRITICAL (blockers)  → schema, auth, type defs, env/config, dependencies
HIGH (core)          → endpoints, DB queries, business logic, integrations
MEDIUM (features)    → UI, forms, loading states, responsive design
LOW (polish)         → animations, cleanup, docs
```

## Lazy Skill Loading

Load these only when the task requires them:
- New feature or scope change → `/gir:load-specgates` before proceeding
- Multi-step or delegated work → `/gir:load-workflows`
- Parallel work detected → `/gir:parallel` (auto-triggered by parallel trigger phrases above)
- Iterative refinement or high uncertainty → `/gir:load-ralph`

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
