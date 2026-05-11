# Dashboard setup

Read when the user hasn't yet completed Entrig account / Supabase / FCM / APNs setup. These steps happen **outside the codebase** at https://entrig.com.

## What the user should complete

### 1. Create an Entrig account

Sign up at https://entrig.com.

### 2. Connect Supabase

In onboarding, click "Connect Supabase," sign in to Supabase, authorize Entrig, and select the project.

### 3. Upload FCM service account JSON (Android targets)

Required if the app ships on Android.

1. Create a Firebase project at https://console.firebase.google.com
2. Add the Android app to the project
3. Project Settings → Service Accounts → Firebase Admin SDK → Generate new private key
4. Download the JSON and upload it to the Entrig dashboard
5. Provide the **Application ID** (the Android package name, e.g., `com.example.myapp`) — found in `android/app/build.gradle` under `applicationId`

> If iOS is also configured in the Firebase console, FCM can deliver to iOS too — APNs upload becomes optional.

### 4. Upload APNs `.p8` key (iOS targets)

Required if the app ships on iOS and FCM is not handling iOS delivery.

1. Apple Developer → Certificates, Identifiers & Profiles → Keys → +
2. Enable "Apple Push Notifications service (APNs)" → Continue → Register
3. Download the `.p8` file (one-time download — save it)
4. Note the **Key ID** (10 chars, on the confirmation page)
5. Note the **Team ID** (10 chars, in Membership section)
6. Note the **Bundle ID** (Xcode project settings or `Info.plist`)
7. Upload `.p8` + Team ID + Bundle ID + Key ID to the Entrig dashboard
8. Configure for **Production**, **Sandbox**, or both — Sandbox for Xcode debug builds, Production for App Store / TestFlight

### 3. Copy the API key

Dashboard → project settings → copy the API Key. This is what `Entrig.init(apiKey: ...)` and the MCP server need.

## When to use this guide

Use this guide when the user wants help completing dashboard setup for push delivery.

For SDK integration, the API key is enough to initialize Entrig. FCM/APNs configuration is needed to send push notifications.
