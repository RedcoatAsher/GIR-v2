# /gir-migrate:migrate-from-gsd

Migrate a GSD 2.0 project (`.gsd/`) to GIR format (`.gir/`). Reads all GSD files, transforms them to GIR's schema, writes `.gir/` files, and produces a migration log. Never modifies or deletes `.gsd/`.

Supports dry-run mode: `/gir-migrate:migrate-from-gsd --dry-run`

---

## Dry-Run Mode

When invoked as `/gir-migrate:migrate-from-gsd --dry-run`:

- Run Phases 1 and 2 (Discovery + Parse) normally
- Run Phase 3 (Transform) in memory — do not write any files
- Skip Phases 4 and 5 entirely (no conflict detection, no writes)
- Output a full **Migration Preview Report** (see below) instead of Phase 6

**Nothing is written to disk in dry-run mode.** Not even the `.gsd/.gir-migration-done` marker.

### Migration Preview Report

After dry-run parsing and transform, output this report:

```
GSD 2.0 → GIR Migration Preview (DRY RUN — no files written)
═════════════════════════════════════════════════════════════

Source: .gsd/  ([N] files found)
Target: .gir/  ([exists with N files / will be created])

FILES THAT WOULD BE WRITTEN
────────────────────────────
.gir/MISSION.md
  Source: MILESTONE-ROADMAP.md + PLANNING-STATUS.md + COMPLETE-TASK-PLANNING.md
  Content: [N] active milestones as phases, [N] completed milestones in history, QUEUE deferred section
  Conflict: [none / "file exists — GSD content would be appended under ## Migrated from GSD 2.0"]

.gir/CLAUDE-decisions.md
  Source: DECISIONS.md
  Content: [N] decisions (D001–DXXX) reformatted to GIR heading style
  Conflict: [none / "file exists — decisions would be appended"]

.gir/CLAUDE-patterns.md
  Source: codebase/CONVENTIONS.md + KNOWLEDGE.md (pattern entries) + research/SUMMARY.md
  Content: [N] conventions, [N] experience patterns, [N] research-derived patterns
  Conflict: [none / ...]

.gir/CLAUDE-troubleshooting.md
  Source: KNOWLEDGE.md (gotcha/bug/fix entries)
  Content: [N] entries across [N] sections
  Conflict: [none / ...]

.gir/CLAUDE-resources.md
  Source: research/SUMMARY.md + research/STACK.md + codebase/INTEGRATIONS.md
  Content: [N] URLs, [N] recommended library versions, [N] integration patterns
  Conflict: [none / ...]

.gir/CLAUDE-activeContext.md
  Source: state-manifest.json + completed-units-*.json + journal/ (last events)
  Content: Current milestone [X], in-progress slice [Y], [N] recent completions
  State quality: [good — state-manifest readable / partial — state-manifest truncated / limited — journal only]
  Conflict: [none / ...]

.gir/DOD.md
  Source: REQUIREMENTS.md
  Content: [N] active requirements as checklist items, [N] validated items noted
  Dropped: [N] out-of-scope requirements (see Data Loss section below)
  Conflict: [none / ...]

.gir/POLICY.md
  Source: preferences.md + CLAUDE.md / CLAUDE-project.md git config
  Content: git main_branch, model preferences, project conventions
  Conflict: [none / ...]

.gir/ESCALATION.md
  Source: REQUIREMENTS.md (constraint/compliance class) + codebase/CONCERNS.md
  Content: [N] escalation conditions derived from constraints
  Note: [scaffolded with defaults / [N] conditions found]
  Conflict: [none / ...]

.gir/REVIEW-LOG.md
  Source: journal/*.jsonl (significant events) + milestone summaries
  Content: [N] events extracted, [N] milestone completions
  Conflict: [none / ...]

CLAUDE-project.md
  Source: codebase/STACK.md + codebase/ARCHITECTURE.md
  Action: [gaps filled — N fields would be added / "file not found — run /gir:init-project first"]

.gir/GSD-MIGRATION-LOG.md  [new file]
  Content: Full transform decision trail, dropped data inventory, conflict resolutions


DATA THAT WOULD BE DROPPED
────────────────────────────
The following GSD data has no GIR equivalent and will NOT be migrated:

  routing-history.json          — execution analytics (no GIR equivalent)
  reports/*.html ([N] files)    — historical visual reports (artefacts only)
  doctor-history.jsonl          — GSD-internal diagnostics (no GIR equivalent)
  event-log.jsonl (bulk)        — only [N] significant events extracted; raw log not preserved
  journal/*.jsonl (bulk)        — only last 100 lines + significant events per file; full history not preserved
  gsd.db (raw tables)           — [extracted N rows from N tables / sqlite3 not available — raw DB not read]

  Requirements dropped (Out-of-scope):
  [list each requirement ID and description that would be dropped]

POTENTIAL DATA LOSS RISKS
──────────────────────────
  [List any specific risks found during parse, e.g.:]

  ⚠  state-manifest.json was [N]KB — only partial read. Current task state may be incomplete.
     Impact: CLAUDE-activeContext.md in-progress section may be missing recent work.
     Mitigation: Review .gir/CLAUDE-activeContext.md after migration and fill gaps manually.

  ⚠  [N] milestone variants found (M011-15meep, M011-8vjslx, etc.)
     Impact: Variant execution history partially merged into base milestone. Some task detail may be lost.
     Mitigation: Originals preserved in .gsd/milestones/ — reference if needed.

  ⚠  journal/[date].jsonl had [N] lines — only last 100 read.
     Impact: Earlier execution events not captured in REVIEW-LOG.md.
     Mitigation: Full journal preserved in .gsd/journal/ — reference for deep history.

  [If no risks:]
  ✓  No significant data loss risks detected.

SUMMARY
────────
  Files that would be written:     [N]
  Files with conflicts (append):   [N]
  Data items preserved:            ~[N] (decisions, requirements, knowledge entries, tasks)
  Data items dropped:              ~[N] (see above)
  Estimated completeness:          [high ~95% / medium ~80% / limited ~60%]

To run the actual migration:
  /gir-migrate:migrate-from-gsd

To uninstall after migrating:
  claude plugin uninstall gir-migrate
```

