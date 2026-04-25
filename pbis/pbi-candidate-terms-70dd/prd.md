# pbi-candidate-terms-70dd: Candidate Terms — A Memory Layer for Semantic Discovery

[View in Backlog](mdc:../../backlog.md#user-content-pbi-candidate-terms-70dd)

## Control-Plane Metadata

- `pbi_id`: `pbi-candidate-terms-70dd`
- `title`: `Candidate Terms — A Memory Layer for Semantic Discovery`
- `status`: `Proposed`
- `repos`:
  - `peddlerr`
- `created_at`: `2026-04-25 15:21:39`
- `updated_at`: `2026-04-25 15:21:39`

## Overview

Phase 2 of the research orchestration (`reconcile-canonical-language`) currently produces deferral and promotion decisions that exist only in the conversation transcript. There is no memory across runs: a term deferred during one health system's Phase 2 is reconsidered from scratch on the next, with no awareness of prior reasoning or accumulating cross-source signal.

This PBI introduces a persistent candidate-term system that allows semantic discovery to accumulate across runs, enabling cross-source comparison, controlled promotion into canonical concepts, and explicit lifecycle management of vocabulary evolution.

> A candidate term represents a possible future canonical concept, derived from one or more source vocabulary observations. Candidate terms accumulate across discovery runs and provide the review surface for deciding whether a source-derived term should be promoted, merged, rejected, or deferred.

Candidate terms are **not** canonical concepts. They are a modeling memory layer that allows Brian's language to emerge through comparison across source systems. Therefore:

- Candidate terms may be source-derived.
- Canonical concepts should be source-neutral.
- Promotion should minimize source-specific leakage.
- Deferral is a valid and expected outcome.

## Problem Statement

During the Task 8 E2E run for `pbi-research-orch-2e0b`, Phase 2 produced 38 deferred terms for `adena-health-system`. None of those decisions were persisted. Concrete observed gaps:

1. **No persistence.** No table holds deferred or candidate terms. Any rationale for deferral lives only in the operator's conversation.
2. **No follow-up mechanism.** Phase 2's existing diagnostic Q7 finds undermapped terms but has no awareness of prior evaluations. It treats a term deferred six runs ago identically to one never seen before.
3. **No cross-source signal.** The reconcile-canonical-language skill is explicitly cross-source by design, but there is currently no place where cross-source convergence accumulates.
4. **No formal promotion criteria.** Past promotion decisions used implicit agent judgment — "cross-system generality," "structural distinctness." Two runs against the same vocabulary could produce different canonical trees.

The architectural contradiction: a skill meant to operate over accumulated cross-source evidence has no mechanism to accumulate that evidence.

## User Stories

- **Operator (Brian):** As the operator of the canonical language, I want to see which terms have been deferred across multiple health systems so that I can identify cross-source convergence and decide when to promote.
- **Phase 2 agent:** As the reconcile-canonical-language skill, I want prior deferral rationale surfaced when I encounter a previously-seen term so that I do not redo evaluations from scratch.
- **Future operator:** As someone reading the canonical concept tree, I want to trace the origin of any concept back to the source observations that justified its promotion so that the canonical language remains auditable.

## Technical Approach

1. **New schema `semantic_modeling`** — separate from `core_semantics` (authoritative configuration). The new schema holds modeling workflow state. Semantics = authoritative configuration; candidate terms = modeling workflow state.
2. **Two tables:**
   - `semantic_modeling.candidate_terms` — durable modeling object: normalized form, candidate role, lifecycle status, resolution targets, evaluation notes, first/last seen version IDs.
   - `semantic_modeling.candidate_term_observations` — junction linking each candidate to one or more `core_semantics.vocabulary_terms` observations across versions. `UNIQUE (vocabulary_term_id, version_id)`.
3. **Lifecycle states:** `candidate`, `deferred`, `promoted`, `merged`, `rejected`, `obsolete`. Initial state is `candidate`; `deferred` means explicitly evaluated and held.
4. **Allowed transitions:**
   - `candidate → deferred | promoted | merged | rejected`
   - `deferred → promoted | merged | rejected | obsolete`
   - No other transitions; no destructive deletes.
5. **Resolution columns are mutually exclusive:**
   - `merged_into_candidate_id` (self-FK) for merge exits.
   - `resolved_concept_key` for promotion exits.
   - Enforced by CHECK constraint.
6. **Normalized form is immutable.** Renaming a candidate is represented as a new candidate plus a merge of the old one. Preserves the audit trail of normalization decisions.
7. **Phase 2 is split into two scripts:**
   - `record_candidate_terms.py` — creates candidates, links observations, advances `last_seen_version_id`. Idempotent.
   - `promote_term_mappings.py` (existing, extended) — evaluates the candidate set; writes promotions, merges, rejections, deferrals; populates `evaluation_notes` on every transition.
8. **Q8 diagnostic** — partitions undermapped terms across the six lifecycle states (never-seen, candidate, deferred, merged, rejected, promoted). Drives Check 2 of reconcile-canonical-language.
9. **Obsolete sweep** — separate maintenance pass. Transitions deferred candidates to `obsolete` when all observations reference superseded source vocabulary versions. Not bundled into Phase 2.
10. **Promotion heuristic** (informally encoded for v1; see Open Questions for hardening): a candidate may be promoted when it appears across ≥2 source systems OR is clearly part of a widely understood domain vocabulary; its meaning is stable across contexts; it maps cleanly into a canonical role; it removes source-specific noise; and it improves downstream interpretability.

## Parallel Execution Waves

- **Wave 0:** Task 1 — schema migration. Blocking for everything downstream.
- **Wave 1:** Tasks 2, 3, 4, 5, 6 — Q8 diagnostic, recording script, promotion script update, obsolete sweep, skill update. All depend only on the schema and can run in parallel.
- **Wave 2:** Task 7 — backfill 38 Adena deferrals. Depends on Tasks 1 and 3.
- **Wave 3:** Task 8 — E2E validation against a follow-up Phase 2 run.

## UX/UI Considerations

- Q8 output must be operator-readable in a SQL client and structured enough for skill consumption.
- Candidate observations should expose `source_system` (joined or denormalized) so cross-source counts are trivially queryable.
- Backfilled rationale must remain legible after the originating conversation is gone — `evaluation_notes` carries the reasoning.
- No UI is in scope for v1; the data model should not preclude one.

## Acceptance Criteria

1. `semantic_modeling.candidate_terms` and `semantic_modeling.candidate_term_observations` tables exist, applied via numbered migration.
2. CHECK constraint enforces resolution-column exclusivity by status: `merged` requires `merged_into_candidate_id` and forbids `resolved_concept_key`; `promoted` requires `resolved_concept_key` and forbids `merged_into_candidate_id`; other statuses require both NULL.
3. Partial unique index on `normalized_term` for active states (`candidate`, `deferred`, `promoted`) prevents duplicate active candidates while permitting `merged`/`rejected`/`obsolete` namespace reuse.
4. `(vocabulary_term_id, version_id)` is unique on `candidate_term_observations`.
5. `record_candidate_terms.py` is idempotent: re-running on the same Phase 1 output produces no duplicate observations and no spurious state transitions.
6. `promote_term_mappings.py` writes lifecycle transitions for every candidate it reasons about during Phase 2, with `evaluation_notes` populated on each transition.
7. Q8 partitions Phase 2's undermapped term set into the six lifecycle states and surfaces prior `evaluation_notes` for already-evaluated candidates.
8. The reconcile-canonical-language skill runs Q8 at the start of Check 2 and operates over the accumulated candidate set, not just the current run's terms.
9. The 38 Adena deferrals from `pbi-research-orch-2e0b` Task 8 are backfilled as `status='deferred'` candidates with one observation each; cross-source count is 1 for all.
10. Obsolete sweep correctly transitions deferred candidates whose observations reference no `is_current` version.
11. No deletion of candidate or observation rows under any normal operation.

## Dependencies

- `peddlerr/api/db/migrations/` — next migration number (currently 041 is latest; this PBI introduces 042).
- `peddlerr/workbench/scripts/promote_term_mappings.py` — existing script being extended.
- `peddlerr/workbench/docs/operations/diagnostics/semantic_config_inventory.sql` — Q8 added here.
- `peddlerr/.agents/skills/reconcile-canonical-language/SKILL.md` — Check 2 update.
- `peddlerr/.claude/commands/reconcile-canonical-language.md` — mirror of skill update.
- `pbi-research-orch-2e0b` Task 8 E2E — source of the 38 backfill records.

## Open Questions

1. **Promotion criteria precision.** The "≥2 source systems OR widely understood" heuristic is currently informal. Should the promotion script encode this as a hard gate, or surface it as a recommendation that requires operator approval? Settled answer needed before Task 4 implementation.
2. **What constitutes "the same candidate" for dedup?** When `record_candidate_terms.py` sees `"Cardiology"` in source A and `"Cardiology"` in source B, it should attach both observations to the same candidate. But what about `"Cardiology Services"` vs `"Cardiology"`? Recommendation for v1: exact normalized-string match only, with merge as the explicit operator action for fuzzy matches. Confirm before Task 3 implementation.
3. **Operator review surface.** Q8 plus diagnostic queries vs. eventual UI? Out of scope for v1, but flag for downstream PBI consideration.

## Control-Plane Links

- [Task List](mdc:tasks.md)
- [Handoffs](mdc:handoffs.md)

## Related Tasks

- [Task List](mdc:tasks.md)
- [Task 1: Schema migration for `semantic_modeling.candidate_terms`](mdc:1.md)
- [Task 2: Q8 — candidate term review set diagnostic](mdc:2.md)
- [Task 3: `record_candidate_terms.py` — observation recording](mdc:3.md)
- [Task 4: `promote_term_mappings.py` — lifecycle evaluation](mdc:4.md)
- [Task 5: Obsolete sweep job](mdc:5.md)
- [Task 6: reconcile-canonical-language skill update for Q8](mdc:6.md)
- [Task 7: Backfill 38 Adena deferrals](mdc:7.md)
- [Task 8: E2E validation of accumulating candidate set](mdc:8.md)
