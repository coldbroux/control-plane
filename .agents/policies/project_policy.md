
# Project Policy

This policy provides a single, authoritative, and machine-readable source of truth for AI coding agents and humans, ensuring that all work is governed by clear, unambiguous rules and workflows. It aims to eliminate ambiguity, reduce supervision needs, and facilitate automation while maintaining accountability and compliance with best practices.

# 1. Introduction

> Rationale: Sets the context, actors, and compliance requirements for the policy, ensuring all participants understand their roles and responsibilities.

## 1.1 Actors

> Rationale: Defines who is involved in the process and what their roles are.

- **User**: The individual responsible for defining requirements, prioritising work, approving changes, and ultimately accountable for all code modifications.
- **AI_Agent**: The delegate responsible for executing the User's instructions precisely as defined by PBIs and tasks.

## 1.2 Architectural Compliance

> Rationale: Ensures all work aligns with architectural standards and references, promoting consistency and quality across the project.

- **Policy Reference**: This document adheres to the AI Coding Agent Policy Document.
- **Includes**:
  - All tasks must be explicitly defined and agreed upon before implementation.
  - All code changes must be associated with a specific task.
  - All PBIs must be aligned with the PRD when applicable.

# 2. Fundamental Principles

> Rationale: Establishes the foundational rules that govern all work, preventing scope creep, enforcing accountability, and ensuring the integrity of the development process.

## 2.1 Core Principles

> Rationale: Lists the essential guiding principles for all work, such as task-driven development, user authority, and prohibition of unapproved changes.

1. Task-Driven Development: No code shall be changed in the codebase unless there is an agreed-upon task explicitly authorising that change.
2. PBI Association: No task shall be created unless it is directly associated with an agreed-upon Product Backlog Item (PBI).
3. PRD Alignment: If a Product Requirements Document (PRD) is linked to the product backlog, PBI features must be sense-checked to ensure they align with the PRD's scope.
4. User Authority: The User is the sole decider for the scope and design of ALL work.
5. User Responsibility: Responsibility for all code changes remains with the User, regardless of whether the AI Agent performed the implementation.
6. Prohibition of Unapproved Changes: Any changes outside the explicit scope of an agreed task are EXPRESSLY PROHIBITED.
7. Task Status Synchronisation: The status of tasks in the tasks index (1-tasks.md) must always match the status in the individual task file. When a task's status changes, both locations must be updated immediately.
8. **Workstream Isolation**: When work is being performed concurrently, each active AI_Agent session must operate in its own isolated workstream consisting of one branch, one worktree, one active task or PBI scope, and one AI_Agent session. Concurrent sessions must not share a worktree.
9. **Promotion Boundary**: Implementation work and promotion work are distinct responsibilities. Promotion to `main` must occur only through the PR/CI flow and must not be performed as an implicit side effect of implementation.
10. **Control-Plane as Coordination Layer**: Cross-session coordination and handoffs must be recorded in the control-plane repository rather than relying on uncommitted repo-local files or conversational context alone.
11. **Controlled File Creation**: The AI_Agent shall not create any files, including standalone documentation files, that are outside the explicitly defined structures for PBIs (see 3.6), tasks (see 4.1), or source code, unless the User has given explicit prior confirmation for the creation of each specific file. This principle is to prevent the generation of unrequested or unmanaged documents.
12. **External Package Research and Documentation**: For any proposed tasks that involve external packages, to avoid hallucinations, use the web to research the documentation first to ensure it's 100% clear how to use the API of the package. Then for each package, a document should be created `<task id>-<package>-guide.md` that contains a fresh cache of the information needed to use the API. It should be date-stamped and link to the original docs provided. E.g., if pg-boss is a library to add as part of task 2-1 then a file `tasks/2-1-pg-boss-guide.md` should be created. This documents foundational assumptions about how to use the package, with example snippets, in the language being used in the project.
13. **Task Granularity**: Tasks must be defined to be as small as practicable while still representing a cohesive, testable unit of work. Large or complex features should be broken down into multiple smaller tasks.
14. **Don't Repeat Yourself (DRY)**: Information should be defined in a single location and referenced elsewhere to avoid duplication and reduce the risk of inconsistencies. Specifically:
    - Task information should be fully detailed in their dedicated task files and only referenced from other documents.
    - PBI documents should reference the task list rather than duplicating task details.
    - The only exception to this rule is for titles or names (e.g., task names in lists).
    - Any documentation that needs to exist in multiple places should be maintained in a single source of truth and referenced elsewhere.
15. **Use of Constants for Repeated or Special Values**: Any value (number, string, etc.) used more than once in generated code—especially "magic numbers" or values with special significance—**must** be defined as a named constant.
    - Example: Instead of `for (let i = 0; i < 10; i++) { ... }`, define `const numWebsites = 10;` and use `for (let i = 0; i < numWebsites; i++) { ... }`.
    - All subsequent uses must reference the constant, not the literal value.
    - This applies to all code generation, automation, and manual implementation within the project.
