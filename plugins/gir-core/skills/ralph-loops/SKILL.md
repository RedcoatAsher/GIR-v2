---
name: ralph-loops
description: Continuous autonomous iteration patterns (Ralph Loops) for Claude Code. Enables self-directed build-test-fix cycles, progressive refinement, and autonomous quality improvement. Use when the user wants Claude to iterate autonomously on a task until completion.
---
# Ralph Loops

> Continuous AI iteration until completion criteria met

## What Are Ralph Loops?

Self-referential feedback loops where Claude:
1. Implements changes
2. Verifies results (tests, builds)
3. Analyzes failures, iterates
4. Repeats until criteria met or max iterations

Named after Ralph Wiggum — persistent iteration despite setbacks.

## Quick Start

### Claude Code CLI (Recommended)
```bash
claude code plugins install anthropics/claude-code/plugins/ralph-wiggum

/ralph-loop "Implement feature X.
Completion criteria:
- Tests pass (npm test)
- Build succeeds (npm run build)
Output <promise>COMPLETE</promise> when done."
--completion-promise "COMPLETE"
--max-iterations 20

/cancel-ralph  # If needed
```

### Manual Pattern (Any Interface)
```
I need you to work iteratively on this until completion.

[TASK]

Completion Criteria (verify each):
- [Criterion 1] (verify: [command])
- [Criterion 2] (verify: [command])

Process:
1. Implement changes
2. Run verification
3. If not passing, debug and iterate
4. If passing: check `.gir/DOD.md` if it exists — all items must pass — then output <promise>COMPLETE</promise>

Use TodoWrite to track iterations.
Keep iterating without permission until:
- All criteria met, OR
- 20 iterations (document blockers)

Start now.
```

## Templates

See [templates/RALPH-loop-prompts.md.template](templates/RALPH-loop-prompts.md.template)

| Template | Use Case |
|----------|----------|
| TDD Implementation | Test-driven development |
| Build Fix | Compilation/type errors |
| API Implementation | REST API with CRUD + tests |
| Refactor Until Quality | Quality metrics |
| Migration | Library/framework upgrades |
| Feature Until Deployed | End-to-end delivery |

## When to Use

**Good for**: TDD, build fixes, API implementations, long-running features (4-12h), migrations, quality improvements

**Not for**: Frequent design decisions, ambiguous requirements, production debugging, unpredictable iterations (>40)

## Best Practices

### Clear Completion Criteria

**Bad**: `Implement authentication`

**Good**:
```
Implement JWT authentication
Criteria:
- Login endpoint works (POST /api/auth/login)
- All tests pass (npm test -- auth)
- Build succeeds (npm run build)
```

### Max Iterations
- Simple (bug fixes): 10-15
- Medium (features, APIs): 15-25
- Complex (migrations): 25-40

### Verification Commands
```
npm test              # Tests
npm run build         # Build
npm run type-check    # TypeScript
npm run lint          # Quality
```

### Completion Promises

Before emitting any promise, check `.gir/DOD.md` if it exists. All DOD items must pass — do not emit `COMPLETE` if they do not.

```
<promise>COMPLETE</promise>
<promise>TESTS_PASSING</promise>
<promise>BUILD_CLEAN</promise>
<promise>API_READY</promise>
<promise>DEPLOYED</promise>
```

### Fallback Instructions
```
If stuck after 15 iterations:
1. Document blockers
2. List what was tried
3. Suggest alternatives
4. Output: <promise>BLOCKED</promise>
```

## Integration with E-P-C-C

```
Phase 1: Explore — Understand codebase
Phase 2: Plan — Define criteria, set verification
Phase 3: Code — RALPH LOOP HERE (iterate until complete)
Phase 4: Commit — Review (already verified), commit, deploy
```

## Expected Results

| Metric | Result |
|--------|--------|
| Success Rate | 6 repos overnight (YC hackathon) |
| Cost Efficiency | $50k contract for $297 API |
| Long-running | Programming language over 3 months |
| Typical | 5-20 iterations for medium features |

## Troubleshooting

**Not completing**: Break task smaller, make criteria specific, check commands, increase iterations

**Exits early**: Use unique promise tag, explicit criteria, add verification commands

**Stuck on same error**: Add fallback instructions, lower max iterations, switch to manual

## Decision Tree

```
Clear completion criteria? → No → Define first
Can verify automatically? → No → Don't use Ralph
Well-scoped (<40 iterations)? → No → Break into smaller
Benefits from autonomous iteration? → Yes → USE RALPH
```

## Resources

- [CLAUDE-delegation.md](CLAUDE-delegation.md) — Ralph as Tier 2 Recommended
- [CLAUDE-workflows.md](CLAUDE-workflows.md) — Workflow details
- Original: https://ghuntley.com/ralph/
- Plugin: https://github.com/anthropics/claude-code/tree/main/plugins/ralph-wiggum
- Orchestrator: https://github.com/mikeyobrien/ralph-orchestrator

**Updated**: 2026-01-12 | **Status**: Production-Ready
