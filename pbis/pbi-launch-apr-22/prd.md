# pbi-launch-apr-22: Launch Attempt Iteration After DB Lock Guard and Guard-Snapshot Retry Fixes
[View in Backlog](mdc:../../backlog.md#user-content-pbi-launch-apr-22)

## Control-Plane Metadata

- `pbi_id`: `pbi-launch-apr-22`
- `title`: `Launch Attempt Iteration After DB Lock Guard and Guard-Snapshot Retry Fixes`
- `status`: `Done`
- `repos`:
  - `crypto-trading-engine`
- `created_at`: `2026-04-22 12:00:00`
- `updated_at`: `2026-04-26 15:35:00`

> Migration note: This PBI was originally drafted on 2026-04-22 under the pre-control-plane delivery model and persisted on the `docs/pbi-launch-apr-22` branch in `crypto-trading-engine` at commit `74ce1d02`. It was migrated into the control-plane on 2026-04-26 after Task 1 executed, so the canonical record now lives here. The legacy `docs/delivery/pbi-launch-apr-22/` directory on that branch remains as a historical artifact.

## Overview

Run the next launch-attempt iteration now that PR #64 (merged 2026-04-21) has landed DB lock timeout guards, an 8-retry guard-snapshot write loop, a lock-release commit before the live loop starts, and a scanner stale-run query fix. These changes directly target the `runner_liveness_failed` failure mode from the April 14 attempt. Apply the standard launch-monitoring contract with explicit evidence capture.

## Problem Statement

The April 14 launch attempt (pbi-launch-apr-14 Task 1) failed because `_order_reconciliation_loop` crashed silently within the first candle cycle. With no done-callback logging, the exception was swallowed, heartbeats and guard snapshots were never written, the health gate was permanently blocked, and the liveness watchdog self-terminated the runner within 8 minutes. Zero orders, zero fills, zero capital at risk — but zero validation of the live pipeline either.

PR #64 (2026-04-21) addressed the root causes:

1. **Guard-snapshot retry loop** (`live_trading_control_service.py`): 8 retries with 10 s delay so transient DB contention no longer silently drops snapshots.
2. **Lock timeout guards** (`live_trading_session_repository.py`): `SET LOCAL lock_timeout` on write methods so isolated writers fail fast instead of blocking indefinitely.
3. **Commit-before-loop** (`live_trading_gateway.py`): `await session.commit()` after `_persist_launch_telemetry` releases the session row lock before the live loop begins.
4. **Scanner stale-run fix**: `_apply_run_discovery_filters` split from `_apply_filters` so gate events no longer corrupt the latest-run subquery.

Additionally, pbi-trig-contract-7a2e (Tasks 1–4) rationalized the live trigger contract and structured flatten-reason persistence, making exit attribution cleaner in this attempt.

Before any further strategy or infrastructure changes, we needed one dedicated launch-attempt loop under this post-fix baseline that produced auditable evidence of whether the reliability improvements resolved the April 14 failure mode.

## User Stories

- **Operator:** As an operator, I want one explicit task that governs the next candidate launch attempt after the PR #64 reliability fixes so heartbeat/guard-snapshot behavior is verified in production and outcomes are auditable.
- **Trader:** As a trader, I want the launch attempt to draw candidates from the consolidated `e206` scanner so I can confirm that DB reliability fixes did not disturb signal quality or scanner state.
- **Maintainer:** As a maintainer, I want launch findings tied to a single tracked PBI so any residual reliability regressions are immediately distinguishable from new issues.

## Technical Approach

1. Define a dedicated launch-attempt PBI for April 22 referencing the post-PR-#64 baseline.
2. Add one execution task covering preflight, launch, CP1 heartbeat validation (the key regression gate), and the full 30-minute z-contract monitoring protocol.
3. Require explicit CP1 evidence: `runtime_heartbeat_age_seconds < 60` and `guard_snapshot_age_seconds` populated within 2 minutes.
4. Reuse the default monitoring contract (30 minutes, 1-minute checkpoints) with flat-lane vs open-position field separation and z-contract fields enforced per checkpoint.
5. Record intervention events (if any) with timestamp, reason, and evidence.
6. Capture whether the guard-snapshot retry loop or lock-timeout guards were exercised (visible in service logs).

## UX/UI Considerations

- CP1 heartbeat/guard-snapshot evidence is the primary regression gate — log it explicitly before continuing to CP2.
- Flat-lane and open-position metrics must be clearly separated at each checkpoint.
- Per-checkpoint z-contract fields (`z_score` + lane-appropriate `z_entry` / `z_exit`) are mandatory.
- Closure note must explicitly state whether the April 14 failure mode was resolved and whether any new blockers emerged.

## Acceptance Criteria

1. New PBI exists at the canonical control-plane location with `prd.md`, `tasks.md`, `handoffs.md`, and at least one linked task file. ✓ (this folder, post-migration)
2. Task 1 explicitly tracks the next candidate launch attempt under the post-PR-#64 reliability-fixed system. ✓
3. Task 1 includes CP1 regression gate: `runtime_heartbeat_age_seconds < 60` and `guard_snapshot_age_seconds` populated within 2 minutes. ✓
4. Task 1 includes the required 30-minute monitoring contract and z-contract field checklist. ✓ (executed at 2-min cadence per operator request, deviation logged in Task 1 evidence)
5. Task 1 includes intervention logging requirements and closure evidence requirements. ✓
6. Backlog includes `pbi-launch-apr-22` with history entries for creation, proposal, start, and completion. ✓

## Outcome Summary (post-execution, 2026-04-26)

- **April 14 failure mode resolved.** PR #64 reliability fixes confirmed working: session ran 22 min with continuous healthy heartbeats (`runtime_heartbeat_age_seconds` 7–10 s) and guard snapshots (`guard_snapshot_age_seconds` 45–57 s) until the unrelated crash.
- **New failure mode discovered.** Pair-leg execution failed when Coinbase rejected leg-B (XRP) for `PREVIEW_INSUFFICIENT_FUNDS_FOR_FUTURES`; the strategy executor crashed on the order-submission exception, the runner liveness gate fired (`runner_executor_missing`, `runner_approval_missing`), and the runner exited with `runner_liveness_failed`.
- **Auto-flatten worked.** `CANDLE_FAIL_SAFE` flatten reason fired; the BTC long leg was unwound at $78,095. **Zero residual exposure** in `live_trading_positions` and the lineage trail confirms the flatten filled.
- **Capital at risk:** $0 net; **Realized P&L:** ~$0 (entry and exit at the same price within ~1 second).
- **Follow-ups recorded as handoffs** (see `handoffs.md`). Four are open: pair-leg partial-fill resilience, pair-leg margin preflight, PR #66 cleanup debt, and CLI ergonomics for `start-candidate`.

## API Impact Summary

### Change Classification
- **BREAKING**: None (planning/operational tracking only).
- **ADDED**: None (no runtime code change in this PBI's scope).
- **CHANGED**: Launch-attempt operational planning and evidence tracking. Note: a small unblocking code fix to `src/utils/guardrails.py` was required to clear a regression gate on launch (PR #70, merged 2026-04-26 as commit `f66d4175`); that change was scoped to a separate fix PR rather than absorbed into this PBI's Task 1.
- **REMOVED**: None.

### Affected Endpoints
- `GET /v1/live/sessions*`
- `GET /v1/scanner/status`
- `GET /v1/scanner/session-signals`
- `GET /v1/scanner/candidate-states`

## Dependencies

- [pbi-launch-apr-14](mdc:../pbi-launch-apr-14/prd.md) — predecessor; April 14 failure root cause and PR #64 scope (legacy delivery, not yet migrated to control-plane)
- [pbi-trig-contract-7a2e](mdc:../pbi-trig-contract-7a2e/prd.md) — live trigger contract rationalization (Tasks 1–4 merged in PR #66, Tasks 6–9 merged 2026-04-23)
- PR #64: `d95dee5d` — DB lock guards, guard-snapshot retry, scanner stale-run fix
- PR #66: `6816d553` — Live Trigger Contract Enforcement and Audit Remediation (Tasks 6–9)
- PR #70: `f66d4175` — `fix(guardrails): tolerate legacy economics_guard_* keys in DB policy payloads` (unblocker for this PBI's launch)
- `crypto-trading-engine`/docs/runbooks/live_session_launch.md
- `crypto-trading-engine`/src/cli/commands/live.py
- `crypto-trading-engine`/src/services/live_trading_control_service.py

## Open Questions

1. Which candidate ID should be used for this attempt (sourced from the `e206` scanner)? — **Resolved**: `859aa142-6214-5c64-89c3-97920e4d1733` (BTC-USD/XRP-USD) selected by operator.
2. Should notional remain at the current default (`$1,500`) for this run? — **Resolved**: operator chose `$1,600` (auto-scaled to $800/leg).
3. Are any temporary launch overrides needed, or should this attempt run with pure defaults? — **Resolved**: pure defaults.
4. Were the guard-snapshot retry paths or lock-timeout guards exercised during any service-level tests after PR #64 merged? — **Resolved**: not exercised in this run (no DB contention triggered them); the fixes still demonstrably prevented the April 14 mode from recurring.

## Control-Plane Links

- [Task List](mdc:tasks.md)
- [Handoffs](mdc:handoffs.md)

## Related Tasks

- [Task List](mdc:tasks.md)
- [Task 1: Launch Next Available Candidate and Validate PR #64 Reliability Fixes via 30-Minute Monitoring](mdc:1.md)
