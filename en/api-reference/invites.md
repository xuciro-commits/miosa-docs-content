---
slug: "api-reference/invites"
title: "Invites API"
description: "API reference for the workspace invite and org invite flows - send, list, revoke, preview, and accept email invitations."
section: "API reference"
order: 12045
---

MIOSA has two parallel invite flows:

- **Workspace invites**  -  invite a user to a specific workspace. If the email already belongs to an org member, the user is added directly (no email sent). If not, an invite email is dispatched and accepting it creates both the `tenant_members` and `workspace_members` rows atomically.
- **Org invites**  -  invite a user to join the org (tenant) without pre-selecting a workspace. The accept step only creates the `tenant_members` row.

> **Note**
>
> Both flows produce opaque tokens. The `GET /invites/:token` and `GET /workspace-invites/:token` preview endpoints are **public**  -  they can be called without an API key to render a landing page before the user signs in.

Email invite links should point at the MIOSA frontend, not the API host.
Use `https://miosa.ai/invites/{token}` for org invites and `https://miosa.ai/workspace-invites/{token}` for workspace invites.
Legacy `https://miosa.ai/invite/{token}` links redirect to the org invite landing page for compatibility.

---

## Workspace Invite Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `POST` | `/workspaces/{id}/invites` | Required | Send a workspace invite |
| `GET` | `/workspaces/{id}/invites` | Required | List pending invites |
| `DELETE` | `/workspaces/{id}/invites/{invite_id}` | Required | Revoke an invite |
| `GET` | `/workspace-invites/{token}` | None (public) | Preview invite details |
| `POST` | `/workspace-invites/{token}/accept` | Required | Accept the invite |

## Org Invite Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `POST` | `/tenants/{id}/invites` | Required (admin/owner) | Send an org invite |
| `GET` | `/tenants/{id}/invites` | Required (admin/owner) | List pending org invites |
| `DELETE` | `/tenants/{id}/invites/{invite_id}` | Required (admin/owner) | Revoke an org invite |
| `GET` | `/invites/{token}` | None (public) | Preview org invite details |
| `POST` | `/invites/{token}/accept` | Required | Accept the org invite |

---

## Workspace Invite Flow

### Send Workspace Invite

**`POST /api/v1/workspaces/{id}/invites`**

Sends an invite email and returns the invite record. If the `email` already belongs to a tenant member the user is added directly (response `type = "added"`, no email sent).

#### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `email` | string | Yes | Email address to invite |
| `role` | string | Yes | `owner`, `admin`, `member`, or `viewer` |

#### Response  -  `201 Created` (new invite)

```json
{
  "type": "invited",
  "data": {
    "id": "inv_abc123",
    "workspace_id": "ws-uuid",
    "tenant_id": "ten-uuid",
    "email": "alice@example.com",
    "role": "member",
    "invited_by": "usr_owner",
    "expires_at": "2026-05-29T09:00:00Z",
    "accepted_at": null,
    "inserted_at": "2026-05-22T09:00:00Z"
  }
}
```

#### Response  -  `200 OK` (existing member added directly)

```json
{
  "type": "added",
  "data": {
    "user_id": "usr_def456",
    "workspace_id": "ws-uuid",
    "role": "member",
    "joined_at": "2026-05-22T09:00:00Z",
    "added_by": "usr_owner"
  }
}
```

**curl**

```bash
curl -X POST https://api.miosa.ai/api/v1/workspaces/{id}/invites \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"email": "alice@example.com", "role": "member"}'
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';
const client = new Miosa();

const result = await client.workspaceInvites.create('ws-uuid', {
  email: 'alice@example.com',
  role: 'member',
});
// result.type === 'invited' | 'added'
```

**Python**

```python
import miosa
client = miosa.Miosa()

result = client.workspace_invites.create("ws-uuid", "alice@example.com", "member")
print(result["type"])  # "invited" or "added"
```

**Go**

```go
result, err := client.WorkspaceInvites.Create(ctx, "ws-uuid", miosa.CreateWorkspaceInviteInput{
    Email: "alice@example.com",
    Role:  "member",
})
```

