---
name: specgates
description: Spec-driven quality gates (SpecKit) for Claude Code. Enforces specification verification before implementation, test coverage requirements, and acceptance criteria checks. Use when building features that require formal specifications or quality assurance workflows.
---
# CLAUDE-specgates.md

<!-- speckit-version: 1.0.0 -->

> Spec-driven quality gates and conventions
>
> Inspired by GitHub Spec Kit. These are native GIR conventions, not SpecKit dependencies.

## Clarification Gate

**When**: Between Plan and Code phases. Before any non-trivial feature implementation begins.

**Skip for**: Single-file bug fixes, styling changes, copy updates, config changes.

Scan planned work against these 10 categories. Rate each Clear / Partial / Missing. Implementation MUST NOT start if any category is Missing.

| # | Category | What to Verify |
|---|---|---|
| 1 | Functional scope | All user actions defined? |
| 2 | Data model | Entities, fields, relationships specified? |
| 3 | UX flow | User paths from entry to completion documented? |
| 4 | Non-functional | Performance, security, accessibility targets set? |
| 5 | Integrations | External API contracts and error handling defined? |
| 6 | Edge cases | Failure modes and boundary conditions enumerated? |
| 7 | Constraints | Budget, timeline, tech constraints stated? |
| 8 | Terminology | Ambiguous terms defined consistently? |
| 9 | Completion signals | How do we know this feature is done? |
| 10 | State transitions | For stateful features: lifecycle documented? |

### `[NEEDS CLARIFICATION]` Markers

When a requirement is ambiguous during planning, mark it inline with `[NEEDS CLARIFICATION]`.

**Rules**:
- Max 3 per feature
- Priority: scope > security > UX > technical
- ALL must be resolved before coding begins
- Use `AskUserQuestion` to resolve — never guess

---

## Acceptance Criteria Convention

Every non-trivial feature MUST include Given/When/Then acceptance scenarios. These are the definition of done.

**Format** (use inline in TodoWrite items or task descriptions):
```markdown
## Feature: [Name]
### Acceptance
- Given [precondition]
  When [action]
  Then [expected outcome]
- Given [edge case]
  When [action]
  Then [expected outcome]
```

**Rules**:
- Minimum 2 scenarios per feature (happy path + one edge case)
- Each scenario must be independently testable
- At least one negative/error scenario for user-facing features
- Reference specific business rules from .gir/CLAUDE-activeContext.md where applicable

---

## Task Format Conventions

### Parallel Safety Markers

Mark parallelizable tasks with `[P]`. Always include target file paths.

```markdown
- [ ] [P] Implement feature A — src/components/FeatureA.tsx
- [ ] [P] Add analytics metrics — src/pages/analytics.tsx
- [ ] Shared middleware update — src/middleware.ts (NOT parallel — shares file)
```

Tasks without `[P]` are sequential by default. Tasks marked `[P]` MUST NOT share files with other `[P]` tasks.

### Phased Task Structure

For multi-task features, organize into 4 phases:

| Phase | Name | Execution | Contents |
|---|---|---|---|
| 1 | **Setup** | Sequential | Dependencies, config, scaffolding |
| 2 | **Foundational** | Sequential, blocking | DB migrations, shared types, API contracts |
| 3 | **User Stories** | Parallelizable `[P]` | Feature work grouped P1→P2→P3 |
| 4 | **Polish** | Sequential | Cross-cutting improvements, cleanup |

**Rule**: No Phase 3 work begins until Phase 2 is complete.

**Example**:
```markdown
## Phase 1: Setup
- [ ] Install dependencies — package.json

## Phase 2: Foundational (blocking)
- [ ] Create DB migration — migrations/
- [ ] Add shared types — types/

## Phase 3: User Stories
- [ ] [P] [US1] Feature page — src/pages/feature.tsx
- [ ] [P] [US2] Settings panel — src/pages/settings.tsx
- [ ] [US3] API handler — src/api/handler.ts

## Phase 4: Polish
- [ ] Verify security policies
- [ ] Update .gir/CLAUDE-activeContext.md
```

---

## State Machine Convention

For any feature with >3 states, document the state machine BEFORE implementation.

**Format**:
```text
[state_a] --trigger--> [state_b]
[state_b] --trigger--> [state_c]
```

Then add a table showing user-facing impact per state:
```markdown
| State | What user sees | What's allowed | Notes |
|---|---|---|---|
| state_a | ... | ... | ... |
```

Projects should maintain a `CLAUDE-statemachines.md` file for their state machine reference.

---

## spec-analyst Agent

Cross-artifact consistency auditor. Detects drift, gaps, and contradictions across GIR docs.

**When**: Before phase transitions, after major scope changes, when docs feel stale.

**Process**:
1. Read: ROADMAP.md, active TASKS.md files, .gir/CLAUDE-decisions.md, .gir/CLAUDE-activeContext.md
2. Run 6 analysis passes (cap at 30 findings):
   - **Coverage**: Every ROADMAP item has corresponding task entries
   - **Consistency**: No terminology drift between docs
   - **Decision compliance**: Patterns match .gir/CLAUDE-decisions.md
   - **Orphans**: No task items that don't trace to a roadmap item
   - **Staleness**: No completed items still listed as pending
   - **Business rule alignment**: .gir/CLAUDE-activeContext.md rules reflected in task acceptance criteria
3. Output findings table with severity (CRITICAL / HIGH / MEDIUM / LOW)
4. Suggest specific fixes with file:line references

**Output format**:
```markdown
| # | Severity | File | Finding | Suggested Fix |
|---|---|---|---|---|
| 1 | CRITICAL | ROADMAP.md:42 | Feature X has no task entries | Create tasks |
| 2 | HIGH | .gir/CLAUDE-activeContext.md:125 | Terminology drift | Standardize |
```

**Principles**: Strict read-only. Never edit files during analysis. Cite file:line for every finding.

**Invocation**: "Run spec-analyst" or "Audit GIR docs for consistency"

---

**Updated**: 2026-02-22
