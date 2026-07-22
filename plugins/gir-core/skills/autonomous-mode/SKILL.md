---
name: autonomous-mode
description: Autonomous execution mode for GIR — unattended operation, cap fail-safe, checkpoint/resume protocol. On-demand skill loaded by /gir:run.
---

# Autonomous Mode

Governs unattended plan execution, usage-cap recovery, and checkpoint/resume protocol.

## MISSION.md Schema

The `autonomous_run` block lives at the bottom of MISSION.md:

```yaml
autonomous_run:
  status: idle | in-progress | escalated | complete
  unattended: true | false
  resume_after_cap: true | false
  started_at: "2026-04-28T10:00:00Z"
  current_phase: 2
  completed_phases: [1]
  current_task: "implementing auth middleware"
  checkpoint_at: "2026-04-28T10:23:00Z"
  scheduled_resume_id: "agent-xyz-123"
  escalated_at: null
  escalation_reason: null
  resume_instructions: null
```

Never edit this block manually during a run. The autonomous-executor owns it.

## Cap Fail-Safe Protocol

### Why pre-emptive scheduling

Claude's session dies instantly at usage cap — no hooks fire. The fail-safe must be scheduled at run **start**, not at cap time.

### Schedule at run start

When `/gir:run` starts with `resume_after_cap: true`:
1. Record `started_at` timestamp
2. Schedule remote agent for `started_at + 70 minutes`
3. Remote agent checks MISSION.md `status` field:
   - `in-progress` → resume via `/gir:run`
   - `complete` or `idle` → do nothing, exit

### SessionStart detection

On every SessionStart, after reading MISSION.md:

If `autonomous_run.status == "in-progress"`:
1. Calculate elapsed: `now - checkpoint_at`
2. If elapsed > 65 minutes → usage cap likely; auto-resume if `resume_after_cap: true`
3. If elapsed < 65 minutes → likely an interrupted normal session; ask user before resuming
4. If `unattended: true` and elapsed > 65 min → resume without asking

### Cancel on completion

When run completes cleanly:
1. Read `scheduled_resume_id` from MISSION.md
2. Cancel the scheduled agent
3. Set `autonomous_run.status = complete`

If cancellation fails (agent already fired and exited): safe to ignore.

## Unattended Mode Rules

When `unattended: true` is set:

| Normal behavior | Unattended behavior |
|---|---|
| "Proceed? yes/no" | Skip — proceed, log decision |
| Parallel decomposition confirmation | Skip — proceed with best decomposition |
| Phase boundary check-in | Skip — continue to next phase |
| Minor ambiguity | Make reasonable call, log to REVIEW-LOG.md |
| ESCALATION condition | Always stop — never skip escalation |
| DOD failure | Always fix — never skip DOD |

Escalation and DOD are never suppressed by unattended mode.

## Resume Protocol

When resuming after cap:

1. Read MISSION.md `autonomous_run` block
2. Load `current_phase` and `current_task`
3. Load `completed_phases` — skip these
4. Resume from `current_task` within `current_phase`
5. If `current_task` is ambiguous, restart the current phase from the beginning (safe — phases should be idempotent)
6. Log resume event to REVIEW-LOG.md:
   ```
   [timestamp] RESUMED after cap interruption. Resuming phase [N] at: [current_task]
   ```

## Context Checkpointing

Long runs outlive their context. At every phase boundary, rewrite `.gir/CLAUDE-activeContext.md` — current phase, decisions made, next steps — so a fresh session can resume from files alone.

- On any resume, treat `.gir/` file state as authoritative over remembered conversation
- Context feels heavy mid-run → checkpoint first, then continue
- Never checkpoint after compaction — by then detail is already lost

## Idempotency Requirement

Phases must be safe to re-run. autonomous-executor should:
- Check if work is already done before redoing it
- Use existence checks (file exists? test passes?) before re-executing
- Never create duplicate output from a re-run
