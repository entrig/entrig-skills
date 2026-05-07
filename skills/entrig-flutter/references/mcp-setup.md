# Adding the Entrig MCP server

Read when the Entrig MCP tools (`get_context`, `create_notification`, etc.) are not available and the user wants to create or manage notifications.

## What needs to happen

The Entrig MCP server must be added to the agent's MCP configuration. Here are the connection details:

```
transport: http
url: https://mcp.entrig.com/beta
headers:
  Authorization: Bearer <entrig_api_key>
```

Use the API key already found in the project. If it hasn't been confirmed yet, ask for it once.

## mcp.json / .mcp.json format

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

If the MCP was added to a project-level file (`.mcp.json`), add it to `.gitignore` — it contains the API key.

A full restart of the agent is required — the tool list is fixed at session start. Tell the user to fully quit and relaunch, then ask again to create the notification.

