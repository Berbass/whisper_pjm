---
id: T-007
title: "Local Notifications & Timezone Initialization"
status: todo
assigned_to: unassigned
dependencies: [T-001]
completion_percentage: 0%
last_updated: 2026-05-28
---

# Objective
Initialize the `flutter_local_notifications` package and configure timezone-aware local scheduling. This lays the groundwork for triggering reminder notifications.

# Acceptance Criteria
- [ ] Create `lib/data/services/notification_service.dart`.
- [ ] Initialize the `timezone` package using `tz.initializeTimeZones()` on app startup.
- [ ] Configure `FlutterLocalNotificationsPlugin` initialization settings for both Android and iOS.
- [ ] Implement runtime permission requests for iOS (UserNotifications) and Android 13+ (`POST_NOTIFICATIONS`, `SCHEDULE_EXACT_ALARM`).
- [ ] Create a dummy test function to trigger an immediate local notification to verify the setup.
