---
slug: "develop/sandboxes/events"
title: "Sandbox events"
description: "Subscribe to sandbox lifecycle and activity events over SSE to monitor agent runs and detect unexpected exits."
section: "Sandboxes"
order: 2008
---

The `events.stream()` endpoint emits lifecycle and activity SSE events for a
sandbox, which is useful for monitoring agent runs.

**Python**

```python
for event in sbx.events.stream():
    print(event["type"], event.get("data"))
    # types: "sandbox.started", "sandbox.destroyed", "exec.started", "exec.completed", ...
```

**TypeScript**

```typescript
for await (const event of sbx.events.stream()) {
  console.log(event.type, event.data);
}
```

**cURL**

```bash
curl https://api.miosa.ai/api/v1/sandboxes/$SBX/events \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Accept: text/event-stream"
```

## Logs, metrics, and export

Every sandbox exposes three observability surfaces:

| Surface | Endpoint |
|---|---|
| Logs | `GET /api/v1/sandboxes/{id}/logs` |
| Per-sandbox metrics | `GET /api/v1/sandboxes/{id}/metrics` |
| Lifecycle event stream | `GET /api/v1/sandboxes/{id}/events` - the stream on this page |

A stopped sandbox's disk can be exported as an archive without resuming it.
`GET /api/v1/sandboxes/{id}/exports` reads the saved disk, so it works while the sandbox is `stopped` and does not start compute.

> **Note: Related**
>
> The [Events API reference](/docs/api-reference/events) documents the full event catalog and payload shapes, and the [production checklist](/docs/develop/sandboxes/production-checklist) shows how to wire the stream into retry logic.
