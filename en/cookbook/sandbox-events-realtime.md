---
slug: "cookbook/sandbox-events-realtime"
title: "Sandbox events in real time"
description: "React to sandbox lifecycle events using either HMAC-signed webhooks (server-to-server) or the SSE event stream (in-process). Covers both patterns, idempotency, and when to choose each."
section: "Platform builders"
order: 9017
---

`~10 min` `Python` `TypeScript`

> **Goal**: React to sandbox lifecycle events  -  `sandbox.running`, `sandbox.destroyed`, `deployment.live`, and others  -  using either webhooks (durable, server-to-server) or the SSE stream (ephemeral, in-process).
>
> **What you'll use**: Webhooks, Tenant SSE event stream

## Two patterns  -  choose one

| | Webhooks | SSE stream |
|---|---|---|
| Delivery | MIOSA calls your URL | Your process pulls |
| Durability | Retried on failure | Lost if process dies |
| Auth | HMAC-SHA256 signature | Your `msk_*` key |
| Latency | ~100ms | ~50ms |
| Best for | Background jobs, billing triggers, Slack alerts | Dashboard UIs, dev-time debugging |

Both patterns carry the same event payloads. You can use both simultaneously  -  webhooks for the durable server-side reaction, SSE for the live frontend widget.

## Pattern A  -  Webhooks

### Register a webhook endpoint

**Python**

```python
import os
from miosa import Miosa

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

hook = client.webhooks.create(
    url="https://api.example.net/webhooks/miosa",
    events=[
        "sandbox.running",
        "sandbox.destroyed",
        "sandbox.error",
        "deployment.live",
        "deployment.failed",
    ],
    description="Acme sandbox lifecycle",
)

# IMPORTANT: save hook["secret"] immediately  -  it is shown only once
print(f"Webhook ID: {hook['id']}")
print(f"Secret (save this): {hook.get('secret')}")
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

const hook = await client.webhooks.create({
  url: 'https://api.example.net/webhooks/miosa',
  events: [
    'sandbox.running',
    'sandbox.destroyed',
    'sandbox.error',
    'deployment.live',
    'deployment.failed',
  ],
  description: 'Acme sandbox lifecycle',
});

// IMPORTANT: save hook.secret immediately  -  shown only once
console.log('Webhook ID:', hook.id);
console.log('Secret (save this):', (hook as any).secret);
```

### Verify the signature

Every delivery includes a `Miosa-Signature` header: `t=<unix-timestamp>,v1=<hex-digest>`. The signed payload is `<timestamp>.<raw-body>`.

**Python (FastAPI)**

```python
import hashlib, hmac, time, json
from fastapi import FastAPI, Request, HTTPException

app = FastAPI()
WEBHOOK_SECRET = os.environ["MIOSA_WEBHOOK_SECRET"]

def verify_signature(body: bytes, header: str, secret: str) -> bool:
    """Return True if the signature is valid and not a replay."""
    parts = dict(item.split("=", 1) for item in header.split(","))
    ts = parts.get("t", "")
    sig = parts.get("v1", "")

    # Replay guard: reject events older than 5 minutes
    if abs(time.time() - int(ts)) > 300:
        return False

    expected = hmac.new(
        secret.encode(),
        f"{ts}.{body.decode()}".encode(),
        hashlib.sha256,
    ).hexdigest()

    return hmac.compare_digest(expected, sig)

@app.post("/webhooks/miosa")
async def receive_webhook(request: Request):
    body = await request.body()
    header = request.headers.get("Miosa-Signature", "")

    if not verify_signature(body, header, WEBHOOK_SECRET):
        raise HTTPException(400, "Invalid signature")

    event = json.loads(body)
    event_type = event.get("type")

    # Idempotency: check event["id"] against your DB before processing
    event_id = event.get("id")

    if event_type == "sandbox.running":
        sandbox_id = event.get("data", {}).get("id")
        print(f"Sandbox {sandbox_id} is running  -  start your downstream job")

    elif event_type == "sandbox.destroyed":
        sandbox_id = event.get("data", {}).get("id")
        print(f"Sandbox {sandbox_id} destroyed  -  record billing event")

    elif event_type == "deployment.live":
        url = event.get("data", {}).get("url")
        print(f"Deployment live at {url}  -  notify user")

    # Always return 200 quickly; do heavy work asynchronously
    return {"ok": True}
```

**TypeScript (Express)**

```typescript
import express from 'express';
import { createHmac, timingSafeEqual } from 'crypto';

const app = express();
app.use(express.raw({ type: 'application/json' })); // raw body for signature check

const WEBHOOK_SECRET = process.env.MIOSA_WEBHOOK_SECRET!;

function verifySignature(body: Buffer, header: string, secret: string): boolean {
  const parts = Object.fromEntries(header.split(',').map(p => p.split('=')));
  const { t: ts, v1: sig } = parts;
  if (!ts || !sig) return false;

  // Replay guard: reject events older than 5 minutes
  if (Math.abs(Date.now() / 1000 - Number(ts)) > 300) return false;

  const expected = createHmac('sha256', secret)
    .update(`${ts}.${body.toString()}`)
    .digest('hex');

  return timingSafeEqual(Buffer.from(expected, 'hex'), Buffer.from(sig, 'hex'));
}

app.post('/webhooks/miosa', (req, res) => {
  const body = req.body as Buffer;
  const header = req.headers['miosa-signature'] as string ?? '';

  if (!verifySignature(body, header, WEBHOOK_SECRET)) {
    res.status(400).json({ error: 'Invalid signature' });
    return;
  }

  const event = JSON.parse(body.toString());
  const { type: eventType, id: eventId, data } = event;

  // Idempotency: check eventId against your DB before processing

  switch (eventType) {
    case 'sandbox.running':
      console.log(`Sandbox ${data?.id} is running  -  start downstream job`);
      break;
    case 'sandbox.destroyed':
      console.log(`Sandbox ${data?.id} destroyed  -  record billing event`);
      break;
    case 'deployment.live':
      console.log(`Deployment live at ${data?.url}  -  notify user`);
      break;
  }

  // Always return 200 quickly
  res.json({ ok: true });
});
```

