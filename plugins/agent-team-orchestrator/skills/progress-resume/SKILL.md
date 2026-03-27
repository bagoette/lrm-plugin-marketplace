---
name: progress-resume
description: Activates when the user asks to "resume", "continue", "check status", or "pick up" a multi-agent orchestrated task, or when a progress.md file is detected in the project directory. Provides guidance for reading orchestration state and resuming execution from the correct phase.
---

# Progress Resume

When the user wants to resume or check on an orchestrated task:

## Step 1: Read the Progress File

Look for `progress.md` in the current directory (or a subdirectory if the user specifies). Read it fully to extract:
- **Project name** and **started date**
- **Current Phase** — where execution paused
- **Agent Team** — what roles are defined
- **Timeline** — completed work and when
- **Blockers** — anything unresolved
- **Decisions Log** — context for choices already made

## Step 2: Summarize State

Present a brief status to the user:

```
Orchestration in progress: **[project name]**
Started: [date]
Current phase: [phase]
Last activity: [most recent timeline entry — one line]
Completed phases: [N of M]
[If blockers exist]: ⚠️ Pending blocker: [description]
```

## Step 3: Ask What to Do

> "Resume from [current phase]? Or would you like to:
> - [R] Resume from current phase
> - [O] Jump to a specific phase
> - [S] See the full timeline
> - [X] Start over with the same requirements"

## Step 4: Resuming Execution

If resuming:
- Use `/orchestrate` workflow starting at Phase 3 (Agent Execution Loop) for the current phase
- Pass each agent the relevant context summary from prior timeline entries
- Do NOT re-run phases already marked complete unless the user explicitly requests it

If the user wants the full timeline: read and display the Timeline section of `progress.md`.

If there are open blockers: present them and ask the user to resolve before continuing.

## When No progress.md Exists

If the user asks to resume but no `progress.md` is found:
> "No progress.md found in this directory. Use `/orchestrate [task description]` to start a new orchestrated task."
