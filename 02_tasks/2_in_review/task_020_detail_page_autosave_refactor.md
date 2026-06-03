---
id: T-020
title: "Refactor Reminder Detail Page UI & Auto-Save"
status: in_review
assigned_to: agent_zed
dependencies: []
completion_percentage: 100%
last_updated: 2026-06-04
---

# Objective
Streamline the Reminder Detail screen by removing manual save actions in favor of auto-save, and reorganizing the primary actions to focus on Acknowledgement and Deletion.

# Acceptance Criteria
- [x] Implement auto-save: save the reminder values automatically whenever a field loses focus or a value (time/date) is changed.
- [x] Remove the "Save" button from the bottom of the details page.
- [x] Replace the top-right AppBar "Save" button with a "Delete" icon (which triggers the deletion flow).
- [x] Keep only the "Acknowledge" button at the bottom of the screen.

# Agent Execution Log

## 2026-06-04 — agent_zed

### Changes Made

**`lib/presentation/screens/reminder_detail_screen.dart`**
- Replaced the manual `_save()` method with `_autoSave()`: a silent, guard-protected save that skips gracefully when the title is empty or a save is already in progress.
- Added `FocusNode` instances (`_titleFocusNode`, `_detailsFocusNode`) to both `TextField`s; each node has a listener that calls `_autoSave()` on focus loss.
- Updated `_pickTime()` and `_pickDate()` to call `_autoSave()` immediately after committing the new `_selectedDateTime` to state.
- Removed the `FilledButton.icon("Save")` from the body and the `OutlinedButton.icon("Delete")` row; the body now contains only the single `FilledButton.icon("Acknowledge")`, promoted to the primary action.
- Replaced the AppBar trailing `IconButton(Icons.check_rounded, tooltip: 'Save')` with `IconButton(Icons.delete_outline_rounded, tooltip: 'Delete')` wired to `_delete()`.
- Updated the class-level doc comment to reflect the new auto-save and action layout.
- Removed the haptic feedback call and post-save SnackBar that were part of the old manual save flow.
- Properly disposed both `FocusNode` instances in `dispose()`.

**`lib/data/services/notification_service.dart`** (pre-existing blocker fixed)
- Removed `DartPluginRegistrant.ensureInitialized()` — this API was removed from the Flutter SDK in recent versions; `WidgetsFlutterBinding.ensureInitialized()` alone is sufficient.
- Removed the now-redundant `package:flutter/foundation.dart` import (all used symbols are re-exported by `package:flutter/widgets.dart`).

**`test/reminder_detail_screen_test.dart`**
- Updated rendering tests: `'shows a Save button in the body'` → `'does NOT show a Save button in the body'`; `'shows a Save icon button in the AppBar'` → `'shows a Delete icon button in the AppBar'` + `'does NOT show a Save icon button in the AppBar'`; `'shows the Delete button'` → `'does NOT show a Delete button in the body'`; `'shows the Acknowledge button'` → `'shows the Acknowledge button as the sole bottom action'`.
- Replaced the `'save path'` group with an `'auto-save'` group: all three tests now simulate focus loss via `FocusManager.instance.primaryFocus?.unfocus()` instead of tapping a Save button.

### Validation
- `flutter analyze` — no issues ✅
- `flutter test` — 138/138 tests passed ✅
