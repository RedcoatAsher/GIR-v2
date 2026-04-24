# /init-memory-bank

Scaffold the `.gir/` memory bank directory in the current project with all 5 template files.

---

## Instructions

Follow these steps in order:

### Step 1: Check for existing `.gir/` directory

Check if `.gir/` already exists in the current project directory.

- If `.gir/` **does not exist**: proceed directly to Step 2.
- If `.gir/` **does exist**: warn the user and ask for confirmation before continuing:

  ```
  Warning: .gir/ already exists in this directory.

  Existing files:
  [list any files currently in .gir/]

  Proceeding will overwrite these files with fresh templates. Any customizations you've made will be lost.

  Continue? (yes/no)
  ```

  Wait for user confirmation. If the user says no or anything other than "yes", stop and do not create any files.

### Step 2: Create `.gir/` directory and write all 10 template files

Write each file with the exact template content shown below. Do not modify the template content — write it exactly as provided.

---

## Template Files

### File 1: `.gir/CLAUDE-activeContext.md`

```markdown
# Active Context

> **Session State**: [In Progress | Planning | Implementing | Reviewing | Debugging | Deployed]
> **Last Updated**: [Date/Time]

---

## Current Goals

1. [Primary goal for this session]
2. [Secondary goal]
3. [Tertiary goal]

---

## In Progress

### [Task Name]
- **Status**: [Not Started | In Progress | Blocked | Complete]
- **Branch**: [branch-name or "main"]
- **Files Modified**: [list key files changed]
- **Blockers**: [any blockers or issues preventing progress]

---

## Recent Decisions

| Decision | Rationale | Files Affected |
|----------|-----------|----------------|
| [What was decided] | [Why this approach] | [Which files changed] |

---

## Next Steps

- [ ] [Next immediate action]
- [ ] [Following action]
- [ ] [Cleanup or verification step]

---

## Notes

[Free-form notes, links to documentation, references, temporary observations]
```

---

### File 2: `.gir/CLAUDE-decisions.md`

```markdown
# Architecture Decisions

> Record of major architectural decisions and their rationale

---

## Decision Log

### [YYYY-MM-DD] [Decision Title]

**Status**: [Proposed | Accepted | Superseded | Deprecated]

**Context**:
[What situation led to needing this decision? What problem are we solving?]

**Decision**:
[What did we decide to do?]

**Rationale**:
[Why did we choose this approach over alternatives?]

**Alternatives Considered**:
1. **[Alternative 1]** — [Why this was rejected]
2. **[Alternative 2]** — [Why this was rejected]
3. **[Alternative 3]** — [Why this was rejected]

**Consequences**:
- Benefits:
  - [Positive outcome 1]
  - [Positive outcome 2]
- Trade-offs:
  - [Trade-off 1]
  - [Trade-off 2]
- Costs:
  - [Cost or limitation 1]
  - [Cost or limitation 2]

**Implementation Details**:
- **Files Affected**: [List key files]
- **Dependencies Added**: [Any new dependencies]
- **Configuration Changes**: [Config updates needed]

**Related Decisions**: [Links to related decision entries]
```

---

### File 3: `.gir/CLAUDE-patterns.md`

```markdown
# Code Patterns & Conventions

> Established coding patterns for this project

---

## File Organization

[Describe how files are organized in your project]

**Example**:
```
components/
├── ui/              # Reusable primitives (buttons, inputs)
├── features/        # Feature-specific components
└── layouts/         # Page layouts and wrappers
```

---

## Naming Conventions

- **Components**: [e.g., PascalCase, kebab-case]
- **Files**: [e.g., ComponentName.tsx, component-name.ts]
- **Functions**: [e.g., camelCase, snake_case]
- **Constants**: [e.g., SCREAMING_SNAKE_CASE, UPPER_CASE]
- **CSS Classes**: [e.g., kebab-case, camelCase, BEM]

---

## Component Patterns

### [Pattern Name]
**When to use**: [Description of when this pattern applies]

**Structure**:
```[language]
[Code example showing the pattern]
```

**Example Usage**:
```[language]
[Real-world usage example]
```

---

## State Management

[Describe how state is managed in your project]

**Patterns**:
- **Local state**: [When and how to use useState, etc.]
- **Global state**: [Context, Zustand, Redux, etc.]
- **Server state**: [React Query, SWR, etc.]

---

## API Patterns

[Describe API conventions]

**Request Format**:
```[language]
[Example API request structure]
```

**Response Format**:
```[language]
[Example API response structure]
```

**Error Handling**:
```[language]
[How errors are handled and structured]
```

---

## Testing Patterns

[Describe testing approach]

**Unit Tests**:
- Location: [Where unit tests live]
- Naming: [Test file naming convention]
- Structure: [How tests are organized]

**Integration Tests**:
[Integration test patterns]

---

## Styling Patterns

[Describe styling conventions]

**Approach**: [Tailwind, CSS Modules, styled-components, etc.]

**Examples**:
```[language]
[Example of proper styling usage]
```

**Don'ts**:
```[language]
[Example of anti-patterns to avoid]
```

---

## Import Order

[Standardized import order]

**Example**:
```typescript
// 1. External libraries
import React from 'react'
import { useState } from 'react'

