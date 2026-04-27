# Handoffs

This file records cross-session handoff instances for pbi-candidate-terms-70dd.

## Usage

- Add a new section for each handoff.
- Do not delete old handoffs.
- Update the `status` field when the handoff progresses or is superseded.
- Include branch, worktree, session, PR, and merge commit when relevant.
- The goal is to give a fresh session enough context to pick up the correct handoff without re-discovering the situation.

---

## handoff-20260427-013000-001

- `handoff_id`: `handoff-20260427-013000-001`
- `timestamp`: `2026-04-27 01:30:00`
- `from`: `pbi-candidate-terms-70dd / Tasks 8 and 9 / closure session`
- `to`: `next PBI that touches the promotion path (or PRD update closing OQ1)`
- `status`: `Open`
- `repos`:
  - `peddlerr`
- `related_task_ids`:
  - `4`
  - `8`
  - `9`
- `pr`: `#9, #11, #12 (merged)`

### Reason

PRD Open Question 1 asked whether the cross-source-count check (`cross_source_count >= 2`) should become a hard gate alongside the placement gate, or stay a heuristic. Tasks 8 and 9 produced enough empirical evidence to recommend a definite answer; this handoff captures it so the next PBI in this area can close the question.

### Summary

The placement gate (the half settled at PRD time and codified in Task 4) is a hard application-layer block: a candidate cannot transition to `status='promoted'` unless its target canonical concept has `parent_concept_id IS NOT NULL`, OR an explicit `--allow-root` flag is supplied with operator-authored rationale.

The cross-source-count check, by contrast, is a heuristic warning. Empirical evidence from Tasks 8 and 9:

- Task 8 (Adena solo): 73 active candidates, 0 single-source promotions occurred. The placement gate alone was sufficient — when an operator might have considered promoting a single-source candidate, the requirement to provide a `parent_concept_key` and rationale made the cost-benefit explicit, and `defer` was naturally chosen.
- Task 9 (Adena + AdventHealth): 124 active candidates, 11 promotions executed, all with `cross_source_count = 2`. Zero `--allow-root` invocations. The heuristic warning never had to suppress an operator decision.

Two reasons NOT to harden the cross-source-count check:

