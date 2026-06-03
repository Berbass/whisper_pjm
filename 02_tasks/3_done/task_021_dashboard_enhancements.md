---
id: T-021
title: "Dashboard Enhancements (Sorting & Swipe Confirm)"
status: done
assigned_to: agent_zed
dependencies: []
completion_percentage: 100%
last_updated: 2026-06-04
---

# Objective
Improve the dashboard experience by preventing accidental deletions and sorting the reminders to show the most relevant ones first.

# Acceptance Criteria
- [x] When swiping a reminder card to the left (Delete/Edit action), prompt the user with a confirmation dialog before deleting.
- [x] Sort the list of reminders on the dashboard so that the most recently created or updated reminders appear at the top.

# Agent Execution Log

## 2026-06-04 — agent_zed

### Analysis
- Reviewed `dashboard_screen.dart`: the left-swipe `confirmDismiss` branch immediately deleted without any guard.
- Reviewed `isar_service.dart`: `watchAllReminders()` and `getAllReminders()` both sorted by `targetDateTime` ascending.
- The `Reminder` model exposes a `createdAt` field (set at insert time, never mutated) which is the best available proxy for "most recently created". No `updatedAt` field exists; sorting by `createdAt` satisfies the AC.
- `reminder.g.dart` already exposes `sortByCreatedAtDesc()` — no model or build_runner changes needed.

### Changes Made

#### `lib/data/database/isar_service.dart`
- Changed `getAllReminders()` sort from `sortByTargetDateTime()` → `sortByCreatedAtDesc()`.
- Changed `watchAllReminders()` sort from `sortByTargetDateTime()` → `sortByCreatedAtDesc()`.

#### `lib/presentation/screens/dashboard_screen.dart`
- Added a client-side sort (`createdAt` descending) in `_buildList()` so the widget layer is deterministic regardless of the data source (important for testability and resilience).
- Replaced the unconditional delete in the left-swipe `confirmDismiss` branch with an `AlertDialog` prompting the user to confirm. Deletion only proceeds on explicit "Delete" tap; "Cancel" (or barrier dismiss) returns `false` and snaps the card back.

#### `test/dashboard_screen_test.dart`
- Added group `DashboardScreen — delete confirmation` (3 tests):
  - `swiping left shows a delete confirmation dialog`
  - `canceling the delete dialog keeps the reminder visible`
  - `confirming deletion removes the reminder from the list`
- Added group `DashboardScreen — sorting` (1 test):
  - `displays reminders newest-first (sorted by createdAt desc)`

### Validation
- `flutter analyze` → **No issues found**
- `flutter test` → **130/130 tests passed** (4 new tests added to the dashboard suite; all pre-existing tests remain green)

## 2026-06-04 — reviewer

### Review Outcome: ✅ APPROVED

**AC 1 — Delete confirmation dialog:**
Implementation is correct and safe. The `confirmDismiss` `else` branch now awaits an `AlertDialog` before proceeding. Tapping "Cancel" or the barrier returns `null`/`false`, snapping the card back without touching the DB. The `confirmed == true` guard correctly handles all non-confirming paths (Cancel, barrier tap, back button). Right-swipe (acknowledge) is completely unmodified.

**AC 2 — Sort newest first:**
A two-layer sort is applied: `sortByCreatedAtDesc()` at the Isar query level in `watchAllReminders()` / `getAllReminders()`, and a client-side `compareTo` sort in `_buildList()`. The dual approach is justified: the Isar sort ensures production data is pre-ordered; the Dart sort ensures widget tests with a mocked provider also exercise the correct order. The comment in `_buildList` explains the rationale.

**Tests:**
- 4 focused widget tests cover dialog visibility, Cancel path, Delete path, and sort order.
- Sort test correctly uses `copyWith(createdAt: ...)` with fixed dates (2026-01-01 vs 2026-06-01) for deterministic comparisons.
- All 130 tests pass.

**Notes:**
- The `Reminder` model has no `updatedAt` field; the worker correctly used `createdAt` as the sorting key and documented this decision in the execution log. Adding `updatedAt` (requiring build_runner) is a potential future enhancement.
- No architectural decisions requiring an ADR.