// 2. Internal libraries / utils
import { cn } from '@/lib/utils'

// 3. Components
import { Button } from '@/components/ui/button'

// 4. Types
import type { User } from '@/types'

// 5. Styles (if applicable)
import styles from './Component.module.css'
```

---

## Error Handling

[How errors should be handled]

**Client-side**:
```[language]
[Error handling pattern for client code]
```

**Server-side**:
```[language]
[Error handling pattern for server/API code]
```

---

## Performance Patterns

[Performance best practices]

- **[Pattern]**: [Description]
- **[Pattern]**: [Description]

---

## Accessibility Patterns

[A11y conventions]

- **[Pattern]**: [Description]
- **[Pattern]**: [Description]
```

---

### File 4: `.gir/CLAUDE-resources.md`

```markdown
# CLAUDE-resources.md

> **External references for on-demand fetching** — Use WebFetch to retrieve when relevant

---

## Purpose

This file stores URLs and references that Claude can fetch on-demand when relevant to the current task. Unlike CLAUDE-project.md (always loaded), these resources are fetched only when needed to save context tokens.

**When to use this file**:
- Competitive intelligence and market research
- External API documentation not covered by Context7
- Industry standards and specifications
- Reference architectures and design patterns
- Third-party analysis and blog posts

**When NOT to use this file**:
- Library/framework docs (use Context7 MCP instead)
- Project-specific details (use CLAUDE-project.md)
- Internal documentation (use CLAUDE-patterns.md or CLAUDE-decisions.md)

---

## Competitive Intelligence

### [Category Name]

| Resource | Description |
|----------|-------------|
| [Resource Title](https://example.com/resource) | Brief description of what this resource contains |
| [Another Resource](https://example.com/another) | Description of content and why it's relevant |

---

## Industry Standards

### [Standard Category]

| Resource | Description |
|----------|-------------|
| [Standard Name](https://example.com/standard) | What this standard covers |

---

## Reference Architectures

### [Architecture Type]

| Resource | Description |
|----------|-------------|
| [Architecture Guide](https://example.com/arch) | Architecture patterns and recommendations |

---

## External APIs

### [API Category]

| Resource | Description |
|----------|-------------|
| [API Docs](https://example.com/api) | API reference not available via Context7 |

---

## How Claude Uses This File

1. **Discovery**: When working on related tasks, Claude reads this file to find relevant resources
2. **Fetch on-demand**: Claude uses WebFetch tool to retrieve specific URLs when needed
3. **Context-efficient**: URLs are only fetched when directly relevant, saving context tokens

**Example Claude behavior**:
```
User: "How do competitors handle X?"
Claude: *reads CLAUDE-resources.md* → *finds relevant competitor docs* → *WebFetch to retrieve* → *summarizes findings*
```
```

---

### File 5: `.gir/CLAUDE-troubleshooting.md`

```markdown
# Troubleshooting Guide

> Known issues and proven solutions

---

## Common Issues

### [Issue Title]

**Symptoms**:
- [Observable symptom 1]
- [Observable symptom 2]
- [Observable symptom 3]

**Root Cause**:
[Explanation of what's actually causing this issue]

**Solution**:
```bash
[Commands to run or steps to fix]
```

**Alternative Solutions**:
1. [Alternative fix 1]
2. [Alternative fix 2]

**Prevention**:
[How to avoid this issue in the future]

**Related Files**: [List files involved]

**Last Seen**: [Date]

**Frequency**: [Common | Occasional | Rare]

---

## Error Reference

Quick lookup table for common errors:

| Error Message | Solution | File |
|---------------|----------|------|
| [Error snippet] | [Quick fix] | [File location] |

---

## Build Issues

### [Build Issue Title]

[Same structure as Common Issues above]

---

## Deployment Issues

### [Deployment Issue Title]

[Same structure as Common Issues above]

---

## Development Environment Issues

### [Dev Environment Issue Title]

[Same structure as Common Issues above]

---

## Performance Issues

### [Performance Issue Title]

**Symptoms**:
- [Observable performance problem]

**Profiling Results**:
```
[Performance metrics or profiler output]
```

**Solution**:
[Optimization steps]

**Before/After Metrics**:
- Before: [Metric]
- After: [Metric]
- Improvement: [Percentage or absolute]

---

## Database Issues

### [Database Issue Title]

[Same structure as Common Issues above]

---

## Integration Issues

### [Third-Party Integration Issue Title]

**Service**: [e.g., Stripe, Auth0, Vercel, etc.]

**Symptoms**:
[Observable issues]

**Root Cause**:
[Explanation]

**Solution**:
[Steps to fix]

**Documentation Reference**: [Link to service docs]
```

---

### File 6: `.gir/MISSION.md`

