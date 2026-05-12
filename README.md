# Entrig Skills

Agent skills for [Entrig](https://entrig.com) — push notifications for Supabase, triggered by database events.

After installing, your AI coding agent (Claude Code, Cursor, etc.) can:

- **Integrate the Entrig SDK** into your app
- **Create and manage notifications** in natural language using the Entrig MCP server
- Avoid the common gotchas: simulator push, AppDelegate quirks, FCM/APNs uploads, recipient path modeling

## Install the skills

```bash
npx skills add entrig/entrig-skills
```

That installs the skill files into your project's `.claude/skills/` (or the equivalent location your agent reads from).

## Add the Entrig MCP server

SDK skills handle app integration. The **`entrig-mcp` skill** handles client-specific MCP setup and notification CRUD workflow.

Use the `entrig-mcp` skill when you need to connect the MCP server or create, update, list, inspect, or delete notification triggers.

Core values: URL `https://mcp.entrig.com/beta`, HTTP / Streamable HTTP transport, bearer/API key auth.

## Available skills

| Skill | What it does |
|---|---|
| [`entrig-mcp`](./skills/entrig-mcp) | Set up the Entrig MCP server and create, update, list, inspect, or delete notification triggers |
| [`entrig-flutter`](./skills/entrig-flutter) | Add the `entrig` Dart package to a Flutter project — install, native setup, code wiring, and tap handling |
| [`entrig-react-native`](./skills/entrig-react-native) | Add `@entrig/react-native` to a **bare** React Native project — setup CLI, pod install, code wiring, and tap handling |
| [`entrig-expo`](./skills/entrig-expo) | Add `@entrig/react-native` to an **Expo** (managed/prebuild) project — config plugin, bundle ID, Apple Developer Portal, code wiring, and tap handling |

More SDKs coming: iOS native, Android native, Capacitor.

Each SDK skill covers its framework integration. The `entrig-mcp` skill covers MCP setup and notification management.

## Prerequisites

Before the agent can do anything useful:

- An Entrig account with **Supabase connected**
- **FCM service account JSON** uploaded (Android targets)
- **APNs `.p8` key** uploaded (iOS targets)
- **API key** from the Entrig dashboard

See [`skills/entrig-flutter/references/dashboard-setup.md`](./skills/entrig-flutter/references/dashboard-setup.md) for the full prerequisite walkthrough.

## License

MIT

## Support

Email: team@entrig.com
