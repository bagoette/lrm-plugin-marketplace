# LRM Plugin Marketplace

A marketplace of reusable plugins for Claude Code. Each plugin is a structured collection of agents, skills, and references that extend Claude Code's capabilities.

## Plugins

### [Agent Team Orchestrator](plugins/agent-team-orchestrator/)

Coordinates a team of specialized AI agents to complete complex, multi-phase tasks with cross-session progress tracking.

**Skills (slash commands):**

- `/orchestrate` — Runs the full pipeline: gather requirements → plan agent team → execute tasks in parallel phases → track decisions → summarize results. Automatically resumes from `progress.md` if one exists.
- `/progress-resume` — Resume, check status, or restart a previously started orchestration.

**Agents:**

| Agent | Role |
|-------|------|
| `requirements-analyst` | Interviews the user to produce a structured requirements document |
| `task-planner` | Designs the agent team composition and phased execution plan |
| `task-executor` | Generic implementation agent that executes a single bounded task |

## Installing a Plugin

```
/install <plugin-url>
```

Use Claude Code's `/install` command with the URL of the plugin you want to add.

## Structure

```
plugins/
  <plugin-name>/
    agents/          ← Sub-agent role definitions (.md with YAML frontmatter)
    skills/
      <command>/
        SKILL.md     ← Slash command entry point
        references/  ← Supporting docs read at runtime by agents
```