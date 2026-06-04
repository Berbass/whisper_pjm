---
id: T-027
title: "Implement JSON Export/Import Backup System"
status: todo
assigned_to: unassigned
dependencies: []
completion_percentage: 0%
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
