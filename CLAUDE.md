# CLAUDE.md

Guidance for Claude when working in this repository.

This repository is the control-plane for multi-agent project coordination.

## Read First

Before acting, read:

- `.agents/policies/project_policy.md`

That file is the source of truth for:

- PBI structure
- task structure
- handoffs
- repo indexes
- work isolation
- promotion/validation boundaries
- testing/task documentation rules

## Critical Branching Rule

This repository operates on `main` only.

Do not create branches or separate worktrees here. The control-plane is intended to behave like a shared real-time coordination database, so all sessions must read and write the same current state.

## Working Tree Discipline

Do not accumulate large local uncommitted changes in this repository.

This repo is a shared coordination layer on `main`, so handoffs, PBI creation, repo-index updates, and status changes should be committed promptly in small logical units.

Before ending a session, either commit the coordination changes or leave the working tree clean.

## What this repo is for

Use this repo to:

- create and maintain PBIs
- create and maintain task files
- record handoffs
- maintain repo lookup indexes
- maintain templates used to scaffold new control-plane artifacts

Do not treat this repo like a product implementation repo.

## Critical Rules

- Stay within the current requested scope
- Do not invent new control-plane file types unless explicitly instructed
- Use existing templates in `templates/` whenever possible
- Keep repo index files append-only
- Record handoffs in `pbis/<PBI-ID>/handoffs.md`
- Prefer plain `.md` files and repo-relative references
- Do not rely on conversational memory when a handoff or PBI record should be updated

## Repository Layout

- `backlog.md`
- `pbis/<PBI-ID>/`
- `repos/<repo-name>/index.md`
- `templates/`
- `.agents/policies/project_policy.md`

## Default Behavior

When asked to create a new control-plane artifact:

1. identify the correct template
2. instantiate it cleanly
3. update related files required by policy
4. keep formatting simple and consistent

When asked to modify policy:

1. make the smallest coherent change
2. preserve numbering and cross-references
3. avoid introducing concepts that are not operationalized elsewhere

When asked to add a handoff:

1. use a distinct entry
2. include required metadata
3. make the next action obvious for a fresh session