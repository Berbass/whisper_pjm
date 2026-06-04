---
id: T-026
title: "Implement Archive Screen & Swipe Actions"
status: todo
assigned_to: unassigned
dependencies: [T-023, T-024, T-025]
completion_percentage: 0%
last_updated: 2026-06-04
---

# Objective
Create a dedicated screen to view archived reminders, complete with its own specific swipe interactions.

# Acceptance Criteria
- [ ] Replace the 3-dots icon in the Dashboard AppBar with a history/archive icon that navigates to the new `ArchiveScreen`.
- [ ] Create `ArchiveScreen` that lists all reminders where `archivedAt != null`, sorted by `archivedAt` descending.
- [ ] Implement left-swipe: permanently deletes the reminder (showing the delete confirmation dialog first).
- [ ] Implement right-swipe: opens the `CreationBottomSheet`, passing the archived reminder's title and details to resurrect the thought as a new reminder.

# Agent Execution Log
