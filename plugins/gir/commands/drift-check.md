# /drift-check

Assess whether the `.gir/` memory bank is stale and recommend what to update.

---

## Instructions

Follow these steps in order:

### Step 1: Check if `.gir/CLAUDE-activeContext.md` exists

Check whether `.gir/CLAUDE-activeContext.md` exists in the current project directory.

**If it does not exist**, respond with:

```
No memory bank found.

.gir/CLAUDE-activeContext.md does not exist in this directory.

To set up a memory bank, run:
  /gir:init-memory-bank

The memory bank helps Claude restore context at the start of each session and track decisions, patterns, and known issues across sessions.
```

Then stop — do not proceed further.

**If it does exist**, continue to Step 2.

### Step 2: Scan all memory bank files

Check which of the following files exist in `.gir/`:

- `CLAUDE-activeContext.md`
- `CLAUDE-decisions.md`
- `CLAUDE-patterns.md`
- `CLAUDE-resources.md`
- `CLAUDE-troubleshooting.md`

For each file that exists, read it.

### Step 3: Assess freshness

For each file that exists, determine its freshness:

**For `CLAUDE-activeContext.md`**:
- Look for a "Last Updated" date/time field near the top
- If it has a date: calculate how long ago that was relative to today
- If it has no date or only placeholder text (`[Date/Time]`): mark as "never updated"
- Check if "Current Goals", "In Progress", and "Next Steps" sections have real content or still contain template placeholders

**For `CLAUDE-decisions.md`**:
- Check if any real decision entries exist (look for actual dates in `### [YYYY-MM-DD]` headings)
- If the file only contains the template placeholder entry, mark as "empty"

**For `CLAUDE-patterns.md`**:
- Check if the naming conventions, component patterns, or other sections have been filled in
- If sections still contain only template placeholder text, mark as "empty"

**For `CLAUDE-resources.md`**:
- Check if any real URLs have been added (look for actual `https://` links beyond the template examples)
- If only placeholder URLs exist, mark as "empty"

**For `CLAUDE-troubleshooting.md`**:
- Check if any real issue entries exist
- If the file only contains the template structure with no real issues, mark as "empty"

### Step 4: Show status report

Present a clean summary:

```
Memory Bank Status
------------------

Files present:
  [checkmark or X]  CLAUDE-activeContext.md    [status: e.g., "updated 3 days ago" | "never updated" | "has placeholder text"]
  [checkmark or X]  CLAUDE-decisions.md        [status: e.g., "2 decisions logged" | "empty (template only)"]
  [checkmark or X]  CLAUDE-patterns.md         [status: e.g., "partially filled" | "empty (template only)"]
  [checkmark or X]  CLAUDE-resources.md        [status: e.g., "3 URLs added" | "empty (template only)"]
  [checkmark or X]  CLAUDE-troubleshooting.md  [status: e.g., "1 issue documented" | "empty (template only)"]

Missing files (if any):
  [list any of the 5 files that do not exist]
```

### Step 5: Provide specific recommendations

Based on what you found, suggest specific actions. Prioritize in this order:

1. **activeContext** — most critical; should be updated every session
2. **patterns** — important once the codebase has established conventions
3. **decisions** — important whenever architectural choices are made
4. **troubleshooting** — fill in as issues are encountered and resolved
5. **resources** — optional; add only if you rely on external references

Format recommendations as actionable items:

```
Recommendations
---------------

[Priority 1 — if activeContext is stale or empty]:
  CLAUDE-activeContext.md hasn't been updated recently.
  Update it now with:
  - Your current session goals
  - What's in progress
  - Any blockers or recent decisions

[Priority 2 — if patterns is empty]:
  CLAUDE-patterns.md is empty.
  As you develop conventions (naming, component structure, API patterns),
  document them here so Claude can follow them consistently.

[Priority 3 — if decisions is empty but the project seems mature]:
  No architectural decisions have been logged.
  If your project has made choices about state management, API design,
  database schema, etc., consider documenting the rationale in CLAUDE-decisions.md.

[If everything looks current]:
  Memory bank looks up to date. No urgent action needed.
  Consider reviewing CLAUDE-activeContext.md at the start of each new session.
```