---

## Instructions

Follow all phases in order. Do not skip phases.

---

### Phase 1: Discovery

**1.1** Check whether `.gsd/` exists in the current working directory.

If not found:
```
No .gsd/ directory found in [current directory].

gir-migrate only migrates GSD 2.0 projects. If your GSD directory is elsewhere,
run this command from the project root that contains .gsd/.
```
Stop.

**1.2** Scan `.gsd/` and report what was found before proceeding:

```
GSD 2.0 Migration — Discovery
──────────────────────────────
Found .gsd/ in: [path]

Root files:
  [list each file with size if determinable]

Directories:
  milestones/   [N milestone folders found]
  research/     [exists / not found]
  codebase/     [exists / not found]
  quick/        [exists / not found]
  journal/      [N files]
  reports/      [exists / not found]

Special files:
  state-manifest.json   [found / not found] [size if found]
  gsd.db                [found / not found]
  routing-history.json  [found / not found]

Existing .gir/:           [exists with N files / not found]
Existing CLAUDE-project.md: [found / not found]

Proceed with migration? (yes / no)
```

Wait for user confirmation before proceeding.

---

### Phase 2: Parse

Read all source files. Rules:

**Markdown files** — read in full:
- `.gsd/PROJECT.md`
- `.gsd/REQUIREMENTS.md`
- `.gsd/DECISIONS.md`
- `.gsd/KNOWLEDGE.md`
- `.gsd/MILESTONE-ROADMAP.md`
- `.gsd/PLANNING-STATUS.md`
- `.gsd/COMPLETE-TASK-PLANNING.md`
- `.gsd/QUEUE.md`
- `.gsd/preferences.md`
- `.gsd/research/SUMMARY.md`
- `.gsd/research/STACK.md`
- `.gsd/research/ARCHITECTURE.md`
- `.gsd/research/FEATURES.md`
- `.gsd/research/PITFALLS.md`
- `.gsd/codebase/STACK.md`
- `.gsd/codebase/ARCHITECTURE.md`
- `.gsd/codebase/STRUCTURE.md`
- `.gsd/codebase/CONVENTIONS.md`
- `.gsd/codebase/INTEGRATIONS.md`
- `.gsd/codebase/CONCERNS.md`
- `.gsd/codebase/TESTING.md`
- All `milestones/MXXX-ROADMAP.md`, `MXXX-SUMMARY.md`, `MXXX-CONTEXT.md`, `MXXX-VALIDATION.md`
- All `milestones/MXXX/slices/SXX-PLAN.md`, `SXX-SUMMARY.md`, `SXX-UAT.md`
- All `quick/*/` markdown files

