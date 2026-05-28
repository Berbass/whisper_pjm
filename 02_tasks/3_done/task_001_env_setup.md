---
id: T-001
title: "Environment Setup & Core Permissions"
status: done
assigned_to: agent_zed
dependencies: []
completion_percentage: 100%
last_updated: 2026-05-28
---

# Objective
Initialize the Flutter environment, clean up the boilerplate, and configure the necessary native iOS and Android permissions required for local notifications and background scheduling.

# Acceptance Criteria
- [x] Flutter project created and default counter app removed.
- [x] Folder structure created according to `whisper_specifications.md` (`lib/core/`, `lib/data/`, etc.).
- [x] Android `AndroidManifest.xml` updated with `POST_NOTIFICATIONS`, `SCHEDULE_EXACT_ALARM`, and `RECEIVE_BOOT_COMPLETED` permissions.
- [x] iOS `Info.plist` updated with relevant notification and background fetch permissions.
- [x] Core dependencies added to `pubspec.yaml` (e.g., `flutter_local_notifications`, `timezone`, `flutter_riverpod`).

# Agent Execution Log
Flutter project scaffolded at `05_deliverables/whisper_app`. Boilerplate counter app replaced. Directory structure created: `lib/core/`, `lib/core/theme/`, `lib/core/constants/`, `lib/data/`, `lib/data/models/`, `lib/presentation/`, `lib/presentation/screens/`, `lib/presentation/widgets/`. All three packages resolved via `flutter pub get`. Android manifest updated with all five required permissions and `flutter_local_notifications` receivers. iOS `Info.plist` updated with `UIBackgroundModes` (fetch + remote-notification) and `NSUserNotificationsUsageDescription`. `flutter analyze` reports no issues.
