# Whisper — Functional & Technical Specifications

This document defines the complete product specifications, technical architecture, and implementation plan for Whisper, a minimalist, highly aesthetic reminder and passive micro-content application built with Flutter and Isar Database.

## 1. Executive Summary

The objective is to build a distraction-free, visually elegant, and highly intuitive mobile application named Whisper. Rather than behaving like a high-pressure to-do list, Whisper functions as a calm companion for passive reminders, daily intentions, inspirational quotes, and optional task check-ins. The application prioritizes fluid micro-interactions, clean typography, adaptive dark/light themes, and robust local scheduling.

### Core Value Propositions

- **Minimalist Aesthetics:** High-end design with plenty of negative space, soft shadows, and subtle transitions to keep the mind calm.
- **Smart & Passive Scheduling:** Support for both specific calendar intervals (daily, weekly, etc.) and frequency-based reminders (e.g., receiving "3 inspirational quotes a week").
- **Local-First & Private:** Fully functional offline, utilizing device-native scheduling to trigger local notifications according to system settings.
- **Reactive Database Architecture:** Powered by Isar to deliver instantaneous, real-time UI updates on the dashboard.

## 2. UI/UX Design System

Whisper's design language focuses on calm, modern UI principles, avoiding cluttered forms or unnecessary inputs.

### 2.1 Color Palette (System Adaptive)

**Light Mode:**
- **Background:** Slate White (`#F8FAFC`)
- **Primary Accent:** Deep Indigo (`#4F46E5`)
- **Text Primary:** Dark Charcoal (`#0F172A`)
- **Text Secondary:** Muted Gray (`#64748B`)
- **Cards/Surface:** Pure White (`#FFFFFF`) with 1px borders (`#E2E8F0`)

**Dark Mode:**
- **Background:** Obsidian Black (`#090A0F`)
- **Primary Accent:** Neon Violet (`#818CF8`)
- **Text Primary:** Off-White (`#F8FAFC`)
- **Text Secondary:** Cool Gray (`#94A3B8`)
- **Cards/Surface:** Dark Navy (`#1E293B`)

### 2.2 Typography

- **Primary Font:** Inter, SF Pro, or Roboto (clean, sans-serif, high legibility).
- **Headings:** Medium/Bold weights with generous line height to make quotes and daily intentions feel impactful.
- **Input Fields:** Large, borderless or minimal bottom-border text entries to preserve a "writing on paper" feel.

### 2.3 User Interactions & Transitions

- **Haptic Feedback:** Gentle haptics on reminder creation, acknowledgement, and removal.
- **Fluid Sheets:** Creating/editing reminders and inspirations happens in an elegant, interactive bottom sheet that pulls up smoothly.
- **Swipe-to-Dismiss:** Swiping a reminder card right triggers "Acknowledge" (silently marking it as seen/read), while swiping left reveals options to edit or delete.

## 3. Functional Specifications

### 3.1 Reminder Creation Flow

The app features a single-screen dashboard with a central Floating Action Button (FAB) or top bar button to add new items.

- **Title Field (Required):** High-prominence text input (ideal for the reminder title, core habit, or the quote itself).
- **Details Field (Optional):** Expandable, multi-line secondary text input for extended descriptions, context, source of a quotation, or personal reflection.
- **Time Selection:** Minimalist wheels or inline sliders for selecting the exact time of day or general delivery windows.
- **Scheduling Engine:** A segmented control or clean toggle interface to switch between Unique (One-time) and Repetitive delivery.

### 3.2 Repetitive Reminder Logic (The Scheduling Core)

Repetitive reminders can be configured via two distinct structures to support both actionable items and passive content delivery:

**Type A: Interval-Based Repetition**
The reminder repeats at exact structural intervals:
- **Daily:** Repeat every $X$ days.
- **Weekly:** Repeat on specific days of the week (e.g., every Monday and Thursday).
- **Monthly:** Repeat on a specific date (e.g., the 15th of every month).
- **Yearly:** Repeat annually on a set calendar date.

**Type B: Frequency-Based Repetition (Occurrences)**
The user defines an expected delivery cadence within a time frame:
- **Goal Pattern:** "Deliver $N$ times per $Day / Week / Month$".

**Implementation Note:** Since native mobile local notification engines (iOS APNs / Android AlarmManager) require specific exact times, Whisper will offer two strategies for frequency-based scheduling:
1. **User-spaced schedule:** The app automatically divides the time frame into equal intervals (e.g., "3 times a week" results in notifications scheduled every Monday, Wednesday, and Friday at a default or chosen time). This is perfect for space-repetition learning or randomized inspirational quote delivery.
2. **Flexible Check-in system:** The app sends a light daily/weekly digest notification prompting the user to acknowledge their $N$ occurrences, tracking progress via an in-app interactive progress bar.

