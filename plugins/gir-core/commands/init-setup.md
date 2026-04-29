# /gir:init-setup

Run full GIR project setup — generates CLAUDE-project.md and scaffolds the .gir/ memory bank.

Runs /gir:init-project followed by /gir:init-memory-bank.

---

## Instructions

Follow these steps in order. Do not skip steps.

### Step 1: Check for existing setup

Before starting, check what already exists in the current directory:

- Does `CLAUDE-project.md` exist?
- Does `.gir/` exist?

If both exist, inform the user:

```
Both CLAUDE-project.md and .gir/ already exist in this directory.

Run /gir:init-project to regenerate CLAUDE-project.md (will overwrite).
Run /gir:init-memory-bank to re-scaffold .gir/ (will not overwrite existing files).
Run /gir:init-setup to do both.

Proceed? (yes / no)
```

Wait for confirmation before continuing if both exist. If the user says no, stop.

If only one exists, note it and proceed — you will skip the step for whichever already exists, unless the user confirms they want to overwrite.

If neither exists, proceed immediately.

### Step 2: Run /gir:init-project

Follow all steps in the /gir:init-project command:
- Detect tech stack
- Ask user to confirm or override detected values
- Generate CLAUDE-project.md

Do not proceed to Step 3 until CLAUDE-project.md is written.

### Step 3: Run /gir:init-memory-bank

Follow all steps in the /gir:init-memory-bank command:
- Scaffold all 11 .gir/ template files
- Add .gir/ to .gitignore if not already present
- Confirm files written

### Step 4: Summary

When both steps complete, show:

```
GIR Setup Complete
------------------

  CLAUDE-project.md    created ✓
  .gir/                scaffolded ✓ (11 files)

Next steps:
  1. Edit CLAUDE-project.md — fill in project description, audience, and any custom conventions
  2. Edit .gir/MISSION.md — set your active priorities for this session cycle
  3. Edit .gir/ESCALATION.md — adjust must-escalate thresholds for your project
  4. Edit .gir/DOD.md — adjust completion criteria for your project
  5. Run /gir:status to verify everything is configured
```
