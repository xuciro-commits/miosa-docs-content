---
slug: "api-reference/errors"
title: "Error Codes"
description: "Every error code the MIOSA API returns, what it means, and how to handle it."
section: "API reference"
order: 12011
---

Most MIOSA API errors follow a single envelope shape, `{error: {code, message, request_id}}`. The HTTP status indicates the broad class; the `code` field is the machine-readable reason. A few families differ: authentication failures return a flat `{error: {type, code}}` shape, and snapshot errors can return a flat message string.

```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "sandbox not found",
    "request_id": "req_01jv..."
  }
}
```

Include `request_id` whenever you contact support. It correlates the request across all internal services.

---

## Authentication errors

Returned by the `ApiKeyAuth` plug before the request reaches any controller.

| Code | HTTP | Meaning | What to do |
|---|---|---|---|
| `invalid_token` | 401 | `Authorization` header missing, malformed, or the key/JWT is invalid | Verify the header is `Authorization: Bearer msk_...` |
| `invalid_token` | 401 | Refresh token used on a non-auth endpoint | Use a workspace API key (`msk_*`), not a refresh token |
| `insufficient_scope` | 403 | Key exists but lacks the required scope for this operation | Adjust key scopes in the dashboard |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests from this key or IP | Back off and retry after `Retry-After` seconds |

> **Caution**
>
> All API endpoints are rate-limited.
> Limits vary by key family, tenant policy, and endpoint family.
> Honor `X-RateLimit-*` and `Retry-After` instead of hardcoding a client-side limit.

---

## General resource errors

These codes appear across all resource types (computers, sandboxes, deployments, workspaces, etc.).

| Code | HTTP | Meaning | What to do |
|---|---|---|---|
| `NOT_FOUND` | 404 | Resource does not exist or belongs to a different tenant | Confirm the ID and that the key's tenant owns the resource |
| `FORBIDDEN` | 403 | Authenticated but not authorized to access this resource | Check membership or ownership |
| `INVALID_ID` | 400 | Path parameter is not a valid UUID | Use a UUID v4 ID, not a slug or name |
| `MISSING_PARAM` | 400 | A required request body field is absent | Check the field name in the request body schema |
| `VALIDATION_ERROR` | 422 | One or more fields failed schema validation | Inspect `details` for per-field messages |
| `VALIDATION_FAILED` | 422 | Changeset validation failed (alias of `VALIDATION_ERROR` in some controllers) | Inspect `details` |
| `TENANT_RESOLUTION_FAILED` | 422 | Cannot determine the tenant for the authenticated identity | The API key may belong to a workspace that has been deleted or suspended |
| `INTERNAL_ERROR` | 500 | Unexpected server-side error | Retry with exponential backoff; report if persistent |

---

## Sandbox errors

| Code | HTTP | Meaning | What to do |
|---|---|---|---|
| `INSUFFICIENT_CREDITS` | 402 | Not enough credits to provision this sandbox | Top up credits in the dashboard or reduce the resource request |
| `SANDBOX_LIMIT_REACHED` | 429 | Tenant has hit the concurrent sandbox cap for its plan (Developer 10, Business 250, Enterprise 500) | Destroy idle sandboxes, upgrade the plan, or contact support to raise the limit |
| `SANDBOX_NOT_RUNNING` | 409 | Operation requires the sandbox to be in `running` state | Wait until state is `running`; poll `GET /sandboxes/{id}` or subscribe to events |
| `AGENT_UNAVAILABLE` | 502 | The computer command service is reachable but not responding | Wait and retry; if persistent, destroy and recreate the sandbox |
| `INVALID_TEMPLATE` | 400 | `template_id` is not a recognized template | Use `GET /api/v1/sandbox-templates` to list valid IDs |
| `MISSING_PATH` | 400 | `path` field is required but absent | Include `path` in the request body |
| `INVALID_PATH` | 400 | `path` is not a valid absolute filesystem path | Use an absolute path (starts with `/`) |
| `MISSING_FILE` | 400 | Multipart upload missing the `file` part | Include the file part in the multipart form |
| `INVALID_BODY` | 400 | Request body could not be parsed or decoded | Verify JSON or base64 encoding |
| `MISSING_MODE` | 400 | `chmod` call missing `mode` field | Include an octal mode string, e.g. `"0644"` |
| `INVALID_PORT` | 422 | Expose port is outside `1`-`65535` | Use a port in the valid range |
| `MISSING_PARAM` | 400 | `port` omitted on `/expose` and template has no default preview port | Provide `port` explicitly |
| `SANDBOX_NOT_RUNNING` | 409 | `/deploy` called but sandbox is not running | Confirm sandbox state before promoting |
| `SANDBOX_PLACEMENT_UNCERTAIN` | 409 | A create without an `Idempotency-Key` could not confirm its placement outcome, so the sandbox may or may not exist | List your sandboxes to reconcile before creating another; retrying without a key risks a duplicate |
| `SANDBOX_PLACEMENT_RECONCILING` | 503 | A create that carried an `Idempotency-Key` could not confirm its placement outcome yet | Retry with the same `Idempotency-Key` after `Retry-After` seconds; the retry resolves to the same sandbox |
| `INVALID_SANDBOX_RUNTIME_PROFILE` | 422 | `runtime_profile` is not `standard`, `soma`, or `auto` | Send one of the allowed values; `details.allowed` names them |
| `SOMA_RUNTIME_UNSUPPORTED_REQUEST` | 422 | The requested shape is above the runtime's maximum, or the runtime cannot serve it | Request a smaller shape, or use `runtime_profile "standard"`; `details.max_shape` names the ceiling |

