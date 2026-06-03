# 00 — Project Dashboard

> Last updated: 2026-06-04 · Updated by: reviewer

## Sprint Summary

| Status        | Count |
|---------------|-------|
| ✅ Done        | 19    |
| 🔄 In Progress | 0     |
| 🔍 In Review   | 0     |
| 📋 To Do       | 2     |

---

## Task Overview

| ID    | Title                                       | Status | Assigned To |
|-------|---------------------------------------------|--------|-------------|
| T-001 | Environment Setup & Core Permissions        | ✅ done | agent_zed   |
| T-002 | Implement Custom Minimalist UI Theme        | ✅ done | agent_zed   |
| T-003 | Create Dashboard Screen UI                  | ✅ done | agent_zed   |
| T-004 | Implement Elegant Creation Bottom Sheet     | ✅ done | agent_zed   |
| T-005 | Git Setup                                   | ✅ done | agent_zed   |
| T-006 | Isar Database Setup & Models                | ✅ done | agent_zed   |
| T-007 | Local Notifications & Timezone Init         | ✅ done | agent_zed   |
| T-008 | Core Scheduling Logic (Interval-Based)      | ✅ done | agent_zed   |
| T-009 | Implement Frequency-Based Scheduling (Phase 3)| ✅ done | agent_zed   |
| T-010 | Implement Deep-Linking from Notifications   | ✅ done | agent_zed   |
| T-011 | Implement Unit Tests for Interval Scheduling| ✅ done | agent_zed   |
| T-012 | Implement Widget Tests for UI Components    | ✅ done | agent_zed   |
| T-013 | Implement Unit Tests for Domain Models      | ✅ done | agent_zed   |
| T-014 | Fix UI/UX Issues in Creation Form           | ✅ done | unassigned  |
| T-015 | Enhance Notification Actions                | ✅ done | agent_zed   |
| T-016 | Interactive Reminder Detail & Inline Editing| ✅ done | agent_zed   |
| T-017 | Update Date Selector for Repetitions        | ✅ done | agent_zed   |
| T-018 | Fix Background Notification Actions         | ✅ done | agent_zed   |
| T-019 | Improve Time Logic for "Once" Reminders     | 📋 todo | unassigned  |
| T-020 | Refactor Detail Page UI & Auto-Save         | ✅ done | agent_zed   |
| T-021 | Dashboard Enhancements                      | ✅ done | agent_zed   |
| T-022 | Creation Form UI/UX Polish                  | 📋 todo | unassigned  |

---

## Deliverable

Flutter project at `04_deliverables/whisper_app/`

### Structure

```
lib/
├── core/
│   ├── constants/app_constants.dart       # Layout constants & durations
│   ├── theme/
│   │   ├── app_colors.dart                # All design-system color tokens
│   │   └── app_theme.dart                 # Light & dark ThemeData (Material 3)
│   └── utils/
│       └── scheduler.dart                 # Interval & frequency notification scheduler
├── data/
│   ├── database/
│   │   └── isar_service.dart              # Isar singleton + CRUD + reactive stream
│   ├── models/
│   │   ├── reminder.dart                  # @collection Reminder + @embedded RecurrenceRule
│   │   └── reminder.g.dart                # Isar-generated schema (build_runner)
│   └── services/
│       └── notification_service.dart      # flutter_local_notifications + deep-link handling
├── presentation/
│   ├── navigation/
│   │   └── app_router.dart                # GlobalKey<NavigatorState> + goToReminderDetail
│   ├── providers/
│   │   └── reminder_provider.dart         # Riverpod StreamProvider for reminders list
│   ├── screens/
│   │   ├── dashboard_screen.dart          # Main list view + FAB + cold-start deep-link
│   │   └── reminder_detail_screen.dart    # Full detail view (deep-link target)
│   └── widgets/
│       ├── reminder_card.dart             # Card with border/dark-navy variants
│       └── creation_bottom_sheet.dart     # Elegant creation sheet
└── main.dart                              # IsarService + NotificationService init + ProviderScope
```

test/
├── widget_test.dart                           # App shell smoke test
├── scheduler_frequency_test.dart              # Frequency scheduling unit tests (20 tests)
├── scheduler_interval_test.dart               # Interval scheduling unit tests (22 tests)
├── reminder_model_test.dart                   # Domain model unit tests (27 tests)
├── creation_bottom_sheet_test.dart            # CreationBottomSheet widget tests (6 tests)
├── dashboard_screen_test.dart                 # DashboardScreen widget tests (7 tests)
└── reminder_detail_screen_test.dart           # ReminderDetailScreen widget tests (17 tests)
```

### Validation

- `flutter pub get` — all dependencies resolved ✅
- `flutter pub run build_runner build` — `reminder.g.dart` generated ✅
- `flutter analyze` — no issues ✅
- `flutter test` — 138/138 tests passed ✅
