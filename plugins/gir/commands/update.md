# /gir:update

Check for updates to installed GIR plugins and apply them.

---

## Instructions

Follow these steps in order:

### Step 1: Check installed GIR plugins

Run the following to see what GIR plugins are currently installed and if updates are available:

```bash
claude plugin list
```

Read the output. Identify which GIR plugins are installed (gir, gir-web, gir-automation, gir-tools, gir-database, gir-ai, gir-qa) and whether any show available updates.

### Step 2: Show current state

Present a brief summary before updating:

```
GIR Plugin Update Check
-----------------------

Installed plugins:
  gir             v[current] → [latest or "up to date"]
  [other installed plugins...]

Plugins not installed:
  [list any GIR plugins not found]
```

### Step 3: Apply updates

Run:

```bash
claude plugin upgrade gir
```

If other GIR spoke plugins are installed, upgrade them too:

```bash
claude plugin upgrade gir-web        # if installed
claude plugin upgrade gir-automation # if installed
claude plugin upgrade gir-tools      # if installed
claude plugin upgrade gir-database   # if installed
claude plugin upgrade gir-ai         # if installed
claude plugin upgrade gir-qa         # if installed
```

Only run upgrade for plugins that are actually installed.

### Step 4: Confirm result

After upgrades complete, run:

```bash
claude plugin list
```

Report what was updated:

```
Update Complete
---------------

Updated:
  gir             v2.1.0 → v2.2.0
  [other updated plugins...]

Already up to date:
  [plugins that were already current]

Your CLAUDE-project.md and .gir/ files were not touched.
Memory bank files persist across updates.
```

If nothing needed updating, say so clearly.
