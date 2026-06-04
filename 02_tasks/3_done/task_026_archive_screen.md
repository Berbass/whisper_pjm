---
id: T-026
title: "Implement Archive Screen & Swipe Actions"
status: done
assigned_to: agent
dependencies: [T-023, T-024, T-025]
completion_percentage: 100%
last_updated: 2026-06-04
---

# Objective
Create a dedicated screen to view archived reminders, complete with its own specific swipe interactions.

# Acceptance Criteria
- [x] Replace the 3-dots icon in the Dashboard AppBar with a history/archive icon that navigates to the new `ArchiveScreen`.
- [x] Create `ArchiveScreen` that lists all reminders where `archivedAt != null`, sorted by `archivedAt` descending.
- [x] Implement left-swipe: permanently deletes the reminder (showing the delete confirmation dialog first).
- [x] Implement right-swipe: opens the `CreationBottomSheet`, passing the archived reminder's title and details to resurrect the thought as a new reminder.

# Agent Execution Log

## 2026-06-04 — Worker

### Changes Made

**`lib/data/database/isar_service.dart`**
- Added `watchArchivedReminders()` static stream method: queries reminders where `archivedAt` is not null, sorted by `archivedAt` descending. Used by the new provider and `ArchiveScreen`.

**`lib/presentation/providers/reminder_provider.dart`**
- Added `archivedRemindersProvider` (`StreamProvider<List<Reminder>>`) backed by the new `watchArchivedReminders()` stream.

**`lib/presentation/navigation/app_router.dart`**
- Added `goToArchive()` static method that pushes `ArchiveScreen` onto the navigator stack. Route name: `/archive`.
- Added import for `archive_screen.dart`.

**`lib/presentation/screens/archive_screen.dart`** _(new file)_
- `ConsumerWidget` that watches `archivedRemindersProvider`.
- Renders a `ListView` of `ReminderCard` widgets wrapped in `Dismissible`.
- **Right-swipe** (`startToEnd`): calls `_resurrect()`, which opens `CreationBottomSheet` with `initialTitle` and `initialDetails` pre-filled; returns `false` so the tile is not dismissed.
- **Left-swipe** (`endToStart`): calls `_confirmDelete()` showing an `AlertDialog` with Cancel/Delete actions. On confirmation, calls `IsarService.deleteReminder()` inside a try-catch (mirrors Dashboard pattern); returns `true` to dismiss the tile.
- Empty state shows `Icons.inventory_2_outlined` with appropriate copy.
- No FAB (archive is read-only navigation).

**`lib/presentation/screens/dashboard_screen.dart`**
- Replaced `Icons.more_horiz_rounded` (tooltip: "Options", `onPressed: () {}`) with `Icons.history_rounded` (tooltip: "Archive", `onPressed: () => AppRouter.goToArchive()`).

**`test/archive_screen_test.dart`** _(new file)_
- 12 widget tests covering: empty state, AppBar title, list rendering, details text, left-swipe delete dialog, Cancel/Delete dialog actions, right-swipe resurrection (sheet opens), right-swipe tile persistence.

### Validation
- `flutter analyze`: 1 pre-existing warning (`unused_import` in `notification_service.dart`) — not introduced by this task.
- `flutter test`: **167/167 tests passed** (12 new, 155 pre-existing).

## 2026-06-04 — Reviewer

### Review Outcome: APPROVED ✅

**Acceptance Criteria**: All 4 criteria verified against the implementation.

**Code Quality**:
- Architecture is clean and additive. `watchArchivedReminders()` correctly queries the Isar index on `archivedAt` with `.archivedAtIsNotNull().sortByArchivedAtDesc()`.
- `ArchiveScreen` uses `ConsumerWidget` (correct choice — no local mutable state needed). All helper methods as instance methods on the widget follow established patterns.
- Error handling mirrors the Dashboard pattern: `try-catch` around `IsarService` calls with silent swallow.
- `Dismissible` keys use `'archive_${reminder.id}'` prefix to avoid key conflicts.
- Visual design is consistent: `AppConstants` layout constants, `theme.colorScheme` tokens, same `_dismissBg` helper shape.

**Tests**: 12 widget tests cover all states and swipe flows. Provider override pattern is consistent with existing tests. The "Delete dismisses tile" test comment correctly explains why it only checks dialog dismissal (Isar not available in test env, try-catch swallows).

**No ADR required** — no structural or architectural decisions; this is additive feature work following established patterns.
