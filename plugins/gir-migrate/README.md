# gir-migrate

One-time migration tool. Converts GSD 2.0 project files (`.gsd/`) to GIR format (`.gir/`).

**Install once. Migrate. Uninstall.**

## Install

```bash
claude plugin install gir-migrate
```

## Run

From your project root (the directory containing `.gsd/`):

**Dry run first (recommended):**
```
/gir-migrate:migrate-from-gsd --dry-run
```

Reads and parses all GSD files, runs all transforms in memory, then produces a full preview report showing:
- Every file that would be written and what it would contain
- Exactly which GSD data maps to which GIR target
- Data that would be dropped (with reasons)
- Potential data loss risks with mitigations
- An estimated completeness score

Nothing is written to disk. Safe to run as many times as needed.

**Run the actual migration:**
```
/gir-migrate:migrate-from-gsd
```

The command will:
1. Discover all GSD 2.0 files
2. Ask for confirmation before writing anything
3. Transform and write to `.gir/`
4. Never modify or delete `.gsd/`
5. Produce a full migration log at `.gir/GSD-MIGRATION-LOG.md`

## Uninstall

After migration, uninstall — you won't need it again:

```bash
claude plugin uninstall gir-migrate
```

## What Gets Migrated

| GSD source | GIR target |
|---|---|
| `PROJECT.md` + milestone roadmap | `.gir/MISSION.md` |
| `DECISIONS.md` (all 99+ entries) | `.gir/CLAUDE-decisions.md` |
| `KNOWLEDGE.md` patterns | `.gir/CLAUDE-patterns.md` |
| `KNOWLEDGE.md` gotchas/bugs | `.gir/CLAUDE-troubleshooting.md` |
| `REQUIREMENTS.md` active items | `.gir/DOD.md` |
| `research/` + `codebase/` | `.gir/CLAUDE-resources.md` |
| `state-manifest.json` + journals | `.gir/CLAUDE-activeContext.md` |
| `preferences.md` | `.gir/POLICY.md` |
| Constraint requirements + concerns | `.gir/ESCALATION.md` |
| Journal events + milestone completions | `.gir/REVIEW-LOG.md` |
| `codebase/STACK.md` + `ARCHITECTURE.md` | `CLAUDE-project.md` (gaps filled) |

## What's Not Migrated

- `routing-history.json` — GIR has no equivalent analytics layer
- `reports/*.html` — historical visual artefacts
- `doctor-history.jsonl` — GSD-internal diagnostic tool
- Out-of-scope requirements — not actionable

## Safe to Re-run?

The command checks for `.gsd/.gir-migration-done` before writing. If migration was already run, it will warn and ask for explicit confirmation before overwriting.

## Requirements

- `gir` plugin must be installed (`claude plugin install gir`)
- `sqlite3` CLI recommended for full database extraction (optional — migration works without it)
