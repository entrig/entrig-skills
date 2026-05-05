# iOS setup details

What `dart run entrig:setup ios` does, when it fails, how to recover.

## What the CLI does

When run from the Flutter project root:

1. Reads `ios/Runner/AppDelegate.swift`. If `EntrigPlugin.checkLaunchNotification` is already there, exits cleanly (no-op).
2. Adds `import UserNotifications` and `import entrig` to the top.
3. In `didFinishLaunchingWithOptions`: inserts `UNUserNotificationCenter.current().delegate = self` and `EntrigPlugin.checkLaunchNotification(launchOptions)` before the `return`.
4. Appends four delegate methods to the class (`didRegisterForRemoteNotifications...`, `didFailToRegister...`, `userNotificationCenter willPresent`, `userNotificationCenter didReceive`).
5. Reads `ios/Runner/Runner.entitlements`. Adds `<key>aps-environment</key><string>development</string>` if not present.
6. Reads `ios/Runner/Info.plist`. Adds `UIBackgroundModes` with `remote-notification` and `fetch` if not present.
7. Creates `.backup` files for each modified file.

## Failure mode 1: `Runner.entitlements` doesn't exist

The CLI prints a warning and skips entitlements. The user must create the file via Xcode (it can't be created cleanly from the command line because Xcode also needs to register the capability):

1. Open `ios/Runner.xcworkspace` in Xcode (not `.xcodeproj`)
2. Select the Runner target → Signing & Capabilities tab
3. Click `+ Capability` → Push Notifications
4. Xcode creates `Runner.entitlements` with the `aps-environment` key

Then re-run `dart run entrig:setup ios` to verify everything is now in place.

## Failure mode 2: AppDelegate has existing delegate methods

If `AppDelegate.swift` already defines any of:
- `didRegisterForRemoteNotificationsWithDeviceToken`
- `didFailToRegisterForRemoteNotificationsWithError`
- `userNotificationCenter ... willPresent`
- `userNotificationCenter ... didReceive`

…the CLI prints "manual integration needed" and exits **without modifying** the AppDelegate. (Imports and `didFinishLaunchingWithOptions` are also not modified in this case.)

You patch manually — see [manual-appdelegate.md](manual-appdelegate.md) for the exact edits. Show the user the diff before applying.

## Failure mode 3: Not running from project root

If `ios/Runner/AppDelegate.swift` doesn't exist at the relative path, the CLI exits with an error. Make sure you're in the Flutter project root (the directory containing `pubspec.yaml`), not in `ios/` or anywhere else.

## After setup

- `cd ios && pod install` if pods are stale (or after first-time setup)
- Build to a real iOS device — simulators do not receive push notifications
- The `.backup` files can be deleted after verifying everything works

## If something is broken

CocoaPods errors after install:

```bash
cd ios
rm Podfile.lock
rm -rf Pods
pod deintegrate
pod repo update
pod install
```

Build cache issues: `flutter clean && flutter pub get && cd ios && pod install`.
