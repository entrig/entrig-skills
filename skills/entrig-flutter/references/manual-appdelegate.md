# Manual AppDelegate patching

Read when `dart run entrig:setup ios` reports existing delegate methods and bails. You patch `ios/Runner/AppDelegate.swift` yourself. Show the diff to the user before saving.

## Imports (top of file)

Add if not present:

```swift
import UserNotifications
import entrig
```

## Inside `didFinishLaunchingWithOptions`

Add these two lines **before** the existing `return`:

```swift
UNUserNotificationCenter.current().delegate = self
EntrigPlugin.checkLaunchNotification(launchOptions)
```

## Existing delegate methods — inject calls inline

The CLI bails because these methods already exist. **Do not duplicate them.** Add the Entrig call inside each existing body.

### `didRegisterForRemoteNotificationsWithDeviceToken`

Add inside the existing method body:

```swift
EntrigPlugin.didRegisterForRemoteNotifications(deviceToken: deviceToken)
```

### `didFailToRegisterForRemoteNotificationsWithError`

```swift
EntrigPlugin.didFailToRegisterForRemoteNotifications(error: error)
```

### `userNotificationCenter willPresent`

Add inside the existing body. The completion handler should include Entrig's foreground options — merge with whatever the existing handler is doing:

```swift
EntrigPlugin.willPresentNotification(notification)
// when calling completionHandler:
completionHandler(EntrigPlugin.foregroundPresentationOptions())
```

If the existing method already returns its own `UNNotificationPresentationOptions`, combine them — e.g., union with `EntrigPlugin.foregroundPresentationOptions()`. Ask the user what to do if the existing logic looks intentional.

### `userNotificationCenter didReceive`

```swift
EntrigPlugin.didReceiveNotification(response)
// then call existing completionHandler() if present
```

## After patching

- Verify imports and method bodies compile (open in Xcode if uncertain).
- `cd ios && pod install` if you also added the dependency for the first time.
- Build to a real device.

## Methods to add if NOT present

If only some of the four delegate methods exist, you may need to add the missing ones. Use these full bodies:

```swift
override func application(_ application: UIApplication,
    didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
  EntrigPlugin.didRegisterForRemoteNotifications(deviceToken: deviceToken)
}

override func application(_ application: UIApplication,
    didFailToRegisterForRemoteNotificationsWithError error: Error) {
  EntrigPlugin.didFailToRegisterForRemoteNotifications(error: error)
}

override func userNotificationCenter(_ center: UNUserNotificationCenter,
    willPresent notification: UNNotification,
    withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
  EntrigPlugin.willPresentNotification(notification)
  completionHandler(EntrigPlugin.foregroundPresentationOptions())
}

override func userNotificationCenter(_ center: UNUserNotificationCenter,
    didReceive response: UNNotificationResponse,
    withCompletionHandler completionHandler: @escaping () -> Void) {
  EntrigPlugin.didReceiveNotification(response)
  completionHandler()
}
```
