---
name: entrig-flutter
description: >
  Add Entrig push notifications to a Flutter app (Supabase-backed). Use when the user asks to set up,
  integrate, or add push notifications to a Flutter project — especially with Supabase Auth.
  Triggers: "Entrig" + Flutter, "push notifications" + Flutter + Supabase, adding the `entrig` Dart
  package, configuring iOS push for a Flutter app, registering devices for Supabase-backed push,
  wiring `Entrig.init` / `Entrig.register` / notification listeners, creating notification triggers
  for a Flutter+Supabase app via the Entrig MCP server.
  This skill covers the full Flutter integration: SDK install, native setup, code wiring, and how to
  get the Entrig MCP server loaded so the user can create/manage notifications from the editor.
metadata:
  author: entrig
  version: "0.7.7"
---

# Entrig — Flutter

Wires the `entrig` Dart package into a Flutter project. Push notifications based on database triggers for the apps built with Supabase as backend.

## Pre-flight

Before doing any work, understand the project state:

- Is this a Flutter project? (`pubspec.yaml` with `flutter:` under `dependencies` — if not, stop)
- What platforms are targeted? (check if `ios/` and `android/` directories exist)
- How is auth handled? (read `main.dart` and search for sign-in/sign-out patterns)
- Check current requirements (min iOS/Android versions) — done in Step 1 after `flutter pub get`.

What is needed for SDK setup:
- **Entrig API key** — from https://app.entrig.com → project settings. This is required for `Entrig.init(...)` and implies Supabase is already connected.

FCM/APNs configuration is needed to send push notifications. Ensure the required push provider configuration is completed in the Entrig dashboard before sending notifications.

Read what you can from the project first, then only ask the user about what's genuinely unclear or missing. If the Entrig API key is missing, ask the user to copy it from the Entrig dashboard project settings.

## Quick integration

### 1. Add dependency

```bash
flutter pub add entrig
```

Use the command output to identify the installed latest `entrig` package version.

### 2. iOS setup

Read and edit the three files directly — see [references/ios-setup.md](references/ios-setup.md) for the exact changes needed. Show the user the diff before saving each file.

If `Runner.entitlements` doesn't exist, the user must add Push Notifications capability in Xcode first (Xcode creates the file — it can't be done from the command line). See [references/ios-setup.md](references/ios-setup.md).


### 3. Initialize in `main.dart`

After `WidgetsFlutterBinding.ensureInitialized()` and after `Supabase.initialize(...)` if present:

```dart
import 'package:entrig/entrig.dart';

await Entrig.init(apiKey: const String.fromEnvironment('ENTRIG_API_KEY'));
```

**Never hardcode the key.** If the project uses `flutter_dotenv`, read from `.env` (and add to `.gitignore`). Otherwise use `--dart-define=ENTRIG_API_KEY=...` or follow the project's existing secret pattern.

### 4. Register the device

Call `Entrig.register(userId: ...)` when the app has the identifier it will use for notification targeting. This can be a signed-in user ID, anonymous/device-scoped ID, or another stable identifier. The same identifier must be stored in the database so Entrig can find it from the table row/event that triggers the notification.

**Example:**

```dart
await Entrig.register(userId: identifier);
```

Follow the project's existing auth/session/state pattern. Call `Entrig.unregister()` when the app should stop receiving notifications for the previously registered identifier.

### 5. Listeners

```dart
Entrig.foregroundNotifications.listen((event) {
  // event.title, event.body, event.type, event.data
});

Entrig.onNotificationOpened.listen((event) {
  // navigate based on event.type / event.data
});
```

Read the project's existing navigation pattern and wire `onNotificationOpened` consistently. If there is no pattern yet, a recommended approach is a dedicated `PushNotificationService` class with a `switch` on `event.type` — but follow whatever the project already uses.

When notification triggers are created or updated via the Entrig MCP, the MCP response includes `notification_tap_contract` with the notification `type` and `payload`. Immediately update the existing `onNotificationOpened` handler so tapping that notification opens the correct screen. Do not create a second global listener if one already exists.

When a notification is deleted via the MCP, remove stale `onNotificationOpened` routing for the deleted type if no remaining notification uses that type.

### 6. Creating notifications via the Entrig MCP

After SDK integration, create and manage notification triggers using the Entrig MCP server. The MCP tools return reasoning steps and post-action instructions — follow them.

Before doing notification work, check whether tools like `get_context` and `create_notification` are callable in the current session.

**If the MCP tools are available** → call `get_context` first; it returns schema + existing notifications + reasoning instructions. Follow those, confirm the proposal in plain language, then call `create_notification`, `update_notification`, or `delete_notification`.

After `create_notification` or `update_notification` succeeds:
- Read `notification_tap_contract.type` and `notification_tap_contract.payload` from the MCP response.
- Update the Flutter app's existing `Entrig.onNotificationOpened.listen(...)` handler to route by that `type` using the returned payload fields.
- Follow the project's navigation pattern. If no pattern exists, use a dedicated service with a `switch` on `event.type`.

After `delete_notification` succeeds:
- Read `deleted_notification_tap_contract.type` if present.
- Remove stale tap routing for that type only if no remaining notification uses it.

**If the MCP tools are NOT available** → follow [references/mcp-setup.md](references/mcp-setup.md). A full agent restart is required after adding it. Do **not** call Entrig's REST API directly, write SQL, or set up `pg_net` triggers manually.

### 7. Verify

- iOS: `cd ios && pod install` if pods are stale, then build to a **real device** (simulators don't receive push).
- Android: build to a device or emulator with Google Play Services.

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
| 11 | Creating a notification but not updating tap routing | After MCP create/update, update `Entrig.onNotificationOpened` using `notification_tap_contract.type` and payload. After delete, remove stale routing if unused. |


## References

- [references/dashboard-setup.md](references/dashboard-setup.md) — account / Supabase / FCM / APNs walkthrough
- [references/mcp-setup.md](references/mcp-setup.md) — MCP server installation
- [references/ios-setup.md](references/ios-setup.md) — exact edits for AppDelegate, entitlements, Info.plist; CLI fallback at the bottom
- [references/common-mistakes.md](references/common-mistakes.md) — extended mistakes with deeper explanations
