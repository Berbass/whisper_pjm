---
id: T-014
title: "Fix UI/UX Issues in Creation Form"
status: in_review
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
- Added `_detailsFocusNode` and requested focus in `onTap` for the details section.
- Added `_isSubmitting` state to `_submit()` to prevent duplicate submissions, disabling the button and showing a `CircularProgressIndicator`.
- Reordered widgets in the build method to show the schedule toggle and frequency options above the date/time selectors.
- Added conditional check `if (!(_scheduleMode == _ScheduleMode.repetitive && _repeatFrequency == _RepeatFrequency.weekly))` to conditionally render the DatePicker.
- Changes saved in `creation_bottom_sheet.dart`.
