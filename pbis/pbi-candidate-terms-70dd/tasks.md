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
- `updated_at`: `2026-04-26 22:00:00`

## Task Summary

| Task ID | Name | Status | Description |
|---|---|---|---|
| 1 | `Schema migration for semantic_modeling.candidate_terms` (`1.md`) | InReview | Create the `semantic_modeling` schema with `candidate_terms` and `candidate_term_observations` tables, lifecycle constraints, partial unique index, and supporting indexes. |
| 2 | `Q8 — candidate term review set diagnostic` (`2.md`) | InReview | Add Q8 to the semantic config inventory diagnostic; partitions undermapped terms across the six lifecycle states with prior rationale surfaced. |
| 3 | `record_candidate_terms.py — observation recording` (`3.md`) | InReview | New script that creates candidate rows and observation links for Phase 1 vocabulary terms; idempotent on re-run. |
| 4 | `promote_term_mappings.py — lifecycle evaluation` (`4.md`) | InReview | Extend the promotion script to write lifecycle transitions (deferred / promoted / merged / rejected) with rationale captured in `evaluation_notes`. |
| 5 | `Obsolete sweep job` (`5.md`) | InReview | Maintenance pass that transitions deferred candidates to `obsolete` when all observations reference superseded source vocabulary versions. |
| 6 | `reconcile-canonical-language skill update for Q8` (`6.md`) | InReview | Update the skill and slash-command mirror to run Q8 at Check 2 and operate over the accumulated candidate set. |
| 7 | `Purge Adena-derived canonical language for clean-slate validation` (`7.md`) | InReview | One-shot operator-confirmed reset that removes all Adena-derived canonical-language state (non-seed canonical concepts, mapping definitions, candidate_terms, observations) so Tasks 8 and 9 exercise the new machinery from a true zero state. Source vocabulary preserved. |
| 8 | `Adena solo through the new pipeline` (`8.md`) | InReview | First real exercise of the new pipeline: record Adena's vocabulary, walk the skill, make decisions. Expected outcome — canonical tree stays at seed-roots-only because single-source signal cannot justify promotion. |
| 9 | `Adena + System B; demonstrate cross-source convergence` (`9.md`) | InReview | Add System B (AdventHealth, HSI00000010) as the second source. Walk the pipeline again. Every non-seed canonical concept post-execution must trace back to multi-source observations + explicit operator rationale (or document an absence analysis if no convergence is observed). |

## Notes

- Each task must have a corresponding numbered markdown file in this same folder.
- Task statuses in this file must always match the statuses recorded in the individual task files.
- Update this file whenever a task is added, removed, reordered, or changes status.
