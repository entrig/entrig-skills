# Entrig MCP server

Use this guide to set up the Entrig MCP server so the coding agent can create and manage notifications.

## Setup

MCP setup is client-specific. Do not assume every agent uses `.mcp.json`.

Use these Entrig connection values in whatever MCP setup flow the user's client supports:

- Server name: `entrig`
- Transport/type: HTTP / Streamable HTTP
- Server URL: `https://mcp.entrig.com/beta`
- Authentication: bearer token / API key
- Header: `Authorization: Bearer <entrig_api_key>`

If the client supports a command-line setup, use that. If it supports a UI connector flow, use the UI. If it uses a JSON config file, add the Entrig server to the existing config without removing other servers.

### Claude Code

```bash
claude mcp add --transport http entrig https://mcp.entrig.com/beta \
  --header "Authorization: Bearer YOUR_ENTRIG_API_KEY"
```

### JSON-based MCP clients

Add this to the client's MCP config file. The exact file path varies by client; use the client's docs or existing project config.

```json
{
  "mcpServers": {
    "entrig": {
      "type": "http",
      "url": "https://mcp.entrig.com/beta",
      "headers": {
        "Authorization": "Bearer <entrig_api_key>"
      }
    }
  }
}
```


## After adding

If the MCP config is project-level, ensure that exact config file is in `.gitignore` because it contains the API key. Do not commit API keys.

A full restart of the agent is required because the tool list is fixed at session start. Tell the user to fully quit and relaunch, then ask again to create or manage the notification.
