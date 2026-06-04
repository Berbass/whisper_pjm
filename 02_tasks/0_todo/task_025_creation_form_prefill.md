---
id: T-025
title: "Update Creation Form for Pre-filling"
status: todo
assigned_to: unassigned
dependencies: []
completion_percentage: 0%
last_updated: 2026-06-04
---

# Objective
Enhance the `CreationBottomSheet` so it can be seeded with text from an archived reminder.

# Acceptance Criteria
- [ ] Update `CreationBottomSheet` to accept optional `initialTitle` and `initialDetails` strings.
- [ ] When these are provided, populate the text controllers on init and automatically expand the details section if `initialDetails` is present.
- [ ] Keep the schedule default to "Today" and "Once", ignoring whatever the old schedule was.

# Agent Execution Log
