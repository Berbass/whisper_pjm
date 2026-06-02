---
id: T-015
title: "Enhance Notification Actions"
status: done
assigned_to: agent_zed
dependencies: []
completion_percentage: 100%
last_updated: 2026-06-03
---

# Objective
Improve the notification experience on Android by allowing users to acknowledge and dismiss reminders directly from the system notification without opening the application.

# Acceptance Criteria
- [ ] Local notifications include an "Acknowledge" (or Dismiss) action button.
- [ ] Tapping the "Acknowledge" button dismisses the notification.
- [ ] The action runs in the background and does not open or bring the app UI to the foreground.

# Agent Execution Log

## 2026-06-03 — agent_zed

### Summary
Implemented the "Acknowledge" notification action on Android, allowing users to dismiss a reminder directly from the system notification shade without opening the app.

### Changes Made

**`lib/data/services/notification_service.dart`**

1. **Top-level background handler** — Added `onBackgroundNotificationResponse`, a top-level function annotated with `@pragma('vm:entry-point')`. This is required by `flutter_local_notifications` for handling action button taps when the app is backgrounded or terminated. The `cancelNotification: true` flag on the action means the OS has already dismissed the notification by the time this callback fires, so no additional work is needed.

2. **`kAcknowledgeActionId` constant** — Exposed as `static const String kAcknowledgeActionId = 'acknowledge'` on the class for discoverability and to avoid stringly-typed comparisons.

3. **`_acknowledgeAction`** — Added `AndroidNotificationAction` with:
   - `showsUserInterface: false` → tapping the button does NOT bring the app to the foreground.
   - `cancelNotification: true` → the OS automatically dismisses the notification when tapped.

4. **`_androidDetails`** — Added `actions: [_acknowledgeAction]` so every scheduled notification carries the action button.

5. **`_plugin.initialize()`** — Registered `onDidReceiveBackgroundNotificationResponse: onBackgroundNotificationResponse` to satisfy the plugin requirement for background action handling.

6. **`_onNotificationTap`** — Added an early-return guard: if `response.actionId == kAcknowledgeActionId`, the handler returns immediately without invoking the deep-link callback. This covers the foreground case.

### Acceptance Criteria — Verified
- [x] Local notifications include an "Acknowledge" action button — `actions: [_acknowledgeAction]` in `_androidDetails`.
- [x] Tapping the button dismisses the notification — `cancelNotification: true` on `_acknowledgeAction`.
- [x] The action runs in the background and does not open the app UI — `showsUserInterface: false` + early-return in `_onNotificationTap` + `onBackgroundNotificationResponse` top-level handler.

### Validation
- `flutter analyze lib/data/services/notification_service.dart` — no issues ✅
- `flutter test` — 83/83 tests passed ✅

## 2026-06-03 — Reviewer

### Review Decision: APPROVED ✅

All 3 acceptance criteria verified directly against `lib/data/services/notification_service.dart`.

- **Acknowledge action button present:** `_acknowledgeAction = AndroidNotificationAction('acknowledge', 'Acknowledge', ...)` defined as a `static const`, wired into `_androidDetails` via `actions: [_acknowledgeAction]`. Every scheduled notification carries the button. ✅
- **Button dismisses notification:** `cancelNotification: true` on `_acknowledgeAction` — the OS automatically removes the notification from the shade upon tap. ✅
- **No foreground / background safe:** `showsUserInterface: false` prevents the app from surfacing; `onBackgroundNotificationResponse: onBackgroundNotificationResponse` registers the `@pragma('vm:entry-point')` top-level handler for the background/terminated case; the foreground case is covered by the early-return guard `if (response.actionId == kAcknowledgeActionId) return;` in `_onNotificationTap`. ✅

**Process note:** T-015 was moved to `2_in_review/` during work but `00_DASHBOARD.md` was not updated at that time — corrected as part of this review pass.

**Test coverage note:** No new unit tests were added for the acknowledge guard logic. Given that `flutter_local_notifications` callback machinery cannot be meaningfully exercised without complex mocking and the guard is a single-line early return, this is acceptable. Future test infrastructure improvements may revisit this.

**ADR created:** `01_context/adr/002_notification_action_background_handler.md` documents the `@pragma('vm:entry-point')` pattern and its architectural constraints.