**Elixir**

```elixir
{:ok, result} = Miosa.WorkspaceInvites.create(client, "ws-uuid", "alice@example.com", "member")
IO.inspect(result["type"])
```

**Java**

```java
Map<String,Object> result = miosa.workspaceInvites().create("ws-uuid", "alice@example.com", "member");
String type = (String) result.get("type"); // "invited" or "added"
```

---

### List Workspace Invites

**`GET /api/v1/workspaces/{id}/invites`**

Returns all pending workspace invites (excludes accepted and revoked).

#### Response  -  `200 OK`

```json
{
  "data": [
    {
      "id": "inv_abc123",
      "workspace_id": "ws-uuid",
      "tenant_id": "ten-uuid",
      "email": "alice@example.com",
      "role": "member",
      "invited_by": "usr_owner",
      "expires_at": "2026-05-29T09:00:00Z",
      "accepted_at": null,
      "inserted_at": "2026-05-22T09:00:00Z"
    }
  ]
}
```

**curl**

```bash
curl https://api.miosa.ai/api/v1/workspaces/{id}/invites \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

**TypeScript**

```typescript
const invites = await client.workspaceInvites.list('ws-uuid');
```

**Python**

```python
invites = client.workspace_invites.list("ws-uuid")
```

**Go**

```go
invites, err := client.WorkspaceInvites.List(ctx, "ws-uuid")
```

**Elixir**

```elixir
{:ok, invites} = Miosa.WorkspaceInvites.list(client, "ws-uuid")
```

**Java**

```java
List<Map<String,Object>> invites = miosa.workspaceInvites().list("ws-uuid");
```

---

### Revoke Workspace Invite

**`DELETE /api/v1/workspaces/{id}/invites/{invite_id}`**

Revokes a pending invite. Returns `409` if the invite was already accepted.

#### Response  -  `200 OK`

```json
{ "invite_id": "inv_abc123", "revoked": true }
```

**curl**

```bash
curl -X DELETE https://api.miosa.ai/api/v1/workspaces/{id}/invites/{invite_id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

**TypeScript**

```typescript
await client.workspaceInvites.revoke('ws-uuid', 'inv_abc123');
```

**Python**

```python
client.workspace_invites.revoke("ws-uuid", "inv_abc123")
```

**Go**

```go
_, err := client.WorkspaceInvites.Revoke(ctx, "ws-uuid", "inv_abc123")
```

**Elixir**

```elixir
{:ok, _} = Miosa.WorkspaceInvites.revoke(client, "ws-uuid", "inv_abc123")
```

**Java**

```java
miosa.workspaceInvites().revoke("ws-uuid", "inv_abc123");
```

---

### Preview Workspace Invite (public)

**`GET /api/v1/workspace-invites/{token}`**

Returns invite metadata without requiring authentication. Use this to render the pre-auth landing page.

#### Response  -  `200 OK`

```json
{
  "data": {
    "workspace_name": "Acme Corp",
    "role": "member",
    "expires_at": "2026-05-29T09:00:00Z",
    "expired": false,
    "accepted": false
  }
}
```

**curl**

```bash
curl https://api.miosa.ai/api/v1/workspace-invites/{token}
```

**TypeScript**

```typescript
const preview = await client.workspaceInvites.preview(token);
if (!preview.expired) {
  // show accept button
}
```

**Python**

```python
preview = client.workspace_invites.preview(token)
if not preview["expired"]:
    pass  # show accept button
```

**Go**

```go
preview, err := client.WorkspaceInvites.Preview(ctx, token)
```

**Elixir**

```elixir
{:ok, preview} = Miosa.WorkspaceInvites.preview(client, token)
unless preview["expired"], do: ...
```

**Java**

```java
Map<String,Object> preview = miosa.workspaceInvites().preview(token);
```

---

### Accept Workspace Invite

**`POST /api/v1/workspace-invites/{token}/accept`**

