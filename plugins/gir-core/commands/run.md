# /gir:run

Execute the active plan in MISSION.md autonomously. Agents are dispatched, monitored, and coordinated without human confirmation at each step.

---

## Instructions

### Step 1: Load autonomous-mode skill

Load the `autonomous-mode` skill before proceeding.

### Step 2: Read MISSION.md

Read `.gir/MISSION.md`. Extract:
- Active priorities / plan phases
- `autonomous_run` block if it exists (resume scenario)
- `unattended` setting

If no plan exists in MISSION.md, respond:
```
No plan found in .gir/MISSION.md.

Add your plan under ## Active Priorities with numbered phases, then run /gir:run again.
```
Stop.

### Step 3: Confirm or resume

**If `autonomous_run.status` is `in-progress`** (resume scenario):
- Show current state: phase, completed, next task
- Auto-resume if `resume_after_cap: true`, otherwise ask user

**If fresh run**: show the plan and confirm:
```
Autonomous Run
──────────────
Phases: [N]
Phase 1: [description]
Phase 2: [description]
...

Unattended mode: [yes/no]
Cap fail-safe: [enabled/disabled]

Starting in 5 seconds. Ctrl+C to cancel.
```
Wait 5 seconds (note only — do not actually pause; proceed immediately after displaying).

### Step 4: Initialize run state

Write to `.gir/MISSION.md` autonomous_run block:
```
autonomous_run:
  status: in-progress
  unattended: true
  resume_after_cap: true
  started_at: [ISO timestamp]
  current_phase: 1
  completed_phases: []
  next_task: [first task description]
  checkpoint_at: [ISO timestamp]
  scheduled_resume_id: pending
```

### Step 5: Schedule cap fail-safe

If `resume_after_cap: true`, use the `schedule` skill to create a one-time remote agent that fires 70 minutes from now with this prompt:

```
Check /Volumes/[project]/.gir/MISSION.md autonomous_run block.
If status is "in-progress" and started_at is more than 60 minutes ago:
  Resume the autonomous run by running /gir:run.
  The previous session likely hit the usage cap.
If status is "complete" or "idle": do nothing and exit.
```

Write the scheduled agent's ID to `autonomous_run.scheduled_resume_id` in MISSION.md.

### Step 6: Hand off to autonomous-executor

Pass to `autonomous-executor` agent:
- Full plan from MISSION.md
- Current autonomous_run state
- Contents of .gir/DOD.md
- Contents of .gir/ESCALATION.md
