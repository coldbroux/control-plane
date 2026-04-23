# Handoffs

This file records cross-session handoff instances for this PBI.

## Usage

- Add a new section for each handoff.
- Do not delete old handoffs.
- Update the `status` field when the handoff progresses or is superseded.
- Include branch, worktree, session, PR, and merge commit when relevant.
- The goal is to give a fresh session enough context to pick up the correct handoff without re-discovering the situation.

---

## handoff-YYYYMMDD-HHMMSS-001

- `handoff_id`: `handoff-YYYYMMDD-HHMMSS-001`
- `timestamp`: `YYYY-MM-DD HH:MM:SS`
- `from`: `<source PBI / task / session>`
- `to`: `<target PBI / task / session>`
- `status`: `Open`
- `repos`:
  - `<repo-name>`
- `related_task_ids`:
  - `<task-id>`
- `branch`: `<branch-name-if-relevant>`
- `worktree`: `<worktree-path-if-relevant>`
- `session`: `<agent-or-session-identifier-if-relevant>`
- `pr`: `<pr-number-or-link-if-relevant>`
- `merge_commit`: `<merge-commit-if-relevant>`

### Reason
<Why this handoff exists. State the boundary clearly and briefly.>

### Summary
<What was discovered, completed, or prepared. Include only the information a fresh session actually needs.>

### Next Action
<What the receiving session should do next. Be concrete.>

---

## Example

## handoff-20260424-091000-001

- `handoff_id`: `handoff-20260424-091000-001`
- `timestamp`: `2026-04-24 09:10:00`
- `from`: `pbi-unit-test-perf-b9a8 / Task 6 / writer session`
- `to`: `pbi-trig-contract-7a2e / Task 8 / validation session`
- `status`: `Open`
- `repos`:
  - `crypto-trading-engine`
- `related_task_ids`:
  - `6`
  - `8`
- `branch`: `claude/modest-banzai-1201f5`
- `worktree`: `/Users/briankerschner/repos/crypto-trading-engine/.claude/worktrees/modest-banzai`
- `session`: `claude-writer-20260424`
- `pr`: `#6`
- `merge_commit`: `16242c6`

### Reason
Task 8 depends on promoted code and runtime validation, so responsibility must move from implementation to validation.

### Summary
Tasks 1–7 were completed and promoted. CI passed and the PR was merged. The remaining work is end-to-end validation of the Adena orchestration flow against the acceptance criteria.

### Next Action
Start a fresh validation session, run the Task 8 E2E validation flow, and update this handoff to `Resolved` or `Superseded` when complete.