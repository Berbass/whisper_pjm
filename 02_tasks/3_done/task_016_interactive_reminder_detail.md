---
id: T-016
title: "Interactive Reminder Detail & Inline Editing"
status: done
assigned_to: agent_zed
dependencies: []
completion_percentage: 100%
last_updated: 2026-06-03
---

# Objective
Allow users to easily view and edit their reminders by making the reminder cards interactive and adding inline editing capabilities to the reminder detail widget.

# Acceptance Criteria
- [x] Tapping a `ReminderCard` on the dashboard opens the reminder detail widget/view.
- [x] The reminder's title, content (note/context), and timing are editable directly from the detail view.
- [x] Edits made in the detail view are saved to the database and update the dashboard automatically.

# Agent Execution Log

## 2026-06-03 — agent_zed

### Summary
Implemented interactive reminder cards and inline editing on the detail screen, satisfying all three acceptance criteria.

### Changes Made

**`lib/presentation/widgets/reminder_card.dart`**
- Added optional `VoidCallback? onTap` parameter.
- Replaced the outer `Container` with `Ink` + `InkWell` so that the card decoration (border, radius, surface colour) is preserved while Material ripple feedback works correctly on tap.

**`lib/presentation/screens/dashboard_screen.dart`**
- Passed `onTap: () => AppRouter.goToReminderDetail(reminder.id)` into each `ReminderCard` built inside the `Dismissible` `itemBuilder`. The swipe-to-dismiss behaviour is unaffected because `Dismissible` captures horizontal drag gestures before `InkWell` can resolve a tap.

**`lib/presentation/screens/reminder_detail_screen.dart`** — major refactor:
- Added `Reminder? initialReminder` parameter to allow widget-test injection (bypassing the async DB fetch).
- Replaced `FutureBuilder` with explicit `_isLoading` / `_isSaving` state managed in `initState` + `setState`.
- Added `TextEditingController _titleController` and `_detailsController`; both are borderless `TextField`s styled to match the existing typographic hierarchy (headlineSmall / bodyMedium).
- Added `DateTime _selectedDateTime` mutated by `_pickTime()` (calls `showTimePicker`) and `_pickDate()` (calls `showDatePicker`, only shown for one-time reminders).
- Added `_save()` which calls `IsarService.updateReminder()` then `Scheduler.scheduleReminder()` so notifications are rescheduled. Because `IsarService` uses `watchAllReminders()` as a reactive stream, the dashboard `remindersProvider` emits the updated list automatically — no extra wiring needed.
- Added `_TappableInfoRow` helper widget: renders editable schedule fields (Time, Date) with an underline + pencil icon as visual affordance, and fires a picker on tap.
- Kept `_acknowledge()` and `_delete()` logic intact from the previous read-only version.
- Primary action is a full-width `FilledButton.icon` ("Save") in the body; a `IconButton(Icons.check_rounded)` in the AppBar provides a secondary quick-save path.

**`test/reminder_detail_screen_test.dart`** — new file (14 tests):
- Rendering group (11 tests): verifies title/details TextFields, Save button, Acknowledge, Delete, formatted time, date row visibility (present for one-time, hidden for recurring), Repeat row for interval reminders.
- Inline editing group (3 tests): verifies `enterText` updates the displayed value for title, details, and that an empty details field shows the hint.

**`test/dashboard_screen_test.dart`** — 1 new test:
- Smoke test: tapping a `ReminderCard` does not throw (navigation is a no-op because `AppRouter.navigatorKey` is not wired in the test harness).

### Acceptance Criteria — Verified
- [x] Tapping a `ReminderCard` on the dashboard opens the reminder detail view — `onTap: () => AppRouter.goToReminderDetail(reminder.id)` wired in `DashboardScreen._buildList`.
- [x] Title, content, and timing are editable from the detail view — borderless `TextField`s for text; `showTimePicker` / `showDatePicker` for the schedule card rows.
- [x] Edits are saved to the database and update the dashboard automatically — `IsarService.updateReminder()` + `Scheduler.scheduleReminder()` in `_save()`; the dashboard reacts via the `watchAllReminders()` `StreamProvider`.

### Validation
- `flutter analyze` — no issues ✅
- `flutter test` — 97/97 tests passed ✅

## 2026-06-03 — Reviewer

### Review Decision: APPROVED ✅ (after one revision cycle)

All 3 acceptance criteria verified against the source code and test suite.

**AC1 — Tapping a `ReminderCard` opens the detail view:** `onTap: () => AppRouter.goToReminderDetail(reminder.id)` wired in `DashboardScreen._buildList`; `AppRouter.navigatorKey` is already attached to `MaterialApp` in `main.dart`. `InkWell` tap and `Dismissible` swipe coexist without conflict. ✅

**AC2 — Fields editable from detail view:** Borderless `_titleController` / `_detailsController` `TextField`s styled to match the design system. `showTimePicker` on Time row; `showDatePicker` on Date row (one-time only). ✅

**AC3 — Edits saved + dashboard auto-updates:** `_save()` calls `IsarService.updateReminder` (write transaction) + `Scheduler.scheduleReminder`. `watchAllReminders()` emits on every Isar write, so the `remindersProvider` `StreamProvider` on the dashboard reflects changes without additional wiring. ✅

**Bug found and fixed during review:** The original `_save()` passed `details: rawDetails.isEmpty ? null : rawDetails` to `copyWith`, but `copyWith` uses `null` as "value not provided" so clearing a note was silently ignored. Fixed by assigning `details` directly via cascade after the `copyWith` call: `reminder.copyWith(...)..details = rawDetails.isEmpty ? null : rawDetails`.

**Test coverage expanded during revision:**
- `onSave` callback injection added to `ReminderDetailScreen` for testability.
- 3 new save-path widget tests: updated title, cleared details persists null, empty-title guard shows SnackBar + blocks save.
- Final count: `flutter test` — 100/100 tests passed. ✅

**No ADR required:** changes are UI-layer only with no new architectural patterns or dependency decisions.
