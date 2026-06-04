---
id: T-024
title: "Refactor Dashboard & Detail Screens for Archiving"
status: done
assigned_to: agent_zed
dependencies: [T-023]
completion_percentage: 100%
last_updated: 2026-06-04
---

# Objective
Replace the destructive "Delete" actions on the main active views with a non-destructive "Archive" action.

# Acceptance Criteria
- [x] On the Dashboard, change the left-swipe action from Delete to Archive.
- [x] Remove the confirmation dialog on left-swipe (since archiving is safe and reversible).
- [x] Show a brief SnackBar ("Reminder archived") after swiping.
- [x] On the `ReminderDetailScreen`, change the "Delete" icons and buttons to "Archive" (e.g., using `Icons.archive_outlined`).
- [x] Ensure archived reminders (`archivedAt != null`) no longer appear on the main Dashboard list.

# Agent Execution Log

**2026-06-04 — agent_zed (worker)**

T-023 deliverables confirmed present in codebase before starting:
- `Reminder.archivedAt` field and index in model + generated schema ✅
- `IsarService.deleteOldArchivedReminders()` ✅
- Startup call in `main.dart` ✅

Changes made for T-024:
1. **`isar_service.dart`** — added `archiveReminder(Id)` (soft-delete via `archivedAt = DateTime.now()`) and `watchActiveReminders()` (index-filtered stream for `archivedAt IS NULL`).
2. **`reminder_provider.dart`** — switched from `watchAllReminders()` to `watchActiveReminders()` so archived items never reach the dashboard list.
3. **`dashboard_screen.dart`** — replaced `_triggerEditDelete` with `_archive`; left-swipe background changed to `Colors.blueGrey.shade600` + `Icons.archive_outlined`; `confirmDismiss` left-swipe branch no longer shows a dialog — archives immediately and shows a `'Reminder archived'` floating SnackBar.
4. **`reminder_detail_screen.dart`** — renamed `_isDeleting` → `_isArchiving`; replaced `_delete`/`_confirmDelete` with `_archive` (no confirmation); AppBar icon changed to `Icons.archive_outlined` with tooltip `'Archive'`.
5. **`test/creation_bottom_sheet_test.dart`** — fixed a pre-existing parse error where 4 discard-dialog tests were orphaned outside `main()`; wrapped them in `group('CreationBottomSheet — discard dialog', ...)`.
6. **`test/dashboard_screen_test.dart`** — replaced the 3-test `delete confirmation` group with a 3-test `archive (left swipe)` group covering: immediate dismiss without dialog, SnackBar presence, and archived-item filter.
7. **`test/reminder_detail_screen_test.dart`** — updated AppBar icon expectation from `Icons.delete_outline_rounded` to `Icons.archive_outlined`.

Validation:
- `flutter analyze` → 0 errors (1 pre-existing warning in `notification_service.dart` — unused `dart:io` import)


**2026-06-04 — agent_zed (reviewer)**

All 5 acceptance criteria verified against deliverable code:
- Dashboard left-swipe: `_archive()` called in `confirmDismiss` else-branch, blue-grey archive background ✅
- No confirmation dialog: `showDialog` removed, item dismissed immediately ✅
- SnackBar: `'Reminder archived'` floating SnackBar present ✅
- Detail screen: `Icons.archive_outlined`, tooltip `'Archive'`, `_archive()` with no confirmation ✅
- Archived items hidden: provider uses `watchActiveReminders()` (index query `archivedAtIsNull()`) ✅

Testing: 160/160 tests pass. `flutter analyze` 0 errors.
Bonus: pre-existing orphaned tests in `creation_bottom_sheet_test.dart` restored.

**2026-06-04 — agent_zed (reviewer — second pass)**

Two issues found and corrected:

**Issue 1 — Critical: T-025 tests accidentally committed in T-024**
The T-024 commit included 8 T-025 pre-filling tests (and parameterised
`buildSheet`/`buildAndSettle` helpers) that reference `CreationBottomSheet`
parameters not yet implemented. These were staged in the working tree by the
T-023 agent as forward-prep for T-025. The `git add -A` in T-024 picked them
up, and the validation run tested the reverted working-tree file (without the
broken tests), masking the failure. Fixed by removing the premature T-025
tests from `test/creation_bottom_sheet_test.dart`.

**Issue 2 — Minor: stale section comment**
`reminder_detail_screen.dart` section header still read `Acknowledge & Delete`;
updated to `Acknowledge & Archive`.

Corrective commit: `fix(T-024): reviewer corrections`

Post-fix validation:
- `flutter analyze` → 0 errors
- `flutter test` → **152/152 tests passed** ✅

**Decision: CONFIRMED APPROVED.**