16. **Technical Documentation for APIs and Interfaces**: As part of completing any PBI that creates or modifies APIs, services, or interfaces, technical documentation must be created or updated explaining how to use these components. This documentation should include:
    - API usage examples and patterns
    - Interface contracts and expected behaviors
    - Integration guidelines for other developers
    - Configuration options and defaults
    - Error handling and troubleshooting guidance
    - The documentation must be created in the appropriate location (e.g., `docs/technical/` or inline code documentation) and linked from the PBI detail document.

## 2.2 PRD Alignment Check

All PBIs must be checked for alignment with the PRD. Any discrepancies must be raised with the User.

## 2.3 Integrity and Sense Checking

All data must be sense-checked for consistency and accuracy.

## 2.4 Scope Limitations

> Rationale: Prevents unnecessary work and keeps all efforts focused on agreed tasks, avoiding gold plating and scope creep.

- No gold plating or scope creep is allowed.
- All work must be scoped to the specific task at hand.
- Any identified improvements or optimizations must be proposed as separate tasks.

## 2.5 Change Management Rules

> Rationale: Defines how changes are managed, requiring explicit association with tasks and strict adherence to scope.

- Conversation about any code change must start by ascertaining the linked PBI or Task before proceeding.
- All changes must be associated with a specific task.
- No changes should be made outside the scope of the current task.
- Any scope creep must be identified, rolled back, and addressed in a new task.
- If the User asks to make a change without referring to a task, then the AI Agent must not do the work and must have a conversation about it to determine if it should be associated with an existing task or if a new PBI + task should be created.

## 2.5.1 Workstream and Handoff Rules

> Rationale: Prevents cross-session interference, clarifies ownership boundaries, and ensures promotion and validation occur from explicit handoffs rather than accumulated hidden context.

- Every active implementation session must be associated with exactly one current task and one execution context.
- An execution context (also referred to in this document as a workstream) consists of:
  - one branch
  - one worktree
  - one active task or PBI scope
  - one AI_Agent session
- When concurrent work is occurring, each session must use a distinct worktree.
- If an AI_Agent discovers work that belongs to a different task, PBI, or repository scope:
  - stop
  - record a handoff in the control-plane
  - include branch, worktree, and session details when relevant
  - do not absorb the work into the current task unless the User explicitly approves the scope change
- Before any git write operation (`git add`, `git commit`, `git push`, merge), the AI_Agent must verify:
  - current working directory
  - git top-level directory
  - current branch
  - staged file list
- If the current worktree, branch, or staged files do not match the assigned execution context or task scope, the AI_Agent must stop and surface the mismatch to the User.
- Promotion work should normally be executed in a fresh session that receives a handoff from the implementation session.
- Validation work that depends on promoted or deployed state should normally be executed in a fresh session after promotion completes.

# 3. Product Backlog Item (PBI) Management

> Rationale: Defines how PBIs are managed, ensuring clarity, traceability, and effective prioritisation of all work.

## 3.1 Overview

This section defines rules for Product Backlog Items (PBIs), ensuring clarity, consistency, and effective management throughout the project lifecycle.

## 3.2 Backlog Document Rules

> Rationale: Specifies how the backlog is documented and structured, so that all PBIs are tracked and managed in a consistent way.

- **Location Pattern**: docs/delivery/backlog.md
- **Scope Purpose Description**: The backlog document contains all PBIs for the project, ordered by priority.
- **Structure**:
  > Rationale: Defines the required table format for PBIs, ensuring standardisation and ease of use.
  - **Table**: `| ID | Actor | User Story | Status | Conditions of Satisfaction (CoS) |`

### 3.2.1 PBI Naming Convention

> Rationale: Defines the naming convention for PBIs to eliminate merge conflicts and provide meaningful context while maintaining uniqueness across distributed development.

#### Format
```
pbi-<abbreviated-short-name>-<hash>
```

#### Generation Rules
1. **Abbreviated Short Name**:
   - Extract 2-3 most meaningful words from PBI title/user story
   - Skip articles (a, an, the) and common words (for, with, to, etc.)
   - Convert to lowercase
   - Join with hyphens
   - Minimum 6 characters, maximum 15 characters

2. **Hash Generation**:
   - Take current UTC timestamp in ISO format: `YYYY-MM-DDTHH:mm:ss.sssZ`
   - Calculate SHA-256 hash of timestamp string
   - Take first 4 characters of hexadecimal hash
   - Convert to lowercase

3. **Validation Rules**:
   - Must match pattern: `pbi-[a-z-]{6,15}-[a-f0-9]{4}`
   - Complete PBI ID must not exceed 30 characters
   - Verify uniqueness before finalizing

#### Examples
- "Implement user authentication system" → `pbi-user-auth-a7f3`
- "Add payment processing with Stripe" → `pbi-payment-stripe-b2e9`
- "Fix database connection timeout issues" → `pbi-db-timeout-fix-c4d1`

## 3.3 Principles
1. The backlog is the single source of truth for all PBIs.
2. PBIs must be ordered by priority (highest at the top).

## 3.4 PBI Workflow

> Rationale: Describes the allowed status values and transitions for PBIs, ensuring a controlled and auditable workflow.

### 3.4.1 Status Definitions
- **status(Proposed)**: PBI has been suggested but not yet approved.
- **status(Agreed)**: PBI has been approved and is ready for implementation.
- **status(InProgress)**: PBI is being actively worked on.
- **status(InReview)**: PBI implementation is complete and awaiting review.
- **status(Done)**: PBI has been completed and accepted.
- **status(Rejected)**: PBI has been rejected and requires rework or deprioritization.

