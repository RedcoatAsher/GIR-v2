---
name: parallel-agents
description: Native parallel agent execution using Claude Code's built-in Agent tool with worktree isolation. On-demand skill — load when parallel work is needed.
---

# Parallel Agents

Execute independent tasks concurrently. Each agent runs in an isolated git worktree via Claude Code's native `isolation: "worktree"` parameter — no external tools required.

## When to Use

Trigger automatically when routing-stub detects parallel trigger phrases, or when:
- 2+ truly independent tasks that modify files
- Tasks with no shared file overlap
- Work that can be verified independently before merging

Do NOT parallelize:
- Tasks that share files (race condition)
- Tasks where one depends on the other's output
- Single-file edits under 50 lines
- Tasks requiring shared state decisions

## Decomposition Rules

Split by **problem domain**, not by file count:

| Good split | Bad split |
|---|---|
| Auth flow vs. UI components | File A vs. File B (arbitrary) |
| API layer vs. test suite | First half vs. second half |
| Feature impl vs. docs | Random distribution |

Maximum useful parallelism: **4 agents**. Beyond that, coordination cost exceeds benefit.

## Dispatch Pattern

Spawn all agents in a single message (enables true concurrency):

```
Agent({ prompt: "[focused task A]", isolation: "worktree" })
Agent({ prompt: "[focused task B]", isolation: "worktree" })
Agent({ prompt: "[focused task C]", isolation: "worktree" })
```

`isolation: "worktree"` gives each agent its own git worktree automatically — no manual branch or worktree management needed.

## Agent Prompt Template

Each agent prompt must include:

```markdown
## Task
[Single focused objective — one domain only]

## Scope
[Exact files/directories this agent may touch]

## Must NOT touch
[Files owned by other parallel agents]

## Context
[Only what this agent needs — no unrelated history]

## Completion Gate
Before reporting done:
1. Verify your changes pass lint/typecheck/tests in your scope
2. Check .gir/DOD.md — confirm all applicable items pass
3. Check .gir/ESCALATION.md — if any condition is met, stop and report BLOCKED

## Return format
- Status: DONE | DONE_WITH_CONCERNS | BLOCKED
- Summary: what you did
- Files changed: list
- Concerns (if any): what needs human review
```

## After Agents Return

1. Read each agent's status and summary
2. If any agent returned BLOCKED → escalate to user before merging
3. If any agent returned DONE_WITH_CONCERNS → flag for human review
4. Check for file conflicts across agents (should be none if decomposed correctly)
5. Merge worktrees — Claude Code handles this automatically when agents complete
6. Run full DOD check on merged result

## DOD Gate

Before reporting parallel work complete, check `.gir/DOD.md`. All items must pass on the merged result, not just per-agent.

## Staleness

If agents were dispatched >30 minutes ago without response, re-check status before merging. Stale worktrees can be listed with `git worktree list`.
