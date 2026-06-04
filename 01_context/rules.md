# Whisper Project Rules & Standards

## Roles & Responsibilities

### 1. Worker (Developer)
* **Objective:** Take up tasks to be completed.
* **Responsibilities:**
  - Execute tasks according to the requirements, technical documentation in `01_context/documentation/`, and global context in `01_context/`.
  - **Self-Verification:** Ensure code compiles, `flutter analyze` passes, and all tests succeed (`flutter test`) before declaring work complete. Adhere strictly to the "Test-Driven & Verified Code Changes" rule.
  - **State Management:** When picking up a task, update the task file's YAML frontmatter status to `in_progress`, move the task file to `1_in_progress/`, and reflect this state in `00_DASHBOARD.md`.
  - Append detailed execution comments (logs) on what has been accomplished directly in the task file.
  - **Blockers & Scope:** If a task is blocked or the scope is found to be too large, document this in the execution log and pause to consult the Project Manager.
  - Move the task file to `2_in_review/` and update its status inside the YAML frontmatter to `in_review` when complete, and reflect this state in `00_DASHBOARD.md`.
  - When a task is moved to `in_review`, commit all changes related to the deliverables.

### 2. Reviewer
* **Objective:** Verify and assure the quality of completed work.
* **Responsibilities:**
  - Review tasks currently marked as `in_review` by examining descriptions, developer logs, and final outputs in `04_deliverables/`.
  - Ensure the work aligns perfectly with tasks specifications and enforces project rules (e.g., verifying that adequate tests were actually written).
  - If any structural or important architectural decisions are made during the task, document them in an Architecture Decision Record (ADR) file inside `01_context/adr/`.
  - **If changes are needed:** Add actionable, specific feedback comments (referencing file paths/lines) to the task file. Update the YAML status to `todo`, move it back to `0_todo/`, and update `00_DASHBOARD.md` accordingly.
  - **If the review is successful:** Update the task status to `done`, move it to `3_done/`, and update the `00_DASHBOARD.md` sprint counters and task overview. Finally, commit and push all changes related to the approval, along with the pertinent state of the project repository and deliverable submodules, to the remote repository.

### 3. Project Manager
* **Objective:** Keep the project moving smoothly and strategically.
* **Responsibilities:**
  - Create tasks according to product and engineering specifications, ensuring they are atomic, well-scoped, and have clear Acceptance Criteria.
  - **Dashboard Ownership:** Actively maintain `00_DASHBOARD.md` as the single source of truth for sprint metrics, active queues, and overarching progress.
  - **Dependency Management:** Track and enforce task sequencing based on the `dependencies` array in the task YAML frontmatter to prevent blocked workflows.
  - Perform global reviews of the codebase and project structure to identify improvements or better alignments with the project's goal.
  - Flexible execution: The PM can act as a worker or a reviewer as needed.
  - Apply hotfixes or quick deliverables/documentation changes directly without formal tasks if appropriate.
  - Every action taken by the Project Manager must keep the entire project's context, goal, and stability in consideration.

> **Note on AI Agent Roles:** If an AI agent is ever unsure about which role to assume (Worker, Reviewer, or Project Manager) for a specific prompt or scenario, it **must ask the user** for clarification before proceeding.

## Coding Standards & Quality Constraints

### 1. Test-Driven & Verified Code Changes
* **Rule:** Every code change must be evaluated for the necessity of unit or integration test creation/updates.
* **Requirements:**
  - If a feature introduces new business logic, helper functions, or algorithms, corresponding unit tests **must** be created.
  - If a change modifies existing verified behavior, existing tests **must** be updated alongside it.
  - For UI components that contain dynamic states or multi-stage user flows (e.g., forms, bottom sheets), integration/widget tests should be written where feasible.
  - Always run `flutter test` and `flutter analyze` to ensure no regressions are introduced before completing a task.

### 2. Versioning & Git Hygiene
* **Rule:** Strict commit structure and staging rules must be adhered to during task lifecycles.
* **Requirements:**
  - Before any task is considered completed or moved to `in_review`, all related changes to the project and its deliverables must be fully staged and committed.
  - Every commit message **must** explicitly mention the related task's ID (e.g., `feat(T-009): implement specific weekday scheduling`, `fix(T-010): resolve dismissible crash`) along with clear, descriptive details of what was changed. Multiple lines of context are allowed.
  - Avoid mixing changes from multiple tasks into a single commit. Keep git history clean and task-focused.
