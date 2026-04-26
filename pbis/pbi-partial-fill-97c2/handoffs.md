# Handoffs

This file records cross-session handoff instances for pbi-partial-fill-97c2.

## Usage

- Add a new section for each handoff.
- Do not delete old handoffs.
- Update the `status` field when the handoff progresses or is superseded.
- Include branch, worktree, session, PR, and merge commit when relevant.
- The goal is to give a fresh session enough context to pick up the correct handoff without re-discovering the situation.

## Origin

This PBI was created in response to [pbi-launch-apr-22](mdc:../pbi-launch-apr-22/handoffs.md) handoff `handoff-20260426-153500-001`. That upstream handoff should be marked `Resolved` once this PBI exists and references it (this file).

## Sequencing Note

This PBI is the second-line defense behind [pbi-pair-preflight-153f](mdc:../pbi-pair-preflight-153f/prd.md). The pair-margin preflight reduces the frequency of partial-fill scenarios; this PBI reduces the blast radius when one happens anyway. Per the sequencing in `pbi-launch-apr-22/handoffs.md` (handoff-003 § "Recommended Sequencing Among Sibling Handoffs"), the preflight PBI should land first.

---
