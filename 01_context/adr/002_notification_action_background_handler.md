# ADR-002 — Notification Action Background Handler Pattern

**Date:** 2026-06-03
**Status:** Accepted
**Related Task:** T-015

---

## Context

`flutter_local_notifications` supports adding action buttons to Android notifications (`AndroidNotificationAction`). When a user taps an action button while the app is **backgrounded or terminated**, the plugin spins up a separate Dart isolate to deliver the `NotificationResponse`. This isolate is completely isolated from the main app isolate — it has no access to the widget tree, providers, or any shared in-memory state.

The plugin requires that the callback registered for this case (`onDidReceiveBackgroundNotificationResponse`) must be a **top-level Dart function** (not a method, closure, or anonymous function), and it must be annotated with `@pragma('vm:entry-point')` to prevent the AOT compiler's tree-shaker from removing it.

## Decision

We introduce a dedicated top-level function `onBackgroundNotificationResponse` in `notification_service.dart` to handle background notification action taps.

```dart
@pragma('vm:entry-point')
void onBackgroundNotificationResponse(NotificationResponse response) {
  debugPrint(
    '[NotificationService] Background action — '
    'id: ${response.id}, actionId: ${response.actionId}',
  );
}
```

This function is registered in `NotificationService.init()`:

```dart
await _plugin.initialize(
  initSettings,
  onDidReceiveNotificationResponse: _onNotificationTap,
  onDidReceiveBackgroundNotificationResponse: onBackgroundNotificationResponse,
);
```

For the current "Acknowledge" action, `cancelNotification: true` means the OS already dismisses the notification before the callback fires. No additional work is needed in the background handler beyond logging.

The foreground case is handled in `_onNotificationTap` via an early-return guard:

```dart
if (response.actionId == kAcknowledgeActionId) return;
```

## Consequences

### Positive
- Satisfies the `flutter_local_notifications` plugin contract for background action handling.
- The `showsUserInterface: false` + background-only handler combination ensures the app never surfaces to the foreground when the user taps "Acknowledge".
- Clear separation: foreground logic in `_onNotificationTap`, background logic in the top-level handler.

### Constraints
- **The background handler must remain a top-level function.** It cannot be moved to a class method or converted to a closure — doing so will cause a runtime crash on Android.
- **The `@pragma('vm:entry-point')` annotation is mandatory.** Removing it will cause the release build's tree-shaker to eliminate the function, breaking background action handling in production.
- **The background isolate has no access to app state.** Any future background action logic that requires database writes or complex state changes must initialize its own service instances (e.g., `IsarService`) within the handler itself, rather than using the global singletons from the main isolate.
- **iOS note:** `AndroidNotificationAction` is Android-only. The `DarwinNotificationDetails` configuration does not include action buttons in the current implementation; iOS notification actions would require separate `DarwinNotificationActionInput`/`DarwinNotificationAction` setup if needed in the future.
