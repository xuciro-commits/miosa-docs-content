---
slug: "platform/api-keys"
title: "API Keys"
description: "User and platform API keys for personal tools, backend automation, and scoped MIOSA API access."
section: "Organization & billing"
order: 13002
---

An **API key** authenticates a person or backend service to MIOSA.
User keys are personal credentials.
Platform keys are tenant automation credentials and can optionally be restricted to one workspace.

## Create a key

From the dashboard: **Workspace → Settings → API Keys → New key**. Or via API:
```http
POST /api/v1/api-keys
```

**TypeScript**

```typescript
const key = await miosa.apiKeys.create({
  name: "Lumen backend prod",
  key_type: "platform",
  workspace_id: workspaceId,
  scopes: ["deployments:write", "sandboxes:write", "domains:write"],
})

console.log(key.key)  // shown once
// -> msk_p_a1b2c3d4...
```

**Python**

```python
from miosa import Miosa

miosa = Miosa(api_key="msk_u_...")

key = miosa.api_keys.create(
    name="Lumen backend prod",
    key_type="platform",
    workspace_id=workspace_id,
    scopes=["deployments:write", "sandboxes:write", "domains:write"],
)

print(key["key"])  # shown once
# -> msk_p_a1b2c3d4...

# List all keys (tokens are NOT shown in list responses)
for k in miosa.api_keys.list():
    print(k["id"], k["name"])

# Revoke a key
miosa.api_keys.delete(key["id"])
```

The plaintext `msk_*` value is returned **exactly once** at creation.
The dashboard shows only a prefix and last four characters thereafter.
The server stores only a SHA-256 hash.

If you lose a key, rotate  -  create a new one, swap it in, delete the old.

## Key families
```text

msk_<type>_<random>
```

- `msk_u_*` - user-bound keys for personal CLI and SDK use.
- `msk_p_*` - tenant platform keys for server-side automation.
- `msk_a_*` - operator-admin keys that are not created through the public self-service API.

Environment-labeled key prefixes are not part of the current self-service contract.
Dedicated test-mode billing isolation is planned.

For a server-side platform integration, use a `platform` key created by a tenant owner or administrator.
Never expose it to browser code.

## Scopes

Each key has a list of scopes. MIOSA enforces scopes on every mutation. Reads typically require the matching `:read` scope; writes require `:write`.

Common scopes:

| Scope | Allows |
|---|---|
| `sandboxes:read` | List, get sandboxes |
| `sandboxes:write` | Create, exec, files.write, destroy |
| `deployments:read` | List, get deployments, versions, domains |
| `deployments:write` | Create, publish, rollback, attach domains |
| `domains:write` | Create, verify, delete custom domains |
| `databases:write` | Provision and operate managed databases |
| `storage:write` | Provision and operate object storage |
| `billing:read` | Read billing and credit information |

For server-side platform integration, start with the exact resource scopes the backend needs.
User keys created without an explicit scope list receive the standard product scopes.
Platform keys are trusted tenant automation credentials, so only tenant owners and administrators can create them.

## Permission presets

A preset is a named set of scopes that the platform keeps, so you pick a name instead of maintaining a list.
`GET /api/v1/api-keys/presets` returns every preset with its scopes.
The dashboard offers **Read-only**, **Automation**, **CI**, **Full access** and **Custom** (choose each scope yourself).

```http
POST /api/v1/api-keys
{ "name": "Deploy pipeline", "preset": "ci" }
```

Send either `preset` or `scopes`, not both.
The platform resolves the preset to its current scopes when the key is created, and the key keeps those scopes afterwards.

**Full access** carries every permission.
Use it only for trusted server-side tools, and prefer a service account.
The dashboard offers it only to organization owners and admins.

For production, create keys under a service account.
A service-account key does not depend on one person's membership, and deleting the account revokes all of its keys.

## Rotation

Rotate keys on a schedule (we recommend every 90 days, or on incident). The flow:

1. Create a new key with the same scopes.
2. Roll out the new key to your backend (deploy + restart).
3. Confirm the new key works (recent activity logged with `key_id` of the new key).
### 4. Delete the old key

**TypeScript**

```typescript
// Step 1: create
const newKey = await miosa.apiKeys.create({ name: "...", scopes: [...] })

// Step 2-3: deploy, confirm.

// Step 4: revoke old
await miosa.apiKeys.delete(oldKeyId)
```

**Python**

```python
# Step 1: create
new_key = miosa.api_keys.create(name="...", scopes=[...])

# Step 2-3: deploy, confirm.

# Step 4: revoke old
miosa.api_keys.delete(old_key_id)
```

Deleting a key invalidates it immediately. Any in-flight requests using the old key will start getting 401.

## Rate limits

Each key has a server-provided request-per-minute limit.
The current defaults are 600 rpm for user keys and 30,000 rpm for platform keys, but tenant and endpoint-family limits can be more restrictive.

- Key ID (the key's own bucket)
- Tenant or organization
- Endpoint (publish has its own bucket, separate from list operations)

Limits scale with the plan. Headers in every response tell you where you stand:
```http

X-RateLimit-Limit: 600
X-RateLimit-Remaining: 547
X-RateLimit-Reset: 1715712060
```

429 responses include a `Retry-After` header. SDKs back off automatically.

## Audit

Every action authenticated by a key emits an audit event with:

- `api_key_id` (so you know which key did what)
- `actor` (the organization, plus the key's name)
- `workspace_id`, `project_id`, `external_workspace_id`, `external_user_id` (if passed or resolved in the request)
- The action (e.g. `deployments.publish`, `domains.create`)
- Timestamp + outcome

Read your audit log via:
```http
GET /api/v1/audit-log?api_key_id=550e8400-e29b-41d4-a716-446655440000
GET /api/v1/audit-log?workspace_id=550e8400-e29b-41d4-a716-446655440000
```

## Browser keys (no)

There is no such thing as a browser-safe `msk_*`. If you need a browser to call MIOSA-like functionality, mint a scoped browser token server-side. See [Browser Tokens](/docs/platform/browser-tokens).

> **Caution: Public vs private keys**
>
> Unlike Stripe, MIOSA does NOT issue separate `pk_*` publishable keys. Every MIOSA key is a server-side secret. If a third-party SDK or doc tells you to "just use the publishable key in the browser," that's not a MIOSA pattern  -  they're probably using a scoped browser token.

## See also

- [Attribution](/docs/platform/attribution)  -  the IDs your API key carries on each call
- [Browser Tokens](/docs/platform/browser-tokens)  -  for end-user-facing access
- [Authentication](/docs/authentication)  -  the wider auth model (JWT for sessions, etc.)