### 3.4.2 Event Transitions
- **event_transition on "create_pbi": set to **Proposed**:
  1. Define clear user story and acceptance criteria.
  2. Ensure PBI has a unique ID and clear title.
  3. Log creation in PBI history.

- **event_transition on "propose_for_backlog" from Proposed to Agreed**:
  1. Verify PBI aligns with PRD and project goals.
  2. Ensure all required information is complete.
  3. Create the PBI's control-plane folder and initial coordination records.
     - The control-plane PBI folder must contain:
       - `prd.md`
       - `tasks.md`
       - `handoffs.md`
     - The PBI must declare the repositories it touches using a `repos:` field.
  4. Update every affected repo index document in the control-plane:
     - `repos/<repo-name>.md`
     - Each listed repo must receive an append-only entry for the new PBI including:
       - timestamp
       - pbi_id
       - title
       - repos
  5. Log approval in PBI history.
  6. Notify stakeholders of new approved PBI.

- **event_transition on "start_implementation" from Agreed to InProgress**:
  1. Verify no other PBIs are InProgress for the same component.
  2. Create tasks for implementing the PBI.
  3. Assign initial tasks to team members.
  4. Log start of implementation in PBI history.

- **event_transition on "submit_for_review" from InProgress to InReview**:
  1. Verify all tasks for the PBI are complete.
  2. Ensure all acceptance criteria are met.
  3. Update documentation as needed.
  4. Notify reviewers that PBI is ready for review.
  5. Log submission for review in PBI history.

- **event_transition on "approve" from InReview to Done**:
  1. Verify all acceptance criteria are met.
  2. Ensure all tests pass.
  3. Update PBI status and completion date.
  4. Archive related tasks and documentation.
  5. Log approval and completion in PBI history.
  6. Notify stakeholders of PBI completion.

- **event_transition on "reject" from InReview to Rejected**:
  1. Document reasons for rejection.
  2. Identify required changes or rework.
  3. Update PBI with review feedback.
  4. Log rejection in PBI history.
  5. Notify team of required changes.

- **event_transition on "reopen" from Rejected to InProgress**:
  1. Address all feedback from rejection.
  2. Update PBI with changes made.
  3. Log reopening in PBI history.
  4. Notify team that work has resumed.

- **event_transition on "deprioritize" from (Agreed, InProgress) to Proposed**:
  1. Document reason for deprioritization.
  2. Pause any in-progress work on the PBI.
  3. Update PBI status and priority.
  4. Log deprioritization in PBI history.
  5. Notify stakeholders of the change in priority.

### 3.4.3 Note

All status transitions must be logged in the PBI's history with timestamp and user who initiated the transition.

## 3.5 PBI History Log

> Rationale: Specifies how all changes to PBIs are recorded, providing a complete audit trail.

- **Location Description**: PBI change history is maintained in the backlog.md file.
- **Fields**:
  - **history_field(Timestamp)**: Date and time of the change (YYYYMMDD-HHMMSS).
  - **history_field(PBI_ID)**: ID of the PBI that was changed.
  - **history_field(Event_Type)**: Type of event that occurred.
  - **history_field(Details)**: Description of the change.
  - **history_field(User)**: User who made the change.

## 3.6 PBI Detail Documents

> Rationale: Provides a dedicated space for detailed requirements, technical design, and UX considerations for each PBI, ensuring comprehensive documentation and alignment across the team.

- **Location Pattern**: `docs/delivery/<PBI-ID>/prd.md`
  - **Example**: `docs/delivery/pbi-user-auth-a7f3/prd.md`
- **Purpose**:
  - Serve as a mini-PRD for the PBI
  - Document the problem space and solution approach
  - Provide technical and UX details beyond what's in the backlog
  - Maintain a single source of truth for all PBI-related information
- **Required Sections**:
  - `# <PBI-ID>: <Title>` (e.g., `# pbi-user-auth-a7f3: Implement user authentication system`)
  - `## Overview`
  - `## Problem Statement`
  - `## User Stories`
  - `## Technical Approach`
  - `## UX/UI Considerations`
  - `## Acceptance Criteria`
  - `## Dependencies`
  - `## Open Questions`
  - `## Related Tasks`
- **Linking**:
  - Must link back to the main backlog entry: `[View in Backlog](mdc:backlog.md#user-content-<PBI-ID>)`
  - The backlog entry must link to this document: `[View Details](mdc:<PBI-ID>/prd.md)`
- **Ownership**:
  - Created when a PBI moves from "Proposed" to "Agreed"
  - Maintained by the team member implementing the PBI
  - Reviewed during PBI review process
  
## 3.7 Control-Plane Coordination Records

> Rationale: Provides a branch-independent coordination layer for cross-session handoffs and repo-level navigation without relying on uncommitted repo-local files or conversational memory.

- The control-plane repository is the canonical coordination layer for active and recently created PBIs.
- Control-plane records are organized canonically by PBI.
- When a PBI moves from `Proposed` to `Agreed`, its control-plane folder must be created.
- Each PBI in the control-plane must have a folder containing:
  - `prd.md`
  - `tasks.md`
  - `handoffs.md`
