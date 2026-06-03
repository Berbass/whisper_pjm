---
id: T-021
title: "Dashboard Enhancements (Sorting & Swipe Confirm)"
status: in_review
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
