---
slug: "api-reference/webhooks"
title: "Webhooks API"
description: "Subscribe to platform lifecycle events via outgoing HTTP callbacks with HMAC signature verification."
section: "API reference"
order: 12050
---

Webhooks fire outgoing HTTP POST requests to your endpoint when platform events occur  -  sandbox lifecycle changes, deployment completions, billing events, and more. Each delivery is signed with HMAC-SHA256 so you can verify authenticity.

Base path: `/api/v1/webhooks`

> **Note**
>
> The webhook `secret` is returned **only once**  -  in the response to `POST /webhooks`. Store it immediately in a secure location. It cannot be retrieved later (only a masked preview is shown afterward).

---

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/webhooks` | List all webhooks |
| `POST` | `/api/v1/webhooks` | Create a webhook |
| `GET` | `/api/v1/webhooks/{id}` | Get a webhook |
| `PATCH` | `/api/v1/webhooks/{id}` | Update a webhook |
| `DELETE` | `/api/v1/webhooks/{id}` | Delete a webhook |
| `POST` | `/api/v1/webhooks/{id}/test` | Send a test delivery |
| `GET` | `/api/v1/webhooks/{id}/deliveries` | List delivery attempts |

---

## List Webhooks

**`GET /api/v1/webhooks`**

### Auth

```http
Authorization: Bearer msk_...
```

### Response  -  `200 OK`

```json
{
  "data": [
    {
      "id": "wh_01hwxyz...",
      "name": "My Webhook",
      "url": "https://api.myapp.com/hooks/miosa",
      "events": ["sandbox.created", "sandbox.destroyed"],
      "secret_preview": "sk_live_••••••••abcd",
      "retry_count": 3,
      "created_at": "2026-05-01T00:00:00Z",
      "updated_at": "2026-05-01T00:00:00Z"
    }
  ]
}
```

---

## Create a Webhook

**`POST /api/v1/webhooks`**

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `url` | string | Yes | HTTPS endpoint to POST events to |
| `name` | string | No | Human-readable label |
| `events` | string[] | No | Event type filter; empty = all events |
| `headers` | object | No | Extra headers to include in deliveries |
| `retry_count` | integer | No | Consecutive failed deliveries before the webhook is marked `failed` (0-10, default `3`) |
| `metadata` | object | No | Arbitrary key-value metadata |

```json
{
  "url": "https://api.myapp.com/hooks/miosa",
  "name": "Sandbox lifecycle",
  "events": ["sandbox.created", "sandbox.ready", "sandbox.error", "sandbox.destroyed"],
  "retry_count": 3
}
```

### Response  -  `201 Created`

```json
{
  "data": {
    "id": "wh_01hwxyz...",
    "name": "Sandbox lifecycle",
    "url": "https://api.myapp.com/hooks/miosa",
    "events": ["sandbox.created", "sandbox.ready", "sandbox.error", "sandbox.destroyed"],
    "secret": "sk_live_abc123def456...",
    "secret_preview": "sk_live_••••••••3456",
    "retry_count": 3,
    "created_at": "2026-05-26T10:14:23Z",
    "updated_at": "2026-05-26T10:14:23Z"
  }
}
```

The `secret` field appears only in this response. Save it securely.

---

## Update a Webhook

**`PATCH /api/v1/webhooks/{id}`**

Partial update  -  include only fields you want to change.

### Request Body

Same fields as create, all optional.

### Response  -  `200 OK`

Webhook object without the `secret` field.

---

## Delete a Webhook

**`DELETE /api/v1/webhooks/{id}`**

### Response  -  `204 No Content`

---

## Send a Test Delivery

**`POST /api/v1/webhooks/{id}/test`**

Sends a synthetic `webhook.test` event to the configured URL to verify connectivity and signature verification.

The attempt is recorded in the delivery log like any other delivery, marked as a test, so what the endpoint received is always visible at `GET /api/v1/webhooks/{id}/deliveries`. A test does not count toward `retry_count` and never moves the webhook to `failed`: a test against an endpoint that is temporarily down must not pause a live subscription.

### Response  -  `200 OK`

```json
{
  "delivered": true,
  "status_code": 200,
  "duration_ms": 143
}
```

---

## List Deliveries

**`GET /api/v1/webhooks/{id}/deliveries`**

Returns recent delivery attempts with status codes and timing.

### Response  -  `200 OK`

```json
{
  "data": [
    {
      "id": "del_01hwyza...",
      "event_type": "sandbox.ready",
      "status": "delivered",
      "status_code": 200,
      "attempt": 1,
      "duration_ms": 98,
      "delivered_at": "2026-05-26T10:14:24Z"
    },
    {
      "id": "del_01hwyzb...",
      "event_type": "sandbox.created",
      "status": "failed",
      "status_code": 500,
      "attempt": 3,
      "duration_ms": 5012,
      "delivered_at": "2026-05-26T10:14:20Z"
    }
  ]
}
```

---

## Signature Verification

Every delivery includes an `X-Miosa-Signature` header. Verify it on your server before trusting the payload:

```http
X-Miosa-Signature: sha256=<hex_digest>
```

Compute the expected signature:

```text
hmac = HMAC-SHA256(secret, raw_request_body_bytes)
expected = "sha256=" + hex(hmac)
assert constant_time_equals(expected, request.headers["X-Miosa-Signature"])
```

> **Note**
>
> Always compare signatures using a constant-time function to prevent timing attacks. Never use `==` for string comparison.

### Example verification (Python)

```python
import hmac, hashlib

