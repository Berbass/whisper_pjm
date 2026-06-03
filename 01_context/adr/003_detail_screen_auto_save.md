# ADR-003 — Detail Screen: Auto-Save over Explicit Save

**Date:** 2026-06-04  
**Status:** Accepted  
**Task:** T-020

---

## Context

The Reminder Detail screen previously exposed two explicit "Save" controls: a `FilledButton` in the body and a check-mark `IconButton` in the AppBar. The user had to deliberately trigger a save after making any change (title, details, time, date), which created unnecessary friction for what is fundamentally an edit-in-place screen.

## Decision

Replace all manual save actions with **silent auto-save** triggered by:

1. A text field losing focus (`FocusNode.addListener` — fires `_autoSave()` when `!hasFocus`).
2. Confirming a time or date picker selection (fires `_autoSave()` immediately after the new value is committed to state).

The save is **silent** (no SnackBar, no haptic feedback), consistent with mobile-first UX norms where users expect data to persist automatically (cf. iOS Notes, Reminders, Google Keep).

## Consequences

### Positive
- Removes the cognitive load of "did I remember to save?" from the user.
- The screen's CTA surface is simplified: the body has a single `FilledButton.icon("Acknowledge")` and the AppBar has a single `IconButton(Icons.delete_outline_rounded)`.
- Fewer UI states to manage and test.

### Trade-offs / Constraints
- **Empty-title guard**: Auto-save is skipped silently when the title is empty. The user receives no explicit feedback in this case; if desired, a future task could add a subtle inline validation hint on the title field.
- **Concurrent-save guard**: A boolean `_isSaving` prevents re-entrant saves. The AppBar Delete button does not check `_isSaving`, so a theoretical race exists between an in-flight auto-save and an immediate delete tap. In practice this is harmless (the delete confirmation dialog adds enough latency), but could be tightened if needed.
- **Picker-path not widget-tested**: Time/date picker dialogs are platform-native and difficult to drive from widget tests. The auto-save path after picker confirmation is covered by code review only; the focus-loss path is fully unit-tested.

## Alternatives Considered

- **Keep explicit Save, remove duplicates**: Retain one Save button (AppBar only). Rejected — still adds friction and was inconsistent with a more modern, edit-in-place UX.
- **Debounced keystroke save**: Save on every keystroke with a debounce timer. Rejected — more complex, harder to test, and unnecessary given the focus-loss trigger already captures the end of each edit session.
