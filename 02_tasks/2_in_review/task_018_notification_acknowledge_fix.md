---
id: T-018
title: "Fix Background Notification Actions"
status: in_review
assigned_to: agent_zed
dependencies: []
completion_percentage: 100%
last_updated: 2026-06-04
---

# Objective
Fix the issue where tapping the "Acknowledge" action on a system notification does nothing.

# Acceptance Criteria
- [x] The "Acknowledge" action button on Android/iOS notifications correctly updates the reminder's status in the database.
- [x] The notification is dismissed upon acknowledgement.
- [x] Background execution is properly wired up in `flutter_local_notifications` so the action runs without needing to bring the app to the foreground.

# Agent Execution Log

## 2026-06-04 — agent_zed

### Root Cause Analysis

Two separate code paths handle notification taps, and both were broken:

1. **Background / terminated** (`onBackgroundNotificationResponse` top-level function): The function logged a debug message and returned immediately — no database write occurred.
2. **Foreground** (`_onNotificationTap` static method): On `kAcknowledgeActionId`, the method returned early with a plain `return` statement — again no database write.

### Changes Made

#### `lib/data/database/isar_service.dart`
- Added `IsarService.acknowledgeReminder(Id id)` — a focused helper that fetches the reminder, sets `lastAcknowledgedAt = DateTime.now()` via `copyWith`, and persists it. No-ops safely if the reminder no longer exists.

#### `lib/data/services/notification_service.dart`
- Added imports: `package:flutter/widgets.dart` (for `WidgetsFlutterBinding`) and `../database/isar_service.dart`.
- **`onBackgroundNotificationResponse`** promoted to `async`. Now calls `WidgetsFlutterBinding.ensureInitialized()` (required to activate platform channels in the background isolate), then opens Isar and calls `acknowledgeReminder`. Non-acknowledge actions return early without any DB work.
- **`_onNotificationTap`**: The `return` on `kAcknowledgeActionId` is replaced with a block that parses the payload, fires `IsarService.acknowledgeReminder` (fire-and-forget is safe here since the app is in the foreground), and then returns. Navigation is still suppressed.
- Updated all doc-comments to reflect the corrected behaviour.

#### `test/notification_acknowledge_test.dart` (new file)
- 23 new unit tests covering:
  - `kAcknowledgeActionId` constant value & non-emptiness.
  - Payload → `reminderId` parsing (valid, null, empty, float, non-numeric strings).
  - `Reminder.copyWith(lastAcknowledgedAt: ...)` — field set correctly, no side-effects on other fields, re-acknowledge with newer timestamp, and ordering relative to `createdAt`.

### Validation
- `flutter analyze` — **No issues found**.
- `flutter test` — **130 / 130 tests passed** (107 pre-existing + 23 new).