### Idempotency

MIOSA retries webhook delivery on failure (5xx, timeout). Your handler must be idempotent. The simplest approach is to persist `event.id` in a seen-events table before processing:

**Python**

```python
import sqlite3

conn = sqlite3.connect("/var/lib/acme/events.db")
conn.execute(
    "CREATE TABLE IF NOT EXISTS seen_events (event_id TEXT PRIMARY KEY)"
)

def process_event(event: dict):
    event_id = event["id"]
    try:
        conn.execute("INSERT INTO seen_events VALUES (?)", (event_id,))
        conn.commit()
    except sqlite3.IntegrityError:
        # Already processed  -  skip
        return

    # Safe to process now
    handle_event(event)
```

**TypeScript**

```typescript
const seenEvents = new Set<string>(); // use Redis or a DB table in production

function processEvent(event: Record<string, unknown>) {
  const eventId = String(event.id ?? '');
  if (seenEvents.has(eventId)) return; // already processed
  seenEvents.add(eventId);

  handleEvent(event);
}
```

## Pattern B  -  SSE event stream

Use the tenant SSE stream for in-process listeners  -  dashboard backends, dev-time monitoring, or any scenario where you want to react inside your running process rather than receive callbacks.

**Python**

```python
import os
from miosa import Miosa

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

# Stream all tenant-scoped events  -  runs until cancelled
for event in client.events.stream():
    event_type = event.get("type", "")

    if event_type == "sandbox.running":
        print(f"[running]   {event['data']['id']}")
    elif event_type == "sandbox.destroyed":
        print(f"[destroyed] {event['data']['id']}")
    elif event_type == "sandbox.error":
        print(f"[error]     {event['data']['id']}  -  {event['data'].get('error')}")
    elif event_type.startswith("deployment."):
        print(f"[deploy]    {event_type}  -  {event.get('data', {}).get('url', '')}")
```

**TypeScript**

```typescript
import { Miosa, TenantEvents } from '@miosa/sdk';

const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

// Stream all tenant-scoped events
// (TenantEvents is available as a standalone resource)
const eventStream = new TenantEvents((client as any).http);

for await (const event of eventStream.stream()) {
  const type = event.type ?? '';

  if (type === 'sandbox.running') {
    console.log(`[running]   ${(event as any).data?.id}`);
  } else if (type === 'sandbox.destroyed') {
    console.log(`[destroyed] ${(event as any).data?.id}`);
  } else if (type === 'sandbox.error') {
    console.log(`[error]     ${(event as any).data?.id}  -  ${(event as any).data?.error}`);
  } else if (type.startsWith('deployment.')) {
    console.log(`[deploy]    ${type}  -  ${(event as any).data?.url ?? ''}`);
  }
}
```

### Filter by event type

The stream accepts a `types` filter to reduce noise:

**Python**

```python
# Receive only deployment events
for event in client.events.stream(types=["deployment.live", "deployment.failed"]):
    print(event)
```

**TypeScript**

```typescript
for await (const event of eventStream.stream(['deployment.live', 'deployment.failed'])) {
  console.log(event);
}
```

## Event catalog

| Event type | Trigger |
|---|---|
| `sandbox.running` | Sandbox finished booting |
| `sandbox.paused` | Sandbox suspended (idle timeout or manual pause) |
| `sandbox.destroyed` | Sandbox deleted |
| `sandbox.error` | Unrecoverable sandbox failure |
| `sandbox.snapshot.ready` | Snapshot upload completed |
| `sandbox.share.created` | Preview share token minted |
| `sandbox.share.revoked` | Preview share token revoked |
| `deployment.live` | Deployment promoted to live |
| `deployment.failed` | Deployment build or promotion failed |

## Choosing between patterns

Use **webhooks** when:

- You need durability (your process may restart)
- You're triggering background jobs, billing events, or Slack notifications
- You want MIOSA to push rather than your process to pull

Use the **SSE stream** when:

- You're building a live dashboard widget
- You're writing a short-lived script or dev tool
- You want the simplest possible integration with no webhook endpoint to deploy

## Related

- [Agent builds an app](/docs/cookbook/agent-builds-app): Use events to update your UI as the agent builds and deploys.
- [Per-user quotas and billing](/docs/cookbook/per-user-quotas-billing): Trigger billing actions on sandbox.destroyed events.
- [Webhooks API reference](/docs/api-reference/webhooks): Full event catalog, delivery semantics, retry policy, and signature spec.
