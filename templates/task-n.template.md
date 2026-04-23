# <TASK-ID> <Task Title>

Back to task list: `tasks.md`

## Overview
<Brief summary of what this task does and why it exists within the PBI.>

### What if we didn't do this step?

**Immediate Impact:**
<What functionality, capability, or quality would be missing right now?>

**Downstream Consequences:**
<How would this affect later tasks, features, or the overall system?>

**Technical Debt:**
<What technical problems or limitations would persist or worsen?>

**Alternative Approaches:**
<Are there simpler workarounds or partial solutions that could suffice?>

**Risk Assessment:**
<What risks remain unmitigated? Rate as Low/Medium/High and explain briefly.>

## Description
<Describe the task in concrete terms. State what should be implemented, validated, reconciled, removed, or documented.>

## Control-Plane / Execution Context

- `pbi_id`: `<PBI-ID>`
- `task_id`: `<TASK-ID>`
- `status`: `Proposed`
- `branch`: `<branch-name-when-known>`
- `worktree`: `<worktree-path-when-known>`
- `session`: `<agent-or-session-identifier-when-known>`

> Leave branch/worktree/session as placeholders until work begins. Update them when the task moves to `InProgress`.

## Status History

| Timestamp | Event Type | From Status | To Status | Details | User |
|-----------|------------|-------------|-----------|---------|------|
| YYYY-MM-DD HH:MM:SS | Created | N/A | Proposed | <Task created> | AI_Agent |

## Requirements
1. <Concrete requirement>
2. <Concrete requirement>
3. <Concrete requirement>

## Planned Functional Code Changes
- [ ] `path/to/functional/file.ext` (create/update/delete)
- [ ] `path/to/functional/file.ext` (create/update/delete)

> Use repo-relative paths. Do not use absolute machine-local paths.

## Implementation Plan
1. <Implementation step>
2. <Implementation step>
3. <Implementation step>

## Test Plan

### Testing Objective
<What this task's tests are intended to prove.>

### Tests to Run
- `<exact command>`
- `<exact command>`
- `<exact command>`

### Success Criteria
- <Concrete, testable outcome>
- <Concrete, testable outcome>
- <Concrete, testable outcome>

## Planned Test Files
- [ ] `path/to/test_file.ext` (create/update)
- [ ] `path/to/test_file.ext` (create/update)

> Use repo-relative paths. Do not use absolute machine-local paths.

## Verification
- [ ] <Verification item tied to the task's intended outcome>
- [ ] <Verification item tied to the task's intended outcome>
- [ ] <Verification item tied to the task's intended outcome>

## Handoff Notes

<Optional. Use this section only when this task needs to hand work to a fresh session for promotion, validation, or cross-scope follow-up. The canonical handoff record must still be written in `handoffs.md`.>