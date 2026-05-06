---
name: entrig
description: >
  Use when working with Entrig — push notifications for Supabase, triggered by database events.
  Triggers: "Entrig", "push notifications" + Supabase, "notify users when" + a Supabase table change,
  notification setup for Flutter / Expo / React Native / iOS / Android / Capacitor apps backed by Supabase,
  using the Entrig MCP server (entrig MCP), creating/updating/deleting notification triggers,
  configuring FCM / APNs for a Supabase-backed mobile app.
  This skill covers cross-cutting concepts (recipient model, dashboard prerequisites, MCP usage)
  and routes to framework-specific skills (entrig-flutter, etc.) for SDK integration.
metadata:
  author: entrig
  version: "0.2.0"
  homepage: https://entrig.com
---

# Entrig

Push notifications for Supabase. Database event → device push, configured once, no backend code.

## How Entrig is structured

| Layer | Lives where | Configured by |
|---|---|---|
| **Notification triggers** (table, event, recipients, message) | Entrig backend (DB triggers + pg_net) | Entrig dashboard or `entrig` MCP server |
| **FCM / APNs credentials** | Entrig dashboard | Manual upload by user |
| **SDK integration** (init, register, listeners) | User's app | `entrig-{framework}` skill |

**Three rules that prevent wasted work:**

1. **Never write SQL, edit Supabase triggers, or touch `pg_net`.** Entrig manages the entire DB-side automatically when notifications are created via the dashboard or MCP.
2. **Never configure FCM/APNs in the user's app.** Those credentials live in the Entrig dashboard. The SDK only handles device registration and notification delivery.
3. **Don't hardcode the API key.** Use env vars or the project's secret-management convention.

## What do you need?

| Task | Go to |
|---|---|
| **Add Entrig to a Flutter app** | Activate the `entrig-flutter` skill |
| **Add Entrig to an Expo / React Native / iOS / Android / Capacitor app** | Coming soon — direct user to https://entrig.com/docs |
| **Create a notification trigger** | Use the `entrig` MCP server — see [references/mcp-usage.md](references/mcp-usage.md) |
| **List / update / delete existing notifications** | Use the `entrig` MCP server — see [references/mcp-usage.md](references/mcp-usage.md) |
| **Understand recipient paths, conditions, payloads** | [references/concepts.md](references/concepts.md) |
| **Verify the user has done the dashboard prerequisites** | [references/dashboard-setup.md](references/dashboard-setup.md) |

## Pre-flight checklist

Before doing **any** Entrig work — SDK integration or notification creation — confirm with the user:

1. **Entrig account** with **Supabase connected** at https://entrig.com
2. **API key** copied from the project settings page
3. **FCM service account JSON uploaded** if targeting Android
4. **APNs `.p8` key uploaded** (with Team ID, Bundle ID, Key ID) if targeting iOS

If any of these are missing, stop and direct the user to [references/dashboard-setup.md](references/dashboard-setup.md). Don't write code or call MCP tools without the prerequisites — both will fail.

## Entrig MCP server

Notification creation, updates, and management require the **Entrig MCP server**. Tools:

| Tool | Purpose |
|---|---|
| `get_context` | Fetch live DB schema + existing notifications. **Always call first** when creating/updating. |
| `create_notification` | Create a new notification trigger |
| `update_notification` | Replace an existing notification |
| `list_notifications` | List notifications for the project |
| `delete_notification` | Delete a notification |
| `feature_request` | Log a feature request when the user asks for something unsupported |

**The MCP server's tools have detailed instructions baked in.** Trust them — call `get_context` first, follow the reasoning steps it returns, confirm the proposal with the user in plain language, then call `create_notification`. See [references/mcp-usage.md](references/mcp-usage.md) for the full flow.

### When the MCP server is NOT available

If the Entrig MCP tools (`get_context`, `create_notification`, etc.) are not registered in your current session, **STOP**. Do not improvise:

- Do NOT call Entrig's REST API directly.
- Do NOT write SQL or create Supabase triggers manually.
- Do NOT ask the user to use the dashboard as a substitute (unless they explicitly prefer it).

Instead, tell the user the MCP server isn't loaded, and give them the exact commands to add it. See [references/mcp-setup.md](references/mcp-setup.md) for the message to deliver.

## Common mistakes

| # | Mistake | Fix |
|---|---|---|
| 1 | Writing SQL or `pg_net` setup manually | Entrig manages all DB-side automatically. Just call the MCP. |
| 2 | Configuring FCM/APNs in the user's app code | Those go in the Entrig dashboard, not in the app. |
| 3 | Hardcoding the API key | Use env vars (`ENTRIG_API_KEY`) or the project's secret pattern. |
| 4 | Skipping the dashboard prerequisites | Account + Supabase connection + FCM/APNs upload must happen before SDK work or notification creation. |
| 5 | Mismatched user IDs | The `userId` passed to `Entrig.register()` MUST match the user identifier field configured in the notification trigger. With Supabase Auth, this is `auth.users.id`. |
| 6 | Testing iOS push on a simulator | Push notifications require a real iOS device. Simulators won't receive them. |
| 7 | Calling `list_notifications` after `get_context` | `get_context` already returns existing notifications. Don't duplicate the call. |
| 8 | Promising features not yet supported | Scheduled notifications, batching, digests, custom APNs headers, badge counts, silent push, per-recipient personalization in one notification — none are supported. Call `feature_request` and tell the user. |

## When something doesn't fit

If the user asks for something not supported:
1. Call the MCP `feature_request` tool with a plain-language description.
2. Tell the user: "That's not supported yet — I've logged it as a feature request."
3. For SDK questions outside notification delivery (analytics, in-app messaging, custom UI), point them to team@entrig.com.
