---
id: T-008
title: "Core Scheduling Logic (Interval-Based)"
status: done
assigned_to: agent_zed
dependencies: [T-006, T-007]
completion_percentage: 100%
last_updated: 2026-05-28
---

# Objective
Write the core scheduling engine that reads a `RecurrenceRule` from a `Reminder` object, maps it to future calendar dates, and registers up to 64 local notifications (OS limit).

# Acceptance Criteria
- [x] Create a dedicated scheduling utility or service (`lib/core/utils/scheduler.dart`).
- [x] Implement the algorithm to calculate future dates for Interval-Based recurrence (Daily, Weekly, Monthly, Yearly).
- [x] Write the function to register these calculated future dates as scheduled notifications via `flutter_local_notifications`.
- [x] Ensure the algorithm caps at scheduling 64 notifications per reminder to respect OS limitations.
- [x] Implement logic to cancel previously scheduled notifications when a reminder is updated or deleted.

# Agent Execution Log
- Created `scheduler.dart` with `scheduleReminder(Reminder)` and `cancelReminder(int id)` as the public API.
- `scheduleReminder` cancels existing registrations first, then dispatches to `_scheduleSingle` (RecurrenceType.none) or `_scheduleInterval` (RecurrenceType.interval). RecurrenceType.frequency is a Phase 3 stub.
- `_calculateFutureDates` skips past occurrences and generates up to `maxNotifications = 64` future dates.
- `_addInterval` supports all four RecurrenceUnit values with correct month/year arithmetic.
- `_notificationId` encodes reminder ID + index into a safe signed 32-bit int.
- Added `import 'package:flutter_local_notifications/flutter_local_notifications.dart'` after analyze revealed missing UILocalNotificationDateInterpretation / AndroidScheduleMode identifiers.
- `flutter analyze` — no issues ✅ · `flutter test` — 1/1 passed ✅