- `handoffs.md` is the canonical location for recording handoff instances related to the PBI.
- Each PBI must declare the repositories it touches using a `repos:` field.
- Repo-level index documents are required for navigation.
- The control-plane repository must contain:
  - `repos/<repo-name>.md`
- When a PBI moves from `Proposed` to `Agreed`, every repo listed in the PBI's `repos:` field must be updated with an entry in its repo index document.
- Repo index documents are append-only navigation aids. They do not replace the canonical PBI folder.

### 3.7.1 Repo Index Document Requirements

Each repo index document must be an append-only log of PBIs that touch that repo.

Each entry must include:

- `timestamp`
- `pbi_id`
- `title`
- `repos`

Repo index documents are used only to make it easy to find which PBIs touch which repo(s). They are not responsible for tracking ongoing PBI status.

### 3.7.2 Handoff Entry Requirements

Every handoff recorded in `handoffs.md` must be a distinct entry with enough metadata for a fresh session to identify and act on the correct handoff.

Each handoff entry must include:

- `handoff_id`
- `timestamp`
- `from`
- `to`
- `status`
- `repos`
- `related_task_ids` when applicable
- `branch` when applicable
- `worktree` when applicable
- `session` when applicable
- `reason`
- `summary`
- `next_action`

If the handoff relates to promotion or post-promotion validation, include when relevant:

- `pr`
- `merge_commit`

### 3.7.3 Handoff Status Values

Permitted handoff statuses are:

- `Open`
- `InProgress`
- `Resolved`
- `Superseded`

### 3.7.4 Handoff Purpose

Handoffs are used for:

- cross-scope discoveries
- implementation → promotion transitions
- promotion → validation transitions
- any other situation where one session must stop and transfer responsibility to a fresh session

# 4. Task Management

> Rationale: Defines how tasks are documented, executed, and tracked, ensuring that all work is broken down into manageable, auditable units.

## 4.1 Task Documentation

> Rationale: Specifies the structure and content required for task documentation, supporting transparency and reproducibility.

- **Location Pattern**: docs/delivery/<PBI-ID>/
  - **Example**: `docs/delivery/pbi-user-auth-a7f3/`
- **File Naming**:
  - Task list: `tasks.md`
  - Task details: `1.md`, `2.md`, `3.md`, etc. (simple numeric naming)
- **Required Sections**:
  > Rationale: Required sections ensure all tasks are fully described and verifiable
  - `# [Task-ID] [Task-Name]` (e.g., `# 1 Create User Authentication Database Schema`)
  - `## Overview`
    - `### What if we didn't do this step?` (subsection - see 4.1.1)
  - `## Description`
  - `## Status History`
  - `## Requirements`
  - `## Planned Functional Code Changes` (see 4.1.2)
  - `## Implementation Plan`
  - `## Test Plan` (see 6.3.1)
    - `### Testing Objective`
    - `### Tests to Run`
    - `### Success Criteria`
  - `## Planned Test Files` (see 6.3.1)
  - `## Verification`

### 4.1.1 "What if we didn't do this step?" Subsection

> Rationale: Enables informed decision-making by explicitly documenting the costs and consequences of not implementing a task, helping the User prioritize work and understand trade-offs.

Every task document **must** include a "What if we didn't do this step?" subsection within the Overview section. This subsection helps the User make informed decisions about whether to proceed with the task by clearly articulating what would be lost or compromised without its implementation.

**Required Content**:

1. **Immediate Impact**: What functionality, capability, or quality would be missing right now?
2. **Downstream Consequences**: How would this affect other tasks, features, or the overall system?
3. **Technical Debt Assessment**: What technical problems or limitations would persist or worsen?
4. **Alternative Approaches**: Are there simpler workarounds or partial solutions that could suffice?
5. **Risk Evaluation**: What risks (security, performance, reliability, maintainability) would remain unaddressed?

**Format Template**:
```markdown
### What if we didn't do this step?

**Immediate Impact:**
[Clear statement of what would be missing or broken]

**Downstream Consequences:**
[How this affects other parts of the system or future work]

**Technical Debt:**
[What problems would persist or worsen]

**Alternative Approaches:**
[Any simpler alternatives, or "None identified" if truly essential]

**Risk Assessment:**
[What risks remain unmitigated - rate as Low/Medium/High]
```

**Writing Guidelines**:
- Be honest and objective - don't oversell the task's importance
- Focus on concrete, measurable impacts rather than abstract concerns
- Distinguish between "nice to have" and "critical for system integrity"
- If alternatives exist, explain their trade-offs clearly
- Help the User understand the cost-benefit ratio of proceeding vs. skipping

