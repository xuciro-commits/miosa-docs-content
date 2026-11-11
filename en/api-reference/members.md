---
slug: "api-reference/members"
title: "Workspace Members API"
description: "API reference for managing workspace member rosters - add, remove, and update roles for users inside a workspace."
section: "Organization & billing"
order: 13001
---

Workspace members are the user roster for a specific workspace. A user must already be an org member (`tenant_members` row) before they can be added to a workspace. The last `owner` in a workspace cannot be removed until another member is promoted to owner.

Base path: `/api/v1/workspaces/:id/members`

> **Note**
>
> To invite a new user who is not yet an org member, use the [Workspace Invites API](/docs/api-reference/invites) instead. Accepting a workspace invite automatically creates the `tenant_members` row when the user is new to the org.

---

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/workspaces/{id}/members` | List workspace members |
| `POST` | `/workspaces/{id}/members` | Add an existing org member to the workspace |
| `PATCH` | `/workspaces/{id}/members/{user_id}` | Update a member's role |
| `DELETE` | `/workspaces/{id}/members/{user_id}` | Remove a member |

---

## Roles

| Role | Description |
|------|-------------|
| `owner` | Full admin including transfer and deletion |
| `admin` | Can manage members and resources |
| `member` | Can create and manage resources |
| `viewer` | Read-only access |

---

## List Members

**`GET /api/v1/workspaces/{id}/members`**

Returns all members of the workspace.

### Response  -  `200 OK`

```json
{
  "data": [
    {
      "user_id": "usr_abc123",
      "email": "alice@example.com",
      "name": "Alice",
      "avatar_url": null,
      "role": "owner",
      "joined_at": "2026-05-01T10:00:00Z",
      "added_by": null
    },
    {
      "user_id": "usr_def456",
      "email": "bob@example.com",
      "name": "Bob",
      "avatar_url": null,
      "role": "member",
      "joined_at": "2026-05-10T14:00:00Z",
      "added_by": "usr_abc123"
    }
  ]
}
```

**curl**

```bash
curl https://api.miosa.ai/api/v1/workspaces/{id}/members \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';
const client = new Miosa();

const members = await client.workspaceMembers.list('ws-uuid');
console.log(members);
```

**Python**

```python
import miosa

client = miosa.Miosa()
members = client.workspace_members.list("ws-uuid")
print(members)
```

**Go**

```go
import "github.com/miosa-ai/miosa-go"