1. Some legitimate canonical concepts are domain-specific to a single corpus (e.g., a research consortium's bespoke vocabulary). Forcing `>=2` would block them from ever being promoted, even when the operator has authoritative context.
2. The `--allow-root` escape hatch already exists for placement violations; doubling up the override surface for cross-source-count would just add ceremony without information value.

Recommendation: leave `cross_source_count >= 2` as a heuristic warning. Mark OQ1 as resolved in the PRD when the next PBI in this area lands.

### Next Action

Either (a) a follow-on PBI updates the PRD to mark OQ1 resolved with the recommendation above, or (b) a small documentation-only commit to the PRD does it directly. Either path closes this handoff.

---

## handoff-20260427-013000-002

- `handoff_id`: `handoff-20260427-013000-002`
- `timestamp`: `2026-04-27 01:30:00`
- `from`: `pbi-candidate-terms-70dd / Task 9 / closure session`
- `to`: `pbi-fuzzy-candidate-merge-XXXX (working name) — not yet authored`
- `status`: `Open`
- `repos`:
  - `peddlerr`
- `related_task_ids`:
  - `3`
  - `9`
- `pr`: `#9, #12 (merged)`

### Reason

PRD Open Question 2 asked what constitutes "the same candidate" for dedup in `record_candidate_terms.py`. v1 uses exact normalized-string match (`term.strip().lower()`). Task 9 surfaced concrete examples where this is too strict and produces missed convergences, which a future PBI should address.

### Summary

Empirical normalization gaps from the Adena+AdventHealth Task 9 run:

**Pluralization gap (1 example, but the pattern likely repeats):**
- Adena `Hospital` (singular, in `locations_api_location_types`) vs AdventHealth `Hospitals` (plural, in `find_locations_facility_types`) — same concept, missed convergence due to exact-string match.

**" Care" suffix pattern (the most prominent gap, ~10+ examples):**
AdventHealth's `services_pages` surface uses "X Care" as a consistent suffix; Adena's `specialty_pages` surface uses bare "X". Examples that should converge but did not:
- Adena `Audiology` vs AdventHealth `Audiology Care`
- Adena `Endocrinology` vs AdventHealth `Endocrinology Care`
- Adena `Nephrology` vs AdventHealth `Nephrology Care`
- Adena `Pulmonology` vs AdventHealth `Pulmonary Care`
- Adena `Orthopedics` vs AdventHealth `Orthopedic Care`
- Adena `Sports Medicine` vs AdventHealth `Sports Medicine and Rehab Care`
- Adena `Sleep` vs AdventHealth `Sleep Care`
- Adena `Spine and Back` vs AdventHealth `Spine Care`
- Adena `Urology` vs AdventHealth `Urology Care`
- Adena `Women's Health` vs AdventHealth `Women's Health Care`

**Hierarchical / semantic relationship gaps (need richer logic — partially overlaps OQ4):**
- Adena `Colorectal Cancer`, `Lung Cancer`, `Ovarian Cancer`, etc. all belong under the new `cancer_care` canonical concept (cross-source promoted in Task 9). They should appear as `subtype_of` candidates relative to `cancer_care`, not as separate top-level deferred candidates.
- Adena `Counseling`, `Psychiatry`, `Psychology` all relate to the new `behavioral_health` concept.

**Tradeoff to be careful about:**
Aggressive normalization risks false-positive convergence. Examples to test against:
- "Cardiology" vs "Pediatric Cardiology" — should NOT auto-merge; "Pediatric Cardiology" is a `subtype_of`.
- "Orthopedics" (the medical field) vs "Orthopedic Care" — should auto-merge.
- "Orthopedics" vs "Pediatric Orthopedics" — should NOT auto-merge.

Recommendation: do NOT modify `record_candidate_terms.py` to do fuzzy dedup automatically — that would break the immutable-normalization rule and introduce silent reasoning. Instead, add a fuzzy-match suggestion layer to Q8 (a `suggested_merge_with` column) that surfaces likely matches; the operator authors a `merge` decision explicitly. Recording stays exact-match; merges stay operator-driven.

### Next Action

Spin out a follow-on PBI (working name `pbi-fuzzy-candidate-merge-XXXX`). Required scope:

1. Schema: optional — could live entirely in Q8/SQL.
2. Q8 update: add `suggested_merge_with` (candidate_term_id) and `suggested_merge_score` (float). Use a configurable similarity function (start with `pg_trgm`-based similarity or a small Python normalization layer that strips known suffixes like " Care", " Services").
3. Skill update: `reconcile-canonical-language` walks suggested merges before walking deferred-with-cross-source candidates.
4. Test fixture: use the AdventHealth+Adena examples above. Specifically, the run should produce `merge` proposals for the " Care"-suffix pairs but NOT for the Pediatric vs adult variants.

Close this handoff when that PBI is authored.

---

## handoff-20260427-013000-003

- `handoff_id`: `handoff-20260427-013000-003`
- `timestamp`: `2026-04-27 01:30:00`
- `from`: `pbi-candidate-terms-70dd / Tasks 8 and 9 / closure session`
- `to`: `future PBI when scale demands a review tool`
- `status`: `Open`
- `repos`:
  - `peddlerr`
- `related_task_ids`:
  - `2`
  - `6`
  - `8`
  - `9`
- `pr`: `#9, #11, #12 (merged)`

### Reason

PRD Open Question 3 asked whether Q8 + diagnostic queries would suffice as the operator review surface, or whether a UI was needed. Tasks 8 and 9 ran end-to-end through the existing surface; this handoff captures the threshold at which a tool would actually pay for itself.

### Summary

Task 8 (Adena solo): 73 candidates → 73 hand-authored decisions in a JSON file. Manageable — fit comfortably in working memory; the build script (~30 lines) generated decisions from a flat candidate list.

Task 9 (Adena + AdventHealth): 124 active candidates → 135 decisions (multi-spec for cross-source promotes). Still manageable but starting to feel friction-bound — required cross-referencing Q8's bucket partition, the convergent-candidate query, the polysemy check, and the per-source observation list to author the decisions JSON.

Inflection points worth marking:

| Active candidates | Operator effort | Tool worth building |
|---:|---|---|
| < 200 | hand-authored JSON, 1-2 hours | no (current state) |
| 200 – 1k | hand-authored JSON becomes brittle; suggested-merge column in Q8 closes most of the gap | small CLI: `q8 --review` that walks the buckets interactively |
| 1k – 5k | hand-authored decisions infeasible | proper review CLI with batch defer / promote operations and grouping |
| > 5k | even CLI feels heavy | web UI with Q8 view + filtering + batch decisions |

Estimate: with ~10-15 source systems' vocabularies accumulating at ~100 candidates each, the 1k threshold becomes plausible. Adena (73) + AdventHealth (51 net new) = 124 after two systems, suggesting a scale of ~60 net-new candidates per system after the early systems contribute the most-overlapping vocabulary.

### Next Action

Defer OQ3. Trigger criteria for revisiting:

1. Operator can no longer fit Q8's output in a single screen view at default formatting (~50 rows).
2. A Phase 2 reconciliation session takes more than ~3 hours of operator time.
3. Cross-source convergence count crosses 50 candidates per session.

When any one of these triggers fires, scope a `pbi-q8-review-cli-XXXX` PBI. Recommend CLI before web UI — the marginal UX gain of a web UI is small relative to its scope, and a CLI integrates naturally with the existing JSON-decisions input format. Close this handoff when either the trigger fires or the question is dropped from PRD.

---

## handoff-20260427-013000-004

- `handoff_id`: `handoff-20260427-013000-004`
- `timestamp`: `2026-04-27 01:30:00`
- `from`: `pbi-candidate-terms-70dd / Task 9 / closure session`
- `to`: `pbi-canonical-relationships-XXXX (working name) — not yet authored`
- `status`: `Open`
- `repos`:
  - `peddlerr`
- `related_task_ids`:
  - `9`
- `pr`: `#12 (merged)`

### Reason

PRD Open Question 4 acknowledged that v1's canonical-concept model only supports `subtype_of` (the `parent_concept_id` self-FK). Richer relationship kinds — `equivalent_to`, `part_of`, `related_to`, `sees_also` — were deferred to a follow-on PBI. Task 9 generated the first concrete case where v1's model is insufficient; this handoff captures the empirical motivation and the schema sketch.

### Summary

**The polysemy case (deferred during Task 9 with rationale captured):**

The candidate `urgent care` is convergent across Adena and AdventHealth (`cross_source_count = 2`) but has two distinct semantic roles on AdventHealth:

| Surface | Role | Existing term-level mapping_definitions row |
|---|---|---|
| `find_locations_facility_types` | `location_type` (a kind of clinic/place) | `Urgent Care` → `location_type` |
| `services_pages` | `healthcare_offering` (a service line) | `Urgent Care` → `healthcare_offering` |

Promoting the candidate to a single new canonical concept (e.g., `urgent_care` under `healthcare_offering`) would either:
1. Conflict with the AdventHealth `find_locations_facility_types` term-level mapping (which says `location_type`), or
2. Lose the location-vs-offering distinction by collapsing both interpretations.

Splitting one candidate into two distinct canonical concepts (`urgent_care_offering` under `healthcare_offering` and `urgent_care_location` under `location_type`) is not supported by v1's candidate model — the candidate is a one-to-many container (one normalized term, many observations), and its `resolved_concept_key` is scalar.

Decision in Task 9: defer urgent_care with rationale. The deferral note is preserved in `semantic_modeling.candidate_terms.evaluation_notes` for the urgent_care candidate row and acts as the test case for the follow-on PBI.

**Other examples that this scope would unlock:**

- "lab" as a place (`location_type`) vs "lab services" as an offering (`healthcare_offering`)
- "cardiology" as a department vs cardiology as a service line vs cardiology as a specialty
- Cross-references between the new sibling concepts (e.g., `behavioral_health sees_also psychiatry` once psychiatry gets promoted)
- Part-whole relationships once cancer_care has subtype concepts (`colorectal_cancer part_of cancer_care`)

**Schema sketch:**

```sql
CREATE TABLE core_semantics.canonical_concept_relationships (
    id                    uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    source_concept_id     uuid NOT NULL REFERENCES core_semantics.canonical_concepts(id),
    target_concept_id     uuid NOT NULL REFERENCES core_semantics.canonical_concepts(id),
    relationship_kind     text NOT NULL CHECK (relationship_kind IN
                            ('equivalent_to', 'part_of', 'related_to', 'sees_also')),
    rationale             text,
    created_at            timestamptz NOT NULL DEFAULT now(),
    UNIQUE (source_concept_id, target_concept_id, relationship_kind),
    CHECK (source_concept_id <> target_concept_id)
);
```

(`subtype_of` stays on `canonical_concepts.parent_concept_id` — the most common kind deserves a dedicated FK, not a row.)

**Workflow additions:**

- New decision kind in `promote_term_mappings.py` (or a new sibling script): `relate` with `relationship_kind`, `source_concept_key`, `target_concept_key`, `rationale`.
- Skill update: `reconcile-canonical-language` emits `relate` decisions when it detects polysemy or part-whole structure during Check 2.
- Q8 extension: surface relationship targets (e.g., for a deferred polysemic candidate, suggest `relate` decisions to existing concepts).

### Next Action

Spin out the PBI (working name `pbi-canonical-relationships-XXXX`). Use the urgent_care candidate as the explicit test scenario:

1. PBI lands, schema migration adds `canonical_concept_relationships` and `relate`-kind support.
2. Re-run a slim version of Task 9 against just the urgent_care candidate.
3. Resolve it: promote two new concepts (`urgent_care_offering`, `urgent_care_location`) AND emit a `relate` decision (`urgent_care_offering equivalent_to urgent_care_location` or `related_to`, depending on operator judgment).
4. Provenance check: both new concepts have `source_system_count = 2`; the relationship row exists.

Close this handoff when that PBI lands and urgent_care moves from `deferred` to one of the resolved states.

---

## handoff-20260427-013000-005

- `handoff_id`: `handoff-20260427-013000-005`
- `timestamp`: `2026-04-27 01:30:00`
- `from`: `pbi-candidate-terms-70dd / Task 9 / closure session`
- `to`: `pbi-promote-mapping-coverage-XXXX (working name) — not yet authored, small follow-on`
- `status`: `Open`
- `repos`:
  - `peddlerr`
- `related_task_ids`:
  - `4`
  - `9`
- `pr`: `#9, #12 (merged)`

### Reason

Task 9 surfaced a real but small gap in `promote_term_mappings.py`: when a convergent candidate is promoted, only the FIRST source's `vocab_term` gets a term-level `mapping_definitions` row. Subsequent promote specs targeting the same already-promoted candidate are rejected ("cannot re-promote"). The candidate's cross-source provenance is intact via `candidate_term_observations`, but term-level mapping coverage is single-source. This handoff captures the fix scope.

### Summary

Empirical evidence from Task 9:

After the 11 cross-source promotions, every promoted concept satisfies the Task 9 acceptance criterion (`source_system_count >= 2` via `candidate_term_observations`):

```
behavioral_health    sources=[adena-health-system, adventhealth]
cancer_care          sources=[adena-health-system, adventhealth]
... (all 11)
```

But `mapping_definitions` term-level coverage is single-source — only Adena's `vocab_term` got a row, because the script's "cannot re-promote" rule rejected the AdventHealth promote spec for the same candidate:

```
behavioral_health  mapped_sources=[adena-health-system]   (AdventHealth's "Behavioral Health" term has no term-level mapping)
cancer_care        mapped_sources=[adena-health-system]
... (all 11)
```

12 of Task 9's 23 promote specs were rejected for this reason (the second-source promote spec per convergent candidate).

**Why this matters in practice:**

- Q8 will continue to show the second source's vocab_terms as "undermapped" (filtered out only by surface-level mapping defaults from Phase 1, not term-level mappings to the new canonical concept).
- The Task 6 skill update intentionally documented the `promoted` bucket as a "flag" exposed when an undermapped term traces to an already-promoted candidate. The skill's instruction to operators is "this is a sync/mapping bug worth surfacing."
- After Task 9, that bucket will contain ~12 known-state rows that are not actually bugs — they're the second-source observations of correctly-promoted candidates.

This adds noise to Q8 and dulls the "promoted bucket = real bug" signal.

**Fix scope:**

Modify `promote_term_mappings.py` to detect "candidate is already promoted to the same `target_concept_key`" and treat that case as an idempotent additional-mapping write rather than a rejection.

Pseudocode:
```python
existing = fetch_candidate(cur, candidate_term_id=spec['candidate_term_id'])
if existing and existing['status'] == 'promoted':
    if existing['resolved_concept_key'] == spec['target_concept_key']:
        # Already-promoted candidate, same target. Idempotent extra mapping.
        write_mapping_if_missing(cur, spec)
        summary.mapping_inserted_or_skipped += 1
        return True
    # Else genuinely conflicting target — reject as before.
    summary.reject(index, ident, f"candidate already promoted to {existing['resolved_concept_key']!r}, "
                                  f"different from requested {spec['target_concept_key']!r}")
    return False
```

Test cases:

1. Re-run Task 9's decisions against the current DB state with the fix → AdventHealth-side mappings get written (12 new mapping_definitions rows); a third run is a no-op.
2. Reject if the candidate is promoted to a DIFFERENT `target_concept_key` (real conflict — e.g., operator typos a wrong concept_key).
3. Reject if the candidate is in `merged`/`rejected`/`obsolete` status (existing behavior, unchanged).

### Next Action

Small one-task PBI (or a single-PR follow-up if the project allows ad-hoc PRs in this area). Working name `pbi-promote-mapping-coverage-XXXX`. Should land before the next two-source convergence run, otherwise the Q8 noise compounds. Close this handoff when the PR merges and Task 9's residual undermapped AdventHealth terms are mapped (verify by re-running the provenance query and checking that `mapping_definitions` term-level coverage is now multi-source for all 11 promoted concepts).