**Example**:
```markdown
### What if we didn't do this step?

**Immediate Impact:**
Without the backfill system, data gaps discovered by Task 4 would need to be filled manually using ad-hoc scripts, requiring developer intervention for each gap.

**Downstream Consequences:**
- Historical analysis and backtesting would have incomplete data
- Manual gap-filling is error-prone and time-consuming
- No systematic way to maintain data completeness over time
- Future features requiring complete historical data would be blocked

**Technical Debt:**
Without rate limiting and retry logic, manual backfill attempts could trigger API bans or fail silently, creating data quality issues that are hard to detect.

**Alternative Approaches:**
Could implement a simple script-based approach for immediate needs, but it would:
- Require manual monitoring and execution
- Lack progress tracking and resume capability
- Not integrate with gap detection system
- Need to be rewritten if requirements grow

**Risk Assessment:**
- Data Quality: HIGH - incomplete data affects all downstream analysis
- Operational: MEDIUM - manual intervention required for routine maintenance
- Scalability: HIGH - doesn't scale with multiple exchanges/symbols
- Future-proofing: MEDIUM - limits ability to expand data coverage
```

### 4.1.2 Planned Functional Code Changes Section

> Rationale: Forces task authors to ground plans in concrete code targets and keep a running, append-only ledger of intended functional code changes.

Every task document **must** include a "Planned Functional Code Changes" section. This section lists **functional** code files, i.e. non-test files, that are expected to be created, updated, or deleted for the task. The list is **append-only** and is expected to grow during task execution as additional required work is uncovered. Do not remove entries; instead update their status.

**Required Format**:
```markdown
## Planned Functional Code Changes
- [ ] `path/to/file.py` (create)
- [ ] `path/to/other_file.ts` (update)
- [ ] `path/to/legacy_file.sql` (delete)
```

## 4.2 Principles
1. Each task must have its own dedicated markdown file.
2. Task files must follow the specified naming convention.
3. All required sections must be present and properly filled out.
4. When adding a task to the tasks index, its markdown file MUST be created immediately and linked using the pattern `[description](mdc:<TASK-ID>.md)` (e.g., `[Create Database Schema](mdc:1.md)`).
5. Individual task files must link back to the tasks index using the pattern `[Back to task list](mdc:tasks.md)`.

## 4.3 Task Workflow

> Rationale: Describes the allowed status values and transitions for tasks, ensuring a controlled and auditable workflow.

## 4.4 Task Status Synchronisation

To maintain consistency across the codebase:

1. **Immediate Updates**: When a task's status changes, update both the task file and the tasks index (1-tasks.md) in the same commit.
2. **Status History**: Always add an entry to the task's status history when changing status.
3. **Status Verification**: Before starting work on a task, verify its status in both locations.
4. **Status Mismatch**: If a status mismatch is found, immediately update both locations to the most recent status.

Example of a status update in a task file:
| Timestamp           | Change Type   | From     | To         | Notes             | Author |
|--------------------|---------------|----------|------------|-------------------|--------|
| 2025-05-19 15:02:00 | Created       | N/A      | Proposed   | Task file created | Julian |
| 2025-05-19 16:15:00 | Status Update | Proposed | InProgress | Started work      | Julian |

Example of the corresponding update in tasks.md:
| ID | Task                                   | Status     | Notes                   |
|----|----------------------------------------|------------|-------------------------|
| 1  | [Add pino logging...](mdc:1.md)         | InProgress | Pino logs connection... |

## 4.5 Status Definitions
- **task_status(Proposed)**: The initial state of a newly defined task.
- **task_status(Agreed)**: The User has approved the task description and its place in the priority list.
- **task_status(InProgress)**: The AI Agent is actively working on this task.
- **task_status(Review)**: The AI Agent has completed the work and it awaits User validation.
- **task_status(Done)**: The User has reviewed and approved the task's implementation.
- **task_status(Blocked)**: The task cannot proceed due to an external dependency or issue.

## 4.6 Event Transitions
- **event_transition on "user_approves" from Proposed to Agreed**:
  1. Verify task description is clear and complete.
  2. Ensure task is properly prioritized in the backlog.
  3. Create task documentation file following the _template.md pattern and link it in the tasks index.
   - The file must be named `<TASK-ID>.md` (e.g., `1.md`, `2.md`)
   - The task description in the index must link to this file
   - Analysis and design work must be undertaken and documented in the required sections of task file
  4. Log status change in task history.

- **event_transition on "start_work" from Agreed to InProgress**:
  1. Verify no other tasks are InProgress for the same PBI.
  2. Create a new branch for the task if using version control.
  3. Log start time and assignee in task history.
  4. Record execution context in the task documentation when available, including:
     - branch
     - worktree
     - session or agent identifier
  5. Update task documentation with implementation start details.

- **event_transition on "submit_for_review" from InProgress to Review**:
  1. Ensure all task requirements are met.
  2. Run all relevant tests and ensure they pass.
  3. Update task documentation with implementation details.
  4. Create a pull request or mark as ready for review.
  5. Notify the User that review is needed.
  6. Log submission for review in task history.
  7. **Complete Verification Checklist**: Mark all completed verification checklist items as `[x]` in the task's Verification section to accurately reflect what has been accomplished during implementation.

- **event_transition on "approve" from Review to Done**:
  1. Verify all acceptance criteria are met.
  2. Ensure any required promotion step has been completed if the task depends on merged or deployed state.
  3. If promotion occurred, record the relevant promotion linkage in the task documentation or related handoff when available, including:
     - branch
     - PR
     - merge commit
  4. Update task documentation with completion details.
  5. Update task status and log completion time.
  6. Archive task documentation as needed.
  7. Notify relevant stakeholders of completion.
  8. Log approval in task history.
  9. **Review Next Task Relevance**: Before marking as Done, review the next task(s) in the PBI task list in light of the current task's implementation outcomes. Confirm with the User whether subsequent tasks remain relevant, need modification, or have become redundant due to implementation decisions or scope changes in the current task. Document any task modifications or removals in the task history.

