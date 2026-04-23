# control-plane

The canonical coordination layer for multi-agent work across projects.

This repository does **not** contain product implementation code. It contains the coordination artifacts — PBIs, tasks, handoffs, and repo indexes — that govern work happening in other repositories.

## What lives here

| Path | Purpose |
|---|---|
| `backlog.md` | Top-level PBI backlog and change history |
| `pbis/<PBI-ID>/` | Per-PBI folder: `prd.md`, `tasks.md`, `handoffs.md`, task files |
| `repos/<repo-name>/index.md` | Append-only index of PBIs touching each repo |
| `templates/` | Starter templates for all control-plane artifacts |
| `.agents/policies/project_policy.md` | Primary operating policy — read this first |

## Key rules

- **`main` only** — no feature branches or worktrees in this repo. Every session reads and writes the same state.
- **Commit promptly** — don't accumulate large batches of uncommitted coordination changes.
- **Policy first** — before doing any work, read `.agents/policies/project_policy.md`.
- **Templates first** — use `templates/` as the starting point when creating new artifacts.
- **Append-only indexes** — repo index files only grow; historical entries are never removed.

## Workflow summary

1. PBIs are created in `backlog.md` with status `Proposed`.
2. When a PBI is approved (`Agreed`), its folder is created under `pbis/` and every repo it touches gets an entry in `repos/<repo-name>/index.md`.
3. Tasks are defined in `pbis/<PBI-ID>/tasks.md` and individual task files (`1.md`, `2.md`, …).
4. Handoffs between sessions are recorded in `pbis/<PBI-ID>/handoffs.md`.
5. Implementation work happens in the target product repos, not here.

For full detail on PBI structure, task workflow, testing tiers, and handoff requirements, see [`.agents/policies/project_policy.md`](.agents/policies/project_policy.md).
