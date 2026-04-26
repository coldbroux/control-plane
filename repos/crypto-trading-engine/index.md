# crypto-trading-engine

This file is an append-only log of PBIs that touch this repo.

## Entries

| Timestamp | PBI ID | Title | Repos |
|---|---|---|---|
| 2026-04-22 12:00:00 | pbi-launch-apr-22 | Launch Attempt Iteration After DB Lock Guard and Guard-Snapshot Retry Fixes | crypto-trading-engine |
| 2026-04-26 17:00:00 | pbi-pair-preflight-153f | Pair-Leg Margin Preflight | crypto-trading-engine |
| 2026-04-26 17:00:01 | pbi-partial-fill-97c2 | Pair-Leg Partial-Fill Resilience | crypto-trading-engine |

## Notes

- Add one new row when a PBI moves from `Proposed` to `Agreed` and includes this repo in its `repos:` field.
- Do not delete old rows.
- Do not use this file to track live PBI status.
- The canonical source of truth for PBI details remains `pbis/<PBI-ID>/`.