- **event_transition on "reject" from Review to InProgress**:
  1. Document the reason for rejection in task history.
  2. Update task documentation with review feedback.
  3. Notify the AI Agent of required changes.
  4. Update task status and log the rejection.
  5. Create new tasks if additional work is identified.

- **event_transition on "significant_update" from Review to InProgress**:
  1. Document the nature of significant changes made to task requirements, implementation plan, or test plan.
  2. Update task status to reflect that additional work is now required.
  3. Log the update reason in task history.
  4. Notify stakeholders that the task requires additional implementation work.
  5. Resume development work to address the updated requirements.

- **event_transition on "mark_blocked" from InProgress to Blocked**:
  1. Document the reason for blocking in task history.
  2. Identify any dependencies or issues causing the block.
  3. Update task documentation with blocking details.
  4. Notify relevant stakeholders of the block.
  5. Consider creating new tasks to address blockers if needed.

- **event_transition on "unblock" from Blocked to InProgress**:
  1. Document the resolution of the blocking issue in task history.
  2. Update task documentation with resolution details.
  3. Resume work on the task.
  4. Notify relevant stakeholders that work has resumed.

## 4.7 One In Progress Task Limit

Only one task per PBI should be 'InProgress' at any given time to maintain focus and clarity. In special cases, the User may approve additional concurrent tasks.

## 4.8 Task History Log

> Rationale: Specifies how all changes to tasks are recorded, providing a complete audit trail.

- **Location Description**: Task change history is maintained in the task's markdown file under the 'Status History' section.
- **Required Fields**:
  > Rationale: Defines the required fields for logging task history, ensuring all relevant information is captured.
  - **history_field(Timestamp)**: Date and time of the change (YYYY-MM-DD HH:MM:SS).
  - **history_field(Event_Type)**: Type of event that occurred.
  - **history_field(From_Status)**: Previous status of the task.
  - **history_field(To_Status)**: New status of the task.
  - **history_field(Details)**: Description of the change or action taken.
  - **history_field(User)**: User who initiated the change.

### 4.8.1 Format Example

| Timestamp | Event Type | From Status | To Status | Details | User |
|-----------|------------|-------------|-----------|---------|------|
| 2025-05-16 15:30:00 | Status Change | Proposed | Agreed | Task approved by Product Owner | johndoe |
| 2025-05-16 16:45:00 | Status Change | Agreed | InProgress | Started implementation | ai-agent-1 |

### 4.8.2 Task Validation Rules

> Rationale: Ensures all tasks adhere to required standards and workflows.

1. **Core Rules**:
   - All tasks must be associated with an existing PBI
   - Task IDs must be unique within their parent PBI
   - Tasks must follow the defined workflow states and transitions
   - All required documentation must be completed before marking a task as 'Done'
   - Task history must be maintained for all status changes
   - Only one task per PBI may be 'InProgress' at any time, unless explicitly approved
   - Every task in the tasks index MUST have a corresponding markdown file
   - Task descriptions in the index MUST be linked to their markdown files

2. **Pre-Implementation Checks**:
   - Verify the task exists and is in the correct status before starting work
   - Document the task ID in all related changes
   - List all files that will be modified
   - Get explicit approval before proceeding with implementation

3. **Error Prevention**:
   - If unable to access required files, stop and report the issue
   - For protected files, provide changes in a format that can be manually applied
   - Verify task status in both task file and index before starting work
   - Document all status checks in the task history

4. **Change Management**:
   - Reference the task ID in all commit messages
   - Update task status according to workflow
   - Ensure all changes are properly linked to the task
   - Document any deviations from the planned implementation

### 4.9 Version Control for Task Completion

> Rationale: Ensures consistent version control practices when completing tasks, maintaining traceability and automation.

1. **Commit Message Format**:
   - When task-related changes are committed, the commit message should include the task identifier and a concise description.
   - Preferred format:
     ```
     <task_id> <task_description>
     ```
   - Example: `1-7 Add pino logging to help debug database connection issues`

2. **Pull Request**:
   - When a task's work is promoted through a pull request, the title should use:
     ```
     [<task_id>] <task_description>
     ```
   - Include a link to the task in the description.

3. **Automation**:
   - Automation such as `git acp "<task_id> <task_description>"` may be used when it fits the current workflow, but it must not override the promotion boundary or any required PR/CI flow.

4. **Verification**:
   - Ensure relevant commits appear in the task's history when applicable.
   - Confirm the task status is updated in both the task file and index.
   - Verify commit and PR metadata follow the required format when used.

## 4.10 Task Index File

> Rationale: Defines the standard structure for PBI-specific task index files, ensuring a consistent overview of tasks related to a Product Backlog Item.

*   **Location Pattern**: `docs/delivery/<PBI-ID>/tasks.md`
    *   Example: `docs/delivery/pbi-user-auth-a7f3/tasks.md`
