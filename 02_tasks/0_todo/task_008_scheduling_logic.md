---
id: T-008
title: "Core Scheduling Logic (Interval-Based)"
status: todo
assigned_to: unassigned
dependencies: [T-006, T-007]
completion_percentage: 0%
last_updated: 2026-05-28
---

# Objective
Write the core scheduling engine that reads a `RecurrenceRule` from a `Reminder` object, maps it to future calendar dates, and registers up to 64 local notifications (OS limit).

# Acceptance Criteria
- [ ] Create a dedicated scheduling utility or service (`lib/core/utils/scheduler.dart`).
- [ ] Implement the algorithm to calculate future dates for Interval-Based recurrence (Daily, Weekly, Monthly, Yearly).
- [ ] Write the function to register these calculated future dates as scheduled notifications via `flutter_local_notifications`.
- [ ] Ensure the algorithm caps at scheduling 64 notifications per reminder to respect OS limitations.
- [ ] Implement logic to cancel previously scheduled notifications when a reminder is updated or deleted.
