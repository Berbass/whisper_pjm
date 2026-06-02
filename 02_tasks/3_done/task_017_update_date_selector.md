---
id: T-017
title: "Update Date Selector Behavior for Repetitive Reminders"
status: done
assigned_to: agent_zed
dependencies: []
completion_percentage: 100%
last_updated: 2026-06-03
---

# Objective
Adjust the date selector's visibility and labeling to support defining a clear starting date for repetitive reminders. This reverts a previous change that hid the date selector for weekly repetitions and improves the UI copy.

# Acceptance Criteria
- [x] The date selector remains visible when the "Weekly" repeat option is selected.
- [x] When the repeat toggle is ON (repetitive mode), the default date label reads "From today" instead of "Today".
- [x] When the repeat toggle is ON (repetitive mode) and a specific date is chosen, the label is prefixed with "From " (e.g., "From Oct 12").
- [x] The selected date acts as the starting date for the repetitions in the underlying scheduling logic.

# Agent Execution Log

## 2026-06-03 — agent_zed

### Summary
Reverted the weekly date-picker hide, updated date labels to signal "starting from", and aligned the scheduler so no frequency-based notifications are fired before the chosen start date.

### Changes Made

**`lib/presentation/widgets/creation_bottom_sheet.dart`**
- Removed the `if (!(_scheduleMode == _ScheduleMode.repetitive && _repeatFrequency == _RepeatFrequency.weekly))` guard around `_buildDatePicker` — the date picker is now always rendered regardless of which repeat sub-frequency is selected.
- Updated the label expression inside `_buildDatePicker`:
  - Repetitive + no date selected → `'From today'`
  - Repetitive + date selected → `'From <formatted date>'` (e.g., `'From Oct 12'`)
  - Unique + no date selected → `'Today'` (unchanged)
  - Unique + date selected → `'<formatted date>'` (unchanged)

**`lib/core/utils/scheduler.dart`** — `_calculateFrequencyDates`
- **`RecurrenceUnit.week`:** Changed the scan cursor initialisation from a hardcoded `DateTime(now...)` to `start.isAfter(now) ? DateTime(start...) : DateTime(now...)`. This ensures that when a future start date is selected (e.g., "From Oct 15"), weekly notifications are not scheduled on earlier dates.
- **`RecurrenceUnit.month`:** Applied the same `start`-aware initialisation for `year`/`month` loop variables.
- **`RecurrenceUnit.year`:** Applied the same pattern for the initial `year` variable. In all three cases the existing `candidate.isAfter(now)` guard is kept as a safety net for same-day edge cases.

**`test/creation_bottom_sheet_test.dart`** — 4 new widget tests in a new group `'date picker label'`:
1. `'shows "Today" label in unique (Once) mode'` — baseline unchanged behaviour.
2. `'shows "From today" label when Repeat is selected'` — new behaviour; also asserts plain `'Today'` is absent.
3. `'shows "From today" label when Repeat + Weekly is selected (date picker must remain visible)'` — directly validates the un-hiding requirement.
4. `'reverts to "Today" label when switching back from Repeat to Once'` — round-trip validation.

**`test/scheduler_frequency_test.dart`** — 3 new unit tests:
1. Weekly group — `'does not schedule any date before a future start date'`: passes a 30-day-future start, confirms every returned date falls on or after that day.
2. Monthly group — `'does not schedule any occurrence before a future start month'`: passes a ~60-day-future start, confirms every returned date is in the start month or later.
3. Yearly group — `'does not schedule any occurrence before a future start year'`: passes a ~400-day-future start, confirms every returned year is ≥ start year.

### Acceptance Criteria — Verified
- [x] Date selector remains visible for Weekly — `if` guard removed; date picker always rendered.
- [x] Default label "From today" in repetitive mode — label expression updated.
- [x] Label prefixed "From" for explicit date in repetitive mode — label expression updated.
- [x] Selected date acts as starting date in scheduler — `start`-aware cursor initialisation added to `week`, `month`, `year` cases.

### Validation
- `flutter analyze` — no issues ✅
- `flutter test` — 107/107 tests passed ✅ (+7 new tests)

## 2026-06-03 — Reviewer

### Review Decision: APPROVED ✅

All 4 acceptance criteria verified against source code and test output.

**AC1 — Date selector visible for Weekly:** The `if (!(...weekly...))` guard has been removed from `build()`. `_buildDatePicker` is unconditionally rendered in the column. Widget test `'shows "From today" label when Repeat + Weekly is selected'` directly asserts the picker text is present. ✅

**AC2 — "From today" label in repetitive mode:** Label expression correctly evaluates to `'From today'` when `_scheduleMode == _ScheduleMode.repetitive` and `_selectedDate == null`. Widget tests `'shows "From today" label when Repeat is selected'` and its round-trip counterpart both pass. ✅

**AC3 — "From [date]" label for explicit date in repetitive mode:** Label expression uses string interpolation `'From formatMediumDate'` when `_selectedDate != null` and mode is repetitive. Code path is correct; date-picker dialog interaction is intentionally not mocked in widget tests (standard Flutter practice), but the label logic is exercised through the state the sheet holds. ✅

**AC4 — Scheduler respects start date:** `_calculateFrequencyDates` now initialises the scan cursor/year/month from `start` when `start.isAfter(now)` for all three affected cases (`week`, `month`, `year`). The `day` case already used `start` as its cursor origin. Three new unit tests (`'does not schedule any date/occurrence before a future start date/month/year'`) pass. All 20 pre-existing frequency scheduler tests continue to pass. ✅

**No regressions:** All 100 pre-existing tests pass alongside the 7 new ones (107 total). ✅

**No ADR required:** No new architectural patterns, dependencies, or structural decisions were introduced. The change is scoped to one conditional expression in `creation_bottom_sheet.dart` and three single-line initialisations in `scheduler.dart`.