Accepts the invite on behalf of the authenticated user. If the user is new to the org, a `tenant_members` row is created atomically alongside the `workspace_members` row.

The caller's API key email must match the invite email (case-insensitive).

#### Response  -  `200 OK`

```json
{
  "accepted": true,
  "workspace_id": "ws-uuid",
  "tenant_id": "ten-uuid",
  "role": "member"
}
```

#### Errors

| Status | Code | Cause |
|--------|------|-------|
| 400 | `INVALID_TOKEN` | Token not found or expired |
| 400 | `REVOKED` | Invite was revoked |
| 422 | `EMAIL_MISMATCH` | Authenticated user's email differs from invite email |

**curl**

```bash
curl -X POST https://api.miosa.ai/api/v1/workspace-invites/{token}/accept \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

**TypeScript**

```typescript
const result = await client.workspaceInvites.accept(token);
console.log('joined workspace:', result.workspaceId);
```

**Python**

```python
result = client.workspace_invites.accept(token)
print("joined workspace:", result["workspace_id"])
```

**Go**

```go
result, err := client.WorkspaceInvites.Accept(ctx, token)
```

**Elixir**

```elixir
{:ok, result} = Miosa.WorkspaceInvites.accept(client, token)
IO.puts("joined workspace: #{result["workspace_id"]}")
```

**Java**

```java
Map<String,Object> result = miosa.workspaceInvites().accept(token);
```

---

## Org Invite Flow

Org invites grant membership in the tenant (org) without assigning a specific workspace.
Write operations require `admin` or `owner` role.

### Send Org Invite

**`POST /api/v1/tenants/{id}/invites`**

Dispatches an invite email. The response includes an `invite_url` that is host-aware  -  on white-label tenants it uses the tenant's custom domain.

#### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `email` | string | Yes | Email address to invite |
| `role` | string | Yes | `owner`, `admin`, or `member` |

#### Response  -  `201 Created`

```json
{
  "data": {
    "invite_id": "inv_xyz789",
    "email": "bob@example.com",
    "role": "member",
    "expires_at": "2026-05-29T09:00:00Z",
    "invite_url": "https://miosa.ai/invites/eyJ..."
  }
}
```

**curl**

```bash
curl -X POST https://api.miosa.ai/api/v1/tenants/{id}/invites \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"email": "bob@example.com", "role": "member"}'
```

**TypeScript**

```typescript
const invite = await client.orgInvites.create(tenantId, {
  email: 'bob@example.com',
  role: 'member',
});
console.log('invite URL:', invite.inviteUrl);
```

**Python**

```python
invite = client.org_invites.create(tenant_id, "bob@example.com", "member")
print("invite URL:", invite["invite_url"])
```

**Go**

```go
invite, err := client.OrgInvites.Create(ctx, tenantID, miosa.CreateOrgInviteInput{
    Email: "bob@example.com",
    Role:  "member",
})
```

**Elixir**

```elixir
{:ok, invite} = Miosa.OrgInvites.create(client, tenant_id, "bob@example.com", "member")
IO.puts("invite URL: #{invite["invite_url"]}")
```

**Java**

```java
Map<String,Object> invite = miosa.orgInvites().create(tenantId, "bob@example.com", "member");
System.out.println("invite URL: " + invite.get("invite_url"));
```

---

### List Org Invites

**`GET /api/v1/tenants/{id}/invites`**

Returns all pending org invites.

**curl**

```bash
curl https://api.miosa.ai/api/v1/tenants/{id}/invites \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

**TypeScript**

```typescript
const invites = await client.orgInvites.list(tenantId);
```

**Python**

```python
invites = client.org_invites.list(tenant_id)
```

**Go**

```go
invites, err := client.OrgInvites.List(ctx, tenantID)
```

**Elixir**

```elixir
{:ok, invites} = Miosa.OrgInvites.list(client, tenant_id)
```

**Java**

```java
List<Map<String,Object>> invites = miosa.orgInvites().list(tenantId);
```

---

