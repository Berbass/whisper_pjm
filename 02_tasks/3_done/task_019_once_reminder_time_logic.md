---
id: T-019
title: "Improve Time & Date Logic for 'Once' Reminders"
status: done
assigned_to: agent_zed
dependencies: []
completion_percentage: 100%
last_updated: 2026-06-04
---

# Objective
Fix the automatic date-shifting logic when creating a "Once" reminder so that it behaves intuitively when the default time is in the recent past.

# Acceptance Criteria
- [ ] When creating a "Once" reminder for today with no specific time selected, do not blindly push it to the next day.
- [ ] If the current time selected (or default) is in the past by less than 4 hours, alert the user with a confirmation dialog asking if they intend to schedule it for the next day.
- [ ] Apply the user's choice (either keep it for today/immediately, or shift to tomorrow).

# Agent Execution Log

## 2026-06-04 — agent_zed

**Status:** Complete. Moved to `in_review`.

### What was done

- **Replaced the blind auto-advance logic** in `_CreationBottomSheetState._submit()` with a 4-hour threshold check:
  - If `targetDateTime` is in the past by **< 4 hours** (and no explicit date was chosen): shows a `AlertDialog` titled *"Schedule for tomorrow?"* with two choices — **Keep today** (fires immediately via a 30-second buffer) and **Tomorrow** (shifts date by one day).
  - If `targetDateTime` is in the past by **≥ 4 hours**: silently auto-advances to tomorrow (previous behaviour preserved for stale times).
  - If `targetDateTime` is in the future, or an explicit date was chosen: no change.
- **Extracted `onceShouldConfirmNextDay(DateTime targetDateTime, DateTime now)`** as a `@visibleForTesting` module-level function in `creation_bottom_sheet.dart` to allow deterministic unit testing.
- **Added 10 unit tests** in `test/once_reminder_time_logic_test.dart` covering all boundary conditions (future, now, 1 min past, 1 h past, 3h59m past, exactly 4 h, 5 h, 24 h, second-precision boundaries).

### Files changed
- `lib/presentation/widgets/creation_bottom_sheet.dart`
- `test/once_reminder_time_logic_test.dart` *(new)*

### Validation
- `flutter analyze` — no issues ✅
- `flutter test` — 146/146 passed ✅