def verify_signature(secret: str, body: bytes, header: str) -> bool:
    expected = "sha256=" + hmac.new(
        secret.encode(), body, hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, header)
```

### Example verification (TypeScript)

```typescript
import { createHmac, timingSafeEqual } from 'crypto';

function verifySignature(secret: string, body: Buffer, header: string): boolean {
  const expected = 'sha256=' + createHmac('sha256', secret).update(body).digest('hex');
  return timingSafeEqual(Buffer.from(expected), Buffer.from(header));
}
```

---

## Retry Policy

Failed deliveries are retried on a fixed backoff schedule:

| Attempt | Delay |
|---------|-------|
| 1 (initial) | Immediate |
| 2 | 1 minute |
| 3 | 5 minutes |
| 4 | 30 minutes |
| 5 | 1 hour |
| 6 | 2 hours |
| 7 | 4 hours |
| 8 | 8 hours |

MIOSA makes up to eight delivery attempts over roughly 15.5 hours (within a 24-hour window), then drops the event.

A delivery is considered failed if your endpoint returns a non-2xx status code or does not respond within 10 seconds. `retry_count` (0-10, default 3) is the number of consecutive failed deliveries after which MIOSA marks the webhook `failed`; it is not the per-delivery retry cap. Set it to `0` to mark the webhook failed on the first failed delivery.

---

## Subscribed Event Types

`GET /api/v1/webhooks/events` returns this list, grouped by product surface. It is the same list the API validates a subscription against, so an event that is not here cannot be subscribed to.

| Event | Description |
|-------|-------------|
| `sandbox.created` | Sandbox created |
| `sandbox.starting` | Sandbox boot started |
| `sandbox.ready` | Sandbox became usable: `ready` turned true, whether that was decided on boot or by a readiness probe |
| `sandbox.started` | Sandbox runtime is up and reachable |
| `sandbox.paused` | Sandbox left `running` for `paused` |
| `sandbox.resumed` | A paused sandbox came back up |
| `sandbox.idle_paused` | Sandbox was paused because it sat idle past its timeout |
| `sandbox.error` | Sandbox failed to start or crashed |
| `sandbox.destroyed` | Sandbox deleted |
| `computer.created` | Computer created |
| `computer.started` | Computer finished booting and is usable |
| `computer.stopped` | Computer was asked to stop |
| `computer.deleted` | Computer destroyed |
| `database.created` | Database provisioned |
| `database.ready` | Database accepting connections |
| `database.stopped` | Database stopped |
| `database.deleted` | Database deleted |
| `managed_database.provisioning` | Managed database provisioning started |
| `managed_database.running` | Managed database running |
| `managed_database.failed` | Managed database provisioning failed |
| `managed_database.destroyed` | Managed database destroyed |
| `managed_service.provisioning` | Managed service provisioning started |
| `managed_service.running` | Managed service running |
| `managed_service.failed` | Managed service provisioning failed |
| `managed_service.destroyed` | Managed service destroyed |
| `deployment.created` | Deployment created |
| `deployment.building` | Build started |
| `deployment.build_started` | Build step began executing |
| `deployment.build_succeeded` | Build produced an artifact |
| `deployment.build_failed` | Build failed |
| `deployment.ready` | Deployment serving traffic |
| `deployment.error` | Deployment failed or crashed |
| `deployment.canceled` | Deployment canceled |
| `deployment.rolled_back` | Deployment rolled back to a previous version |
| `preview.created` | Pull-request preview created |
| `preview.active` | Preview serving traffic |
| `preview.destroyed` | Preview torn down |
| `health_check.healthy` | Endpoint health check passed |
| `health_check.unhealthy` | Endpoint health check failed |
| `cron.execution_started` | Scheduled job started |
| `cron.execution_completed` | Scheduled job finished |
| `cron.execution_failed` | Scheduled job failed |
| `billing.credits_low` | Credit balance below the configured threshold |
| `billing.credits_exhausted` | Credit balance reached zero |
| `billing.sandbox.spend_threshold_reached` | Sandbox spend crossed its threshold |
| `run.created` | Agent run created |
| `run.queued` | Agent run queued |
| `run.running` | Agent run started |
| `run.progress` | Agent run reported progress |
| `run.succeeded` | Agent run succeeded |
| `run.failed` | Agent run failed |
| `run.canceled` | Agent run canceled |
| `run.file_ready` | A file produced by an agent run is ready |
| `forge.push` | A branch in a Forge repository received commits |
| `forge.pull_request` | A Forge pull request was opened, reopened, closed or merged |
| `forge.check` | A check reported a result on a Forge commit |

`forge.*` payloads carry `repository_id`, `repository_slug` and `organization_slug`, so one subscription can route each event to the right repository.

`webhook.test` is the synthetic event sent by `POST /api/v1/webhooks/:id/test`. It is not subscribable and appears in the delivery log as a test delivery.

Subscribe to all events by omitting the `events` field. Subscribe to a subset by listing specific event types.

---

## Examples

**cURL**

```bash
# Create
curl -X POST https://api.miosa.ai/api/v1/webhooks \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://api.myapp.com/hooks/miosa",
    "events": ["sandbox.created", "sandbox.destroyed"]
  }'

# Test
curl -X POST https://api.miosa.ai/api/v1/webhooks/wh_01hwxyz/test \
  -H "Authorization: Bearer msk_u_..."

# Deliveries
curl https://api.miosa.ai/api/v1/webhooks/wh_01hwxyz/deliveries \
  -H "Authorization: Bearer msk_u_..."
```

**Python**

```python
import miosa

client = miosa.Miosa()

hook = client.webhooks.create(
    url="https://api.myapp.com/hooks/miosa",
    events=["sandbox.created", "sandbox.destroyed"],
    retry_count=3,
)

# Save hook.secret securely  -  only available now
print(hook.secret)

# Test
result = client.webhooks.test(hook.id)
print(result.delivered, result.status_code)
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

const hook = await client.webhooks.create({
  url: 'https://api.myapp.com/hooks/miosa',
  events: ['sandbox.created', 'sandbox.ready', 'sandbox.destroyed'],
  retryCount: 3,
});

// Store hook.secret in your secrets manager  -  only returned once
console.log(hook.secret);

// Test connectivity
const result = await client.webhooks.test(hook.id);
console.log(result.delivered, result.statusCode);
```
