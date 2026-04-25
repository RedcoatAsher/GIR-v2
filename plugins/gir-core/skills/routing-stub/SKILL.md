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

## Delegation Tiers

**Tier 1 — Always deploy:**
- `sequential-thinking`: complex tasks (>3 steps), architecture, debugging, refactors (3+ files)
- `code-reviewer`: before every commit, after major tasks
- `Explore agent`: task involves >5 files, or "where is X" / "how does X work" questions

**Tier 2 — Usually deploy:**
- `Subtask`: >2 independent file-modifying tasks — parallel worktrees prevent conflicts
- `Debugger agent`: no obvious cause, multi-system issue
- `Ralph loop`: clear completion criteria, iterative work (TDD, build fixes)

**Tier 3 — Never auto-delegate:**
- Final implementation, direct Q&A, architecture decisions, single-file edits (<50 lines), trivial fixes

## Task Priority Order

```
CRITICAL (blockers)  → schema, auth, type defs, env/config, dependencies
HIGH (core)          → endpoints, DB queries, business logic, integrations
MEDIUM (features)    → UI, forms, loading states, responsive design
LOW (polish)         → animations, cleanup, docs
```

## Lazy Skill Loading

Load these only when the task requires them:
- New feature or scope change → `/gir-core:load-specgates` before proceeding
- Multi-step, delegated, or parallel work → `/gir-core:load-workflows`
- Iterative refinement or high uncertainty → `/gir-core:load-ralph`

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
