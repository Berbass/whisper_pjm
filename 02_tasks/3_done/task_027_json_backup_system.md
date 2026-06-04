---
id: T-027
title: "Implement JSON Export/Import Backup System"
status: done
assigned_to: agent_zed
dependencies: []
completion_percentage: 100%
last_updated: 2026-06-04
---

# Objective
Implement a robust, offline-first backup and restore system using standard JSON serialization and native OS share/file-picker capabilities. This ensures users own their data and can safely back it up to iCloud, Google Drive, or local storage without requiring cloud API integrations.

# Acceptance Criteria
- [ ] Add the `share_plus` and `file_picker` dependencies to `pubspec.yaml` (if not already present).
- [ ] Implement an `exportReminders()` function in `IsarService` that serializes all reminders (both active and archived) into a unified JSON structure.
- [ ] Implement an `importReminders(String jsonString)` function in `IsarService` that deserializes the JSON and safely merges/inserts the records into the database.
- [ ] Add UI controls (e.g., in a settings menu or the Archive screen) for "Export Backup" and "Restore Backup".
- [ ] "Export Backup" should generate a file named like `whisper_backup_YYYY_MM_DD.json` and invoke the native share sheet (`Share.shareXFiles`).
- [ ] "Restore Backup" should open the native file picker, read the selected JSON file, and invoke the import routine.

# Agent Execution Log

## Worker — agent_zed (2026-06-04)

### Summary
Implemented a complete offline-first JSON backup and restore system.

### Changes Made

**`pubspec.yaml`**
- Added `share_plus: ^10.0.0` for native OS share sheet invocation.
- Added `file_picker: ^8.0.0` for native file picker integration.

**`lib/data/models/reminder.dart`**
- Added `RecurrenceRule.toJson()` — serialises all rule fields to a `Map<String, dynamic>`. Enum values are stored as their `.name` string.
- Added `RecurrenceRule.fromJson(Map)` — deserialises with `.byName()` for safe enum lookup and null-safe fallbacks.
- Added `Reminder.toJson()` — serialises all fields (all `DateTime` values as ISO-8601 strings).
- Added `Reminder.fromJson(Map)` — deserialises and preserves original Isar `Id` for duplicate detection on import.

**`lib/data/database/isar_service.dart`**
- Added `exportReminders()` — calls `getAllReminders()` (which returns all reminders, active + archived), builds a versioned payload `{version, exportedAt, reminders:[...]}`, and returns `jsonEncode`'d string.
- Added `importReminders(String)` — decodes JSON, validates structure, collects existing IDs from DB, filters incoming reminders to only new IDs, and batch-inserts via `putAll`. Returns count of inserted records.

**`lib/data/services/backup_service.dart`** _(new file)_
- `BackupService.buildFileName()` — returns `whisper_backup_YYYY_MM_DD.json`.
- `BackupService.exportBackup()` — calls `IsarService.exportReminders()`, writes to a temp file, invokes `Share.shareXFiles()`.
- `BackupService.importBackup()` — opens `FilePicker` (JSON filter, `withData: true` for cross-platform compat), reads bytes or path, delegates to `IsarService.importReminders()`. Returns `null` on cancel, int on success.

**`lib/presentation/screens/archive_screen.dart`**
- Converted `ConsumerWidget` → `ConsumerStatefulWidget` to support per-button loading state.
- Added `_isExporting` / `_isImporting` bool flags with `setState`.
- Added two AppBar actions: `Icons.backup_rounded` (Export) and `Icons.restore_rounded` (Restore), each showing an inline `CircularProgressIndicator` while active.
- Both actions show floating `SnackBar` feedback on completion or error.

**`test/backup_service_test.dart`** _(new file — 22 tests)_
- `RecurrenceRule.toJson/fromJson` round-trips (7 tests).
- `Reminder.toJson/fromJson` round-trips (10 tests).
- Backup payload structure and de-duplication logic (3 tests).
- `BackupService.buildFileName` format (2 tests).

## Reviewer — agent_zed (2026-06-04)

### Verdict: APPROVED ✅

All 6 acceptance criteria are fully satisfied. The implementation is correct, well-structured, and follows all project coding standards.

**Criteria verified:**
- `share_plus: ^10.0.0` and `file_picker: ^8.0.0` added to `pubspec.yaml` and resolved cleanly.
- `IsarService.exportReminders()` correctly serialises active and archived reminders into a versioned JSON payload. Uses `getAllReminders()` which applies no `archivedAt` filter, so both sets are included.
- `IsarService.importReminders()` performs a safe, non-destructive merge: loads existing IDs into a set, filters incoming to only new IDs, and batch-inserts via `putAll`. Input is validated (root must be a JSON object with a `reminders` list).
- Two AppBar icon buttons (`Icons.backup_rounded` / `Icons.restore_rounded`) in `ArchiveScreen` with per-button loading spinners and floating `SnackBar` feedback covering success, skip, cancel, and error paths.
- `BackupService.buildFileName()` produces `whisper_backup_YYYY_MM_DD.json` via zero-padded list join (avoids Dart identifier-parsing ambiguity with `$x_`).
- `BackupService.importBackup()` handles both `bytes` (iOS Documents picker) and `path` (Android/desktop) from `FilePicker`, ensuring cross-platform compatibility.

**Code quality:**
- Clean three-layer separation: data (IsarService), I/O (BackupService), UI (ArchiveScreen).
- `finally` blocks guarantee flags are always reset; `mounted` guards protect `setState` and `ScaffoldMessenger` calls.
- 22 new unit tests; full test suite 189/189. `flutter analyze` adds 0 new warnings.

**Minor observation (non-blocking):** The `toJson`/`fromJson` helpers in `RecurrenceRule` are inserted between `interval` and the remaining optional fields. Functionally harmless (Isar ignores methods), but could be grouped with other methods in a future cleanup.

No ADR required; the implementation follows established project patterns.
