---
id: T-023
title: "Database Updates for Archiving & Startup Cleanup"
status: todo
assigned_to: unassigned
dependencies: []
completion_percentage: 0%
last_updated: 2026-06-04
---

# Objective
Update the `Reminder` data model to support archiving and implement a silent cleanup routine that purges old archived items when the app starts.

# Acceptance Criteria
- [ ] Add a `DateTime? archivedAt` field to the `Reminder` Isar model.
- [ ] Re-run `build_runner` to regenerate the Isar schema.
- [ ] Add a method in `IsarService` to delete any reminders where `archivedAt` is older than 90 days.
- [ ] Call this cleanup method silently during the app's initialization phase (e.g., in `main.dart` or immediately after opening the Isar database).

# Agent Execution Log
