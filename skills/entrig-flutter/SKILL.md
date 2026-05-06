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
  version: "0.3.0"
---

# Entrig — Flutter

Wires the `entrig` Dart package into a Flutter project. Supabase-backed push notifications.

## Pre-flight — confirm before any work

Verify this is a Flutter project: `pubspec.yaml` exists at the project root with `flutter:` under `dependencies`. If not, stop.

Then confirm with the user (do not proceed without):

1. **Entrig API key** — from https://entrig.com → project settings.
2. **Supabase connected** at https://entrig.com (during onboarding).
3. **FCM service account JSON uploaded** — required for Android targets.
4. **APNs `.p8` key uploaded** (with Team ID, Bundle ID, Key ID) — required for iOS targets.
5. **Target platforms** — iOS, Android, or both. Android is zero-config natively. iOS needs a CLI run + Xcode capability.
6. **Auth source** — Supabase Auth or custom. Determines where `Entrig.register` / `Entrig.unregister` calls go.

If any prerequisite is missing, send the user to [references/dashboard-setup.md](references/dashboard-setup.md) and stop. The SDK won't deliver and notification creation will fail without these.

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
- Trigger a test notification from the Entrig dashboard or via the MCP (next section).

## Creating notifications via the Entrig MCP

After SDK integration, the user creates notification triggers (table + event + recipients + message) using the **Entrig MCP server**. The MCP tools are self-instructing — they return reasoning steps in their responses. Trust them.

### Check first: is the MCP loaded?

Before doing notification work, check whether the Entrig MCP tools are available in the current session. The signal is whether tools like `get_context` and `create_notification` are callable.

**If the MCP tools are available** → proceed normally. Call `get_context` first; it returns the schema, existing notifications, and detailed reasoning instructions. Follow those instructions, confirm the proposal with the user in plain language, then call `create_notification`.

**If the MCP tools are NOT available** → see [references/mcp-setup.md](references/mcp-setup.md). Walk the user through adding the server. Do **NOT** improvise:
- Do NOT call Entrig's REST API directly.
- Do NOT write SQL or set up `pg_net` triggers.
- Do NOT default to telling them to use the dashboard.

After the user adds the MCP server, they must fully restart the agent for the tools to load.

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
| 11 | Calling Entrig REST API when MCP isn't loaded | Walk the user through MCP setup instead — see references/mcp-setup.md. |
| 12 | Writing SQL or `pg_net` triggers manually | Never. Entrig manages all DB-side automatically when notifications are created via MCP. |

## What this skill does NOT do

- **Notification configuration logic** — recipient paths, conditions, payload syntax, product limits. The MCP tools deliver this in their responses.
- **FCM / APNs credential setup** — those live in the Entrig dashboard, not in code.
- **SQL or Supabase trigger creation** — Entrig manages the DB-side automatically.
- **Scheduled / time-based notifications, batching, digests, silent push, badge counts, custom APNs/FCM headers** — not supported. If the user asks for any of these, call the MCP `feature_request` tool, then tell the user.

## References

- [references/dashboard-setup.md](references/dashboard-setup.md) — account / Supabase / FCM / APNs walkthrough
- [references/mcp-setup.md](references/mcp-setup.md) — exact instructions when MCP isn't loaded
- [references/ios-setup.md](references/ios-setup.md) — what `dart run entrig:setup ios` does, when it fails
- [references/manual-appdelegate.md](references/manual-appdelegate.md) — exact AppDelegate edits when the CLI bails
- [references/common-mistakes.md](references/common-mistakes.md) — extended mistakes with deeper explanations
