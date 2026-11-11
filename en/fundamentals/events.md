---
slug: "fundamentals/events"
title: "Events (SSE)"
description: "Stream progress over Server-Sent Events using one-time tickets - per-resource streams, the tenant-wide stream, and the event catalog."
section: "API reference"
order: 12004
---

Several long-running operations stream progress over [Server-Sent Events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events). The SSE stream for a resource is always at:
```http

GET /api/v1/{resource-type}/{id}/events
```

Because the browser `EventSource` API does not support custom headers, SSE endpoints use a **one-time ticket** for authentication instead of the `Authorization` header:

**Python**

```python
# 1. Mint a ticket (server-side)
ticket = client.sandboxes.sse_ticket(sandbox_id="sbx_abc123")

# 2. Open the stream
import sseclient, urllib.request

url = f"https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/events?ticket={ticket.value}"
with urllib.request.urlopen(url) as resp:
    for event in sseclient.SSEClient(resp):
        print(event.data)
```

**TypeScript**

```typescript
// 1. Mint a ticket (server-side)
const ticket = await miosa.sandboxes.sseTicket('sbx_abc123');

// 2. Open the stream (browser or Node)
const url = `https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/events?ticket=${ticket.value}`;
const source = new EventSource(url);
source.onmessage = (e) => console.log(JSON.parse(e.data));
```

**cURL**

```bash
# 1. Mint a ticket
TICKET=$(curl -s -X POST \
  https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/sse-ticket \
  -H "Authorization: Bearer msk_u_..." | jq -r '.ticket')

# 2. Stream events
curl -N "https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/events?ticket=$TICKET"
```

Event types vary by resource. Common event shapes:
```jsonc
// Build log line
{ "type": "log", "stream": "stdout", "data": "Installing dependencies...", "ts": 1748390460 }

// Status change
{ "type": "status_changed", "previous": "building", "current": "ready", "ts": 1748390462 }

// Error
{ "type": "error", "code": "BUILD_FAILED", "message": "Exit code 1", "ts": 1748390465 }
```

## Sandbox lifecycle events

| Event | When |
|---|---|
| `sandbox.created` | Sandbox resource created (not yet booted) |
| `sandbox.ready` | Boot complete, exec and file endpoints available |
| `sandbox.error` | Fatal boot or runtime error |
| `sandbox.destroyed` | Sandbox stopped and resources released |
| `sandbox.share.created` | A share token was minted for the sandbox |
| `sandbox.share.revoked` | A share token was revoked |

## Deployment lifecycle events

| Event | When |
|---|---|
| `deployment.created` | Deployment object created |
| `deployment.build_started` | Builder VM booted, build in progress |
| `deployment.build_succeeded` | Release artifact ready |
| `deployment.build_failed` | Build exited non-zero |
| `deployment.published` | Active version updated, traffic cut over |
| `deployment.rollback` | Active version reset to a prior version |

## Tenant-wide SSE stream

To receive all lifecycle events for your organization without polling per-resource endpoints, subscribe to the tenant stream:

```bash
# Mint a ticket
TICKET=$(curl -s -X POST https://api.miosa.ai/api/v1/events/ticket \
  -H "Authorization: Bearer msk_u_..." | jq -r '.ticket')

# Stream all org events
curl -N "https://api.miosa.ai/api/v1/events?ticket=$TICKET"
```

Each event on the tenant stream includes a `resource_type` and `resource_id` field so you can route to the right handler:
```jsonc
{
  "type": "sandbox.ready",
  "resource_type": "sandbox",
  "resource_id": "sbx_abc123",
  "external_user_id": "alice@example.net",
  "ts": 1748390462
}
```

See [Events (SSE) API Reference](/docs/api-reference/events) and [Tenant Events](/docs/api-reference/tenant-events) for the complete event catalog and filtering options.
