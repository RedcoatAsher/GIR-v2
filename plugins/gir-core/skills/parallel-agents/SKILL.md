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
Agent({ prompt: "[focused task A]", isolation: "worktree", model: "haiku" })
Agent({ prompt: "[focused task B]", isolation: "worktree", model: "sonnet" })
Agent({ prompt: "[focused task C]", isolation: "worktree", model: "inherit" })
```

`isolation: "worktree"` gives each agent its own git worktree automatically — no manual branch or worktree management needed.

Choose each agent's model per routing-stub Model Routing — mechanical streams get `haiku`, implementation streams `sonnet` or inherit. Pass the chosen value in the `model` field of every `Agent(...)` call, as shown above.

## Agent Prompt Template

Each agent prompt must include:

````markdown
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
End your reply with exactly this fenced block:

```yaml
agent_result:
  status: DONE        # one of: DONE, DONE_WITH_CONCERNS, BLOCKED
  summary: "<one line>"
  files_changed: [<paths>]
  concerns: []            # required non-empty if status != DONE
  blocked_reason: null    # required if BLOCKED
  escalation_hit: false   # true if a .gir/ESCALATION.md condition was met
  retry_safe: true        # false if a re-run could duplicate side effects
```
````

## After Agents Return

1. Parse each agent's fenced `agent_result` block
2. If any agent returned BLOCKED → apply Failure Handling below
3. If any agent returned DONE_WITH_CONCERNS → flag for human review
4. Check for file conflicts across agents — unexpected overlap (should be none if decomposed correctly) is BLOCKED: stop, preserve the affected worktrees, resolve the conflict in the main session before merging
5. Merge worktrees — Claude Code handles this automatically when agents complete, once no unresolved overlap remains
6. Run full DOD check on merged result

## Failure Handling

- `escalation_hit: true` → never retry. Stop and escalate to the user.
- BLOCKED with `retry_safe: false` → skip the retry, go straight to fallback below.
- BLOCKED with `retry_safe: true` → retry once: fresh agent, fresh worktree, same Scope/Must-NOT-touch, failure summary appended to Context. Max 1 retry per stream, 2 per run.
- Retry fails, or `retry_safe: false` → fallback: absorb the stream into the main session and finish it sequentially. Never spawn a third agent for the same stream.
- More than half the streams failed → abandon the parallel run; finish sequentially or escalate.
- Log every retry and fallback to `.gir/REVIEW-LOG.md`: `[timestamp] parallel-retry|parallel-fallback: stream, attempt, reason`. Skip silently if `.gir/` isn't set up (memory bank is optional).

## DOD Gate

Before reporting parallel work complete, check `.gir/DOD.md`. All items must pass on the merged result, not just per-agent.

## Staleness

If agents were dispatched >30 minutes ago without response, re-check status before merging. Stale worktrees can be listed with `git worktree list`. Still unresponsive after the re-check → treat as a failed stream and apply Failure Handling. Never merge a worktree from an agent that returned no `agent_result` block without verifying its diff manually.