*   **Purpose**: To list all tasks associated with a specific PBI, provide a brief description for each, show their current status, and link to their detailed task files.
*   **Required Sections and Content**:
    1.  **Title**:
        *   Format: `# Tasks for <PBI-ID>: <PBI Title>`
        *   Example: `# Tasks for pbi-user-auth-a7f3: Implement user authentication system`
    2.  **Introduction Line**:
        *   Format: `This document lists all tasks associated with <PBI-ID>.`
    3.  **Link to Parent PBI**:
        *   Format: `**Parent PBI**: [<PBI-ID>: <PBI Title>](mdc:prd.md)`
        *   Example: `**Parent PBI**: [pbi-user-auth-a7f3: Implement user authentication system](mdc:prd.md)`
    4.  **Task Summary Section Header**:
        *   Format: `## Task Summary`
    5.  **Task Summary Table**:
        *   Columns: `| Task ID | Name | Status | Description |`
        *   Markdown Table Structure:

| Task ID | Name | Status | Description |
|---|---|---|---|
| 1 | [Task name](mdc:1.md) | Proposed | Brief task description |

*   **`Task ID`**: The unique identifier for the task within the PBI (e.g., `1`, `2`, `3`).
*   **`Name`**: The full name of the task, formatted as a Markdown link to its individual task file (e.g., `[Create User Authentication Database Schema](mdc:1.md)`).
*   **`Status`**: The current status of the task, which must be one of the permitted values defined in Section 4.5 (Status Definitions).
*   **`Description`**: A brief, one-sentence description of the task's objective.
*   **Prohibited Content**:
    *   The Task Summary table in this file must only contain what is specified, and nothing else, unless a User specifically oks it.

# 6. Testing Documentation and Implementation Strategy

> Rationale: Clarifies the comprehensive testing approach across individual tasks, integration testing, and end-to-end validation to ensure proper test coverage distribution and eliminate circular dependencies.
## 6.1 Overview

This section defines how testing should be documented and implemented across PBIs and tasks, ensuring proper separation of concerns and comprehensive coverage without duplication or circular dependencies.

## 6.1.1 General Testing Principles

1. **Risk-Based Approach**: Testing effort must be proportional to the complexity, change surface, and failure risk of the task or PBI.
2. **Clarity and Maintainability**: Tests should be clear, concise, and easy to maintain. Avoid overly complex test logic or brittle scenarios unless the risk profile justifies them.
3. **Automation**: Tests should be automated wherever feasible so verification is repeatable and reliable across sessions and environments.

## 6.2 Three-Tier Testing Architecture

> Rationale: Establishes clear boundaries between unit testing, integration testing, and end-to-end validation to ensure appropriate test coverage at each level.

### 6.2.1 Tier 1: Individual Task Testing (Tasks 1-N-1)
- **Purpose**: Validate specific component functionality in isolation
- **Scope**: Each implementation task creates and runs its own focused tests
- **Responsibility**: Individual task implementer
- **Test Files**: Task creates its own test files as part of implementation

### 6.2.2 Tier 2: Integration Testing (Task N-1)
- **Purpose**: Validate interactions and interfaces between components from previous tasks
- **Scope**: Cross-component integration, not re-testing individual components
- **Responsibility**: Integration task implementer
- **Test Files**: Creates new integration test files that use components built in previous tasks

### 6.2.3 Tier 3: End-to-End CoS Validation (Task N)
- **Purpose**: Validate the complete system against all PBI acceptance criteria.
- **Scope**: Complete system scenarios spanning the entire PBI scope.
- **Responsibility**: E2E validation task implementer.
- **Test Files**: May orchestrate all tests from previous tasks plus system-level scenarios, operational runbooks, deployment verification steps, and runtime validation steps as required by the PBI.
- **Important Distinction**: Task N is a validation task, not inherently a promotion task. If the acceptance criteria require promoted, deployed, migrated, or runtime-integrated state, Task N may depend on a separate promotion step before validation can be completed.

### 6.2.4 Promotion-Dependent Validation

> Rationale: Some acceptance criteria can only be honestly validated after code has been promoted and the relevant runtime environment has been updated.

- If Task N depends on promoted or deployed state, the workflow should be:
  1. implementation tasks complete
  2. promotion handoff is created
  3. promotion is executed in a dedicated promotion session
  4. validation handoff is created
  5. Task N validation is executed in a dedicated validation session
- In this case, the promotion step is a delivery control step, not itself a numbered implementation task, unless the User explicitly creates a task for promotion.
- Task N is not complete merely because CI is green or code is merged.
- Task N is complete only when the required acceptance criteria have been validated in the target state required by the task.

## 6.3 Task-Level Testing Requirements

> Rationale: Ensures each task creates comprehensive tests for its own implementation without creating dependencies on future tasks.

### 6.3.1 Implementation Task Testing (Tasks 1 through N-2)
Each implementation task **must** include:

1. **Test Plan Section**: Detailed testing approach within the task document
2. **Tests to Run Section**: Specific commands that validate task completion
3. **Test File Creation**: Create all test files needed to validate the task
4. **Planned Test Files Section**: List the test files this task expects to create or update. Use checkboxes (`- [ ]`) and mark them complete (`- [x]`) as the files are created/updated during implementation.

