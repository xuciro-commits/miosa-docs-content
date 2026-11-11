---
slug: "fundamentals/requests"
title: "Requests, limits, and errors"
description: "Idempotency keys, rate limits and quota headers, and the MIOSA error envelope with its HTTP status codes."
section: "API reference"
order: 12003
---

## Idempotency

Every mutation endpoint (`POST`, `PUT`, `PATCH`, `DELETE`) accepts an optional `Idempotency-Key` header. Retrying a request with the same key is safe  -  MIOSA returns the original response without re-executing the operation.
```http
POST /api/v1/sandboxes
Authorization: Bearer msk_u_...
Idempotency-Key: create-sandbox-for-alice-2026-05-17
Content-Type: application/json

{ "template_id": "miosa-sandbox", "external_user_id": "alice" }
```

Idempotency keys are:

- Scoped to your workspace and the endpoint path.
- Valid for **24 hours** from first use.
- Stored regardless of whether the original request succeeded or failed.

If the first request timed out before you received a response, retry with the same key. You will receive the original result if the operation completed, or a `202 Accepted` if it is still in progress.

> **Tip: Key generation**
>
> Use a deterministic, collision-resistant key tied to your own business event  -  for example, `"create-sandbox-{your_request_id}"`. UUIDs or hash of the request body both work.

## Rate limits

Rate limits apply per API key, per workspace, and per endpoint family. Every response includes quota headers:
```http
X-RateLimit-Limit: 600
X-RateLimit-Remaining: 587
X-RateLimit-Reset: 1748390460
```

Current self-service defaults:

| Credential or endpoint family | Limit | Window |
|---|---|---|
| User API key | 600 requests | Per minute per key |
| Platform API key | 30,000 requests | Per minute per key |
| Auth (`/auth/login`, `/auth/refresh`) | 60 requests | Per minute per IP |

Tenant and endpoint-family policies can impose lower limits.
Treat the response headers as authoritative.

When you exceed a limit, the API returns `429 Too Many Requests` with a `Retry-After` header containing the number of seconds until your quota resets.

High-volume workloads (AI agent pipelines creating many sandboxes per minute) can request a quota increase from the [platform dashboard](https://miosa.ai/dashboard/workspace/quota) or by contacting support.

## Errors

All errors use a consistent envelope:
```json
{
  "error": {
    "code": "SANDBOX_NOT_FOUND",
    "message": "No sandbox with id sbx_abc123 exists in this workspace.",
    "request_id": "req_01hx9z..."
  }
}
```

| Field | Description |
|---|---|
| `code` | Machine-readable constant. Use this in your `switch` statements, not `message`. |
| `message` | Human-readable explanation. May change between API versions. |
| `request_id` | Include this when contacting support or filing a bug report. |

HTTP status codes map directly to error categories:

| Status | Category |
|---|---|
| `400` | Invalid request body or parameters |
| `401` | Missing or invalid credential |
| `403` | Valid credential, insufficient permissions |
| `404` | Resource not found (or not visible to your workspace) |
| `409` | Conflict  -  resource already exists, duplicate idempotency key with different body |
| `422` | Validation error  -  request parsed correctly but failed domain rules |
| `429` | Rate limited |
| `5xx` | MIOSA-side fault  -  safe to retry with exponential backoff |

The full error code catalog is in the [API Reference](/docs/api-reference).

- [API Reference](/docs/api-reference): Complete endpoint reference with request/response schemas and error codes.
