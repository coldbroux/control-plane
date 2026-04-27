# pbi-cp-commit-rules-76db: Require Stage-and-Commit Discipline for Control-Plane Writes
[View in Backlog](mdc:../../backlog.md#user-content-pbi-cp-commit-rules-76db)

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

## Overview

The `AGENTS.md` and `CLAUDE.md` files in our active product repos tell agents to *write* PBI / task / handoff / repo-index artifacts into the control-plane, but never tell them to *commit and push* those writes. As a result, sessions are leaving uncommitted control-plane changes scattered across local working trees, and the "all sessions share one canonical state" promise in the existing instruction text is silently violated. Add an explicit "Writing to control-plane" subsection in both files in each affected repo that requires immediate stage + commit + push on `main`, and that carves out an exception to the surrounding "non-`main` branch" rule.

## Problem Statement

Both `peddlerr/AGENTS.md` (lines 29–49) and `crypto-trading-engine/AGENTS.md` (lines 8–28) contain an identical "Project Delivery Documentation Writing Policy" block. The same block is duplicated in both `peddlerr/CLAUDE.md` (around line 92) and `crypto-trading-engine/CLAUDE.md` (around line 105). The block correctly tells agents to write coordination artifacts into the control-plane, but it has four gaps:

1. **"Writing" is conflated with "persisted."** The text says writes "share one canonical state," which implies persistence happens by writing the file. It does not — uncommitted writes only exist in the writer's working tree until `git add && git commit && git push`.
2. **No mention that control-plane runs on `main` only.** Both repos teach agents (correctly, for those repos) that "all work must occur on a non-`main` branch." Without an explicit carve-out, agents apply that rule to the control-plane and either refuse to commit or create branches the control-plane forbids.
3. **No "commit per artifact" guidance.** Agents do not know that PBI creation, handoff updates, repo-index appends, and status changes should each be one small commit rather than batched into a sprawling working tree.
4. **No `cd ~/repos/control-plane` step.** Agents may write the file via absolute path from inside their own repo's worktree and never enter the control-plane working directory to run git at all.

`cad-lab` is a fresh repo (zero commits as of 2026-04-26) with an untracked `AGENTS.md` and no `CLAUDE.md`. It must adopt the same instruction set from the start so the gap is not introduced there as a fresh problem.

The downstream symptom is the one the operator is observing: control-plane sessions discover stranded uncommitted artifacts (PBIs, handoffs, index entries) that were intended to be canonical state but were never persisted, and the operator is forced to babysit cross-repo handoffs that should have been atomic.

## User Stories

- **Operator:** As an operator coordinating multiple agent sessions, I want every control-plane write produced by a product-repo session to be committed and pushed before the session reports completion, so I never have to discover stranded coordination artifacts after the fact.
- **Agent:** As an agent working in a product repo, I want clear instructions that distinguish "this repo's git discipline" from "control-plane's git discipline" so I do not accidentally apply a non-`main`-branch rule to a repository that runs on `main` only.
- **Maintainer:** As the maintainer of `AGENTS.md` and `CLAUDE.md` across repos, I want one canonical instruction block I can copy verbatim into each repo so the rule stays consistent over time.

## Technical Approach

1. Draft a single canonical "Writing to control-plane" subsection that:
   - States that every control-plane write requires `git add` (specific paths, never `git add .`), `git commit` (one logical artifact per commit), and `git push` from inside the control-plane working directory.
   - Names the repository's `main`-only constraint and explicitly carves out the surrounding "non-`main` branch" rule.
   - Lists the standard pre-commit verification (`pwd`, `git rev-parse --show-toplevel`, `git status`, `git diff --cached --name-only`).
   - States that a control-plane write is not "done" until it has been pushed.
2. In each of the three repos, replace or extend the existing "Project Delivery Documentation Writing Policy" block in **both** `AGENTS.md` and `CLAUDE.md` with the new subsection. Keep the existing list of artifact types (backlog / prd / tasks / task files / handoffs / repo index).
3. For `cad-lab`, the change lands as part of the repo's first real commit. Coordinate with the existing first-commit-setup work so this addition is included in the initial baseline rather than added retroactively.
4. Add no new instruction file in the control-plane itself — the canonical authority remains `.agents/policies/project_policy.md`. The product-repo files just point at it more loudly and add the missing operational steps.

## Parallel Execution Waves

- **Wave 0:** Draft the canonical "Writing to control-plane" subsection text once and circulate for approval.
- **Wave 1:** Apply the patch to `peddlerr/AGENTS.md` and `peddlerr/CLAUDE.md` (one PR).
- **Wave 1:** Apply the patch to `crypto-trading-engine/AGENTS.md` and `crypto-trading-engine/CLAUDE.md` (one PR).
- **Wave 1:** Add the canonical subsection to `cad-lab/AGENTS.md` and create `cad-lab/CLAUDE.md` if absent (coordinate with `cad-lab` first-commit setup).

> Wave-1 items are independent across repos and may run in parallel once the canonical text in Wave 0 is approved.

## UX/UI Considerations

- The new subsection must read as a clear exception to the surrounding "non-`main` branch" rule, not as a contradiction. Place it adjacent to that rule with a one-line cross-reference.
- The `cd ~/repos/control-plane` step must be explicit. Agents that operate from absolute paths often skip directory changes; the instruction should specifically tell them to enter the directory before running git commands.
- Keep the instruction copy-pasteable. Operators and agents should be able to lift the exact stage/commit/push sequence from the file without reformatting.

## Acceptance Criteria

1. `peddlerr/AGENTS.md`, `peddlerr/CLAUDE.md`, `crypto-trading-engine/AGENTS.md`, `crypto-trading-engine/CLAUDE.md`, `cad-lab/AGENTS.md`, and `cad-lab/CLAUDE.md` all contain the canonical "Writing to control-plane" subsection.
2. Each instance of the subsection states explicitly that the control-plane runs on `main` only and carves out the repo's "non-`main` branch" rule.
3. Each instance includes the stage / commit / push sequence with `cd ~/repos/control-plane` as the first step.
4. Each instance states that one logical artifact = one commit, and that a control-plane write is not done until pushed.
5. The text in all six locations is byte-identical to the canonical version (so the rule cannot drift across repos).
6. `cad-lab` has both files (`AGENTS.md` and `CLAUDE.md`) committed; the first-commit setup absorbs the addition rather than producing a separate follow-up PR.

## Dependencies

- `peddlerr/AGENTS.md` (lines 29–49)
- `peddlerr/CLAUDE.md` (around line 92)
- `crypto-trading-engine/AGENTS.md` (lines 8–28)
- `crypto-trading-engine/CLAUDE.md` (around line 105)
- `cad-lab/AGENTS.md` (currently untracked; will be committed as part of repo's first commit)
- `cad-lab/CLAUDE.md` (does not yet exist; will be created as part of this PBI)
- `control-plane/CLAUDE.md` — already states "commit promptly in small logical units"; this PBI propagates that intent into the product repos.
- `control-plane/.agents/policies/project_policy.md` — remains the canonical authority; the new subsection cross-references it.

## Open Questions

1. Should the canonical subsection be authored once in this PBI's PRD and lifted from there, or should it live in a shared snippet under `control-plane/templates/` for easier reuse in future repos? (Suggested: author here for now, extract to `templates/` only if a fourth repo joins.)
2. Should the subsection mandate a specific commit-message prefix (e.g., `cp:` or `control-plane:`) so cross-repo session logs are easier to filter? (Suggested: yes, propose `cp:` prefix; defer final decision until drafting Wave 0.)
3. For `cad-lab`, do we want to coordinate this addition with the broader first-commit setup as a single PR, or land the AGENTS.md/CLAUDE.md addition first and let the rest of the first-commit work follow? (Suggested: bundle, since both touch the initial baseline.)
4. Do any other repos in the operator's portfolio (e.g., `foundation-local-env`) need the same instruction block? It is registered in the control-plane but does not currently appear to write coordination artifacts itself. Out of scope for now unless confirmed otherwise.

## Control-Plane Links

- [Task List](mdc:tasks.md)
- [Handoffs](mdc:handoffs.md)

## Related Tasks

- [Task List](mdc:tasks.md)