```markdown
# Mission State

_Updated: [Date]_

## Active Mission
<!-- One sentence: what is this project trying to accomplish right now -->

## Active Priorities (ordered)
1. <!-- Highest priority deliverable -->
2. <!-- Second priority -->
3. <!-- Third priority -->

## In Progress
- [ ] Task — assigned to: [agent/human] — started: [Date]

## Blocked
<!-- Item — blocked on: [what] — since: [Date] -->

## Completed This Cycle
<!-- Item — completed: [Date] — commit: [hash] -->

## Out of Scope (do not work on)
<!-- Item — reason: [why excluded] -->

## Next Session: Start Here
<!-- Leave a clear instruction for the next session to pick up exactly where left off -->

## Unattended Operation Rules
- Maximum autonomous iterations before human check-in: [N]
- Stop and escalate if: [condition]
- Stop and wait if CI fails more than 2 consecutive times
```

---

### File 7: `.gir/ESCALATION.md`

```markdown
# Escalation Policy

## Autonomously Allowed
- Bug fixes (non-breaking)
- Refactoring within existing module boundaries
- Test additions or improvements
- Documentation updates
- Dependency bumps (patch or minor, non-breaking)
- Code review and feedback
- Additive schema migrations (new columns only, no removal or type changes)

## Notify Only (log to REVIEW-LOG.md, continue)
- New file added outside existing structure
- New dependency added
- New environment variable added
- Significant refactor spanning multiple modules
- New command or skill added

## Must Escalate (stop, log, wait for human)
- Any destructive data operation
- Breaking change to public API
- Security-sensitive change (auth, tokens, encryption, permissions)
- Production configuration or infrastructure change
- Scope expansion beyond original spec
- Irreversible action (delete, drop, truncate, force push)
- CI failure unresolvable after 2 attempts
- Conflict between specs that cannot be resolved by reading existing decisions
```

---

### File 8: `.gir/DOD.md`

```markdown
# Definition of Done

## Required for every merge
- [ ] Tests pass (CI green)
- [ ] Lint clean (zero errors)
- [ ] Typecheck clean
- [ ] Change matches spec scope — no unrequested additions
- [ ] No new TODO/FIXME without issue reference
- [ ] No secrets or credentials introduced

## Required for new features
- [ ] Tests cover new behavior
- [ ] Public API documented if changed
- [ ] Migration path documented if breaking

## Required for schema changes
- [ ] Migration is reversible or rollback plan documented
- [ ] No production data at risk

## Rollback Safety
Can this deploy be reverted in <15 minutes without data loss?
If no: document why before merging.
```

---

### File 9: `.gir/POLICY.md`

```markdown
# Repo Policy

## Language and Stack
<!-- e.g. TypeScript, Node 20, Postgres 15 -->

## Branching
<!-- e.g. feature/* → dev → prod, PRs required -->

## Commit Style
<!-- e.g. conventional commits: feat/fix/chore/refactor -->

## Testing Requirements
<!-- e.g. unit tests for all new functions, integration tests for API routes -->

## Code Review
<!-- e.g. self-review against DOD.md before any merge -->

## Environment Variables
<!-- e.g. all secrets in .env.local, never committed -->

## Performance Budgets
<!-- e.g. API responses < 200ms p95 -->

## Security Rules
<!-- e.g. no eval(), all inputs validated at boundary, parameterized queries only -->
```

---

### File 10: `.gir/REVIEW-LOG.md`

```markdown
# Review Log

| Date | Task | Outcome | Notes |
|------|------|---------|-------|
| [YYYY-MM-DD] | [Task description] | [Pass / Fail / Escalated] | [Why] |
```

---

### Step 3: Add `.gir/` to `.gitignore`

Check if `.gitignore` exists in the current directory.

- If it exists: read it and check if `.gir/` or `.gir` is already listed. If it is not, append `.gir/` to the file.
- If it does not exist: create a new `.gitignore` containing just `.gir/`.

### Step 4: Confirm success

After writing all files, report:

```
Memory bank initialized in .gir/

Files created:
  .gir/CLAUDE-activeContext.md   — Session state, current goals, and next steps
  .gir/CLAUDE-decisions.md       — Log of architectural decisions and their rationale
  .gir/CLAUDE-patterns.md        — Established code patterns and conventions
  .gir/CLAUDE-resources.md       — External URLs for on-demand fetching
  .gir/CLAUDE-troubleshooting.md — Known issues and proven solutions
  .gir/MISSION.md                — Active priorities and unattended operation state
  .gir/ESCALATION.md             — Must-escalate conditions
  .gir/DOD.md                    — Definition of done checklist
  .gir/POLICY.md                 — Repo policy and conventions
  .gir/REVIEW-LOG.md             — Review outcomes and audit trail

.gitignore: .gir/ added (memory bank stays local, not committed)

Next steps:
  1. Edit MISSION.md with your active priorities
  2. Review ESCALATION.md and adjust must-escalate thresholds for your project
  3. Review DOD.md and adjust completion criteria for your project
  4. Update CLAUDE-activeContext.md with your current session goals
  5. Fill in POLICY.md with project conventions

Tip: Run /gir-core:drift-check at any time to assess how fresh your memory bank is.
```
