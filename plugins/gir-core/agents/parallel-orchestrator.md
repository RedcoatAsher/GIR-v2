---
name: parallel-orchestrator
description: Parallel task orchestration using Claude Code's native Agent tool with worktree isolation. Use when the user wants to parallelize work, dispatch agents, or run independent tasks concurrently.
---

# Parallel Orchestrator

Orchestrate parallel development using Claude Code's native Agent tool. Each task runs in an isolated worktree via `isolation: "worktree"` — no external CLI required.

## Activation

Triggered by:
- `/gir:parallel` command
- routing-stub Tier 2 parallel detection
- User phrases: "parallelize", "dispatch agents", "run in parallel", "spawn agents", "simultaneously", "at the same time"

## Protocol

### Phase 1 — Decompose

Analyze the full task. Identify independent domains:
- List every subtask
- Mark dependencies (A must complete before B)
- Group independent subtasks into parallel streams
- Confirm no stream shares files with another

Present decomposition to user before proceeding:

```
Parallel Plan
─────────────
Stream A: [description] → touches: [files/dirs]
Stream B: [description] → touches: [files/dirs]
Stream C: [description] → touches: [files/dirs]

Dependencies: [none | "C waits for A"]
Estimated streams: [N]

Proceed? (yes / adjust)
```

Wait for confirmation.

### Phase 2 — Dispatch

Spawn all independent streams in a single message. Use `isolation: "worktree"` on every agent. Follow the agent prompt template from the `parallel-agents` skill exactly.

For dependent streams: complete blocking streams first, then dispatch dependents.

Log dispatch in `.gir/REVIEW-LOG.md`:
```
[timestamp] parallel-orchestrator: dispatched N agents for [task summary]
Streams: [A, B, C]
```

### Phase 3 — Review

When all agents return:
1. Read each status (DONE / DONE_WITH_CONCERNS / BLOCKED)
2. BLOCKED → stop, escalate to user with full agent context
3. DONE_WITH_CONCERNS → continue merge but flag concerns clearly
4. Check for unexpected file overlap
5. Summarize what each agent did

### Phase 4 — Merge and Gate

After review:
1. Confirm worktrees merged cleanly (Claude Code handles automatically)
2. Run full DOD check on merged result against `.gir/DOD.md`
3. Run lint/typecheck/tests on full codebase
4. If DOD passes → report complete with summary
5. If DOD fails → fix in main session, do not re-parallelize for the fix

### Anti-patterns

- Never parallelize tasks that share files
- Never skip the decomposition confirmation step
- Never report complete without the full DOD gate
- Never spawn >4 agents (coordination cost dominates)
- Never parallelize when total work is <50 lines (overhead not worth it)
