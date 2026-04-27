---
name: state-machines
description: State machine templates and patterns for Claude Code. Provides structured state management for workflows, UI components, and process orchestration. Use when implementing stateful logic, workflow engines, or multi-step processes.
---
# State Machines

<!-- speckit-version: 1.0.0 -->

> Document state machines for features with >3 states. Synced via GIR — template is kept current; your project state machines are always preserved.

## How to Document State Machines

When implementing a feature with >3 states, add a new section in the PROJECT:state-machines section below following this template:

```markdown
## [Feature Name]

\`\`\`
[state_a] --trigger--> [state_b]
[state_b] --trigger--> [state_c]
\`\`\`

| State | [Impact Col 1] | [Impact Col 2] | Notes |
|---|---|---|---|

### Business Rules
- Rule 1
- Rule 2
```

### Checklist Before Adding

- [ ] Feature has >3 distinct states
- [ ] Each state has clear entry/exit conditions
- [ ] User-facing impact is documented per state
- [ ] Business rules reference activeContext.md where applicable
- [ ] At least one error/failure state is included

See CLAUDE-specgates.md for the full SpecKit convention.

---

