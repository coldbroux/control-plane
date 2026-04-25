# Tasks for pbi-candidate-terms-70dd: Candidate Terms — A Memory Layer for Semantic Discovery

This document lists all tasks associated with pbi-candidate-terms-70dd.

**Parent PBI**: `pbi-candidate-terms-70dd: Candidate Terms — A Memory Layer for Semantic Discovery` (`prd.md`)

## Control-Plane Metadata

- `pbi_id`: `pbi-candidate-terms-70dd`
- `title`: `Candidate Terms — A Memory Layer for Semantic Discovery`
- `status`: `InProgress`
- `repos`:
  - `peddlerr`
- `created_at`: `2026-04-25 15:21:39`
- `updated_at`: `2026-04-25 16:05:00`

## Task Summary

| Task ID | Name | Status | Description |
|---|---|---|---|
| 1 | `Schema migration for semantic_modeling.candidate_terms` (`1.md`) | InProgress | Create the `semantic_modeling` schema with `candidate_terms` and `candidate_term_observations` tables, lifecycle constraints, partial unique index, and supporting indexes. |
| 2 | `Q8 — candidate term review set diagnostic` (`2.md`) | Proposed | Add Q8 to the semantic config inventory diagnostic; partitions undermapped terms across the six lifecycle states with prior rationale surfaced. |
| 3 | `record_candidate_terms.py — observation recording` (`3.md`) | Proposed | New script that creates candidate rows and observation links for Phase 1 vocabulary terms; idempotent on re-run. |
| 4 | `promote_term_mappings.py — lifecycle evaluation` (`4.md`) | Proposed | Extend the promotion script to write lifecycle transitions (deferred / promoted / merged / rejected) with rationale captured in `evaluation_notes`. |
| 5 | `Obsolete sweep job` (`5.md`) | Proposed | Maintenance pass that transitions deferred candidates to `obsolete` when all observations reference superseded source vocabulary versions. |
| 6 | `reconcile-canonical-language skill update for Q8` (`6.md`) | Proposed | Update the skill and slash-command mirror to run Q8 at Check 2 and operate over the accumulated candidate set. |
| 7 | `Backfill 38 Adena deferrals` (`7.md`) | Proposed | Insert deferred-status candidate rows + Adena observations for the 38 terms identified during pbi-research-orch-2e0b Task 8 E2E. |
| 8 | `E2E validation of accumulating candidate set` (`8.md`) | Proposed | Re-run Phase 2 to confirm prior rationale surfaces, recording is idempotent, and Q8 partitions correctly across runs. |
| 9 | `Two-system E2E validation — canonical language convergence` (`9.md`) | Proposed | Run two health systems through Phase 1 + Phase 2 end-to-end and verify that cross-source convergence is observable (or documented as absent) in the canonical concept tree. |

## Notes

- Each task must have a corresponding numbered markdown file in this same folder.
- Task statuses in this file must always match the statuses recorded in the individual task files.
- Update this file whenever a task is added, removed, reordered, or changes status.
