# Handoffs

This file records cross-session handoff instances for pbi-launch-apr-22.

## Usage

- Add a new section for each handoff.
- Do not delete old handoffs.
- Update the `status` field when the handoff progresses or is superseded.
- Include branch, worktree, session, PR, and merge commit when relevant.
- The goal is to give a fresh session enough context to pick up the correct handoff without re-discovering the situation.

## Background Rationale (this PBI's executing session)

Task 1 launched candidate `859aa142-6214-5c64-89c3-97920e4d1733` (BTC-USD/XRP-USD perp, e206 scanner) at $1,600 notional on 2026-04-26. The session ran for **22 minutes** before terminating with `runner_liveness_failed`.

**The April 14 failure mode is resolved.** PR #64's reliability fixes worked: heartbeats wrote continuously (7–10 s ages across all checkpoints), guard snapshots stayed fresh (45–57 s ages), the health gate held `ready`, and the runner survived well past the 8-minute liveness watchdog kill point that ended the April 14 attempt. This validates that PBI as Done.

**A different failure mode triggered the early termination.** At t+22m the strategy fired ENTRY_LONG (z = −3.55, deep in the entry zone with edge ≈ 199.5 bps). The pair-leg execution split:

- BTC leg-A (buy) submitted → filled at $78,095.
- XRP leg-B (sell) REJECTED by Coinbase: `PREVIEW_INSUFFICIENT_FUNDS_FOR_FUTURES`.
- The strategy executor raised on the order-submission exception, the executor was torn down, and the runner-liveness gate fired (`runner_executor_missing`, `runner_approval_missing`) → `runner_liveness_failed`.
- **Auto-flatten via `CANDLE_FAIL_SAFE` succeeded**: the BTC leg was unwound at the same price within ~1 second. **Zero residual exposure** (`live_trading_positions` for the session: 0 rows; lineage shows flatten filled).
- Realized P&L: ~$0; net capital at risk at session end: $0.

The recommended follow-ups below are scoped to keep this PBI's outcome (validation) clean while creating discrete, prioritizable work for the new findings.

---

## handoff-20260426-153500-001

- `handoff_id`: `handoff-20260426-153500-001`
- `timestamp`: `2026-04-26 15:35:00`
- `from`: `pbi-launch-apr-22 / Task 1 / closure session`
- `to`: `New PBI — pair-leg partial-fill resilience`
- `status`: `Open`
- `repos`:
  - `crypto-trading-engine`
- `related_task_ids`:
  - `1`
- `branch`: `n/a (new PBI work; no branch yet)`
- `pr`: `n/a`

### Reason

Pair-leg execution is not atomic enough to survive a single rejected leg. When leg-B is rejected, the order-submission exception kills the strategy executor; without an executor the liveness gate fires (`runner_executor_missing`, `runner_approval_missing`) and the runner exits. The auto-flatten via `CANDLE_FAIL_SAFE` works — exposure is unwound — but the runner's death cascade is over-aggressive: a single rejected order should not also kill the session. This is a separate concern from the April 14 reliability fixes and should be tracked in its own PBI.

### Summary

Observed during pbi-launch-apr-22 Task 1 at session `c84b1a1d-df11-4cca-ac73-e44dc19d20d7`. Lineage:

- `enter` `submitted` buy (leg_a / BTC) → exch `c6ee4779…`
- `enter` `failed` sell (leg_b / XRP) — Coinbase rejected for `PREVIEW_INSUFFICIENT_FUNDS_FOR_FUTURES`
- `stop` `signal_only` (executor torn down on the order-submission exception)
- `flatten` `filled` sell — `flatten_trigger_reason=CANDLE_FAIL_SAFE` → exch `abac4220…` (BTC unwound at $78,095)

Service log fingerprint:

```
ERROR src.services.live_order_execution_engine: Coinbase response missing order_id …
ERROR src.services.live_trading_strategy_runner: LIVE ORDER SUBMISSION FAILED …
CRITICAL src.services.live_trading_strategy_runner: Partial batch rollback for session … may have residual exposure (residual=True uncertain=False).
INFO  src.domain.services.strategy_executor: Stopping strategy execution
ERROR src.services.live_trading.gateway: Live session … marked failed due to runner liveness issues: runner_executor_missing,runner_approval_missing
```