client := miosa.NewClient(os.Getenv("MIOSA_API_KEY"))
members, err := client.WorkspaceMembers.List(ctx, "ws-uuid")
```

**Elixir**

```elixir
client = Miosa.client("msk_u_...")
{:ok, members} = Miosa.WorkspaceMembers.list(client, "ws-uuid")
```

**Java**

```java
MiosaClient miosa = new MiosaClient("msk_u_...");
List<Map<String,Object>> members = miosa.workspaceMembers().list("ws-uuid");
```

---

## Add Member

**`POST /api/v1/workspaces/{id}/members`**

Adds an existing org member to the workspace with the given role.

> **Caution**
>
> The user must already hold a `tenant_members` row for this org. If they are not yet an org member, send a workspace invite via `POST /workspaces/{id}/invites` instead  -  accepting the invite creates the org membership atomically.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `user_id` | UUID | Yes | The user's ID  -  must already be a tenant member |
| `role` | string | Yes | `owner`, `admin`, `member`, or `viewer` |

### Response  -  `201 Created`

```json
{
  "data": {
    "user_id": "usr_def456",
    "workspace_id": "ws-uuid",
    "role": "member",
    "joined_at": "2026-05-22T09:00:00Z",
    "added_by": "usr_abc123"
  }
}
```

### Errors

| Status | Code | Cause |
|--------|------|-------|
| 422 | `NOT_TENANT_MEMBER` | User is not a member of the parent org |
| 422 | `MISSING_USER_ID` | `user_id` field missing |
| 422 | `MISSING_ROLE` | `role` field missing |
| 409 |  -  | User is already a workspace member |

**curl**

```bash
curl -X POST https://api.miosa.ai/api/v1/workspaces/{id}/members \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"user_id": "usr_def456", "role": "member"}'
```

**TypeScript**

```typescript
const record = await client.workspaceMembers.add('ws-uuid', {
  userId: 'usr_def456',
  role: 'member',
});
```

**Python**

```python
record = client.workspace_members.add("ws-uuid", user_id="usr_def456", role="member")
```

**Go**

```go
record, err := client.WorkspaceMembers.Add(ctx, "ws-uuid", miosa.AddWorkspaceMemberInput{
    UserID: "usr_def456",
    Role:   "member",
})
```

**Elixir**

```elixir
{:ok, record} = Miosa.WorkspaceMembers.add(client, "ws-uuid", "usr_def456", "member")
```

**Java**

```java
Map<String,Object> record = miosa.workspaceMembers().add("ws-uuid", "usr_def456", "member");
```

---

## Update Member Role

**`PATCH /api/v1/workspaces/{id}/members/{user_id}`**

Changes a workspace member's role.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `role` | string | Yes | New role: `owner`, `admin`, `member`, or `viewer` |

### Response  -  `200 OK`

Updated workspace member record (same shape as the add response).

### Errors

| Status | Code | Cause |
|--------|------|-------|
| 404 |  -  | User is not a member of this workspace |
| 422 | `MISSING_ROLE` | `role` field missing |

**curl**

```bash
curl -X PATCH https://api.miosa.ai/api/v1/workspaces/{id}/members/{user_id} \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"role": "admin"}'
```

**TypeScript**

```typescript
const updated = await client.workspaceMembers.updateRole('ws-uuid', 'usr_def456', {
  role: 'admin',
});
```

**Python**

```python
updated = client.workspace_members.update_role("ws-uuid", "usr_def456", role="admin")
```

**Go**

```go
updated, err := client.WorkspaceMembers.UpdateRole(ctx, "ws-uuid", "usr_def456",
    miosa.UpdateWorkspaceMemberRoleInput{Role: "admin"})
```

**Elixir**

```elixir
{:ok, updated} = Miosa.WorkspaceMembers.update_role(client, "ws-uuid", "usr_def456", "admin")
```

**Java**

```java
Map<String,Object> updated = miosa.workspaceMembers().updateRole("ws-uuid", "usr_def456", "admin");
```

---

## Remove Member

**`DELETE /api/v1/workspaces/{id}/members/{user_id}`**

Removes a user from the workspace. This does not remove them from the parent org.

### Response  -  `200 OK`

```json
{ "deleted": true }
```

### Errors

| Status | Code | Cause |
|--------|------|-------|
| 404 |  -  | User is not a member of this workspace |
| 409 | `LAST_OWNER` | Cannot remove the last workspace owner  -  promote another member first |

**curl**

```bash
curl -X DELETE https://api.miosa.ai/api/v1/workspaces/{id}/members/{user_id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

**TypeScript**

```typescript
await client.workspaceMembers.remove('ws-uuid', 'usr_def456');
```

**Python**

```python
client.workspace_members.remove("ws-uuid", "usr_def456")
```

**Go**

```go
err := client.WorkspaceMembers.Remove(ctx, "ws-uuid", "usr_def456")
```

**Elixir**

```elixir
{:ok, _} = Miosa.WorkspaceMembers.remove(client, "ws-uuid", "usr_def456")
```

**Java**

```java
miosa.workspaceMembers().remove("ws-uuid", "usr_def456");
```

---

## Common Errors

| Status | Code | Cause |
|--------|------|-------|
| 403 |  -  | Workspace belongs to a different tenant |
| 404 |  -  | Workspace or user not found |
| 409 | `LAST_OWNER` | Refusing to remove the sole workspace owner |
| 422 | `NOT_TENANT_MEMBER` | User must join the org before joining the workspace |
| 422 | `MISSING_USER_ID` | `user_id` body field is required |
| 422 | `MISSING_ROLE` | `role` body field is required |

---

## See also

- [Workspace Invites API](/docs/api-reference/invites)  -  invite users by email (creates org membership on accept)
- [Workspaces API](/docs/api-reference/workspaces)  -  workspace CRUD
- [Error Codes](/docs/api-reference/errors)  -  full error reference
