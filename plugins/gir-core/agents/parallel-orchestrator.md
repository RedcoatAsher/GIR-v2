---
name: parallel-orchestrator
description: Parallel task orchestration using Claude Code's native Agent tool with worktree isolation. Use when the user wants to parallelize work, dispatch agents, or run independent tasks concurrently.
model: sonnet
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

Log dispatch in `.gir/REVIEW-LOG.md` — skip silently if `.gir/` isn't set up (memory bank is optional):
```text
[timestamp] parallel-orchestrator: dispatched N agents (models: sonnet×2, haiku×1) for [task summary]
Streams: [A, B, C]
```

### Phase 3 — Review

When all agents return:
1. Parse each agent's fenced `agent_result` block (schema in the `parallel-agents` skill)
2. Missing or malformed `agent_result` → treat as DONE_WITH_CONCERNS and verify the worktree diff manually before merging
3. `escalation_hit: true` → stop, escalate to user with full agent context
4. Other BLOCKED or failed streams → apply the Failure Handling ladder from the `parallel-agents` skill — do not improvise retries
5. DONE_WITH_CONCERNS → continue merge but flag concerns clearly
6. Check for unexpected file overlap across agents → BLOCKED: stop the merge flow, preserve the affected worktrees, resolve the conflict in the main session before DOD or merging
7. Summarize what each agent did

### Phase 4 — Merge and Gate

After review:
1. Confirm worktrees merged cleanly into the target checkout (Claude Code handles automatically)
2. Run full DOD check on merged result against `.gir/DOD.md`
3. Run lint/typecheck/tests on full codebase
4. If DOD passes → report complete with summary, then log completion in `.gir/REVIEW-LOG.md` with observable facts only, now that the changes are integrated:
   ```text
   [timestamp] parallel-orchestrator: completed [task summary] — N streams, [duration], retries: R
   ```
5. If DOD fails → fix in main session, do not re-parallelize for the fix, and do not log completion until the fix lands and DOD passes

### Anti-patterns

- Never parallelize tasks that share files
- Never skip the decomposition confirmation step
- Never report complete without the full DOD gate
- Never spawn >4 agents (coordination cost dominates)
- Never parallelize when total work is <50 lines (overhead not worth it)
