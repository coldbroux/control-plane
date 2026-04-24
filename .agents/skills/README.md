# Agent Skills

This directory contains shared skill definitions for AI coding agents working in this repository.

## What is a skill?

A skill is a named, reusable workflow — a structured prompt that tells an agent *exactly* how to execute a recurring task: what to check, what commands to run, what "done" looks like, and what guardrails apply.

## How each agent uses this directory

| Agent | How skills are loaded |
|---|---|
| **Codex** | Natively — scans `$REPO_ROOT/.agents/skills/` automatically |
| **Claude Code** | Slash commands in `.claude/commands/<name>.md`; committed to git |
| **Gemini CLI / others** | Plain markdown — reference this directory directly |

## Skill directory format

Each skill is a **directory** containing a `SKILL.md` file.

```
.agents/skills/
  my-skill/
    SKILL.md      ← skill definition (frontmatter + workflow body)
  README.md       ← this file
```

## Skills in this repo

| Skill | Directory | Trigger |
|---|---|---|
| Manage Agent Skills | [`manage-agent-skills/`](manage-agent-skills/) | When creating or updating a skill in `.agents/skills/` |
| Tidy Up | [`tidy-up/`](tidy-up/) | Before starting new work or as a standalone cleanup — scan for stale branches, worktrees, stashes, and WIP; recommend isolate / consolidate / purge; execute only on explicit human approval |
