# Adding the Entrig MCP server

Read when the Entrig MCP tools (`get_context`, `create_notification`, etc.) are not available in the current session and the user wants to create or manage notifications.

## What to tell the user

Deliver this message verbatim (adapt formatting to fit the agent's UI):

> The Entrig MCP server isn't connected in this session. To create notifications from your editor, add it once:
>
> **For Claude Code:**
> ```
> claude mcp add --transport http entrig https://mcp.entrig.com/beta \
>   --header "Authorization: Bearer YOUR_ENTRIG_API_KEY"
> ```
>
> **For Cursor or other MCP-compatible agents**, add this to your `.mcp.json`:
> ```json
> {
>   "mcpServers": {
>     "entrig": {
>       "type": "http",
>       "url": "https://mcp.entrig.com/beta",
>       "headers": {
>         "Authorization": "Bearer YOUR_ENTRIG_API_KEY"
>       }
>     }
>   }
> }
> ```
>
> Replace `YOUR_ENTRIG_API_KEY` with your key from https://entrig.com → project settings.
>
> Then **fully quit and relaunch the agent** (not just clear the conversation — the tool list is fixed for the session). After restart, ask me again to create the notification.

## Why a full restart is required

Most agents fix the tool list at session start. Adding an MCP server mid-session updates the config but doesn't make the tools callable until a fresh session loads them. Telling the user to "reload" or "restart MCP" isn't enough — they need a clean process restart.

## What NOT to do

- **Do not call Entrig's REST API directly to create notifications.** The MCP server runs validation, schema fetching, path validation, and reasoning that REST callers must replicate. You will get it wrong.
- **Do not write SQL or set up `pg_net` triggers manually.** Entrig manages all DB-side automatically. Manual SQL bypasses Entrig's bookkeeping and will leave the project in an inconsistent state.
- **Do not push the user to the dashboard as a default fallback.** Some users prefer it; that's fine if they ask. But the whole point of the MCP is the in-editor flow — defaulting to the dashboard concedes the value proposition.

## Verifying the MCP is loaded after restart

Tell the user: in Claude Code, run `/mcp` and confirm `entrig` appears as connected. If it shows as failed:

1. Confirm the API key is correct (it should match the one in their Entrig dashboard).
2. Confirm the env where the agent was launched has the right value.
3. Check that the URL is reachable: `curl -i https://mcp.entrig.com/beta` should return 401 (server is up, no auth header sent).

If `entrig` is connected but the agent still says tools are unavailable, the session likely didn't pick up the new server — try one more clean restart.
