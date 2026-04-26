# GIR Setup Story — Phase 0–7

> How a new GIR repo gets hardened. Each phase is a discrete prompt. Run them in order on a fresh install.

---

## Phase 0 — Bootstrap

**Goal**: Install gir-core, scaffold memory bank, verify plugin wiring.

```
Run /init-memory-bank to scaffold .gir/ with 10 template files.
Customize: MISSION.md (active priorities), ESCALATION.md (must-escalate thresholds), DOD.md (completion criteria), POLICY.md (project conventions).
Verify: routing-stub and core-practices are in gir-module.json always-load list.
```

---

## Phase 1 — Modularization

**Goal**: Confirm hub-and-spoke structure is clean. No monolithic CLAUDE.md. Each plugin self-registers via SessionStart hook.

**Prompt template**:
```
Audit the GIR plugin structure. Verify:
- gir-core is the mandatory hub (always-load: routing-stub, core-practices)
- spoke plugins are optional and lazy-loaded
- no spoke docs are embedded in core-practices
- SessionStart hooks read MISSION.md + ESCALATION.md
Report findings. Fix any bleed between hub and spokes.
```

---

## Phase 2 — Delegation Cleanup

**Goal**: Consolidate routing and delegation rules into routing-stub. Delete standalone delegation skill if it exists.

**Prompt template**:
```
Audit delegation guidance across all always-loaded skills.
Merge any standalone auto-delegation skill into routing-stub.
Remove duplicate delegation rules from core-practices.
Verify Tier 1/2/3 delegation tiers are clearly defined in routing-stub only.
Gate checks (DOD + ESCALATION) must exist in all core agents before completion claims.
```

---

## Phase 3 — Reference Cleanup

**Goal**: Purge stale references (deleted skills, renamed files, old terminology).

**Prompt template**:
```
Search all plugin files for references to deleted or renamed skills.
Check: README files, gir-module.json, marketplace.json, hooks.json, plugin.json.
Remove dead references. Update terminology to match current skill names.
Do not change behavior — only remove stale pointers.
```

---

## Phase 4 — Graphify Integration

**Goal**: Add Graphify-aware context discipline to routing-stub. Make structural vs inline query decision deterministic.

**Prompt template**:
```
Run /graphify on the repo. Read GRAPH_REPORT.md.
Add a Graphify Rule to routing-stub/SKILL.md with:
- a corpus-size gate (skip Graphify if corpus fits in one context window and task touches ≤1 community)
- explicit "consult when" triggers: God Node edits, cross-community tasks, unfamiliar areas, dependency analysis
- explicit "skip when" conditions: known path, ≤1 community, policy/decisions/code correctness
- staleness rule: re-run /graphify after 7 days or major merges
Add a step to Session Start Protocol: check GRAPH_REPORT.md for structural tasks.
```

---

## Phase 5 — Hardening / Drift Normalization

**Goal**: Fix terminology drift, stale file links, and missing bridges between completion promises and DOD gate.

**Prompt template**:
```
Audit all always-loaded skills for:
- stale file links (files that no longer exist)
- outdated terminology (renamed sections, deleted skills referenced)
- missing DOD bridge: any completion promise (e.g. ralph-loops COMPLETE) that exits without checking .gir/DOD.md
Fix findings. No behavior changes — normalization only.
Verify each fix with a re-read before committing.
```

---

## Phase 6 — Rule Authority / Deduplication

**Goal**: One source per rule. Eliminate any rule stated in two places — it will drift.

**Prompt template**:
```
Audit always-loaded skills for duplicated rules.
Common candidates:
- session-start steps (routing-stub owns these — remove from core-practices)
- sequential-thinking triggers (routing-stub Tier 1 owns these — collapse in core-practices to a pointer)
- escalation/completion workflow steps (routing-stub owns — remove restatements elsewhere)
Replace duplicates with one-line pointers to the authoritative source.
No new behavior. Rule authority map: routing-stub owns execution rules; core-practices owns dev practices.
```

---

## Phase 7 — Operating Model Extraction

**Goal**: Capture the hardened setup as a reusable reference. New repos can follow the same arc.

**Prompt template**:
```
Create docs/OPERATING-MODEL.md: concise reference for session-start, routing, Graphify gate, escalation, DOD, and rule authority map.
Create docs/SETUP-STORY.md: Phase 0–7 arc with reusable prompt templates for each phase.
Source material: routing-stub/SKILL.md, core-practices/SKILL.md, commands/init-memory-bank.md.
No changes to core skills. Documentation only.
```

---

## Phase 8 — Codification

**Goal**: Wire the operating model into discoverable entry points and close any scaffold gaps surfaced by earlier phases.

**Prompt template**:
```
Link docs/OPERATING-MODEL.md and docs/SETUP-STORY.md from the gir-core README under an "Operating Model & Setup" section.
Add ESCALATION-LOG.md to the init-memory-bank scaffold as File 11 with a header + audit table.
Fix any stale file counts in init-memory-bank's intro and Step 2 header to match actual file count.
Confirm ESCALATION-LOG.md template does not duplicate resume instructions — those go to CLAUDE-activeContext.md per routing-stub.
Update MISSION.md to log Phase 8 complete.
No new features. Documentation and scaffold corrections only.
```

---

## Reuse Notes

- Run phases sequentially on a new repo. Skip any phase that doesn't apply (e.g. Phase 4 if you don't use Graphify).
- Phase 3 can be re-run any time to clean up stale references after major refactors.
- Phase 5 can be re-run after any large merge to catch drift.
- The operating model in `docs/OPERATING-MODEL.md` is the source of truth for how the repo works day-to-day.