### 3.3 Native System Notifications

- Notifications will utilize the native operating system channels (iOS UserNotifications Framework & Android NotificationCompat API).
- Respects OS focus modes, Do Not Disturb, and customized sound settings.
- Includes "Quick Action" buttons inside the notification payload (e.g., "Acknowledge", "Snooze 15m"). Clicking "Acknowledge" silences the notification and logs the interaction without requiring the user to open the app.

## 4. Technical Architecture (Flutter)

To build a robust, scalable, and modular app, we will use a clean, decoupled folder structure.

```text
lib/
├── core/
│   ├── theme/          # App styles, light/dark custom ThemeData
│   └── utils/          # Date formatters, timezone helpers
├── data/
│   ├── database/       # Isar database local configuration
│   ├── models/         # Isar annotated models (Reminder, RecurrenceRule)
│   └── services/       # Local Notifications & Native Channels
├── providers/          # State management (Riverpod, Provider or Bloc)
└── presentation/
    ├── screens/        # Dashboard, Details, Creation Sheets
    └── widgets/        # Custom cards, custom input pickers, list items
```

### 4.1 Key Dependencies

- **Local Database:** `isar` + `isar_flutter_libs` + `path_provider` (Highly performant, modern, type-safe NoSQL database for Flutter with native FFI bindings).
- **Local Database Generator:** `isar_generator` & `build_runner` (Dev dependencies for compiling type-safe database access code).
- **Local Notifications:** `flutter_local_notifications` (Handling foreground, background, and scheduled notification tasks).
- **Timezones:** `timezone` (Crucial for handling daylight savings changes and scheduling repetitive notifications accurately).
- **State Management:** `flutter_riverpod` or simple native `ValueNotifier` for keeping the footprint low.

## 5. Data Models (Dart)

By utilizing Isar, we can store nested collections natively. The `RecurrenceRule` is marked as `@embedded`, meaning it will be saved directly inside the `Reminder` document, avoiding complex relational multi-table database joins.

```dart
import 'package:isar/isar.dart';

part 'reminder.g.dart';

enum RecurrenceType {
  none,       // One-time
  interval,   // Daily, Weekly, Monthly, Yearly
  frequency   // N times per Day/Week/Month
}

enum RecurrenceUnit {
  day,
  week,
  month,
  year
}

@embedded
class RecurrenceRule {
  @enumerated
  RecurrenceType type = RecurrenceType.none;
  
  // For Interval-Based (Type A)
  @enumerated
  RecurrenceUnit? unit;
  
  int interval = 1; // e.g., Repeat every "2" (interval) "weeks" (unit)
  
  List<int>? daysOfWeek; // 1 = Monday, 7 = Sunday
  
  int? dayOfMonth;

  // For Frequency-Based (Type B)
  int? occurrencesCount; // e.g., "3"
  
  @enumerated
  RecurrenceUnit? frequencyUnit; // e.g., "week"
}

@collection
class Reminder {
  Id id = Isar.autoIncrement; // Isar uses 64-bit integer IDs

  late String title;
  
  String? details;
  
  late DateTime targetDateTime; // Starting point / Target execution date
  
  late RecurrenceRule recurrenceRule;
  
  late bool isEnabled;
  
  late DateTime createdAt;
  
  // Field used to track if this has been acknowledged today / for its current period
  late DateTime? lastAcknowledgedAt;
}
```

## 6. Implementation Roadmap

### Phase 1: Environment Setup & Core UI Shell
- Configure Flutter environment, set up Android/iOS manifest permissions (`POST_NOTIFICATIONS` for Android 13+, background fetch, and native alarm permissions).
- Implement the custom minimalist UI theme (Dark/Light themes optimized for readability of poetry, quotes, and intentions).
- Create the Dashboard showing scheduled passive reminders and an elegant bottom sheet for entering new items.

### Phase 2: Local Storage & Local Notification Integration
- Set up the Isar database layer. Write the initialization routine using `path_provider` to locate the local application documents directory.
- Run `flutter pub run build_runner build` to compile the generated `reminder.g.dart` database logic.
- Initialize `flutter_local_notifications` and setup timezone-aware local scheduling.
- Write the scheduling logic function that maps `RecurrenceRule` instances to future calendar dates and schedules up to 64 local notifications (OS limit).

### Phase 3: Frequency-Based Scheduling Logic (The USP)
- Build the algorithm that processes "N times per Day/Week/Month" and converts it into concrete notification schedules.
- Implement user settings to let them choose delivery windows (e.g., Morning, Afternoon, Evening) so inspirations arrive during mindful hours.

### Phase 4: Polish & Micro-Interactions
- Add custom physics-based scroll animations.
- Integrate haptic feedback routines for the "Acknowledge" gesture.
- Conduct extensive testing across iOS and Android background states (testing notification delivery while device is asleep).
