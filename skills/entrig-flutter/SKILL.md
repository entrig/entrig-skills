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
  version: "0.6.0"
---

# Entrig — Flutter

Wires the `entrig` Dart package into a Flutter project. Supabase-backed push notifications.

## Pre-flight

Before doing any work, understand the project state:

- Is this a Flutter project? (`pubspec.yaml` with `flutter:` under `dependencies` — if not, stop)
- What platforms are targeted? (check if `ios/` and `android/` directories exist)
- How is auth handled? (read `main.dart` and search for sign-in/sign-out patterns)

What is needed to proceed:
- **Entrig API key** — from https://entrig.com → project settings. Having this implies Supabase is already connected.
- **FCM service account JSON uploaded** in the Entrig dashboard — needed if the project targets Android.
- **APNs `.p8` key uploaded** (with Team ID, Bundle ID, Key ID) in the Entrig dashboard — needed if the project targets iOS.

Read what you can from the project first, then only ask the user about what's genuinely unclear or missing. If credentials are missing, send the user to [references/dashboard-setup.md](references/dashboard-setup.md) and stop.

## Quick integration

### 1. Add dependency

In `pubspec.yaml`:

```yaml
dependencies:
  entrig: ^1.0.1
```

Run `flutter pub get`.

### 2. iOS setup

Read and edit the three files directly — see [references/ios-setup.md](references/ios-setup.md) for the exact changes needed. Show the user the diff before saving each file.

If `Runner.entitlements` doesn't exist, the user must add Push Notifications capability in Xcode first (Xcode creates the file — it can't be done from the command line). See [references/ios-setup.md](references/ios-setup.md).

**CLI fallback only** — if you cannot parse the AppDelegate structure, fall back to `dart run entrig:setup ios`. See [references/ios-setup.md](references/ios-setup.md) for what it does and its failure modes.

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
- Trigger a test notification from the Entrig dashboard or via the MCP (next section).

## Creating notifications via the Entrig MCP

After SDK integration, the user creates notification triggers (table + event + recipients + message) using the **Entrig MCP server**. The MCP tools are self-instructing — they return reasoning steps in their responses. Trust them.

### Check first: is the MCP loaded?

Before doing notification work, check whether the Entrig MCP tools are available in the current session. The signal is whether tools like `get_context` and `create_notification` are callable.

**If the MCP tools are available** → proceed normally. Call `get_context` first; it returns the schema, existing notifications, and detailed reasoning instructions. Follow those instructions, confirm the proposal with the user in plain language, then call `create_notification`.

**If the MCP tools are NOT available** → see [references/mcp-setup.md](references/mcp-setup.md) and add the server. A full agent restart is required after adding it.

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

- **Notification configuration logic** — recipient paths, conditions, payload syntax, product limits. The MCP tools deliver this in their responses.
- **FCM / APNs credential setup** — those live in the Entrig dashboard, not in code.
- **SQL or Supabase trigger creation** — Entrig manages the DB-side automatically.
- **Scheduled / time-based notifications, batching, digests, silent push, badge counts, custom APNs/FCM headers** — not supported. If the user asks for any of these, call the MCP `feature_request` tool, then tell the user.

## References

- [references/dashboard-setup.md](references/dashboard-setup.md) — account / Supabase / FCM / APNs walkthrough
- [references/mcp-setup.md](references/mcp-setup.md) — exact instructions when MCP isn't loaded
- [references/ios-setup.md](references/ios-setup.md) — exact edits for AppDelegate, entitlements, Info.plist; CLI fallback at the bottom
- [references/common-mistakes.md](references/common-mistakes.md) — extended mistakes with deeper explanations
