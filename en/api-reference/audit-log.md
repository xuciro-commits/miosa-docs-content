---
slug: "api-reference/audit-log"
title: "Audit Log API"
description: "Retrieve a cursor-paginated audit log of all actions taken within your tenant."
section: "API reference"
order: 12049
---

The audit log records all significant actions performed within your tenant  -  resource creation and deletion, API key usage, settings changes, and more. Results are cursor-paginated and ordered newest-first.

Base path: `/api/v1/audit-log`

> **Note**
>
> Audit events are immutable. There is no write API. The log is append-only.

---

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/audit-log` | Fetch recent audit events |

---

## List Audit Events

**`GET /api/v1/audit-log`**

### Auth

```http
Authorization: Bearer msk_...
```

### Query Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `limit` | integer | No | `50` | Events per page (max `100`) |
| `after` | string | No |  -  | Cursor from previous response's `next_cursor` |
| `type` | string | No |  -  | Filter by event type prefix (e.g. `sandbox.`) |
| `actor_id` | string | No |  -  | Filter by actor user ID |
| `resource_type` | string | No |  -  | Filter by resource type (e.g. `sandbox`) |
| `resource_id` | string | No |  -  | Filter by resource ID |

### Response  -  `200 OK`

```json
{
  "data": [
    {
      "id": "evt_01hwxyz...",
      "type": "sandbox.created",
      "actor": {
        "type": "api_key",
        "id": "user_42"
      },
      "resource": {
        "type": "sandbox",
        "id": "sbx_abc123"
      },
      "ts": "2026-05-26T10:14:23Z",
      "metadata": {
        "image": "debian-12-sandbox-v8",
        "workspace_id": "ws_01hwxyz..."
      }
    }
  ],
  "next_cursor": "MjAyNi0wNS0yNlQxMDoxNDoyM1o"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `data[].id` | string | Unique event ID |
| `data[].type` | string | Dot-namespaced event type |
| `data[].actor.type` | string | `"user"` or `"api_key"` |
| `data[].actor.id` | string | User ID or `external_user_id` for API key actors |
| `data[].resource.type` | string | Resource type (e.g. `sandbox`, `computer`, `webhook`) |
| `data[].resource.id` | string | Resource ID |
| `data[].ts` | string | ISO 8601 timestamp |
| `data[].metadata` | object | Event-specific detail fields |
| `next_cursor` | string | Opaque cursor; omitted when no more pages |

### Common Event Types

| Type | Trigger |
|------|---------|
| `sandbox.created` | Sandbox provisioned |
| `sandbox.destroyed` | Sandbox deleted |
| `sandbox.exec` | `POST /exec` called |
| `computer.created` | Computer provisioned |
| `computer.stopped` | Computer stopped |
| `computer.deleted` | Computer deleted |
| `snapshot.created` | Snapshot initiated |
| `api_key.created` | API key minted |
| `api_key.deleted` | API key revoked |
| `tenant.preview_domain.set` | Preview domain configured |
| `webhook.created` | Outgoing webhook created |
| `quota.upserted` | Per-user quota set |
| `settings.updated` | Tenant settings changed |

Use the `type` filter with a prefix to narrow events (e.g. `type=sandbox.` returns all sandbox events).

---

## Pagination

Audit events are returned newest-first. To page forward:

1. Issue the initial request without `after`.
2. If `next_cursor` is present, re-issue with `after={next_cursor}`.
3. Stop when `next_cursor` is absent.

Cursors are opaque base64url strings encoding a timestamp. Do not parse or construct them.

---

## Errors

| Status | Code | Cause |
|--------|------|-------|
| 400 |  -  | Invalid cursor format |

---

## Examples

**cURL**

```bash
# First page
curl "https://api.miosa.ai/api/v1/audit-log?limit=50&type=sandbox." \
  -H "Authorization: Bearer msk_u_..."

# Next page
curl "https://api.miosa.ai/api/v1/audit-log?limit=50&after=MjAyNi0wNS0yNlQxMDoxNDoyM1o" \
  -H "Authorization: Bearer msk_u_..."

# Filter by resource
curl "https://api.miosa.ai/api/v1/audit-log?resource_type=computer&resource_id=cmp_xyz789" \
  -H "Authorization: Bearer msk_u_..."
```

**Python**

```python
import miosa

client = miosa.Miosa()

# Page through all sandbox events in the last page
cursor = None
while True:
    page = client.audit_log.list(
        type="sandbox.",
        limit=100,
        after=cursor,
    )
    for event in page.data:
        print(event.type, event.resource.id, event.ts)
    if not page.next_cursor:
        break
    cursor = page.next_cursor
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

let cursor: string | undefined;

do {
  const page = await client.auditLog.list({
    type: 'sandbox.',
    limit: 100,
    after: cursor,
  });

  for (const event of page.data) {
    console.log(event.type, event.resource.id, event.ts);
  }

  cursor = page.nextCursor;
} while (cursor);
```
