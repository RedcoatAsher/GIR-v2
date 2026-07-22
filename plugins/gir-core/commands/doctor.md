# /doctor

Deterministic GIR ecosystem health check. Every check below is an exact command with an expected result — run them as written, report PASS/FAIL, do not improvise substitutes.

---

## Instructions

Follow these steps in order:

### Step 1: Detect context

Check whether `.claude-plugin/marketplace.json` exists in the current directory.

- **Exists** → this is the GIR development repo. Run Step 2 and Step 3.
- **Does not exist** → this is a user project. Run Step 2 only.

### Step 2: Project checks (any project)

**Check 2.1 — Memory bank present**

```bash
ls .gir/ 2>/dev/null
```

- `.gir/` missing entirely → SKIP: memory bank is optional (see README). Remedy if the project wants one: `run /gir:init-memory-bank`
- `.gir/` exists → verify the core files: `MISSION.md`, `CLAUDE-activeContext.md`, `ESCALATION.md`, `DOD.md`, `REVIEW-LOG.md`. Any missing → FAIL, list them (a partial memory bank is worse than none).

**Check 2.2 — Module registry coherent**

Read `.gir/GIR.modules` (missing → FAIL with remedy: restart the session so hooks regenerate it). For each `### <module> (vX.Y.Z)` entry, the module should correspond to an installed GIR plugin. Stale entries from uninstalled spokes are harmless (WARN, not FAIL) — never delete them yourself.

**Check 2.3 — Escalation conditions defined**

```bash
[ -f .gir/ESCALATION.md ] && grep -c "Must Escalate" .gir/ESCALATION.md || echo "MISSING"
```

`.gir/ESCALATION.md` missing → WARN: no escalation file, autonomous runs will have no stop conditions. File present but count is zero → WARN: escalation gates are unset. Count ≥1 → PASS.

### Step 3: Repo checks (GIR development repo only)

**Check 3.1 — Manifest JSON validity**

```bash
for f in .claude-plugin/marketplace.json plugins/*/.claude-plugin/plugin.json plugins/*/gir-module.json plugins/*/hooks/hooks.json plugins/*/.mcp.json; do
  [ -f "$f" ] && { jq empty "$f" 2>/dev/null || echo "INVALID: $f"; }
done
```

Expected: no `INVALID:` lines. Any line → FAIL, name the file.

**Check 3.2 — Version lock-step**

```bash
jq -r '.metadata.version' .claude-plugin/marketplace.json
jq -r '"\(input_filename): \(.version)"' plugins/*/.claude-plugin/plugin.json plugins/*/gir-module.json
grep -ho "v[0-9]\+\.[0-9]\+\.[0-9]\+" plugins/*/hooks/hooks.json | sort -u
```

Expected: every version equals `metadata.version`, except `gir-migrate` (independent track). Any other mismatch → FAIL, name file and version.

**Check 3.3 — Marketplace entries match plugin directories**

```bash
jq -r '.plugins[].source' .claude-plugin/marketplace.json | sed 's|^\./||' | sort
ls -d plugins/*/ | sed 's|/$||' | sort
```

Expected: identical lists. Difference → FAIL, name the orphan.

**Check 3.4 — Hook registry drift**

For gir-core: the agents, skills, and commands listed in the `hooks/hooks.json` GIR.modules entry must match `gir-module.json` `provides`. Diff deterministically:

```bash
prompt=$(jq -r '.hooks.SessionStart[0].hooks[0].prompt' plugins/gir-core/hooks/hooks.json)
diff <(jq -r '.provides.agents + .provides.skills + .provides.skills_on_demand + .provides.commands | .[]' plugins/gir-core/gir-module.json | sort) \
     <(echo "$prompt" | grep -E '^- \*\*(Agents|Skills|Commands)' | sed -E 's/^- \*\*[^*]+\*\*: //' | tr ', ' '\n' | grep -v '^$' | sort)
```

Non-empty diff → FAIL, list the missing/extra names (`<` = in gir-module.json only, `>` = in hooks.json only).

### Step 4: Report

Present one table, checks in order:

```text
GIR Doctor
──────────
  [PASS|FAIL|WARN|SKIP]  2.1 Memory bank present
  [PASS|FAIL|WARN|SKIP]  2.2 Module registry coherent
  [PASS|FAIL|WARN|SKIP]  2.3 Escalation conditions defined
  [PASS|FAIL|SKIP]       3.1 Manifest JSON validity
  [PASS|FAIL|SKIP]       3.2 Version lock-step
  [PASS|FAIL|SKIP]       3.3 Marketplace entries match directories
  [PASS|FAIL|SKIP]       3.4 Hook registry drift

Result: [HEALTHY | N failures]
```

Every FAIL gets one remedy line beneath the table. Do not fix anything automatically — report only.
