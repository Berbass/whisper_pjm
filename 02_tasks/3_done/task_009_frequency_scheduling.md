---
id: T-009
title: "Implement Frequency-Based Scheduling (Phase 3)"
status: done
completion_percentage: 100%
last_updated: 2026-06-01
completed_at: 2026-06-01
assigned_to: agent_zed
dependencies: [T-008]
---

# Objective
Implement the logic to schedule notifications based on specific frequencies (e.g., specific days of the week, monthly on a certain date). This replaces the current stub for `RecurrenceType.frequency` in the `Scheduler` service.

# Acceptance Criteria
- [x] Implement `RecurrenceType.frequency` logic in the `Scheduler` service.
- [x] Add support for scheduling on specific days of the week.
- [x] Add support for scheduling on specific days of the month.
- [x] Ensure previously scheduled notifications with this pattern are updated/re-scheduled if modified.
- [x] Write unit/integration tests validating the correct next-fire dates for frequency-based reminders.

# Agent Execution Log

**2026-06-01 — agent_zed**

### Analysis
- Reviewed `RecurrenceRule` model: `daysOfWeek`, `dayOfMonth`, `occurrencesCount`,
  and `frequencyUnit` fields were already in place from T-006.
- The `Scheduler` already cancelled then re-scheduled on every `scheduleReminder`
  call, satisfying the update/re-schedule acceptance criterion structurally —
  only the date-generation logic was missing.

### Implementation (`lib/core/utils/scheduler.dart`)
- Added `_scheduleFrequency(Reminder)` — parallel to `_scheduleInterval`;
  guards against null `frequencyUnit`.
- Added `_calculateFrequencyDates(DateTime, RecurrenceRule)` — pure function;
  four sub-modes keyed on `frequencyUnit`:
  - **week**: day-by-day scan collecting weekdays present in `daysOfWeek`.
  - **month**: month-by-month scan; clamps `dayOfMonth` to each month’s last day.
  - **day**: evenly-spaced intervals of `(24×60) ÷ occurrencesCount` minutes.
  - **year**: year-by-year scan preserving `start.month`/`start.day`.
- All returned dates are strictly after `DateTime.now()`.
- Exposed `@visibleForTesting calculateFrequencyDates` shim for unit tests.

### Tests (`test/scheduler_frequency_test.dart`)
20 unit tests across four groups:
- `weekly — daysOfWeek` (6): weekday filtering, future-only, time-of-day
  preservation, ascending order, empty/null guards.
- `monthly — dayOfMonth` (5): day matching, future-only, day-31 clamping,
  1-month gap, null guard.
- `daily — occurrencesCount` (5): count=4/2/0 spacing, future-only, maxNotifications count.
- `yearly` (4): month+day preservation, 1-year gap, future-only,
  leap-year Feb-29 clamping.

### Validation
- `flutter analyze` — no issues.
- `flutter test` — 21/21 tests passed (20 new + 1 pre-existing widget test).

**2026-06-01 — review pass**

- AC-1 ✅ `RecurrenceType.frequency` switch case wired to `_scheduleFrequency`.
- AC-2 ✅ Weekly: day-by-day scan with `daysOfWeek`; `DateTime.weekday` convention (1=Mon, 7=Sun) matches model.
- AC-3 ✅ Monthly: month-by-month scan; `dayOfMonth` clamped via `DateTime(y, m+1, 0).day` — short months handled correctly.
- AC-4 ✅ `scheduleReminder` always calls `cancelReminder` first — re-schedule on edit covered for all `RecurrenceType` branches.
- AC-5 ✅ 20 unit tests: weekday targeting, monthly gap, day-31/Feb-29 clamping, spacing precision, future-only guarantee, null/empty guards.
- `flutter analyze` — no issues.
- `flutter test` — 21/21 passed.
- Code note: `payload: reminder.id.toString()` consistently set across all `zonedSchedule` call sites; ready for T-010 deep-linking.

**Status: APPROVED → done.**
