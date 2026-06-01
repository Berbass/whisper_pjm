---
id: T-007
title: "Local Notifications & Timezone Initialization"
status: done
assigned_to: agent_zed
dependencies: [T-001]
completion_percentage: 100%
last_updated: 2026-05-28
---

# Objective
Initialize the `flutter_local_notifications` package and configure timezone-aware local scheduling. This lays the groundwork for triggering reminder notifications.

# Acceptance Criteria
- [x] Create `lib/data/services/notification_service.dart`.
- [x] Initialize the `timezone` package using `tz.initializeTimeZones()` on app startup.
- [x] Configure `FlutterLocalNotificationsPlugin` initialization settings for both Android and iOS.
- [x] Implement runtime permission requests for iOS (UserNotifications) and Android 13+ (`POST_NOTIFICATIONS`, `SCHEDULE_EXACT_ALARM`).
- [x] Create a dummy test function to trigger an immediate local notification to verify the setup.

# Agent Execution Log
- Created `notification_service.dart` with static `init()`, `requestPermissions()`, and `showTestNotification()`.
- `init()` calls `tz_data.initializeTimeZones()` then initialises the plugin for Android (`@mipmap/ic_launcher`) and iOS (no auto-permission prompt — deferred to `requestPermissions()`).
- `requestPermissions()` resolves Android/iOS platform implementations separately; Android requests both `POST_NOTIFICATIONS` and `SCHEDULE_EXACT_ALARM`.
- Exposed `plugin` and `notificationDetails` as static getters for use by `Scheduler`.
- Added `_onNotificationTap` callback stub for future deep-linking.
- Updated `main.dart` to call `IsarService.open()`, `NotificationService.init()`, and `NotificationService.requestPermissions()` before `runApp`.
- `flutter analyze` — no issues ✅ · `flutter test` — 1/1 passed ✅
