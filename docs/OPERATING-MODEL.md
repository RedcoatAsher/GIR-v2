# GIR Operating Model

> How GIR works in practice. Reference this when setting up a new repo or verifying an existing one.

---

## Core Files

| File | Role | Authority |
|------|------|-----------|
| `routing-stub/SKILL.md` | Session-start, routing, Graphify gate, escalation, completion | **Always-loaded. Owns all execution rules.** |
| `core-practices/SKILL.md` | MCP tools, command efficiency, anti-patterns, git conventions | Always-loaded. Owns dev practices. |
| `.gir/MISSION.md` | Active priorities, in-progress work, next-session handoff | Committed. Updated each session. |
| `.gir/ESCALATION.md` | Must-escalate conditions — stop/log/wait | Local. Set per project via `/init-memory-bank`. |
| `.gir/DOD.md` | Definition of done checklist — required for every merge | Local. Set per project via `/init-memory-bank`. |
| `graphify-out/GRAPH_REPORT.md` | Knowledge graph of repo — God Nodes, communities, surprising connections | Generated. Re-run `/graphify` after major merges or every 7 days. |

---

## Session-Start Protocol (runs silently every session)

1. Read `.gir/MISSION.md` → load active priorities
2. Read `.gir/CLAUDE-activeContext.md` → restore session state
3. Read `.gir/ESCALATION.md` → hold must-escalate conditions in context
4. Structural/cross-community task? → check `graphify-out/GRAPH_REPORT.md` before raw reads

*Defined in routing-stub. Do not duplicate in other files.*

---

## Graphify Gate (when to use GRAPH_REPORT.md vs inline reads)

```
corpus fits in one context window?
  yes + task touches ≤1 community → inline reads
  yes + task touches a God Node or crosses communities → check GRAPH_REPORT.md first
  no (large corpus) → always check GRAPH_REPORT.md

GRAPH_REPORT.md is stale after 7 days or major merges → re-run /graphify
```

The report header tells you corpus size. The "fits in a single context window" warning is the trigger condition. No interpretation required.

---

## Execution Routing

```
Default                                    → Single session
Narrow isolated result, only output matters → Subagent
≥3 interdependent parallel streams         → Agent Teams (log justification in .gir/REVIEW-LOG.md)
```

Agent Teams cost a full context window per member. Justify before spawning.

---

## Delegation Tiers

**Tier 1 — Always:**
- `sequential-thinking`: complex tasks (>3 steps), architecture, debugging, refactors (3+ files)
- `code-reviewer`: before every commit, after major tasks
- `Explore agent`: task involves >5 files or "where/how does X work" questions

**Tier 2 — Usually:**
- `Subtask`: >2 independent file-modifying tasks
- `Debugger agent`: no obvious cause, multi-system issue
- `Ralph loop`: clear completion criteria, iterative work (TDD, build fixes)

**Tier 3 — Never auto-delegate:**
- Final implementation, direct Q&A, architecture decisions, single-file edits, trivial fixes

---

## Completion Gate

Task is complete only when:
- All items in `.gir/DOD.md` pass
- CI / lint / typecheck / tests pass
- No unresolved must-escalate conditions

Check `.gir/DOD.md` before every completion claim. ralph-loops must also check DOD before emitting `COMPLETE`.

---

## Rule Authority Map

One source per rule. No duplicates.

| Rule | Defined in | Do not restate in |
|------|-----------|-------------------|
| Session-start steps | routing-stub | core-practices |
| Graphify gate | routing-stub | anywhere else |
| Delegation tiers + sequential-thinking | routing-stub | core-practices |
| Escalation trigger | routing-stub | core-practices |
| Completion gate | routing-stub + DOD.md | core-practices |
| MCP tool strategy | core-practices | routing-stub |
| Anti-patterns | core-practices | routing-stub |
| Git/file conventions | core-practices | routing-stub |

---

## Bootstrapping a New GIR Repo

See [`docs/SETUP-STORY.md`](SETUP-STORY.md) for the Phase 0–7 arc with reusable prompt templates for each phase, including the Graphify integration (Phase 4), drift normalization (Phase 5), and rule deduplication (Phase 6) prompts.