For files >100KB: read first 200 lines only, note truncation in migration log.

**JSON files** — targeted reads:
- `routing-history.json` — read fully (small)
- `completed-units-*.json` — read fully
- `state-manifest.json` — if >500KB: attempt to read first 300 lines to extract schema, then search for keys: `currentMilestone`, `currentSlice`, `currentTask`, `completedTasks`, `inProgressTasks`. Read surrounding context for each key found.
- `milestones/*/slices/*/tasks/T*.VERIFY.json` — read all (task completion state)
- `reports/reports.json` — read fully (small)

**JSONL files** — selective reads:
- Each `journal/*.jsonl`: read last 100 lines only
- `event-log.jsonl`: read last 100 lines only
- Scan for lines containing `"type":"task_complete"`, `"type":"decision"`, `"type":"milestone_complete"` — extract those lines regardless of position

**SQLite database** — attempt extraction:
```bash
sqlite3 .gsd/gsd.db ".schema"
```
If sqlite3 available: also run:
```bash
sqlite3 .gsd/gsd.db "SELECT name FROM sqlite_master WHERE type='table';"
```
Then for each table found, run:
```bash
sqlite3 .gsd/gsd.db "SELECT * FROM [table] LIMIT 50;"
```
If sqlite3 not available: note in migration log, skip.

---

### Phase 3: Transform

Apply these transforms. Track every decision in the migration log.

#### 3.1 → `.gir/MISSION.md`

Structure:
```markdown
# MISSION.md
> Migrated from GSD 2.0 on [date]. Source: .gsd/MILESTONE-ROADMAP.md + PLANNING-STATUS.md + COMPLETE-TASK-PLANNING.md

## Active Priorities

[For each incomplete milestone from MILESTONE-ROADMAP.md, in priority order:]
### Phase [N]: [Milestone name] ([Milestone ID])
[Description from milestone ROADMAP/SUMMARY]

Slices:
- [ ] [Slice name] — [task count] tasks, [hour estimate]
- [ ] ...

## Deferred

[Content from QUEUE.md — future milestones not yet planned]

## Completed This Cycle

[List completed milestones M001–M006 with one-line summary each]

## Autonomous Run State

autonomous_run:
  status: idle
  unattended: false
  resume_after_cap: false
  started_at: null
  current_phase: null
  completed_phases: []
  current_task: null
  checkpoint_at: null
  scheduled_resume_id: null
```

Rules:
- Milestones with all slices complete → Completed section
- Milestones with any incomplete slice → Active Priorities
- Preserve milestone IDs (M010, M011 etc.) in phase headings for traceability

#### 3.2 → `.gir/CLAUDE-decisions.md`

Source: `.gsd/DECISIONS.md`

Transform format:
```markdown
# CLAUDE-decisions.md
> Migrated from GSD 2.0 — [N] decisions preserved from .gsd/DECISIONS.md

[For each decision D001–DXXX:]
### [Date from When column] — [Decision title]
**Choice:** [Choice column]
**Rationale:** [Rationale column]
**Revisable:** [yes/no]
**GSD ID:** D[NNN] | Milestone: [When]
```