### Next Action

Create a new PBI in the control-plane covering pair-leg partial-fill resilience. Suggested scope:

1. After a successful `CANDLE_FAIL_SAFE` flatten, the runner should remain alive in a quarantined state (no new entries) rather than tear down the executor and trigger the liveness gate.
2. Distinguish between "executor crashed unexpectedly" (current `runner_executor_missing` semantics) and "executor was torn down deliberately as part of a partial-fill recovery" — the latter should not count as a liveness failure.
3. Reconcile the order-fill drift warnings observed during the recovery window (`session_total_fills` lagged `filled_orders` for ~5 s during teardown). This is a metric-accuracy issue, not a correctness issue, but it muddies post-incident analysis.
4. Acceptance: a forced leg-B rejection in a controlled live or staging session keeps the runner alive, completes the auto-flatten cleanly, posts a single coherent incident summary, and emits no `runner_liveness_failed` event.

Update this handoff to `Resolved` once the new PBI is created and references this entry.

---

## handoff-20260426-153500-002

- `handoff_id`: `handoff-20260426-153500-002`
- `timestamp`: `2026-04-26 15:35:00`
- `from`: `pbi-launch-apr-22 / Task 1 / closure session`
- `to`: `New PBI — pair-leg margin preflight`
- `status`: `Open`
- `repos`:
  - `crypto-trading-engine`
- `related_task_ids`:
  - `1`
- `branch`: `n/a`
- `pr`: `n/a`

### Reason

The trigger for the runner crash was Coinbase rejecting leg-B for insufficient futures margin (`PREVIEW_INSUFFICIENT_FUNDS_FOR_FUTURES`). The launch CLI logged a scaling step ("Scaled candidate notional sizing for launch capital: 1000 -> 800.00000000 USD per leg") but did not preflight whether the resulting per-leg requirement was actually fundable on the CFM portfolio. Discovering this only after leg-A has already filled is the worst possible time — the system has live exposure that must be unwound, and the strategy state machine has to recover from a partial fill instead of refusing the entry up front.

### Summary

Per-leg notional was $800; Coinbase rejected the sell leg with `PREVIEW_INSUFFICIENT_FUNDS_FOR_FUTURES`. The Coinbase CFM balance/positions endpoints were already being polled every ~15 s (visible in the launch log `httpx GET … cfm/balance_summary`). The data needed for a preflight is therefore already in scope — what is missing is the gating logic that consults it before submitting leg-A. Reasonable causes: (a) initial-margin requirement on this product currently > $800 worth of buying power, or (b) the BTC leg's margin reservation consumed headroom needed for the XRP leg.

### Next Action

Create a new PBI for pair-leg margin preflight. Suggested scope:

1. Compute the **pair total** initial-margin requirement *and* per-leg requirements before submitting leg-A.
2. Compare against current available CFM equity (and any pending margin holds for in-flight orders).
3. If either leg would fail margin, refuse the entry signal at the gateway layer with a structured `entry_blocked_insufficient_margin` event in the guard counters; do not submit any leg.
4. Surface a clear `block_reason` value on the next status snapshot so monitoring sees the refusal explicitly.
5. Acceptance: a candidate that would fail leg-B margin generates an entry-block event and zero exchange orders; a candidate that fits proceeds normally.

Update this handoff to `Resolved` once the new PBI is created and references this entry.

---

## handoff-20260426-153500-003

- `handoff_id`: `handoff-20260426-153500-003`
- `timestamp`: `2026-04-26 15:35:00`
- `from`: `pbi-launch-apr-22 / Task 1 / closure session`
- `to`: `Cleanup PBI — PR #66 economics_guard_* leftovers`
- `status`: `Open`
- `repos`:
  - `crypto-trading-engine`
- `related_task_ids`:
  - `1`
- `branch`: `n/a (cleanup work; new branch when scheduled)`
- `pr`: `#70 (related — partial fix; unblocks launches but defers full cleanup)`
- `merge_commit`: `f66d4175`

### Reason

PR #66 (pbi-trig-contract-7a2e, merged 2026-04-23 as `6816d553`) removed the six `economics_guard_*` fields from the `_GUARDRAIL_FIELDS` registry in `src/utils/guardrails.py` but did not remove them from two other places:

