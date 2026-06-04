---
id: T-023
title: "Database Updates for Archiving & Startup Cleanup"
status: done
assigned_to: agent_zed
dependencies: []
completion_percentage: 100%
last_updated: 2026-06-04
---

# Objective
Update the `Reminder` data model to support archiving and implement a silent cleanup routine that purges old archived items when the app starts.

# Acceptance Criteria
- [x] Add a `DateTime? archivedAt` field to the `Reminder` Isar model.
- [x] Re-run `build_runner` to regenerate the Isar schema.
- [x] Add a method in `IsarService` to delete any reminders where `archivedAt` is older than 90 days.
- [x] Call this cleanup method silently during the app's initialization phase (e.g., in `main.dart` or immediately after opening the Isar database).

# Agent Execution Log

## 2026-06-04 — agent_zed

### Changes Made

**`lib/data/models/reminder.dart`**
- Added `DateTime? archivedAt` field annotated with `@Index()` for efficient querying in the cleanup routine.
- Extended `Reminder.create()` factory with an optional `archivedAt` parameter (defaults to `null`).
- Extended `Reminder.copyWith()` with `archivedAt` parameter, following the existing `??` preservation pattern.

**`lib/data/models/reminder.g.dart`**
- Regenerated via `flutter pub run build_runner build --delete-conflicting-outputs`. Build succeeded with 26 outputs.

**`lib/data/database/isar_service.dart`**
- Added `static Future<int> deleteOldArchivedReminders()` method. Calculates a 90-day cutoff from `DateTime.now()`, then deletes all reminders where `archivedAt` is not null and is before the cutoff, within a single write transaction. Returns the count of deleted records.

**`lib/main.dart`**
- Called `await IsarService.deleteOldArchivedReminders()` immediately after `IsarService.open()` so the purge runs silently on every cold start before the UI builds.

**`test/reminder_model_test.dart`**
- Added a `Reminder.archivedAt` group with 6 unit tests covering:
  - Default value is `null` via `create()`
  - Can be set via `create(archivedAt: ...)`
  - `copyWith` sets `archivedAt` correctly
  - `copyWith` does not mutate the original
  - `copyWith` preserves `archivedAt` when not overridden
  - `copyWith` on a non-archived reminder keeps `archivedAt` as `null`

### Validation
- `flutter pub run build_runner build --delete-conflicting-outputs` — succeeded (26 outputs) ✅
- `flutter analyze` — 0 errors; 1 pre-existing warning (`unused_import` in `notification_service.dart`, unrelated to this task) ✅
- `flutter test` — 152/152 tests passed (146 pre-existing + 6 new) ✅
