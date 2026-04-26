# pbi-partial-fill-97c2: Pair-Leg Partial-Fill Resilience
[View in Backlog](mdc:../../backlog.md#user-content-pbi-partial-fill-97c2)

## Control-Plane Metadata

- `pbi_id`: `pbi-partial-fill-97c2`
- `title`: `Pair-Leg Partial-Fill Resilience`
- `status`: `Proposed`
- `repos`:
  - `crypto-trading-engine`
- `created_at`: `2026-04-26 17:00:01`
- `updated_at`: `2026-04-26 17:00:01`

> Spun out of [pbi-launch-apr-22](mdc:../pbi-launch-apr-22/handoffs.md) handoff-001 (`handoff-20260426-153500-001`). Sibling PBI to [pbi-pair-preflight-153f](mdc:../pbi-pair-preflight-153f/prd.md) — pair-margin preflight reduces the *frequency* of this scenario; this PBI reduces the *blast radius* if it happens anyway.

## Overview

When a pair-leg entry has one leg accepted by Coinbase and the other rejected, the auto-flatten via `CANDLE_FAIL_SAFE` already unwinds residual exposure correctly. The problem is what happens *next*: the strategy executor crashes on the order-submission exception, the runner-liveness gate fires (`runner_executor_missing`, `runner_approval_missing`), and the runner exits with `runner_liveness_failed`. A single rejected order should not also kill the session. This PBI keeps the runner alive in a quarantined state after a successful auto-flatten so the failure is surgical rather than session-ending.

## Problem Statement

Observed during pbi-launch-apr-22 Task 1, session `c84b1a1d-df11-4cca-ac73-e44dc19d20d7`. Lineage:

- `enter` `submitted` buy (leg_a / BTC) → exch `c6ee4779…`
- `enter` `failed` sell (leg_b / XRP) — Coinbase rejected for `PREVIEW_INSUFFICIENT_FUNDS_FOR_FUTURES`
- `stop` `signal_only` (executor torn down on the order-submission exception)
- `flatten` `filled` sell — `flatten_trigger_reason=CANDLE_FAIL_SAFE` → exch `abac4220…` (BTC unwound at $78,095)
- Runner exit reason: `runner_liveness_failed` due to `runner_executor_missing`, `runner_approval_missing`

Service log fingerprint:

```
ERROR src.services.live_order_execution_engine: Coinbase response missing order_id …
ERROR src.services.live_trading_strategy_runner: LIVE ORDER SUBMISSION FAILED …
CRITICAL src.services.live_trading_strategy_runner: Partial batch rollback for session … may have residual exposure (residual=True uncertain=False).
INFO  src.domain.services.strategy_executor: Stopping strategy execution
ERROR src.services.live_trading.gateway: Live session … marked failed due to runner liveness issues: runner_executor_missing,runner_approval_missing
```

Realized P&L was ~$0 and zero residual exposure was confirmed in `live_trading_positions`, so correctness held. The runner death was over-aggressive: the auto-flatten path already proved the system can recover from a partial fill — the executor teardown that follows is the gratuitous step.

Additionally, the order-fill drift warnings observed during the recovery window (`session_total_fills` lagged `filled_orders` for ~5 s during teardown) are a metric-accuracy issue, not a correctness issue, but they muddy post-incident analysis.

## User Stories

- **Operator:** As an operator, I want a single rejected order to NOT terminate the live session so that one-off exchange rejections are absorbed surgically rather than burning a launch attempt.
- **Trader:** As a trader, I want the runner to enter a quarantined "no new entries" state after a partial-fill recovery so I can decide whether to manually resume, manually flatten, or end the session — without the runner deciding for me.
- **Maintainer:** As a maintainer, I want the runner-liveness gate to distinguish "executor crashed unexpectedly" from "executor was torn down deliberately as part of partial-fill recovery" so post-incident analysis is unambiguous.

## Technical Approach

1. After a successful `CANDLE_FAIL_SAFE` flatten that fully unwinds the partial leg, transition the runner into a new `quarantined` state (no new entries; existing approval and monitoring stay alive) instead of tearing down the executor.
2. Add a runner-liveness gate distinction:
   - `executor_crashed_unexpectedly` → liveness failure (current behavior).
   - `executor_torn_down_for_recovery` → suppressed; counts as a structured incident, not a liveness failure.
3. Reconcile the `session_total_fills` vs `filled_orders` drift during teardown — likely a publish-ordering issue between the order-engine update path and the session aggregator. Fix or document with a tolerance window.
4. Emit one coherent incident summary (single structured event with: rejected leg, rejection reason, accepted leg, flatten reason, flatten fill price, residual exposure check, quarantine state) instead of the current scatter of CRITICAL/ERROR log lines.

## UX/UI Considerations

- The quarantined state must be visible in `trade live status` output and in the session status snapshot — the operator must be able to tell at a glance that the runner is alive but will not enter.
- The single incident summary should be machine-readable (structured fields) so dashboards can attribute incidents without log-grepping.
- Existing `runner_liveness_failed` semantics for genuine executor crashes must remain unchanged — this PBI adds a discrimination, not a softening.

## Acceptance Criteria

1. A forced leg-B rejection in a controlled live or staging session keeps the runner alive in `quarantined` state, completes the auto-flatten cleanly, and emits zero `runner_liveness_failed` events.
2. The same scenario produces exactly one structured incident summary event containing the rejected leg, the flatten outcome, and a residual-exposure confirmation.
3. A genuine executor crash (simulated by killing the executor task) still produces a `runner_liveness_failed` event — i.e., the discrimination does not silently swallow real failures.
4. After the runner enters `quarantined`, `trade live status` reports the state and the session status snapshot exposes a `runner_state=quarantined` (or equivalent) field.
5. `session_total_fills` and `filled_orders` agree (or differ by at most one configurable tolerance step) at the end of the recovery window in the test scenario.
6. End-to-end: a staging session with an injected leg-B rejection produces a single coherent incident record, zero residual exposure, no liveness-failure event, and a quarantined runner that the operator manually ends.

## Dependencies

- [pbi-launch-apr-22](mdc:../pbi-launch-apr-22/prd.md) — observed failure; see handoff-001 in `pbi-launch-apr-22/handoffs.md`.
- [pbi-pair-preflight-153f](mdc:../pbi-pair-preflight-153f/prd.md) — sibling PBI; reduces the *frequency* of this scenario. This PBI is the second-line defense.
- `crypto-trading-engine`/src/services/live_trading_strategy_runner.py
- `crypto-trading-engine`/src/services/live_trading/gateway.py
- `crypto-trading-engine`/src/domain/services/strategy_executor.py
- `crypto-trading-engine`/src/services/live_order_execution_engine.py
- `crypto-trading-engine`/docs/runbooks/live_session_launch.md (will need an update to describe the new quarantined state)

## Open Questions

1. How should the operator exit a quarantined session — a new `trade live resume`/`trade live end` verb, or reuse existing `trade live stop` + a `--resume` flag?
2. Is "quarantined = no new entries, existing approval intact" the right semantics, or should the approval also be paused so even manual operator entries are blocked until explicit resume?
3. Should a configurable max-quarantine-duration auto-flatten and end the session if the operator does not act within N minutes? (Defensive default; risk is leaving a stalled session indefinitely.)
4. What is the right tolerance for `session_total_fills` vs `filled_orders` drift before flagging it as a real reconciliation error?

## Control-Plane Links

- [Task List](mdc:tasks.md)
- [Handoffs](mdc:handoffs.md)

## Related Tasks

- [Task List](mdc:tasks.md)
