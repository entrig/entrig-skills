# Entrig concepts

Read when you need to model recipients, conditions, or payloads for a notification trigger.

## Trigger anatomy

A notification trigger has three parts:

| Part | What it answers | Where it's defined |
|---|---|---|
| **Trigger** | When does it fire? | `table` + `event` (insert/update/delete) + optional `eventConditions` |
| **Recipients** | Who receives it? | `path` (FK route from event table to recipient identity) + optional `recipientFilters` |
| **Content** | What does it say? | `title`, `body`, `type`, `payload` |

## Recipient paths

The `path` is a route through foreign keys from the **event table** to the **recipient identity column** (e.g., `users.id` or `profiles.id`).

| Pattern | Path shape |
|---|---|
| Direct FK on the event row | `messages.user_id` |
| FK chain | `messages.room_id -> rooms.owner_id -> users.id` |
| Junction table | `messages.room_id -> room_members(room_id -> user_id) -> users.id` |
| Repeated junction on same table | `messages.room_id -> room_members(room_id -> org_id) -> room_members(org_id -> user_id) -> users.id` |
| Broadcast (everyone) | `broadcast -> users.id` |

**The path's last hop MUST be the recipient identity column** (e.g., `users.id`), not a foreign key column that points to it. If your last column is an FK referencing the recipient table, add one more hop.

## Conditions

Filter when the trigger fires (`eventConditions`) or which recipients receive it (`recipientFilters`).

```json
{
  "all": [
    { "field": "orders.status", "operator": "=", "value": "shipped" },
    {
      "any": [
        { "field": "orders.priority", "operator": "=", "value": "high" },
        { "field": "orders.amount", "operator": ">", "value": 1000 }
      ]
    }
  ]
}
```

- `"all": [...]` is AND, `"any": [...]` is OR
- `"value"` for literals, `"valueField"` for field-to-field comparison
- For UPDATE events: `"orders.old.status"` references the previous value

Operators: `=`, `!=`, `>`, `<`, `>=`, `<=`, `LIKE`, `ILIKE`, `IN`, `NOT IN`, `IS NULL`, `IS NOT NULL`, `@>`, `<@`, `?`, `?|`, `?&`

## Placeholders and payload

Title, body, and payload entries use dotted refs inside `{{...}}`:

- `{{messages.content}}` — column on event row
- `{{messages.user_id.name}}` — through FK
- `{{orders.old.status}}` — previous value (UPDATE only)

**Important:** Placeholders cannot reference recipient/join-table values. One notification fanout uses one shared rendered message — per-recipient personalization in a single trigger is not supported. If the user wants per-recipient content, propose multiple triggers.

## Limits to know

- One trigger = one recipient path. For multiple audiences, propose multiple triggers.
- One trigger = one shared rendered message across all recipients.
- No scheduled / time-based triggers — only direct DB events.
- No batching, digests, badge counts, silent push, custom APNs/FCM headers, collapse keys, TTLs, or media/image options.

When the user asks for any of these, call the MCP `feature_request` tool.
