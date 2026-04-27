# Tasks for pbi-cp-commit-rules-76db: Require Stage-and-Commit Discipline for Control-Plane Writes

This document lists all tasks associated with pbi-cp-commit-rules-76db.

**Parent PBI**: [pbi-cp-commit-rules-76db: Require Stage-and-Commit Discipline for Control-Plane Writes](mdc:prd.md)

## Control-Plane Metadata

- `pbi_id`: `pbi-cp-commit-rules-76db`
- `title`: `Require Stage-and-Commit Discipline for Control-Plane Writes`
- `status`: `Proposed`
- `repos`:
  - `cad-lab`
  - `crypto-trading-engine`
  - `peddlerr`
- `created_at`: `2026-04-26 17:30:00`
- `updated_at`: `2026-04-26 17:30:00`

## Task Summary

| Task ID | Name | Status | Description |
|---|---|---|---|

## Notes

- This PBI is `Proposed`. Tasks will be created when the PBI is approved and moves to `Agreed` → `InProgress` (per policy 3.4.2 `start_implementation`).
- Anticipated task shape (do not pre-create — recorded here for design context only):
  - **Task 1** — Author the canonical "Writing to control-plane" subsection text and get operator sign-off (Wave 0).
  - **Task 2** — Apply the canonical text to `peddlerr/AGENTS.md` and `peddlerr/CLAUDE.md` via PR (Wave 1, parallel).
  - **Task 3** — Apply the canonical text to `crypto-trading-engine/AGENTS.md` and `crypto-trading-engine/CLAUDE.md` via PR (Wave 1, parallel).
  - **Task 4** — Apply the canonical text to `cad-lab/AGENTS.md`, create `cad-lab/CLAUDE.md`, and bundle into the first-commit setup for `cad-lab` (Wave 1, parallel; coordinate with whoever owns the cad-lab initial commit).
- Each task must have a corresponding numbered markdown file in this same folder once added.
- Task statuses in this file must always match the statuses recorded in the individual task files.
