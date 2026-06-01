# T-010 — Deep-Linking from Notifications: Implementation Plan

## Problem
When a user taps a local notification, the app opens but lands on the `DashboardScreen` root. There is no navigation to the specific reminder that fired. The `_onNotificationTap` stub in `NotificationService` only prints a log line.

## Architecture Decisions

### Dependency Direction
The `data/` layer must NOT import from `presentation/`. To enable navigation from a static `NotificationService` callback without coupling layers:
- `NotificationService` exposes a **static callback setter** (`setDeepLinkCallback`).
- `main.dart` injects the navigation function when wiring the app together.
- `NotificationService` also stores a `_pendingLaunchReminderId` for cold-start cases.

### Navigator Key
A `GlobalKey<NavigatorState>` in `AppRouter` (presentation layer) is passed to `MaterialApp`. This allows navigation from outside the widget tree once the first frame renders.

## Files Changed

| File | Change |
|------|--------|
| `lib/presentation/navigation/app_router.dart` | **NEW** — navigator key + `goToReminderDetail(int)` |
| `lib/presentation/screens/reminder_detail_screen.dart` | **NEW** — full detail screen |
| `lib/data/services/notification_service.dart` | Add deep-link callback, pending cold-start ID, cold-start check in `init()` |
| `lib/core/utils/scheduler.dart` | Add `payload: reminder.id.toString()` to all `zonedSchedule` calls |
| `lib/main.dart` | Pass navigator key, register deep-link callback |
| `lib/presentation/screens/dashboard_screen.dart` | `initState` + `addPostFrameCallback` for cold-start navigation |

## Flow Diagrams

### Warm Start (app running/minimized)
```
User taps notification
  → flutter_local_notifications fires onDidReceiveNotificationResponse
  → NotificationService._onNotificationTap(response)
  → _deepLinkCallback?.call(int.parse(response.payload!))
  → AppRouter.goToReminderDetail(id)
  → navigatorKey.currentState!.push(ReminderDetailScreen(reminderId: id))
```

### Cold Start (app terminated)
```
User taps notification → OS launches app
  → main() runs → NotificationService.init()
  → getNotificationAppLaunchDetails() returns payload
  → NotificationService._pendingLaunchReminderId = id
  → runApp(WhisperApp) → DashboardScreen renders
  → DashboardScreen.initState → addPostFrameCallback
  → NotificationService.pendingLaunchReminderId != null
  → AppRouter.goToReminderDetail(id)  [navigator now exists]
```

## Payload Encoding
Reminder ID is encoded as a plain decimal string in the notification payload:
`payload: reminder.id.toString()`

The `_notificationId(reminderId, i)` helper already encodes occurrence index into the Android/iOS notification slot, which is separate from the payload. The payload always carries just the `reminderId` so all occurrences of the same reminder deep-link to the same detail screen.
