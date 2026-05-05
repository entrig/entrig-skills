---
name: entrig-flutter
description: >
  Add Entrig push notifications to a Flutter app (Supabase-backed). Use when the user asks to set up,
  integrate, or add push notifications in a Flutter project — especially with Supabase Auth.
  Triggers: "Entrig" + Flutter, "push notifications" + Flutter + Supabase, adding the `entrig` Dart
  package, configuring iOS push for a Flutter app, registering devices for Supabase-backed push,
  wiring `Entrig.init` / `Entrig.register` / notification listeners in a Flutter app.
  This skill handles SDK integration only. For notification creation / management, use the `entrig`
  skill (MCP-driven). For Entrig concepts (recipient paths, payloads), see the `entrig` skill.
metadata:
  author: entrig
  version: "0.1.0"
---

# Entrig — Flutter

Wires the `entrig` Dart package into a Flutter project. Supabase-backed push notifications.

## Pre-flight

Verify this is a Flutter project: `pubspec.yaml` exists at the project root and has `flutter:` under `dependencies`. If not, stop.

Confirm with the user (don't proceed without):

1. **Entrig API key** — from https://entrig.com → project settings. If they don't have one, see [`entrig` skill → references/dashboard-setup.md](../entrig/references/dashboard-setup.md).
2. **Target platforms** — iOS, Android, or both. Android is zero-config. iOS needs CLI + Xcode capability.
3. **Auth source** — Supabase Auth or custom. Determines where `register` / `unregister` calls go.

## Quick integration

### 1. Add dependency

In `pubspec.yaml`:

```yaml
dependencies:
  entrig: ^1.0.1
```

Run `flutter pub get`.

### 2. iOS setup (skip if Android-only)

```bash
dart run entrig:setup ios
```

Patches `AppDelegate.swift`, `Runner.entitlements`, `Info.plist`. Creates `.backup` files.

**Two failure modes** — see [references/ios-setup.md](references/ios-setup.md):
- `Runner.entitlements` doesn't exist → user must add Push Notifications capability in Xcode first
- AppDelegate already has custom delegate methods → manual patch needed, see [references/manual-appdelegate.md](references/manual-appdelegate.md)

### 3. Initialize in `main.dart`

After `WidgetsFlutterBinding.ensureInitialized()` and after `Supabase.initialize(...)` if present:

```dart
import 'package:entrig/entrig.dart';

await Entrig.init(apiKey: const String.fromEnvironment('ENTRIG_API_KEY'));
```

**Never hardcode the key.** If the project uses `flutter_dotenv`, read from `.env` (and add to `.gitignore`). Otherwise use `--dart-define=ENTRIG_API_KEY=...` or follow the project's existing secret pattern.

### 4. Register on auth

**Supabase Auth (recommended):**

```dart
Supabase.instance.client.auth.onAuthStateChange.listen((data) {
  final session = data.session;
  if (session != null) {
    Entrig.register(userId: session.user.id);
  } else {
    Entrig.unregister();
  }
});
```

Place in the root widget's `initState`. Convert `MyApp` to `StatefulWidget` if needed. If an `onAuthStateChange` listener already exists, extend it — don't add a second one.

**Custom auth:** find the sign-in success path → `Entrig.register(userId: <theirUserId>)`. Find sign-out → `Entrig.unregister()`. The `userId` MUST match the user identifier field in the notification trigger.

### 5. Listeners

```dart
Entrig.foregroundNotifications.listen((event) {
  // event.title, event.body, event.type, event.data
});

Entrig.onNotificationOpened.listen((event) {
  // navigate based on event.type / event.data
});
```

Ask the user if they want navigation wired now, or stub listeners with a TODO.

### 6. Verify

- iOS: `cd ios && pod install` if pods are stale, then build to a **real device** (simulators don't receive push).
- Android: build to a device or emulator with Google Play Services.
- Trigger a test notification from the Entrig dashboard or MCP.

## Common mistakes

| # | Mistake | Fix |
|---|---|---|
| 1 | Testing iOS push on a simulator | Real device only — simulators won't receive. |
| 2 | Hardcoding the API key | Use `--dart-define`, `flutter_dotenv`, or the project's secret pattern. |
| 3 | `userId` mismatch with notification trigger | The userId in `Entrig.register()` must match the user identifier field in the dashboard. With Supabase Auth, that's `auth.users.id`. |
| 4 | Skipping `Runner.entitlements` Xcode step | If file doesn't exist, the CLI bails. User must add Push Notifications capability in Xcode first. |
| 5 | Re-running `flutter create` or pod regen wipes iOS setup | Re-run `dart run entrig:setup ios` after any iOS regeneration. |
| 6 | Forgetting `flutter pub get` after `pubspec.yaml` edit | Always run after dep changes. |
| 7 | Stale build after pod/dep changes | `flutter clean` then `flutter run` if behavior is flaky. |
| 8 | Configuring FCM/APNs in Flutter code | Those go in the Entrig dashboard, not in the app. |
| 9 | Multiple `onAuthStateChange` listeners | Extend the existing one — don't add a second. |
| 10 | Adding `firebase_messaging` or other push packages | Entrig handles delivery itself. Don't combine with another push SDK unless you know the conflict surface. |

## What this skill does NOT do

- Create notification triggers — use the `entrig` MCP server (see the `entrig` skill).
- Configure FCM / APNs credentials — those live in the Entrig dashboard.
- Write SQL or Supabase triggers — Entrig manages the DB-side automatically.
- Handle scheduled / time-based notifications, batching, digests — not supported.

For unsupported requests, call the `feature_request` MCP tool, then tell the user.

## References

- [references/ios-setup.md](references/ios-setup.md) — what `dart run entrig:setup ios` does, when it fails, how to fix
- [references/manual-appdelegate.md](references/manual-appdelegate.md) — exact AppDelegate edits when the CLI bails
- [references/common-mistakes.md](references/common-mistakes.md) — extended mistakes table with deeper explanations
