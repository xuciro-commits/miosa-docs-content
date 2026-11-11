---
slug: "api-reference/sandbox-env"
title: "Sandbox Env API"
description: "CRUD for per-sandbox environment variables, encrypted at rest."
section: "API reference"
order: 12024
---

The sandbox env API manages a key-value store of environment variables attached to a sandbox. Variables are encrypted at rest and injected into the sandbox process environment on start. The API never returns plaintext values  -  only key names and metadata.

Base path: `/api/v1/sandboxes/{id}/env`

> **Note**
>
> Variable names must match `[A-Z][A-Z0-9_]*`  -  uppercase letters, digits, and underscores, starting with a letter. This mirrors POSIX shell environment variable conventions.

---

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/sandboxes/{id}/env` | List all env vars (names only, no values) |
| `PUT` | `/api/v1/sandboxes/{id}/env` | Bulk set or update env vars |
| `DELETE` | `/api/v1/sandboxes/{id}/env/{key}` | Delete a single env var |

---

## List Env Vars

**`GET /api/v1/sandboxes/{id}/env`**

Returns all variable names. Values are never returned.

### Auth

Requires `sandboxes:read` scope.

```http
Authorization: Bearer msk_...
```

### Response  -  `200 OK`

```json
{
  "data": [
    { "key": "DATABASE_URL", "encrypted": true },
    { "key": "API_SECRET", "encrypted": true },
    { "key": "LOG_LEVEL", "encrypted": false }
  ]
}
```

---

## Set Env Vars

**`PUT /api/v1/sandboxes/{id}/env`**

Bulk upsert. Existing keys are updated; new keys are created. Keys not present in the request are left unchanged.

### Auth

Requires `sandboxes:write` scope.

```http
Authorization: Bearer msk_...
```

### Request Body

```json
{
  "vars": [
    { "key": "DATABASE_URL", "value": "postgres://...", "encrypted": true },
    { "key": "LOG_LEVEL", "value": "debug" }
  ]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `vars[].key` | string | Yes | Variable name (`[A-Z][A-Z0-9_]*`) |
| `vars[].value` | string | Yes | Variable value |
| `vars[].encrypted` | boolean | No | Store encrypted at rest (default: `true`) |

### Response  -  `200 OK`

```json
{
  "data": [
    { "key": "DATABASE_URL", "encrypted": true },
    { "key": "LOG_LEVEL", "encrypted": false }
  ]
}
```

---

## Delete Env Var

**`DELETE /api/v1/sandboxes/{id}/env/{key}`**

Removes a single variable.

### Response  -  `200 OK`

```json
{
  "deleted": true,
  "name": "LOG_LEVEL"
}
```

---

## Errors

| Status | Code | Cause |
|--------|------|-------|
| 400 | `INVALID_ID` | Sandbox ID is not a valid UUID |
| 400 | `invalid vars` | `vars` missing or a key fails name validation |
| 404 | `NOT_FOUND` | Sandbox does not exist or wrong tenant |
| 404 | `NOT_FOUND` | Key does not exist (on DELETE) |

---

## Examples

**cURL**

```bash
# Set
curl -X PUT https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/env \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{
    "vars": [
      {"key": "DATABASE_URL", "value": "postgres://user:pass@host/db"},
      {"key": "LOG_LEVEL", "value": "info"}
    ]
  }'

# List
curl https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/env \
  -H "Authorization: Bearer msk_u_..."

# Delete
curl -X DELETE https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/env/LOG_LEVEL \
  -H "Authorization: Bearer msk_u_..."
```

**Python**

```python
import miosa

client = miosa.Miosa()

# Set
client.sandboxes.env.set("sbx_abc123", {
    "DATABASE_URL": "postgres://user:pass@host/db",
    "LOG_LEVEL": "info",
})

# List (names only)
keys = client.sandboxes.env.list("sbx_abc123")

# Delete one
client.sandboxes.env.delete("sbx_abc123", "LOG_LEVEL")
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// Set
await client.sandboxes.env.set('sbx_abc123', {
  DATABASE_URL: 'postgres://user:pass@host/db',
  LOG_LEVEL: 'info',
});

// List
const { data } = await client.sandboxes.env.list('sbx_abc123');

// Delete
await client.sandboxes.env.delete('sbx_abc123', 'LOG_LEVEL');
```