Rules:
- Preserve all decisions — none dropped
- If date is milestone reference (not calendar date), use the milestone as context: `### [M010] — [Title]`
- Preserve GSD IDs for traceability

#### 3.3 → `.gir/CLAUDE-patterns.md`

Sources: `.gsd/codebase/CONVENTIONS.md` + pattern entries from `.gsd/KNOWLEDGE.md` + `.gsd/research/SUMMARY.md`

Rules:
- `codebase/CONVENTIONS.md` → direct carry-over under `## Conventions`
- `KNOWLEDGE.md` entries matching: "pattern", "always", "never", "convention", "standard", "prefer", "use X not Y" → `## Patterns from Experience`
- `research/SUMMARY.md` recommendations section → `## Research-Derived Patterns`
- Add section header noting GSD origin

#### 3.4 → `.gir/CLAUDE-troubleshooting.md`

Source: `.gsd/KNOWLEDGE.md`

Rules:
- KNOWLEDGE.md entries matching: "bug", "error", "gotcha", "issue", "fix", "workaround", "broken", "fails", "warning", "caveat" → include
- Preserve section structure from KNOWLEDGE.md
- Add header: `> Migrated from GSD 2.0 .gsd/KNOWLEDGE.md`
- Entries that fit both patterns AND troubleshooting → copy to both files

#### 3.5 → `.gir/CLAUDE-resources.md`

Sources: `.gsd/research/SUMMARY.md`, `.gsd/research/STACK.md`, `.gsd/codebase/INTEGRATIONS.md`

Rules:
- Extract all URLs from these files → resources list
- Extract recommended library versions table from `research/STACK.md` → `## Recommended Versions`
- Extract integration patterns from `codebase/INTEGRATIONS.md` → `## Integration Patterns`

#### 3.6 → `.gir/CLAUDE-activeContext.md`

Sources: `state-manifest.json` (targeted keys), `completed-units-*.json`, last journal entries

Structure:
```markdown
# CLAUDE-activeContext.md
> Migrated from GSD 2.0 on [date]

## Last Updated
[date of migration]

## Current Goals
[Active milestone from state-manifest currentMilestone, or from PLANNING-STATUS.md]

## In Progress
[currentSlice + currentTask from state-manifest if found]
[Or: last task_complete event from journal]

## Recently Completed
[completed-units entries — last 10 significant completions]

## Next Steps
[First incomplete slice of first active milestone]

## Known Blockers
[Any BLOCKED status from task VERIFY.json files]
```

#### 3.7 → `.gir/DOD.md`

Source: `.gsd/REQUIREMENTS.md`

Rules:
- **Status: Active** requirements → checklist items: `- [ ] [Requirement description]`
- **Status: Validated** requirements → note section: `> Already validated: [list]`
- **Status: Out-of-scope** → dropped entirely (logged in migration log)
- Strip R-IDs from checklist items but add them as inline references: `- [ ] [Description] _(R012)_`
- Group by Class (core-capability first, then primary-user-loop, etc.)

#### 3.8 → `.gir/POLICY.md`

Source: `.gsd/preferences.md` + git config from `CLAUDE-project.md` or `CLAUDE.md`

```markdown
# POLICY.md
> Migrated from GSD 2.0 preferences.md

## Git
- Main branch: [from preferences.md git.main_branch]
- [Any other git conventions from CLAUDE.md or CLAUDE-project.md]

## Models
[Model preferences from preferences.md comments]

## Conventions
[Any project-level conventions from CLAUDE.md]
```

#### 3.9 → `.gir/ESCALATION.md`

Source: `.gsd/REQUIREMENTS.md` constraint-class requirements + `codebase/CONCERNS.md`

