# /gir:parallel

Decompose the current task into independent parallel workstreams and execute them concurrently using isolated agents.

---

## Instructions

Follow these steps in order:

### Step 1: Load the parallel-agents skill

Load `parallel-agents` skill before proceeding. It contains the full orchestration protocol.

### Step 2: Invoke the parallel-orchestrator agent

Hand off to the `parallel-orchestrator` agent with:
- Full task description
- Any relevant file context
- Contents of `.gir/DOD.md` if it exists
- Contents of `.gir/ESCALATION.md` if it exists

The parallel-orchestrator will handle decomposition, dispatch, isolation, and merge.
