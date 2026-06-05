---
id: T-028
title: "Add Interactive Tooltip for Repeat Schedules"
status: todo
assigned_to: unassigned
dependencies: []
completion_percentage: 0%
last_updated: 2026-06-04
---

# Objective
Provide users with a quick, human-readable breakdown of complex repetition schedules. When a user taps the repeat label on a non-daily recurring reminder, an ephemeral tooltip should display the exact pattern.

# Acceptance Criteria
- [ ] Implement a formatter that translates a `RecurrenceRule` into a human-readable string (e.g., "Every week, Mon. and Thu.", "Every month, the 15th").
- [ ] On the UI (Reminder Cards / Detail Screen), wrap the Repeat indicator in a `Tooltip` (or custom overlay) configured to trigger on tap (`TooltipTriggerMode.tap`).
- [ ] The tooltip should only be active for reminders that repeat, explicitly excluding "Daily" (as daily is already self-explanatory).
- [ ] The hovering text styling should match the app's minimalist aesthetic (e.g., soft shadows, calm colors).

# Agent Execution Log