### Revoke Org Invite

**`DELETE /api/v1/tenants/{id}/invites/{invite_id}`**

Revokes a pending org invite. Returns `409` if the invite was already accepted.

**curl**

```bash
curl -X DELETE https://api.miosa.ai/api/v1/tenants/{id}/invites/{invite_id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

**TypeScript**

```typescript
await client.orgInvites.revoke(tenantId, inviteId);
```

**Python**

```python
client.org_invites.revoke(tenant_id, invite_id)
```

**Go**

```go
_, err := client.OrgInvites.Revoke(ctx, tenantID, inviteID)
```

**Elixir**

```elixir
{:ok, _} = Miosa.OrgInvites.revoke(client, tenant_id, invite_id)
```

**Java**

```java
miosa.orgInvites().revoke(tenantId, inviteId);
```

---

### Preview Org Invite (public)

**`GET /api/v1/invites/{token}`**

Returns invite metadata without authentication.

#### Response  -  `200 OK`

```json
{
  "data": {
    "email": "bob@example.com",
    "tenant_name": "Acme Corp",
    "role": "member",
    "expires_at": "2026-05-29T09:00:00Z",
    "expired": false,
    "accepted": false
  }
}
```

**curl**

```bash
curl https://api.miosa.ai/api/v1/invites/{token}
```

**TypeScript**

```typescript
const preview = await client.orgInvites.preview(token);
console.log('invited to org:', preview.tenantName);
```

**Python**

```python
preview = client.org_invites.preview(token)
print("invited to org:", preview["tenant_name"])
```

**Go**

```go
preview, err := client.OrgInvites.Preview(ctx, token)
```

**Elixir**

```elixir
{:ok, preview} = Miosa.OrgInvites.preview(client, token)
IO.inspect(preview["tenant_name"])
```

**Java**

```java
Map<String,Object> preview = miosa.orgInvites().preview(token);
```

---

### Accept Org Invite

**`POST /api/v1/invites/{token}/accept`**

Accepts an org invite on behalf of the authenticated user. Creates a `tenant_members` row.

The caller's API key email must match the invite email (case-insensitive).

#### Response  -  `200 OK`

```json
{
  "accepted": true,
  "tenant_id": "ten-uuid",
  "tenant": {
    "id": "ten-uuid",
    "name": "Acme Corp"
  }
}
```

#### Errors

| Status | Code | Cause |
|--------|------|-------|
| 400 | `invalid_token` | Token not found or expired |
| 422 | `EMAIL_MISMATCH` | Session email differs from invite email |

**curl**

```bash
curl -X POST https://api.miosa.ai/api/v1/invites/{token}/accept \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

**TypeScript**

```typescript
const result = await client.orgInvites.accept(token);
console.log('joined org:', result.tenantId);
```

**Python**

```python
result = client.org_invites.accept(token)
print("joined org:", result["tenant_id"])
```

**Go**

```go
result, err := client.OrgInvites.Accept(ctx, token)
```

**Elixir**

```elixir
{:ok, result} = Miosa.OrgInvites.accept(client, token)
IO.puts("joined org: #{result["tenant_id"]}")
```

**Java**

```java
Map<String,Object> result = miosa.orgInvites().accept(token);
```

---

## Common Errors

| Status | Code | Cause |
|--------|------|-------|
| 400 | `INVALID_TOKEN` / `invalid_token` | Token not found, expired, or revoked |
| 400 | `REVOKED` | Invite was explicitly revoked |
| 409 | `ALREADY_ACCEPTED` | Cannot revoke an invite that was already accepted |
| 422 | `EMAIL_MISMATCH` | Authenticated user's email does not match the invite email |
| 422 | `MISSING_EMAIL` | `email` field missing from request body |

---

## See also

- [Workspace Members API](/docs/api-reference/members)  -  manage the roster directly (requires prior org membership)
- [Workspaces API](/docs/api-reference/workspaces)  -  workspace CRUD
- [Error Codes](/docs/api-reference/errors)  -  full error reference
