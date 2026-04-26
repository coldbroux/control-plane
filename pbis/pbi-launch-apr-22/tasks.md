# Tasks for pbi-launch-apr-22: Launch Attempt Iteration After DB Lock Guard and Guard-Snapshot Retry Fixes

This document lists all tasks associated with pbi-launch-apr-22.

**Parent PBI**: [pbi-launch-apr-22: Launch Attempt Iteration After DB Lock Guard and Guard-Snapshot Retry Fixes](mdc:prd.md)

## Control-Plane Metadata

- `pbi_id`: `pbi-launch-apr-22`
- `title`: `Launch Attempt Iteration After DB Lock Guard and Guard-Snapshot Retry Fixes`
- `status`: `Done`
- `repos`:
  - `crypto-trading-engine`
- `created_at`: `2026-04-22 12:00:00`
- `updated_at`: `2026-04-26 15:35:00`

## Task Summary

| Task ID | Name | Status | Description |
|---|---|---|---|
| 1 | [Launch Next Available Candidate and Validate PR #64 Reliability Fixes via 30-Minute Monitoring](mdc:1.md) | Done | Launch the next `e206` candidate, confirm CP1 regression gate (heartbeat age < 60 s, guard snapshot populated within 2 min), then run the 30-minute z-contract monitoring protocol to produce closure evidence on the April 14 failure mode. |

## Notes

- Task 1 was the only task in this PBI; outcome and follow-up recommendations are recorded in `handoffs.md`.
- Each task must have a corresponding numbered markdown file in this same folder.
- Task statuses in this file must always match the statuses recorded in the individual task files.
- Update this file whenever a task is added, removed, reordered, or changes status.
