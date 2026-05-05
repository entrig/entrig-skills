# Using the Entrig MCP server

Read when the user asks to create, update, list, or delete a notification trigger.

## Setup check

The Entrig MCP server is bundled with this plugin. The user needs:

```bash
export ENTRIG_API_KEY=<their_key>
```

If MCP tools aren't available (`get_context`, `create_notification`, etc.), tell the user to:
1. Set `ENTRIG_API_KEY` in their environment.
2. Restart their agent / reload the MCP connection.

## Standard flow for create/update

**Always follow this order. Don't skip steps.**

### 1. Call `get_context` first

This fetches the live DB schema **and** existing notifications in one call. Pass `table` and/or `event` if the user already mentioned them — pre-filters the existing notifications.

```
get_context({ table: "messages", event: "insert" })
```

The response includes built-in reasoning instructions. **Read them** and follow them. They cover duplicate detection, recipient modeling, and what to ask the user.

### 2. Check for duplicates

If `existing_notifications` includes a trigger on the same table+event the user wants, surface it:

> "There's already a notification on `messages` insert: 'New message'. Do you want to create another, or update the existing one?"

### 3. Reason through WHEN / WHO / WHAT

The instructions returned by `get_context` walk through this. The structured reasoning is:

- **WHEN** — table, event, optional `eventConditions`. For UPDATE events, decide if the trigger is gated on a specific field changing (use `.old.` refs).
- **WHO** — name the recipient table + identity column FIRST. Then build the path to it. Last hop must be that exact column.
- **WHAT** — pick the payload fields, then write title/body using dotted-ref placeholders.

See `concepts.md` for path patterns, condition syntax, and placeholder rules.

### 4. Confirm with the user in plain language

Show what the notification will look like on a phone (title + body with example values). Ask: "Does this look right, or would you like to change anything?"

**Only call `create_notification` after the user confirms.**

### 5. Call the tool

```
create_notification({ table, event, path, eventConditions?, recipientFilters?, title, body, type, payload? })
```

If validation fails (e.g., invalid path), the tool returns a clear error. Fix the path or ask the user a clarifying question — don't retry with the same input.

## Listing / deleting

- **`list_notifications`** — only when the user explicitly wants to browse existing notifications. If you've already called `get_context`, the data is in that response — don't call `list_notifications` again.
- **`delete_notification`** — requires `notification_id`. Get it from `get_context` or `list_notifications`.

## Updating

`update_notification` requires both `notification_id` AND `trigger_name`. Both are returned by `get_context` / `list_notifications`. The rest of the input is the same shape as `create_notification`.

## Unsupported requests

When the user asks for something not supported (scheduled triggers, batching, digests, per-recipient content, silent push, etc.):

1. Call `feature_request({ description: "<what they asked for, in plain language>" })`.
2. Tell the user: "That's not supported yet — I've logged it as a feature request."

Do this **before** telling the user it's not supported, so the request is captured.

## Things NOT to do

- Don't write SQL or call Supabase directly to create triggers — Entrig manages all DB-side automatically.
- Don't call `get_tables` after `get_context` — schema is already in the response.
- Don't call `list_notifications` after `get_context` — notifications are already there.
- Don't promise per-recipient personalization, push delivery options (silent, badge, TTL), or anything beyond the fields in `create_notification`.
