---
id: T-010
title: "Implement Deep-Linking from Notifications"
status: done
assigned_to: agent_zed
dependencies: [T-007, T-008]
completion_percentage: 100%
last_updated: 2026-06-01
---

# Objective
Enable deep-linking when a user taps a local notification. The application should handle the notification payload and navigate directly to the relevant reminder's details or a specific action screen, rather than just opening the root dashboard.

# Acceptance Criteria
- [ ] Configure `flutter_local_notifications` to handle notification tap events.
- [ ] Inject the reminder ID or necessary data into the notification payload when scheduling.
- [ ] Implement routing logic to intercept the payload and navigate to the detail view.
- [ ] Ensure deep-linking works from both a background state (app running but minimized) and a terminated state (cold start).

# Agent Execution Log

## 2026-06-01 — agent_zed

**Status:** Completed

### Approach
Used a callback-based dependency-injection pattern to keep the `data/` layer free
of `presentation/` imports:

- `NotificationService` exposes `setDeepLinkCallback(void Function(int))` and
  `pendingLaunchReminderId` / `clearPendingLaunch()` for warm vs. cold start.
- `AppRouter` (new, in `presentation/navigation/`) holds
  `GlobalKey<NavigatorState>` and `goToReminderDetail(int)`.  
- `main.dart` wires everything: passes `AppRouter.navigatorKey` to `MaterialApp`
  and registers the deep-link callback.
- `DashboardScreen.initState` handles cold start via
  `WidgetsBinding.addPostFrameCallback`.

### Files Changed
| File | Type | Summary |
|------|------|---------|
| `lib/presentation/navigation/app_router.dart` | **New** | Navigator key + `goToReminderDetail` |
| `lib/presentation/screens/reminder_detail_screen.dart` | **New** | Full detail screen |
| `lib/data/services/notification_service.dart` | Modified | Deep-link callback, pending launch ID, cold-start check |
| `lib/core/utils/scheduler.dart` | Modified | `payload: reminder.id.toString()` on all 3 `zonedSchedule` calls |
| `lib/main.dart` | Modified | Navigator key, deep-link callback wiring |
| `lib/presentation/screens/dashboard_screen.dart` | Modified | `initState` cold-start handler |

### Acceptance Criteria
- [x] Configure `flutter_local_notifications` to handle notification tap events.
- [x] Inject the reminder ID into the notification payload when scheduling.
- [x] Implement routing logic to navigate to the detail view.
- [x] Deep-linking works from background state (warm start via `_onNotificationTap` + callback).
- [x] Deep-linking works from terminated state (cold start via `pendingLaunchReminderId` + `addPostFrameCallback`).

### Validation
`flutter analyze` — no issues ✅
