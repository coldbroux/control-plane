# Product Backlog

This document contains all PBIs for the project, ordered by priority.

## PBI Summary

| ID | Actor | User Story | Status | Conditions of Satisfaction (CoS) |
|:---|:------|:-----------|:-------|:----------------------------------|
| pbi-candidate-terms-70dd | Operator | As the operator of the canonical language, I want a persistent candidate-term layer so that semantic discovery accumulates across runs, cross-source convergence becomes queryable, and promotion into the canonical language is grounded in evidence rather than transient agent judgment. | InProgress | `pbis/pbi-candidate-terms-70dd/prd.md` |
| pbi-launch-apr-22 | Operator | As an operator, I want one explicit task that governs the next candidate launch attempt after the PR #64 reliability fixes so heartbeat/guard-snapshot behavior is verified in production and outcomes are auditable. | Done | `pbis/pbi-launch-apr-22/prd.md` |

## PBI Change History

| Timestamp | PBI_ID | Event_Type | Details | User |
|:----------|:-------|:-----------|:--------|:-----|
| 20260425-152139 | pbi-candidate-terms-70dd | create_pbi | Created PBI for a candidate-term memory layer enabling cross-run accumulation of semantic discovery; spun out of pbi-research-orch-2e0b Task 9 scope. | AI_Agent |
| 20260425-160500 | pbi-candidate-terms-70dd | propose_for_backlog | User approved the PBI scope, design refinements, and the semantic-placement gate; folder + repo index already created during proposal. | User |
| 20260425-160500 | pbi-candidate-terms-70dd | start_implementation | Beginning execution with Task 1 (schema migration). | User |
| 20260422-120000 | pbi-launch-apr-22 | create_pbi | Created PBI (legacy delivery model) to validate PR #64 reliability fixes (guard-snapshot retry, lock guards, commit-before-loop) via a real launch attempt. Originally drafted on `docs/pbi-launch-apr-22` branch in `crypto-trading-engine`. | AI_Agent |
| 20260422-120000 | pbi-launch-apr-22 | propose_for_backlog | Approved at draft time under legacy model; control-plane folder created post-hoc on 2026-04-26 during migration. | User |
| 20260426-094600 | pbi-launch-apr-22 | start_implementation | Operator authorized launch of candidate `859aa142-…` at $1,600 capital under post-PR-#64 baseline. Task 1 InProgress. | User |
| 20260426-103500 | pbi-launch-apr-22 | submit_for_review | Closure summary delivered: April 14 mode resolved (PR #64 confirmed); session failed at t+22m via NEW pair-leg partial-fill failure mode; auto-flatten succeeded; zero residual exposure. Follow-ups recorded as handoffs. | AI_Agent |
| 20260426-153500 | pbi-launch-apr-22 | approve | Operator approved closure during control-plane migration. Acceptance criteria met. | User |
