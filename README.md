# Entrig Skills

Agent skills for [Entrig](https://entrig.com) — push notifications for Supabase, triggered by database events.

After installing, your AI coding agent (Claude Code, Cursor, etc.) can:

- **Integrate the Entrig SDK** into your app (Flutter — more SDKs coming)
- **Create and manage notifications** in natural language using the Entrig MCP server
- Avoid the common gotchas: simulator push, AppDelegate quirks, FCM/APNs uploads, recipient path modeling

## Install the skills

```bash
npx skills add entrig/entrig-skills
```

That installs the skill files into your project's `.claude/skills/` (or the equivalent location your agent reads from).

## Add the Entrig MCP server

Skills handle SDK integration. The **MCP server** is what lets your agent create, list, and update notifications. Add it to your agent's MCP config.

### Claude Code

Add the Entrig MCP server with one command:

```bash
claude mcp add --transport http entrig https://mcp.entrig.com/beta \
  --header "Authorization: Bearer YOUR_ENTRIG_API_KEY"
```

Replace `YOUR_ENTRIG_API_KEY` with your key from [entrig.com](https://entrig.com) → project settings.

Then restart Claude Code and confirm with `/mcp` — `entrig` should appear as connected.

### Cursor / other agents

Add to your `.mcp.json` (or your agent's equivalent MCP config):

```json
{
  "mcpServers": {
    "entrig": {
      "type": "http",
      "url": "https://mcp.entrig.com/beta",
      "headers": {
        "Authorization": "Bearer YOUR_ENTRIG_API_KEY"
      }
    }
  }
}
```

Restart your agent to load the server.

## Available skills

| Skill | What it does |
|---|---|
| [`entrig-flutter`](./skills/entrig-flutter) | Add the `entrig` Dart package to a Flutter project — install, native setup, code wiring, and using the Entrig MCP for notifications |
| [`entrig-react-native`](./skills/entrig-react-native) | Add `@entrig/react-native` to a **bare** React Native project — setup CLI, pod install, code wiring, MCP usage |
| [`entrig-expo`](./skills/entrig-expo) | Add `@entrig/react-native` to an **Expo** (managed/prebuild) project — config plugin, bundle ID, Apple Developer Portal, code wiring, MCP usage |

More SDKs coming: iOS native, Android native, Capacitor.

Each SDK skill is self-contained — it covers its own framework's integration plus how to use the Entrig MCP server to create and manage notifications.

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
