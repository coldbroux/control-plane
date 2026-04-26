# pbi-pair-preflight-153f: Pair-Leg Margin Preflight
[View in Backlog](mdc:../../backlog.md#user-content-pbi-pair-preflight-153f)

## Control-Plane Metadata

- `pbi_id`: `pbi-pair-preflight-153f`
- `title`: `Pair-Leg Margin Preflight`
- `status`: `Proposed`
- `repos`:
  - `crypto-trading-engine`
- `created_at`: `2026-04-26 17:00:00`
- `updated_at`: `2026-04-26 17:00:00`

> Spun out of [pbi-launch-apr-22](mdc:../pbi-launch-apr-22/handoffs.md) handoff-002 (`handoff-20260426-153500-002`). The launch session at `c84b1a1d-df11-4cca-ac73-e44dc19d20d7` failed because Coinbase rejected leg-B (XRP) for `PREVIEW_INSUFFICIENT_FUNDS_FOR_FUTURES` *after* leg-A (BTC) had already filled. This PBI exists to prevent that failure mode from being reachable in the first place.

## Overview

Refuse pair-leg entries at the gateway layer when the candidate would fail Coinbase futures (CFM) margin requirements for either leg. Today we discover the margin shortfall only after leg-A has filled, which forces a partial-fill recovery and live capital at risk. The data needed for a preflight is already polled (CFM `balance_summary` runs every ~15 s); what is missing is the gating logic that consults it before submitting leg-A.

## Problem Statement

During pbi-launch-apr-22 Task 1, the launch CLI logged a notional scaling step ("Scaled candidate notional sizing for launch capital: 1000 -> 800.00000000 USD per leg") but did not preflight whether the resulting per-leg requirement was actually fundable on the CFM portfolio. ENTRY_LONG fired with z = −3.55 and edge ≈ 199.5 bps; BTC leg-A submitted and filled at $78,095; XRP leg-B was rejected by Coinbase for insufficient futures funds. The strategy executor raised on the order-submission exception and the runner exited via `runner_liveness_failed`. Auto-flatten via `CANDLE_FAIL_SAFE` succeeded with zero residual exposure, but discovering a margin shortfall only after leg-A is already on the book is the worst possible time — every such event creates live capital at risk and burns recovery cycles.

Reasonable causes observed or hypothesized:
- Initial-margin requirement on the rejected leg's product currently exceeds the per-leg notional after scaling.
- The accepted leg's margin reservation consumed CFM headroom needed for the rejected leg.
- Pending margin holds for in-flight orders are not netted when computing available equity.

## User Stories

- **Operator:** As an operator, I want pair-leg entries that cannot be funded to be refused before any leg is submitted so I never have to recover from a partial-fill caused by an avoidable margin rejection.
- **Trader:** As a trader, I want a clear `block_reason` surfaced when the gateway refuses an entry so I can distinguish margin-blocked candidates from absent signals during post-session analysis.
- **Maintainer:** As a maintainer, I want refused entries to emit a structured guard event so monitoring can count and alert on entry blocks distinctly from successful entries and runner failures.

## Technical Approach

1. Add a margin-preflight step inside the gateway's pair-leg entry pipeline, executed *before* leg-A submission.
2. Compute pair-total *and* per-leg initial-margin requirements using the contract specs already available in the products registry.
3. Compare against current available CFM equity, netting any pending margin holds for in-flight orders.
4. On insufficient margin (either leg or pair): refuse the entry, do not submit any exchange order, and emit a structured `entry_blocked_insufficient_margin` event in the guard counters.
5. Surface `block_reason` on the next session status snapshot so monitoring and the live status CLI see the refusal explicitly.
6. Reuse the existing CFM `balance_summary` polling output rather than introducing new exchange calls; if data is too stale to trust, fall back to refusing the entry with a distinct `block_reason` (e.g., `margin_data_stale`).

## UX/UI Considerations

- The block must be visible in the standard live-session status snapshot — no operator should have to grep service logs to see why an entry was refused.
- The structured guard event name must be stable so dashboards and runbooks can reference it.
- Refusal should be silent at the strategy layer (no z-contract or executor-level error events) — it is a gateway decision, not a strategy or executor failure.

## Acceptance Criteria

1. A candidate that would fail leg-B (or leg-A, or pair-total) margin generates an `entry_blocked_insufficient_margin` event and produces zero exchange orders.
2. A candidate that fits available margin proceeds normally and the event is not emitted.
3. The next status snapshot after a refusal contains a non-null `block_reason` matching the guard event.
4. Pending margin holds for in-flight orders are netted into available equity (verified by a test that simulates a queued order then a new entry attempt against the residual headroom).
5. If `balance_summary` data is older than a configurable freshness threshold, the entry is refused with `block_reason=margin_data_stale` rather than proceeding on stale data.
6. End-to-end: in a controlled staging session, a candidate engineered to exceed available margin produces the block event, posts a coherent `block_reason`, and the runner remains healthy with no `runner_liveness_failed`.

## Dependencies

- [pbi-launch-apr-22](mdc:../pbi-launch-apr-22/prd.md) — observed failure that motivated this PBI; see handoff-002 in `pbi-launch-apr-22/handoffs.md`.
- [pbi-partial-fill-97c2](mdc:../pbi-partial-fill-97c2/prd.md) — sibling PBI that hardens the runner if a margin rejection ever slips past this preflight.
- `crypto-trading-engine`/src/services/live_trading/gateway.py
- `crypto-trading-engine`/src/services/live_order_execution_engine.py
- `crypto-trading-engine`/src/services/live_trading_strategy_runner.py
- `crypto-trading-engine`/configs/products/ (initial-margin metadata)
- Coinbase CFM `balance_summary` integration (already polled; no new dependency)

## Open Questions

1. What freshness threshold is acceptable for `balance_summary` data before we refuse on `margin_data_stale`? Suggested starting point: 60 s.
2. Should we cache the most recent balance snapshot inside the gateway, or re-query at preflight time? (Latter is safer; former is faster.)
3. Do we need to model exchange-side margin haircuts beyond the published initial-margin requirement, or is the published number sufficient for the preflight gate?
4. Should the preflight also validate that the candidate's per-leg notional is non-zero post-scaling (defensive check), or is that already guaranteed by the launch CLI?

## Control-Plane Links

- [Task List](mdc:tasks.md)
- [Handoffs](mdc:handoffs.md)

## Related Tasks

- [Task List](mdc:tasks.md)
