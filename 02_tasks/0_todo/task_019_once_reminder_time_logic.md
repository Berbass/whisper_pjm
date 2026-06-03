---
id: T-019
title: "Improve Time & Date Logic for 'Once' Reminders"
status: todo
assigned_to: unassigned
dependencies: []
completion_percentage: 0%
last_updated: 2026-06-03
---

# Objective
Fix the automatic date-shifting logic when creating a "Once" reminder so that it behaves intuitively when the default time is in the recent past.

# Acceptance Criteria
- [ ] When creating a "Once" reminder for today with no specific time selected, do not blindly push it to the next day.
- [ ] If the current time selected (or default) is in the past by less than 4 hours, alert the user with a confirmation dialog asking if they intend to schedule it for the next day.
- [ ] Apply the user's choice (either keep it for today/immediately, or shift to tomorrow).

# Agent Execution Log
