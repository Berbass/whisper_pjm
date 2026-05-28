---
id: T-006
title: "Isar Database Setup & Models"
status: todo
assigned_to: unassigned
dependencies: [T-001]
completion_percentage: 0%
last_updated: 2026-05-28
---

# Objective
Set up the local Isar database layer. Define the core data models (`Reminder` and `RecurrenceRule`) and generate the type-safe data access logic.

# Acceptance Criteria
- [ ] Create `lib/data/models/reminder.dart` following the schema defined in `whisper_specifications.md`.
- [ ] Implement the `@collection` class for `Reminder` and the `@embedded` class for `RecurrenceRule`.
- [ ] Create a database service (`lib/data/database/isar_service.dart`) with an initialization routine using `path_provider` to locate the application documents directory.
- [ ] Run `flutter pub run build_runner build` to generate `reminder.g.dart`.
- [ ] Implement basic CRUD (Create, Read, Update, Delete) methods in the database service.
