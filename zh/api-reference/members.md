---
slug: "api-reference/members"
title: "工作区成员 API"
description: "用于管理工作区成员名册的 API 参考 - 添加、删除和更新工作区内用户的角色。"
section: "Organization & billing"
order: 13001
---

工作区成员是特定工作区的用户名册。用户必须已经是组织成员（`tenant_members` 行）才能添加到工作区。在另一个成员晋升为所有者之前，无法删除工作区中的最后一个 `owner`。

基本路径：`/api/v1/workspaces/:id/members`

> **注意**
>
> 要邀请还不是组织成员的新用户，请改用 [Workspace Invites API](/docs/api-reference/invites)。当用户是组织的新用户时，接受工作区邀请会自动创建 `tenant_members` 行。

---

## 端点

|方法|路径|描述 |
|--------|------|-------------|
| `GET` | `/workspaces/{id}/members` |列出工作区成员 |
| `POST` | `/workspaces/{id}/members` |将现有组织成员添加到工作区 |
| `PATCH` | `/workspaces/{id}/members/{user_id}` |更新成员的角色 |
| `DELETE` | `/workspaces/{id}/members/{user_id}` |删除成员 |

---

## 角色

|角色 |描述 |
|------|-------------|
| `owner` |完整的管理，包括传输和删除 |
| `admin` |可以管理会员和资源 |
| `member` |可以创建和管理资源|
| `viewer` |只读访问 |

---

## 列出成员

**`GET /api/v1/workspaces/{id}/members`**

返回工作区的所有成员。

### 响应 - `200 OK`

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

**打字稿**

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

**去**

```go
import "github.com/miosa-ai/miosa-go"

client := miosa.NewClient(os.Getenv("MIOSA_API_KEY"))
members, err := client.WorkspaceMembers.List(ctx, "ws-uuid")
```

**长生不老药**

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

## 添加会员

**`POST /api/v1/workspaces/{id}/members`**

将现有组织成员添加到具有给定角色的工作区。

> **注意**
>
> 用户必须已持有该组织的 `tenant_members` 行。如果他们还不是组织成员，请通过 `POST /workspaces/{id}/invites` 发送工作区邀请 - 接受邀请会自动创建组织成员资格。

### 请求正文

| 字段 | 类型 | 必填 | 描述 |
|--------|------|----------|-------------|
| `user_id` | UUID |是的 |用户 ID - 必须已经是租户成员 |
| `role` |字符串|是的 | `owner`、`admin`、`member` 或 `viewer` |

### 响应 - `201 Created`

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

### 错误

| 状态 | 代码 | 原因 |
|--------|------|--------|
| 422 | `NOT_TENANT_MEMBER` | 用户不是父组织的成员 |
| 422 | `MISSING_USER_ID` | `user_id` 字段缺失 |
| 422 | `MISSING_ROLE` | `role` 字段缺失 |
| 409 |  -  | 用户已经是工作区成员 |

**curl**

```bash
curl -X POST https://api.miosa.ai/api/v1/workspaces/{id}/members \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"user_id": "usr_def456", "role": "member"}'
```

**打字稿**

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

**去**

```go
record, err := client.WorkspaceMembers.Add(ctx, "ws-uuid", miosa.AddWorkspaceMemberInput{
    UserID: "usr_def456",
    Role:   "member",
})
```

**长生不老药**

```elixir
{:ok, record} = Miosa.WorkspaceMembers.add(client, "ws-uuid", "usr_def456", "member")
```

**Java**

```java
Map<String,Object> record = miosa.workspaceMembers().add("ws-uuid", "usr_def456", "member");
```

---

## 更新会员角色

**`PATCH /api/v1/workspaces/{id}/members/{user_id}`**

更改工作区成员的角色。

### 请求正文

| 字段 | 类型 | 必填 | 描述 |
|--------|------|----------|-------------|
| `role` |字符串|是的 |新角色：`owner`、`admin`、`member` 或 `viewer` |

### 响应 - `200 OK`

更新了工作区成员记录（与添加响应的形状相同）。

### 错误

| 状态 | 代码 | 原因 |
|--------|------|--------|
| 404 |  -  | 用户不是此工作区的成员 |
| 422 | `MISSING_ROLE` | `role` 字段缺失 |

**curl**

```bash
curl -X PATCH https://api.miosa.ai/api/v1/workspaces/{id}/members/{user_id} \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"role": "admin"}'
```

**打字稿**

```typescript
const updated = await client.workspaceMembers.updateRole('ws-uuid', 'usr_def456', {
  role: 'admin',
});
```

**Python**

```python
updated = client.workspace_members.update_role("ws-uuid", "usr_def456", role="admin")
```

**去**

```go
updated, err := client.WorkspaceMembers.UpdateRole(ctx, "ws-uuid", "usr_def456",
    miosa.UpdateWorkspaceMemberRoleInput{Role: "admin"})
```

**长生不老药**

```elixir
{:ok, updated} = Miosa.WorkspaceMembers.update_role(client, "ws-uuid", "usr_def456", "admin")
```

**Java**

```java
Map<String,Object> updated = miosa.workspaceMembers().updateRole("ws-uuid", "usr_def456", "admin");
```

---

## 删除成员

**`DELETE /api/v1/workspaces/{id}/members/{user_id}`**

从工作区中删除用户。这不会将它们从父组织中删除。

### 响应 - `200 OK`

```json
{ "deleted": true }
```

### 错误

| 状态 | 代码 | 原因 |
|--------|------|--------|
| 404 |  -  | 用户不是此工作区的成员 |
| 409 | `LAST_OWNER` | 无法移除最后一个工作区所有者 - 请先提升另一个成员 |

**curl**

```bash
curl -X DELETE https://api.miosa.ai/api/v1/workspaces/{id}/members/{user_id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

**打字稿**

```typescript
await client.workspaceMembers.remove('ws-uuid', 'usr_def456');
```

**Python**

```python
client.workspace_members.remove("ws-uuid", "usr_def456")
```

**去**

```go
err := client.WorkspaceMembers.Remove(ctx, "ws-uuid", "usr_def456")
```

**长生不老药**

```elixir
{:ok, _} = Miosa.WorkspaceMembers.remove(client, "ws-uuid", "usr_def456")
```

**Java**

```java
miosa.workspaceMembers().remove("ws-uuid", "usr_def456");
```

---

## 常见错误

| 状态 | 代码 | 原因 |
|--------|------|-------|
| 403 |  -  | 工作区属于其他租户 |
| 404 |  -  | 未找到工作区或用户 |
| 409 | `LAST_OWNER` | 拒绝移除唯一的工作区所有者 |
| 422 | `NOT_TENANT_MEMBER` | 用户必须先加入组织，才能加入工作区 |
| 422 | `MISSING_USER_ID` | `user_id` 请求体字段为必填 |
| 422 | `MISSING_ROLE` | `role` 请求体字段为必填 |

---

## 另请参阅

- [Workspace Invites API](/docs/api-reference/invites) - 通过电子邮件邀请用户（接受后创建组织成员资格）
- [工作区 API](/docs/api-reference/workspaces) - 工作区 CRUD
- [错误代码](/docs/api-reference/errors) - 完整错误参考