Rules:
- Requirements with class `compliance/security` or `constraint` → escalation candidates
- `codebase/CONCERNS.md` entries → review for escalation conditions
- Format as GIR escalation conditions: `Escalate when: [condition]`
- If no clear escalation conditions found: scaffold with GIR default template + note

#### 3.10 → `.gir/REVIEW-LOG.md`

Sources: `journal/*.jsonl` (significant events), `reports/reports.json`, completed milestone summaries

Rules:
- Extract `task_complete`, `milestone_complete`, `decision` event types from journal
- Format as: `[timestamp] [event_type]: [description]`
- Add completed milestone summaries from MXXX-SUMMARY.md files
- Most recent entries first

#### 3.11 → `CLAUDE-project.md` (update only)

Source: `.gsd/codebase/STACK.md` + `.gsd/codebase/ARCHITECTURE.md`

Rules:
- If `CLAUDE-project.md` already exists: compare tech stack sections. Fill any gaps from GSD codebase analysis. Never overwrite existing content — only add missing fields.
- If `CLAUDE-project.md` does not exist: note in migration log — run `/gir:init-project` first.
- Add `## Architecture Notes` section from `codebase/ARCHITECTURE.md` if not present.

---

### Phase 4: Conflict Detection

For each `.gir/` target file:

| Situation | Action |
|---|---|
| `.gir/` doesn't exist | Create directory, write all files |
| Target file doesn't exist | Write directly |
| Target file is empty or scaffold-only | Overwrite |
| Target file has real content | Append GSD content under `## Migrated from GSD 2.0` section — never overwrite |

Always show conflicts before writing:
```
Conflict detected: .gir/CLAUDE-decisions.md already has content.
Action: GSD decisions will be appended under "## Migrated from GSD 2.0" section.
```

---

### Phase 5: Write

Write all transformed files. Add `.gir/` to `.gitignore` if not already present (same as init-memory-bank).

Write `.gsd/.gir-migration-done` marker file:
```
migrated_at: [ISO timestamp]
gir_version: 2.2.0
files_written: [list]
```

This prevents accidental re-migration and gives init-memory-bank a signal to skip re-scaffolding files already written.

**Never modify or delete any file in `.gsd/`** — the only write inside `.gsd/` is the `.gir-migration-done` marker.

---

### Phase 6: Migration Report

Write `.gir/GSD-MIGRATION-LOG.md`:

```markdown
# GSD Migration Log
Migrated: [timestamp]
Source: .gsd/ (GSD 2.0)
Target: .gir/ (GIR v2.2.0)

## Files Written
[list each .gir/ file created/updated with source mapping]

## Transform Decisions
[Every non-obvious decision made during transform — e.g., "D045 had no date; used milestone M010 as context"]

## Conflicts Resolved
[Each conflict and how it was handled]

## Data Dropped
[What was not migrated and why]
| File | Reason |
|---|---|
| routing-history.json | No GIR equivalent — execution analytics |
| reports/*.html | Historical artefacts — no GIR equivalent |
| doctor-history.jsonl | GSD-internal diagnostic — no GIR equivalent |
| Out-of-scope requirements (R0XX, R0XX) | Not actionable in current phase |

## SQLite Extraction
[Result of gsd.db extraction attempt — tables found, data extracted, or "sqlite3 not available"]

## Truncated Files
[Any files that were too large and were read partially]

## Recommended Next Steps
1. Review .gir/MISSION.md — verify active phase ordering matches your priorities
2. Review .gir/DOD.md — confirm checklist items match your current definition of done
3. Review .gir/ESCALATION.md — add project-specific escalation thresholds
4. Run /gir:status to verify full GIR setup
5. Run /gir:drift-check to assess memory bank freshness
6. Uninstall gir-migrate: claude plugin uninstall gir-migrate
```

Print the uninstall reminder at the end of every successful migration:
```
Migration complete. .gsd/ was not modified.

To uninstall gir-migrate (recommended — you won't need it again):
  claude plugin uninstall gir-migrate
```