#### Required Content for Implementation Tasks

    ## Test Plan

    ### Testing Objective
    [What this task's tests aim to verify]

    ### Tests to Run
    # Specific commands that validate this task's implementation
    pytest tests/unit/test_[component].py -v
    [other specific test commands]

    ### Success Criteria
    - [Concrete, testable criteria for task completion]

    ## Planned Test Files
    - [ ] `tests/unit/test_[component].py` (create)

### 6.3.2 Integration Testing Task (Task N-1)
The integration testing task focuses on **component interactions**, not individual component functionality.

#### Integration Task Requirements
- **Cross-Component Testing**: Test interfaces between components from previous tasks
- **No Unit Test Duplication**: Do not re-test individual component functionality
- **Interface Validation**: Focus on how components work together
- **System Integration**: Test complete flows across multiple components

#### Integration Task Structure

    ## Implementation Plan

    1. **[Component A] + [Component B] Integration**:
       - Test that Component A can interact with Component B
       - Test data flow between components
       - Test error handling across component boundaries

    2. **[Multi-Component] Integration**:
       - Test complete workflows spanning multiple components
       - Test system behavior under integration scenarios

    ## Tests to Run
    # Run integration tests that THIS task creates
    pytest tests/integration/test_[specific_integration].py -v

    # Verify all unit tests from previous tasks still pass
    pytest tests/unit/ -v

    # Run complete integration suite
    pytest tests/integration/ -v

### 6.3.3 End-to-End CoS Testing Task (Task N)
The E2E task validates **complete system functionality** against all PBI acceptance criteria.

#### E2E Task Requirements
- **Acceptance Criteria Validation**: Every acceptance criteria from PBI PRD must be tested
- **Clean Environment Testing**: Test complete system deployment from scratch
- **System-Level Scenarios**: Test workflows that span the entire PBI scope
- **Comprehensive Coverage**: Ensure all PBI functionality is validated

## 6.4 Test File Organization and Dependencies

> Rationale: Establishes clear file organization that supports the three-tier testing architecture without circular dependencies.

### 6.4.1 Test File Creation Responsibility
- **Implementation Tasks**: Create `tests/unit/test_[component].py` files
- **Integration Task**: Create `tests/integration/test_[integration_scenario].py` files
- **E2E Task**: Create `tests/e2e/test_[pbi_id]_cos.py` files

### 6.4.2 Test Execution Order
1. **During Implementation Tasks**: Run unit tests for the specific component being implemented
2. **During Integration Task**: Run integration tests + verify all unit tests still pass
3. **During E2E Task**: Run complete test suite (unit + integration + e2e)

### 6.4.3 Dependency Management
- **No Forward Dependencies**: Tasks cannot reference test files from future tasks
- **Cumulative Testing**: Later tasks can reference and run tests from earlier tasks
- **Self-Contained**: Each task creates all test files it needs for validation

## 6.5 Test Documentation Standards

> Rationale: Ensures consistent, actionable test documentation that enables reliable task completion validation.

### 6.5.1 Required Test Documentation Elements
Every task with testing requirements must include:

1. **Test Objective**: Clear statement of what the tests validate
2. **Tests to Run**: Executable commands that validate task completion
3. **Success Criteria**: Concrete, measurable criteria for test success
4. **Test Files**: List of test files created or updated by the task

### 6.5.2 Test Command Specificity
Test commands must be:
- **Executable**: Can be copy-pasted and run directly
- **Specific**: Target the exact functionality implemented by the task
- **Validating**: Directly validate the task's verification criteria
- **Self-Contained**: Do not depend on test files not yet created

### 6.5.3 Example Test Documentation

    ## Test Plan

    ### Test Objective
    Verify database connection management provides reliable async connectivity with proper error handling.

    ### Tests to Run
    # Test database connection establishment
    pytest tests/unit/test_database_connection.py::test_async_connection -v

    # Test connection pool behavior
    pytest tests/unit/test_database_pool.py -v

    # Manual connection verification
    python -c "from src.utils.database import get_connection; print('Connection successful')"

    ### Success Criteria
    - Async database connections establish within 5 seconds
    - Connection pool handles 10 concurrent connections
    - Retry logic recovers from transient connection failures

## 6.6 Quality Assurance for Test Documentation

> Rationale: Ensures test documentation is accurate, complete, and enables reliable task completion validation.

### 6.6.1 Test Documentation Review Criteria
- **Completeness**: All required elements present and properly filled out
- **Executability**: All test commands can be executed as documented
- **Coverage**: Tests adequately cover the task's verification criteria
- **Independence**: Task tests do not depend on files from future tasks

### 6.6.2 Common Test Documentation Anti-Patterns
- **Forward Dependencies**: Referencing test files not yet created
- **Generic Commands**: Using vague commands like "run tests" without specificity
- **Missing Files**: Listing test commands without creating the test files
- **Circular References**: Integration tasks creating unit tests that should be in implementation tasks

### 6.6.3 Test Documentation Validation
Before marking a task as complete:
- [ ] All test commands execute successfully
- [ ] All referenced test files exist and are created by the task
- [ ] Success criteria are met by the test execution
- [ ] No dependencies on files from future tasks
