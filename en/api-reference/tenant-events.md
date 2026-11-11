---
slug: "api-reference/tenant-events"
title: "Tenant Events (SSE) API"
description: "Subscribe to a real-time server-sent event stream of all tenant-wide platform events."
section: "API reference"
order: 12053
---

The tenant events stream delivers real-time Server-Sent Events (SSE) for all activity across your tenant  -  sandbox lifecycle changes, deployments, webhook deliveries, and more. Subscribe once and receive events for all your resources.

Base path: `/api/v1/events/stream`

> **Note**
>
> This endpoint uses `text/event-stream` (SSE). Set `Accept: text/event-stream` on your request, or use a native `EventSource` client.

> **Note**
>
> The `Authorization` header cannot be set by browser `EventSource`. Use a server-side client or pass an SSE ticket obtained from `POST /api/v1/auth/sse-ticket` for browser usage.

---

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/events/stream` | Open tenant-wide SSE event stream |

---

## Open Event Stream

**`GET /api/v1/events/stream`**

Upgrades to an SSE stream. The connection stays open until the client disconnects or the server closes it. A `: heartbeat` comment is sent every 15 seconds to keep the connection alive through proxies.

### Auth

```http
Authorization: Bearer msk_...
```

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `types` | string | No | Comma-separated glob patterns to filter events (e.g. `sandbox.*,webhook.delivered`) |

### Response  -  `200 OK` (`text/event-stream`)

Each event is a standard SSE frame:

```text
id: evt_01hwxyz...
event: sandbox.ready
data: {"type":"sandbox.ready","id":"sbx_abc123","status":"running","tenant_id":"...","ts":"2026-05-26T10:14:23Z"}

id: evt_01hwyza...
event: deployment.succeeded
data: {"type":"deployment.succeeded","id":"dep_xyz456","url":"https://my-app.miosa.app","ts":"2026-05-26T10:15:01Z"}

: heartbeat
```

### Event Shape

All events share a common envelope:

```json
{
  "type": "sandbox.ready",
  "id": "sbx_abc123",
  "tenant_id": "ten_...",
  "ts": "2026-05-26T10:14:23Z"
}
```

Additional fields vary by event type (see table below).

---

## Event Types

### Sandbox Events

| Type | Description | Extra Fields |
|------|-------------|-------------|
| `sandbox.created` | Sandbox provisioned | `image`, `workspace_id` |
| `sandbox.ready` | Sandbox reached `running` status | `status` |
| `sandbox.error` | Sandbox failed to start | `error`, `reason` |
| `sandbox.destroyed` | Sandbox deleted |  -  |
| `sandbox.paused` | Sandbox vCPUs suspended |  -  |
| `sandbox.resumed` | Sandbox vCPUs resumed |  -  |

### Computer Events

| Type | Description | Extra Fields |
|------|-------------|-------------|
| `computer.created` | Computer provisioned | `size`, `workspace_id` |
| `computer.running` | Computer reached `running` status |  -  |
| `computer.stopped` | Computer stopped |  -  |
| `computer.deleted` | Computer deleted |  -  |

### Deployment Events

| Type | Description | Extra Fields |
|------|-------------|-------------|
| `deployment.build_started` | Build job enqueued | `version_id` |
| `deployment.succeeded` | Build completed and promoted | `url`, `version_id` |
| `deployment.failed` | Build or promotion failed | `error` |
| `deployment.rollback` | Version rolled back | `from_version`, `to_version` |

### Webhook Events

| Type | Description | Extra Fields |
|------|-------------|-------------|
| `webhook.delivered` | Outgoing webhook POST succeeded | `webhook_id`, `status_code` |
| `webhook.failed` | Outgoing webhook POST failed | `webhook_id`, `attempt`, `error` |

### Filtering with `types`

Pass a comma-separated list of glob patterns. The `*` wildcard matches any suffix segment:

```text
?types=sandbox.*           # all sandbox events
?types=sandbox.*,computer.running
?types=deployment.succeeded,deployment.failed
```

An empty `types` parameter (or no parameter) delivers all events.

---

## Reconnection

On disconnect, reconnect using the `Last-Event-ID` header set to the last `id` you received. The server will resume from that cursor, replaying up to 60 seconds of backlogged events.

```http
GET /api/v1/events/stream
Authorization: Bearer msk_...
Last-Event-ID: evt_01hwxyz...
```

---

## Examples

**cURL**

```bash
# All events
curl -N -H "Authorization: Bearer msk_u_..." \
  -H "Accept: text/event-stream" \
  "https://api.miosa.ai/api/v1/events/stream"

# Sandbox events only
curl -N -H "Authorization: Bearer msk_u_..." \
  -H "Accept: text/event-stream" \
  "https://api.miosa.ai/api/v1/events/stream?types=sandbox.*"
```

**Python**

```python
import httpx

headers = {
    "Authorization": "Bearer msk_u_...",
    "Accept": "text/event-stream",
}

with httpx.stream("GET", "https://api.miosa.ai/api/v1/events/stream",
                  params={"types": "sandbox.*"}, headers=headers) as r:
    for line in r.iter_lines():
        if line.startswith("data:"):
            import json
            event = json.loads(line[5:].strip())
            print(event["type"], event["id"])
```

**TypeScript**

```typescript
// Server-side (Node.js / Deno)  -  EventSource doesn't support custom headers in browsers
import EventSource from 'eventsource';

const es = new EventSource(
  'https://api.miosa.ai/api/v1/events/stream?types=sandbox.*',
  { headers: { Authorization: 'Bearer msk_u_...' } }
);

es.addEventListener('sandbox.ready', (e) => {
  const data = JSON.parse(e.data);
  console.log('Sandbox ready:', data.id);
});

es.addEventListener('sandbox.error', (e) => {
  const data = JSON.parse(e.data);
  console.error('Sandbox failed:', data.id, data.error);
});

es.onerror = () => es.close();
```
