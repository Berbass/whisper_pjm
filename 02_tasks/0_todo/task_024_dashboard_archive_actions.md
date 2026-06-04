---
id: T-024
title: "Refactor Dashboard & Detail Screens for Archiving"
status: todo
assigned_to: unassigned
dependencies: [T-023]
completion_percentage: 0%
last_updated: 2026-06-04
---

# Objective
Replace the destructive "Delete" actions on the main active views with a non-destructive "Archive" action.

# Acceptance Criteria
- [ ] On the Dashboard, change the left-swipe action from Delete to Archive.
- [ ] Remove the confirmation dialog on left-swipe (since archiving is safe and reversible).
- [ ] Show a brief SnackBar ("Reminder archived") after swiping.
- [ ] On the `ReminderDetailScreen`, change the "Delete" icons and buttons to "Archive" (e.g., using `Icons.archive_outlined`).
- [ ] Ensure archived reminders (`archivedAt != null`) no longer appear on the main Dashboard list.

# Agent Execution Log
