# AGENTS.md

# Control-Plane Repository Guidelines

This repository is the canonical coordination layer for multi-agent work across projects.

## Purpose

Use this repo to manage:

- backlog-level PBI tracking
- per-PBI control-plane records
- repo-to-PBI lookup indexes
- handoffs between fresh sessions
- reusable markdown templates for new PBIs and tasks

This repo does **not** contain implementation code for product systems. It contains the coordination artifacts that govern that work.

## Canonical Policy

Before doing any work, read:

- `.agents/policies/project_policy.md`

That file is the primary operating policy for PBIs, tasks, handoffs, work isolation, promotion boundaries, and testing structure.

## Branching Model

This repository operates on `main` only.

The control-plane is intended to behave like a shared, real-time coordination layer across agent sessions. Its value depends on all sessions reading and writing the same current state rather than diverging across feature branches.

Rules:
- Do not create or use feature branches in this repository.
- Do not create or use separate worktrees in this repository.
- Read from and write to `main` only.
- Keep changes small, append-friendly, and safe for concurrent coordination use.
- If a requested change would be risky to apply directly on `main`, stop and ask the user before proceeding.

## Working Tree and Commit Discipline

Because this repository operates on `main` only and acts as a shared coordination layer, local uncommitted state must be minimized.

Rules:
- Do not accumulate large batches of uncommitted changes in this repository.
- Prefer small, atomic commits that publish coordination changes promptly.
- Do not leave handoffs, new PBI folders, repo index updates, or task-state changes only in local uncommitted form.
- Keep the working tree clean between logical operations whenever possible.
- If a coordinated change is only partially complete and would leave shared state inconsistent, either:
  - finish and commit it immediately, or
  - revert the partial local change before stopping.
- Before ending a session, either commit the coordination changes or return the repository to a clean working tree.

## Repository Structure

- `backlog.md` — top-level backlog and PBI change history
- `pbis/` — canonical PBI folders
- `repos/` — append-only repo lookup indexes
- `templates/` — template source files for control-plane artifacts
- `.agents/policies/project_policy.md` — governing policy for this repo and its workflow

## Execution Model

When working in this repo:

- Treat every change as coordination/documentation work, not product-code work
- Prefer small, precise edits
- Do not invent new file structures unless the policy explicitly allows them
- Keep records append-friendly and easy for fresh sessions to scan
- Do not delete historical handoff or backlog records unless the user explicitly requests cleanup

## File Creation Rules

You may create or update only files that fit the policy-defined structure, including:

- `backlog.md`
- `pbis/<PBI-ID>/prd.md`
- `pbis/<PBI-ID>/tasks.md`
- `pbis/<PBI-ID>/handoffs.md`
- `pbis/<PBI-ID>/<TASK-ID>.md`
- `repos/<repo-name>/index.md`
- `templates/*.template.md`

Do not create additional coordination files outside this structure unless explicitly requested.

## Editing Rules

- Keep backlog rows concise
- Keep repo index entries append-only
- Keep handoff entries discrete and metadata-rich
- Keep templates generic and placeholder-based unless a concrete example is explicitly helpful
- Prefer repo-relative paths over machine-local absolute paths

## Handoffs

When you discover work that belongs to another task, PBI, or repo scope:

- stop
- record the handoff in the relevant `pbis/<PBI-ID>/handoffs.md`
- include branch, worktree, session, and promotion metadata when relevant
- do not absorb the work into the current scope unless explicitly instructed

## Templates

When generating new artifacts, prefer using the files in `templates/` as the starting point rather than inventing structure from scratch.

## Change Discipline

- Keep changes aligned with the current requested task
- Do not reorganize the repo opportunistically
- Do not rewrite historical records for style alone
- Ask whether a requested structural change belongs in the policy, a template, or a live PBI artifact before applying it