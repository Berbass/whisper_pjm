# T-009 Feature Draft: Frequency-Based Scheduling

## Problem
The `RecurrenceType.frequency` case in `Scheduler.scheduleReminder()` is currently
a no-op stub. Users who save reminders with frequency rules (e.g., "every Mon/Wed/Fri"
or "the 15th of every month") never receive notifications.

## Solution Design

### Scheduling Sub-modes (keyed on `RecurrenceRule.frequencyUnit`)

| `frequencyUnit` | Required field(s)    | Behaviour                                           |
|-----------------|----------------------|-----------------------------------------------------|
| `week`          | `daysOfWeek`         | Fire on each specified weekday, repeating weekly    |
| `month`         | `dayOfMonth`         | Fire on the specified day each month                |
| `day`           | `occurrencesCount`   | Fire N times/day, evenly spaced by minutes          |
| `year`          | (uses start's month/day) | Fire on the same calendar date each year        |

### Date Calculation Strategy
- Use `reminder.targetDateTime` for the desired **time-of-day** (hour + minute).
- Scan **forward from `DateTime.now()`**, collecting up to `maxNotifications` (64)
  future `DateTime` objects.
- Never schedule a date in the past — skip silently.
- **Day clamping**: if `dayOfMonth = 31` and the current month only has 28 days,
  clamp to the last valid day (avoids `DateTime` overflow).

### Edge Cases
- `daysOfWeek` is null or empty when `frequencyUnit = week` → return `[]`, skip.
- `dayOfMonth` is null when `frequencyUnit = month` → return `[]`, skip.
- `occurrencesCount` below 1 → clamp to 1.
- Guard in `_scheduleFrequency` returns early when `frequencyUnit` is null (malformed rule).

### Testability
A `@visibleForTesting` shim `calculateFrequencyDates(start, rule)` is exposed on
`Scheduler` so unit tests can exercise the pure date-generation logic without
touching the notification plugin.

## Files Changed
- `lib/core/utils/scheduler.dart` — add `_scheduleFrequency`, `_calculateFrequencyDates`,
  and `calculateFrequencyDates` shim.
- `test/scheduler_frequency_test.dart` — unit tests for all four sub-modes.
