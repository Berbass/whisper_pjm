---
id: T-018
title: "Fix Background Notification Actions"
status: todo
assigned_to: unassigned
dependencies: []
completion_percentage: 0%
last_updated: 2026-06-03
---

# Objective
Fix the issue where tapping the "Acknowledge" action on a system notification does nothing.

# Acceptance Criteria
- [ ] The "Acknowledge" action button on Android/iOS notifications correctly updates the reminder's status in the database.
- [ ] The notification is dismissed upon acknowledgement.
- [ ] Background execution is properly wired up in `flutter_local_notifications` so the action runs without needing to bring the app to the foreground.

# Agent Execution Log
