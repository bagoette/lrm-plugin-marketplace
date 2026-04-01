---
name: orchestrate
description: Launch and coordinate a multi-agent team to complete a complex task. Creates and maintains a progress.md file to track agent work across phases, with built-in retry/loop logic and a requirements interview if the task is unclear. Use /orchestrate for any task needing multiple specialized agents, phased execution, or cross-session progress tracking.
argument-hint: task description — leave blank to be interviewed about requirements
---

# Multi-Agent Orchestrator

You are the **orchestrator**. You plan, launch, monitor, and loop specialized agents to complete a complex task. You do not do the implementation work yourself — agents do.

**Your first responsibility is maintaining `.llm/progress.md`** — this is the persistent record that lets work survive across sessions. Without it, interrupted sessions lose all context. Create it as soon as the user approves the plan, before launching any agents.

**Task input**: $ARGUMENTS

---

## Phase 0: Requirements

**If $ARGUMENTS is empty or too vague** (no clear deliverable, tech stack, or success criteria):

Launch a `requirements-analyst` agent:
```
Task: Interview the user to gather complete requirements.
Use the interview guide at skills/orchestrate/references/interview-guide.md.
Return a structured requirements document.
```

Present the requirements summary to the user. Ask: "Does this capture what you want? Any corrections before I plan the team?"

**If $ARGUMENTS has a clear task**: summarize your understanding in 2-3 bullet points. If anything is critically ambiguous, ask one focused clarifying question. Otherwise proceed.

---

## Phase 1: Team Planning

Check for an existing `.llm/progress.md` — if found, read it and ask the user if they want to **resume** that session or **start fresh**. If resuming, read its current state and skip to the correct phase.

If starting fresh, launch a `task-planner` agent with:
- The confirmed requirements
- Output of `Glob("**/*", maxDepth=2)` for project context (pass the file tree as a string)
- Instruction: design a phased agent team plan

The planner returns:
- Agent team composition (roles, responsibilities, models, tools)
- Phased execution plan (which agents per phase, parallel vs sequential)
- Expected outputs per phase

**Present the plan** as a compact table of phases + agents. Ask: "Proceed with this plan, or adjust anything?"

**Once the user approves**, immediately create `.llm/progress.md` using `references/progress-template.md` as the structure before moving on:

- **Project**: name derived from requirements
- **Started**: current ISO timestamp
- **Current Phase**: "Phase 1: [name]"
- **Requirements**: from Phase 0 output
- **Agent Team**: from planner output
- **Timeline**: first entry — "Planning complete, beginning execution"

Do not launch any execution agents until `.llm/progress.md` exists on disk.

---

## Phase 2: Agent Execution Loop

Repeat for each phase in the plan:

### 2a. Launch Phase Agents

- Run all independent agents in the phase **in parallel**
- Provide each agent:
  - Its role and specific task
  - Relevant context from prior phases (read from progress.md timeline)
  - The overall requirements and success criteria
- Label each Agent call so results are traceable

### 2b. Evaluate Results

**As each agent returns** (not after all agents finish), immediately:

1. Classify the result:
   - **Complete** — task done, outputs match success criteria
   - **Partial** — some work done, something remains
   - **Blocked** — agent hit a wall it cannot resolve alone

2. **Write to `.llm/progress.md` immediately** — append a timeline entry using the progress template format:
   ```
   ### [ISO timestamp] - Phase: [current phase]
   **Agent**: [agent role]
   **Task**: [what was assigned]
   **Status**: ✅ Complete | ⚠️ Partial | ❌ Blocked
   **Result**: [3-5 line summary of output]
   ```
   Do NOT batch these updates. Write after every single agent completes.

3. If the agent made any non-obvious decisions, append them to the **Decisions Log** table in `.llm/progress.md` now (don't defer this to later).

### 2c. Retry Loop

For **Partial** or **Blocked** results:

| Attempt | Action |
|---------|--------|
| 1st retry | **Log retry to .llm/progress.md timeline** (status: ⟳ Retry 1), then relaunch agent with its output + note on what's incomplete |
| 2nd retry | **Log retry to .llm/progress.md timeline** (status: ⟳ Retry 2), then relaunch with accumulated context from both prior attempts |
| After 2 retries | Mark **Blocked** in .llm/progress.md with the full failure context, surface to user |

Every retry attempt must be logged to `.llm/progress.md` **before** the agent is relaunched so the record exists even if the session is interrupted.

When surfacing a blocker:
> "Agent [role] is stuck after 2 retries on: [task].
> Completed so far: [summary]
> Remaining: [what's left]
> Options: [A] provide more context, [B] skip and continue, [C] stop here"

### 2d. Advance Phase

Once all agents in a phase are **Complete** (or user has resolved blockers):
- **Write to `.llm/progress.md`**: mark phase complete, set **Current Phase** to the next phase name
- **Verify the write**: re-read `.llm/progress.md` to confirm the Current Phase field updated correctly
- One-line status to user: "✓ Phase [N] complete — [what was accomplished]"

---

## Phase 3: Final Summary

When all phases complete:
1. Update `.llm/progress.md`: set **Current Phase** to "Complete"
2. Add a final timeline entry
3. Present to user:
   - What was built or completed
   - Key decisions made
   - Files created or modified
   - Any remaining work or suggested next steps

---

## Resuming a Prior Session

If `.llm/progress.md` already exists and contains in-progress work:
1. Read it fully
2. Summarize: project name, current phase, last activity, any blockers
3. Ask: "Resume from [current phase], or start a specific phase over?"
4. Continue from the current phase — do NOT re-run completed phases

---

## Orchestrator Rules

- **`.llm/progress.md` is your responsibility** — create it after the user approves the plan, update it after every agent completes, and read it before each phase. This file is the only artifact that survives a session interruption — if it doesn't reflect the current state, context is lost. Never batch updates; write immediately.
- **You coordinate, agents implement** — don't write code or project files yourself. The only file you create and maintain is `.llm/progress.md`.
- **Log decisions inline** — when any agent reports a non-obvious choice, or the user makes a decision, append it to the Decisions Log table in `.llm/progress.md` immediately (architecture choices, user decisions, blocker resolutions, scope changes)
- **Parallel by default** — agents in the same phase run simultaneously unless there's an explicit dependency
- **Status updates are brief** — one line per phase unless the user asks for detail
- **Respect the loop budget** — 2 retries max per agent before escalating; don't spin indefinitely
- **Flexible team sizing** — if the task is simpler than planned, collapse phases; if more complex, add agents
