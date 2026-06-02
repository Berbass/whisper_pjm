---
id: T-014
title: "Fix UI/UX Issues in Creation Form"
status: done
assigned_to: zed_agent
dependencies: []
completion_percentage: 100%
last_updated: 2026-06-03
---

# Objective
Address user feedback regarding the UX and UI flow in the reminder creation form. This includes fixing input interaction, preventing duplicate submissions, reordering form fields, and implementing conditional visibility for date selection.

# Acceptance Criteria
- [x] Tapping the note/context input field requires only one tap to focus and bring up the keyboard.
- [x] The "Save reminder" button displays a loading state or disables immediately upon tap to prevent duplicate reminder creation.
- [x] The form field order is strictly: Once/Repeat Toggle -> Date Selector -> Time.
- [x] The Date Selector is entirely hidden from the UI when "Weekly" repeat is selected.

# Agent Execution Log

## 2026-06-03 — agent_zed (worker)

- Added `_detailsFocusNode` and requested focus in `onTap` for the details section.
- Added `_isSubmitting` state to `_submit()` to prevent duplicate submissions, disabling the button and showing a `CircularProgressIndicator`.
- Reordered widgets in the build method to show the schedule toggle and frequency options above the date/time selectors.
- Added conditional check `if (!(_scheduleMode == _ScheduleMode.repetitive && _repeatFrequency == _RepeatFrequency.weekly))` to conditionally render the DatePicker.
- Changes saved in `creation_bottom_sheet.dart`.

## 2026-06-03 — Reviewer

### Review Decision: APPROVED ✅

All 4 acceptance criteria verified directly against `lib/presentation/widgets/creation_bottom_sheet.dart`.

- **Single-tap focus:** `GestureDetector.onTap` in `_buildDetailsSection` calls `_detailsFocusNode.requestFocus()` immediately. The `FocusNode` is properly declared, disposed, and wired to the `TextField`. ✅
- **Duplicate-submit prevention:** `_isSubmitting` bool is declared. `_submit()` checks `_isSubmitting` at entry and sets it `true`. Button uses `onPressed: _isSubmitting ? null : _submit` and renders a `CircularProgressIndicator` while `true`. ✅
- **Field order (Toggle → Date → Time):** `build()` calls `_buildScheduleToggle` → `_buildFrequencyOptions` → (conditional) `_buildDatePicker` → `_buildTimePicker` in that sequence. ✅
- **Date Selector hidden for Weekly:** `if (!(_scheduleMode == _ScheduleMode.repetitive && _repeatFrequency == _RepeatFrequency.weekly))` correctly gates the `_buildDatePicker` call. ✅

No follow-up changes required.
