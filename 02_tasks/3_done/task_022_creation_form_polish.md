---
id: T-022
title: "Creation Form UI/UX Polish"
status: done
assigned_to: agent_zed
dependencies: []
completion_percentage: 100%
last_updated: 2026-06-04
---

# Objective
Apply final polish to the creation bottom sheet, improving copy, dynamic labels, and preventing accidental data loss.

# Acceptance Criteria
- [ ] Display the actual target time instead of "set time" in the time picker trigger row (e.g., "at 3:45 pm").
- [ ] Rename the submit button from "Save reminder" to "Save".
- [ ] If the user has entered data and attempts to dismiss the creation sheet (via tapping outside or swipe down), display a confirmation dialog asking if they want to discard their changes.

# Agent Execution Log

## 2026-06-04 — agent_zed

**Status:** Complete. Moved to `in_review`.

### What was done

1. **Time picker label** — Changed `_buildTimePicker()` so that instead of rendering the static string *"Set time"*, the row always shows `'at ${(_selectedTime ?? TimeOfDay.now()).format(context)}'`. When no time is explicitly selected the label appears in muted colour (indicating the default); once the user picks a time it switches to the primary `onSurface` colour.

2. **Submit button copy** — Renamed the `FilledButton` label from `'Save reminder'` to `'Save'`.

3. **Discard confirmation dialog** — Wrapped the widget tree in `PopScope(canPop: !_hasUserData, ...)`. `_hasUserData` returns `true` when the title or details field contains text, or when a date/time has been explicitly selected. Added `_onDataChanged()` listeners on both `TextEditingController`s (registered in `initState`, removed in `dispose`) so that `canPop` is always fresh after a rebuild. When `canPop` is `false` and a pop is invoked (barrier tap or swipe-down), `_showDiscardDialog()` presents an `AlertDialog` with **Keep editing** and **Discard** actions.

### Files changed
- `lib/presentation/widgets/creation_bottom_sheet.dart`
- `test/creation_bottom_sheet_test.dart` — 6 new widget-test cases added (time label, button copy, 4 discard-dialog scenarios)

### Validation
- `flutter analyze` — no issues ✅
- `flutter test` — 146/146 passed ✅
