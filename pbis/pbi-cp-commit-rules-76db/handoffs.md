# Handoffs

This file records cross-session handoff instances for pbi-cp-commit-rules-76db.

## Usage

- Add a new section for each handoff.
- Do not delete old handoffs.
- Update the `status` field when the handoff progresses or is superseded.
- Include branch, worktree, session, PR, and merge commit when relevant.
- The goal is to give a fresh session enough context to pick up the correct handoff without re-discovering the situation.

## Origin

Created in response to operator observation (2026-04-26) that product-repo agent sessions have been writing PBI / handoff / index artifacts into the control-plane without committing them. Audit of `peddlerr/AGENTS.md`, `peddlerr/CLAUDE.md`, `crypto-trading-engine/AGENTS.md`, and `crypto-trading-engine/CLAUDE.md` confirmed all four contain the same "Project Delivery Documentation Writing Policy" block but none instruct the agent to stage / commit / push the resulting writes. `cad-lab/AGENTS.md` exists but is untracked; `cad-lab/CLAUDE.md` does not exist. This PBI tracks the fix.

---