- `configs/guardrails/global.yaml` (still defines all six at lines 38–43)
- `live_execution_policy_versions.global-default` v=1 (`is_active=true`, created 2026-04-08) — `policy_payload` still carries all six keys

The result: every live launch since #66 failed at `gateway._guardrail_preflight` → `resolve_guardrail_map(overrides=db_policy_payload)` with `Unknown guardrail 'economics_guard_enabled'`, blocking pbi-launch-apr-22 mid-execution. PR #70 (`f66d4175`, this session) shipped a tactical fix that adds the six keys to `_DEPRECATED_GUARDRAIL_KEYS` so legacy DB rows are silently dropped. That is enough to unblock launches, but the YAML file and the DB row still carry stale config that does not match the in-code registry — a slow-burn cleanup item.

### Summary

The PR #70 fix is intentional debt: it makes the system tolerant of stale DB policies without forcing a coupled DB migration during an unrelated launch attempt. The remaining work is purely housekeeping. Risk is low because the legacy keys are no longer referenced by any code path.

### Next Action

Create a small cleanup PBI (or a single follow-up task on the pbi-trig-contract-7a2e PBI if it is still open) covering:

1. Remove the six `economics_guard_*` keys from `configs/guardrails/global.yaml` (defaults section).
2. Insert a new active version of `live_execution_policy_versions.global-default` whose `policy_payload` drops the six keys; deactivate v=1.
3. Verify all live and staging configs by replaying `gateway._guardrail_preflight` against them.
4. Optionally tighten the validator to fail loudly in dev/staging environments when the YAML and registry disagree, so future registry-shrink commits cannot leave this kind of debt undetected.
5. Acceptance: `set(_GUARDRAIL_FIELDS) == set(load_guardrail_defaults())` and the active DB policy contains no keys outside the registry.

Update this handoff to `Resolved` when the YAML and DB are scrubbed.

---

## handoff-20260426-153500-004

- `handoff_id`: `handoff-20260426-153500-004`
- `timestamp`: `2026-04-26 15:35:00`
- `from`: `pbi-launch-apr-22 / Task 1 / closure session`
- `to`: `New task or small PBI — start-candidate CLI ergonomics`
- `status`: `Open`
- `repos`:
  - `crypto-trading-engine`
- `related_task_ids`:
  - `1`
- `branch`: `n/a`
- `pr`: `n/a`

### Reason

`PRODUCTION_TRADING_ENABLED=1 poetry run trade live start-candidate ...` runs the live runner in-process — the calling shell IS the runner until the session ends. There is no `--detach` flag. This is surprising for a verb named "start" and creates real ops friction:

- An orchestrator (human or agent) cannot easily run other CLI commands against the running session from the same shell.
- The session ID has to be harvested by tailing the launch log file rather than from the command's exit output.
- Real-time CP1 (≤2 min) requires running the launch in a background shell with a parallel monitoring driver. In this session that timing was missed and CP1 evidence had to be reconstructed from a t+11.75m snapshot. The retroactive evidence was sufficient (the session ran 22 min, far past the April 14 8-min watchdog) but the workflow remains fragile.

### Summary

The fix is small and contained. Either:

- Add a `--detach` flag to `trade live start-candidate` that returns immediately after `Session ID: …` is logged, with the runner reparented to a daemon (or a separate `trade live supervise --session-id` verb that picks up the runner loop).
- Or split the verb explicitly: `trade live launch` (synchronous preflight + DB record + return session ID) vs `trade live supervise --session-id` (the runtime loop).

### Next Action

Create a small PBI or task covering:

1. CLI design choice (`--detach` vs split-verb), with operator preference noted.
2. Implementation, including reparenting/daemonization details on macOS (where this rig runs) — `launchd`-managed runner is one option; a `nohup` + PID-file pattern is another.
3. Tests covering: (a) launch returns within N seconds with a stable session ID, (b) the runner survives parent-shell exit, (c) `trade live status` works against a detached session.
4. Update `docs/runbooks/live_session_launch.md` and any agent memory hooks to reflect the new ergonomics.
5. Acceptance: an operator can launch a candidate, harvest the session ID, and run monitoring queries against it from a single non-blocked shell.

Update this handoff to `Resolved` when shipped.

---
