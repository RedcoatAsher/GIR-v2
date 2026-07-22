---
name: autonomous-executor
description: Executes a multi-phase plan from MISSION.md autonomously. Dispatches parallel agents, checkpoints state, enforces DOD and ESCALATION gates at every phase boundary. Use via /gir:run.
---

# Autonomous Executor

Execute a plan from MISSION.md phase by phase. No human confirmation between phases. Checkpoint state frequently. Gate every phase on DOD + ESCALATION.

## Activation

Triggered by `/gir:run` only. Never invoked directly by routing-stub.

## Execution Protocol

### Phase Loop

For each phase in the plan:

1. **Checkpoint** — Write current phase to MISSION.md `autonomous_run.current_phase` and `next_task`
2. **Assess** — Is this phase parallelizable? (>2 independent file-modifying tasks)
   - Yes → invoke `parallel-orchestrator` with `unattended: true`
   - No → execute directly in current session
3. **Execute** — Run the phase work
4. **Gate** — Before marking phase complete:
   - Check `.gir/DOD.md` — all items must pass
   - Check `.gir/ESCALATION.md` — if any condition met, stop immediately (see Escalation)
5. **Log** — Append phase completion to `.gir/REVIEW-LOG.md`
6. **Checkpoint** — Update MISSION.md: move phase to `completed_phases`, advance `current_phase`; refresh `.gir/CLAUDE-activeContext.md` per autonomous-mode Context Checkpointing

### Checkpointing Rules

Write to MISSION.md `autonomous_run` block:
- Before starting each phase
- After each significant action within a phase (file written, agent dispatched, test run)
- After each phase completes
- On any BLOCKED or error state

Checkpoint format in MISSION.md:
```
  checkpoint_at: [ISO timestamp]
  current_phase: [N]
  current_task: [description of what was just done or about to be done]
  completed_phases: [1, 2, ...]
```

### Unattended Mode

When `unattended: true`:
- Skip all "Proceed? yes/no" confirmation prompts
- Skip parallel-orchestrator decomposition confirmation
- Log decisions that would normally need confirmation to `.gir/REVIEW-LOG.md` instead
- Never ask the user anything — make the reasonable call and log it

### BLOCKED Recovery

Parallel streams inside a phase must apply the `parallel-agents` Failure Handling ladder exactly; do not duplicate or reinterpret it here.

For a phase that fails outside of parallel streams (e.g. a single-agent phase), apply the same shape:

1. **Retry** (only if the failure looks safe to re-run): retry the failed task once with additional context from the error
2. **Fallback** (if retry fails, or a retry isn't safe): absorb the phase into the main session and finish it directly — log the fallback to REVIEW-LOG.md
3. **Escalate** (if fallback also fails): Stop the run. Write to MISSION.md:
   ```
   autonomous_run:
     status: escalated
     escalated_at: [timestamp]
     escalation_reason: [what failed and why]
     resume_instructions: [exact steps to resume from this point]
   ```
   Write the same to `.gir/ESCALATION-LOG.md`. Notify user.

### Completion

When all phases complete:

1. Run full DOD check on entire codebase
2. If DOD passes:
   - Update MISSION.md: `autonomous_run.status = complete`
   - Cancel the scheduled resume agent (read `scheduled_resume_id`, cancel it)
   - Report summary: phases completed, files changed, time taken, any concerns flagged
3. If DOD fails:
   - Fix issues in current session (do not re-parallelize)
   - Re-run DOD check
   - Only mark complete when DOD passes

### Summary Report Format

```
Autonomous Run Complete
───────────────────────
Phases completed: [N/N]
Duration: [~X minutes]

Phase results:
  Phase 1: [description] ✓
  Phase 2: [description] ✓
  Phase 3: [description] ✓

Files changed: [count]
DOD: passed
Escalations: [none | list]
Concerns: [none | list for human review]

REVIEW-LOG.md updated with full decision trail.
```

## Anti-patterns

- Never skip DOD gate between phases
- Never proceed past an ESCALATION condition
- Never run >4 parallel agents per phase
- Never checkpoint less than once per phase
- Never mark complete without DOD passing on merged result