> **Note: Idempotent creates retry safely**
>
> When a sandbox create cannot confirm its placement outcome, the answer depends on whether you sent an `Idempotency-Key` header.
>
> - With a key, the API returns a retryable `503 SANDBOX_PLACEMENT_RECONCILING` with `Retry-After`. Retry with the same key and it resolves to the same sandbox.
> - Without a key, the API returns a terminal `409 SANDBOX_PLACEMENT_UNCERTAIN`. List your sandboxes to reconcile before creating another, since a retry could create a duplicate.
>
> Send an `Idempotency-Key` on every create so an uncertain outcome is retryable instead of terminal.

---

## Sandbox template build errors

| Code | HTTP | Meaning | What to do |
|---|---|---|---|
| `TEMPLATE_NOT_FOUND` | 404 | Template ID does not exist | Verify the template ID |
| `BUILD_NOT_FOUND` | 404 | Build record does not exist | Verify the build ID |
| `BUILD_ALREADY_TERMINAL` | 409 | Cannot cancel a build that is already `ready`, `failed`, or `cancelled` | No action needed; build is in a terminal state |
| `BUILD_NOT_RETRYABLE` | 409 | Build is not in a state that allows retry (`failed` or `cancelled` only) | Only `failed` or `cancelled` builds may be retried |
| `UNAUTHORIZED` | 401 | SSE ticket missing, invalid, or expired | Re-issue a ticket via `POST /auth/sse-ticket` and reconnect |

---

## Computer errors

| Code | HTTP | Meaning | What to do |
|---|---|---|---|
| `INSUFFICIENT_CREDITS` | 402 | Not enough credits to start this computer | Top up credits |
| `COMPUTER_NOT_RUNNING` | 409 | Desktop or exec operation requires a running computer | Start the computer first |
| `COMPUTER_NOT_FOUND` | 404 | Computer does not exist in this tenant | Verify the computer ID |
| `AGENT_UNAVAILABLE` | 502 | computer command service unreachable | Computer may be starting; retry after a few seconds |

---

## Deployment errors

| Code | HTTP | Meaning | What to do |
|---|---|---|---|
| `DEPLOYMENT_NOT_FOUND` | 404 | Deployment ID does not exist | Verify the deployment ID |
| `VALIDATION_ERROR` | 422 | Deployment fields failed validation | Check `details` for field-level messages |
| `SANDBOX_NOT_RUNNING` | 409 | Publishing requires a running sandbox | Confirm sandbox state |
| `RUNTIME_TARGET_UNAVAILABLE` | 409 | Sandbox has no IP yet  -  cannot route to it | Retry after a few seconds |
| `RUNTIME_LOGS_UNAVAILABLE` | 502 | Cannot fetch runtime logs from the sandbox | Retry or check sandbox health |
| `EMPTY_ARTIFACT` | 422 | Publish output contained no files | Verify your build step produces output files |
| `SERVICE_UNAVAILABLE` | 503 | A required internal service (e.g. encryption) is not configured | Contact support |

---

## Snapshot errors

Direct snapshot create, list, restore, and delete operations apply to desktop Computers under `/api/v1/computers/{id}/snapshots`.
Canonical sandbox V1 exposes snapshot-backed fork at `/api/v1/sandboxes/{id}/fork`, not customer-managed snapshot IDs.

| Code | HTTP | Meaning | What to do |
|---|---|---|---|
| `NOT_FOUND` (snapshot) | 404 | Snapshot does not exist or belongs to a different computer | Verify snapshot ID and computer ID |
| `FORBIDDEN` | 403 | Authenticated tenant does not own this snapshot | Check ownership |
| `already_deleted` | 409 | Snapshot has already been deleted | No action; already in terminal state |
| `snapshot is not in ready state` | 409 | Restore attempted before snapshot reached `ready` | Poll snapshot status until `ready` |
| `missing sse ticket` | 401 | SSE stream opened without a ticket | Issue a ticket via `POST /auth/sse-ticket` first |

---

## Custom domain errors

| Code | HTTP | Meaning | What to do |
|---|---|---|---|
| `DOMAIN_NOT_FOUND` | 404 | Custom domain record not found | Verify the domain ID |
| `COMPUTER_NOT_FOUND` | 404 | The computer this domain is attached to was not found | Verify the computer ID |
| `FORBIDDEN` | 403 | Domain belongs to a different tenant | Verify ownership |
| `VALIDATION_FAILED` | 422 | `fqdn` missing or invalid | Include a valid fully-qualified domain name |

---

## Workspace errors

| Code | HTTP | Meaning | What to do |
|---|---|---|---|
| `VALIDATION_FAILED` | 422 | Workspace creation or update fields are invalid | Inspect `details` |
| `TEMPLATE_NAME_TAKEN` | 409 | A template with that name already exists in this workspace | Choose a different name |

---

## Rate-limit and quota errors

| Code | HTTP | Meaning | What to do |
|---|---|---|---|
| `RATE_LIMIT_EXCEEDED` | 429 | General API rate limit hit | Respect the `Retry-After` response header |
| `SANDBOX_LIMIT_REACHED` | 429 | Concurrent sandbox cap reached | Destroy idle sandboxes or upgrade plan |
| `INSUFFICIENT_CREDITS` | 402 | Credit balance depleted | Purchase credits from the billing dashboard |

> **Caution**
>
> All 429 responses include a `Retry-After` header (seconds). Honor it  -  repeated violations may result in temporary IP blocks.

---

## See also

- [API Reference overview](/docs/api-reference)  -  base URL, auth, and request format
- [Sandboxes](/docs/api-reference/sandboxes)  -  sandbox lifecycle and file operations
- [Computers](/docs/api-reference/computers)  -  computer CRUD and desktop control
- [Deployments](/docs/api-reference/deployments)  -  deployment and version errors
