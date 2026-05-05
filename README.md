# Entrig Skills

Agent skills + MCP server for [Entrig](https://entrig.com) — push notifications for Supabase, driven by database events.

Install once and your AI coding agent (Claude Code, Cursor, OpenAI Codex, …) can:

- **Integrate the Entrig SDK** into your app (Flutter, Expo, React Native, iOS, Android, Capacitor)
- **Create and manage notifications** in natural language via the Entrig MCP server
- Avoid the common gotchas — simulator push, AppDelegate quirks, FCM/APNs upload steps, recipient path modeling

## Install

```bash
npx skills add entrig/entrig-skills
```

This installs the skills and wires the Entrig MCP server. Set your API key:

```bash
export ENTRIG_API_KEY=your_key_here
```

Get your key at [entrig.com](https://entrig.com) → project settings.

## Available skills

| Skill | What it does |
|---|---|
| [`entrig`](./skills/entrig) | Cross-cutting concepts, MCP usage, dashboard prerequisites |
| [`entrig-flutter`](./skills/entrig-flutter) | Add the `entrig` Dart package to a Flutter project |

More SDKs coming: Expo, React Native, iOS native, Android native, Capacitor.

## Bundled MCP server

Installing this plugin also configures the [Entrig MCP server](../../entrig_mcp/) so agents can create, list, update, and delete notifications without leaving the editor. The MCP and skills work together: skills handle SDK integration, MCP handles notification configuration.

## Prerequisites

- An Entrig account with Supabase connected
- FCM service account JSON uploaded (Android targets)
- APNs `.p8` key uploaded (iOS targets)
- API key from the Entrig dashboard

See [`skills/entrig/references/dashboard-setup.md`](./skills/entrig/references/dashboard-setup.md) for the full prerequisite walkthrough.

## Plugin platforms

This repo also ships as a plugin for:

- **Claude Code** — `.claude-plugin/`
- **Cursor** — `.cursor-plugin/`
- **OpenAI Codex** — `.codex-plugin/`

## License

MIT

## Support

Email: team@entrig.com
