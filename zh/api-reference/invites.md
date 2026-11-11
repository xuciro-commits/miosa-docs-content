---
slug: "api-reference/invites"
title: "邀请API"
description: "工作区邀请和组织邀请流程的 API 参考 - 发送、列出、撤销、预览和接受电子邮件邀请。"
section: "API reference"
order: 12045
---

MIOSA 有两个并行的邀请流程：

- **工作区邀请** - 邀请用户到特定工作区。如果电子邮件已属于组织成员，则直接添加该用户（不发送电子邮件）。如果没有，则会发送邀请电子邮件并接受它，自动创建 `tenant_members` 和 `workspace_members` 行。
- **组织邀请** - 邀请用户加入组织（租户），无需预先选择工作区。接受步骤仅创建 `tenant_members` 行。

> **注意**
>
> 两个流程都会产生不透明的令牌。 `GET /invites/:token` 和 `GET /workspace-invites/:token` 预览端点是 **公开** - 无需 API 密钥即可调用它们，以在用户登录之前呈现登录页面。

电子邮件邀请链接应指向 MIOSA 前端，而不是 API 主机。
使用 `https://miosa.ai/invites/{token}` 进行组织邀请，使用 `https://miosa.ai/workspace-invites/{token}` 进行工作区邀请。
旧版 `https://miosa.ai/invite/{token}` 链接重定向到组织邀请登录页面以实现兼容性。

---

## 工作区邀请端点

|方法|路径|授权 |描述 |
|--------|------|------|-------------|
| `POST` | `/workspaces/{id}/invites` |必填 |发送工作区邀请 |
| `GET` | `/workspaces/{id}/invites` |必填 |列出待处理的邀请 |
| `DELETE` | `/workspaces/{id}/invites/{invite_id}` |必填 |撤销邀请 |
| `GET` | `/workspace-invites/{token}` |无（公共）|预览邀请详情 |
| `POST` | `/workspace-invites/{token}/accept` |必填 |接受邀请 |

## 组织邀请端点

|方法|路径|授权 |描述 |
|--------|------|------|-------------|
| `POST` | `/tenants/{id}/invites` |必需（管理员/所有者）|发送组织邀请 |
| `GET` | `/tenants/{id}/invites` |必需（管理员/所有者）|列出待处理的组织邀请 |
| `DELETE` | `/tenants/{id}/invites/{invite_id}` |必需（管理员/所有者）|撤销组织邀请 |
| `GET` | `/invites/{token}` |无（公共）|预览组织邀请详细信息 |
| `POST` | `/invites/{token}/accept` |必填 |接受组织邀请 |

---

## 工作区邀请流程

### 发送工作区邀请

**`POST /api/v1/workspaces/{id}/invites`**

发送邀请邮件并返回邀请记录。如果 `email` 已经属于租户成员，则直接添加用户（响应 `type = "added"`，不发送电子邮件）。

#### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `email` |字符串|是的 |邀请电子邮件地址 |
| `role` |字符串|是的 | `owner`、`admin`、`member` 或 `viewer` |

#### 响应 - `201 Created`（新邀请）

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

#### 响应 - `200 OK`（直接添加现有成员）

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

**打字稿**

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

**去**

```go
result, err := client.WorkspaceInvites.Create(ctx, "ws-uuid", miosa.CreateWorkspaceInviteInput{
    Email: "alice@example.com",
    Role:  "member",
})
```

**长生不老药**

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

### 列出工作区邀请

**`GET /api/v1/workspaces/{id}/invites`**

返回所有待处理的工作区邀请（不包括已接受和已撤销的邀请）。

#### 响应 - `200 OK`

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

**打字稿**

```typescript
const invites = await client.workspaceInvites.list('ws-uuid');
```

**Python**

```python
invites = client.workspace_invites.list("ws-uuid")
```

**去**

```go
invites, err := client.WorkspaceInvites.List(ctx, "ws-uuid")
```

**长生不老药**

```elixir
{:ok, invites} = Miosa.WorkspaceInvites.list(client, "ws-uuid")
```

**Java**

```java
List<Map<String,Object>> invites = miosa.workspaceInvites().list("ws-uuid");
```

---

### 撤销工作区邀请

**`DELETE /api/v1/workspaces/{id}/invites/{invite_id}`**

撤销待处理的邀请。如果邀请已被接受，则返回 `409`。

#### 响应 - `200 OK`

```json
{ "invite_id": "inv_abc123", "revoked": true }
```

**curl**

```bash
curl -X DELETE https://api.miosa.ai/api/v1/workspaces/{id}/invites/{invite_id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

**打字稿**

```typescript
await client.workspaceInvites.revoke('ws-uuid', 'inv_abc123');
```

**Python**

```python
client.workspace_invites.revoke("ws-uuid", "inv_abc123")
```

**去**

```go
_, err := client.WorkspaceInvites.Revoke(ctx, "ws-uuid", "inv_abc123")
```

**长生不老药**

```elixir
{:ok, _} = Miosa.WorkspaceInvites.revoke(client, "ws-uuid", "inv_abc123")
```

**Java**

```java
miosa.workspaceInvites().revoke("ws-uuid", "inv_abc123");
```

---

### 预览工作区邀请（公开）

**`GET /api/v1/workspace-invites/{token}`**

返回邀请元数据，无需身份验证。使用它来呈现预身份验证登录页面。

#### 响应 - `200 OK`

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

**打字稿**

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

**去**

```go
preview, err := client.WorkspaceInvites.Preview(ctx, token)
```

**长生不老药**

```elixir
{:ok, preview} = Miosa.WorkspaceInvites.preview(client, token)
unless preview["expired"], do: ...
```

**Java**

```java
Map<String,Object> preview = miosa.workspaceInvites().preview(token);
```

---

### 接受工作区邀请

**`POST /api/v1/workspace-invites/{token}/accept`**

代表经过身份验证的用户接受邀请。如果用户是组织的新用户，则会在 `workspace_members` 行旁边自动创建 `tenant_members` 行。

调用者的 API 密钥电子邮件必须与邀请电子邮件匹配（不区分大小写）。

#### 响应 - `200 OK`

```json
{
  "accepted": true,
  "workspace_id": "ws-uuid",
  "tenant_id": "ten-uuid",
  "role": "member"
}
```

#### 错误

| 状态 | 代码 | 原因 |
|--------|------|--------|
| 400 | `INVALID_TOKEN` |令牌未找到或已过期 |
| 400 | `REVOKED` |邀请被撤销 |
| 422 | `EMAIL_MISMATCH` | 经过身份验证的用户的电子邮件与邀请电子邮件不同 |

**curl**

```bash
curl -X POST https://api.miosa.ai/api/v1/workspace-invites/{token}/accept \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

**打字稿**

```typescript
const result = await client.workspaceInvites.accept(token);
console.log('joined workspace:', result.workspaceId);
```

**Python**

```python
result = client.workspace_invites.accept(token)
print("joined workspace:", result["workspace_id"])
```

**去**

```go
result, err := client.WorkspaceInvites.Accept(ctx, token)
```

**长生不老药**

```elixir
{:ok, result} = Miosa.WorkspaceInvites.accept(client, token)
IO.puts("joined workspace: #{result["workspace_id"]}")
```

**Java**

```java
Map<String,Object> result = miosa.workspaceInvites().accept(token);
```

---

## 组织邀请流程

组织邀请授予租户 (组织) 成员资格，而无需分配特定工作区。
写操作需要 `admin` 或 `owner` 角色。

### 发送组织邀请

**`POST /api/v1/tenants/{id}/invites`**

发送邀请电子邮件。响应包括一个主机感知的 `invite_url` - 在白标签租户上，它使用租户的自定义域。

#### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `email` |字符串|是的 |邀请电子邮件地址 |
| `role` |字符串|是的 | `owner`、`admin` 或 `member` |

#### 响应 - `201 Created`

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

**打字稿**

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

**去**

```go
invite, err := client.OrgInvites.Create(ctx, tenantID, miosa.CreateOrgInviteInput{
    Email: "bob@example.com",
    Role:  "member",
})
```

**长生不老药**

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

### 列出组织邀请

**`GET /api/v1/tenants/{id}/invites`**

返回所有待处理的组织邀请。

**curl**

```bash
curl https://api.miosa.ai/api/v1/tenants/{id}/invites \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

**打字稿**

```typescript
const invites = await client.orgInvites.list(tenantId);
```

**Python**

```python
invites = client.org_invites.list(tenant_id)
```

**去**

```go
invites, err := client.OrgInvites.List(ctx, tenantID)
```

**长生不老药**

```elixir
{:ok, invites} = Miosa.OrgInvites.list(client, tenant_id)
```

**Java**

```java
List<Map<String,Object>> invites = miosa.orgInvites().list(tenantId);
```

---

### 撤销组织邀请

**`DELETE /api/v1/tenants/{id}/invites/{invite_id}`**

撤销待处理的组织邀请。如果邀请已被接受，则返回 `409`。

**curl**

```bash
curl -X DELETE https://api.miosa.ai/api/v1/tenants/{id}/invites/{invite_id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

**打字稿**

```typescript
await client.orgInvites.revoke(tenantId, inviteId);
```

**Python**

```python
client.org_invites.revoke(tenant_id, invite_id)
```

**去**

```go
_, err := client.OrgInvites.Revoke(ctx, tenantID, inviteID)
```

**长生不老药**

```elixir
{:ok, _} = Miosa.OrgInvites.revoke(client, tenant_id, invite_id)
```

**Java**

```java
miosa.orgInvites().revoke(tenantId, inviteId);
```

---

### 预览组织邀请（公开）

**`GET /api/v1/invites/{token}`**

返回邀请元数据，无需身份验证。

#### 响应 - `200 OK`

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

**打字稿**

```typescript
const preview = await client.orgInvites.preview(token);
console.log('invited to org:', preview.tenantName);
```

**Python**

```python
preview = client.org_invites.preview(token)
print("invited to org:", preview["tenant_name"])
```

**去**

```go
preview, err := client.OrgInvites.Preview(ctx, token)
```

**长生不老药**

```elixir
{:ok, preview} = Miosa.OrgInvites.preview(client, token)
IO.inspect(preview["tenant_name"])
```

**Java**

```java
Map<String,Object> preview = miosa.orgInvites().preview(token);
```

---

### 接受组织邀请

**`POST /api/v1/invites/{token}/accept`**

代表经过身份验证的用户接受组织邀请。创建 `tenant_members` 行。

调用者的 API 密钥电子邮件必须与邀请电子邮件匹配（不区分大小写）。

#### 响应 - `200 OK`

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

#### 错误

|状态 |代码|原因 |
|--------|------|--------|
| 400 | `invalid_token` |令牌未找到或已过期 |
| 422 | `EMAIL_MISMATCH` | 会话电子邮件与邀请电子邮件不同 |

**curl**

```bash
curl -X POST https://api.miosa.ai/api/v1/invites/{token}/accept \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

**打字稿**

```typescript
const result = await client.orgInvites.accept(token);
console.log('joined org:', result.tenantId);
```

**Python**

```python
result = client.org_invites.accept(token)
print("joined org:", result["tenant_id"])
```

**去**

```go
result, err := client.OrgInvites.Accept(ctx, token)
```

**长生不老药**

```elixir
{:ok, result} = Miosa.OrgInvites.accept(client, token)
IO.puts("joined org: #{result["tenant_id"]}")
```

**Java**

```java
Map<String,Object> result = miosa.orgInvites().accept(token);
```

---

## 常见错误

|状态 |代码|原因 |
|--------|------|--------|
| 400 | `INVALID_TOKEN` / `invalid_token` |令牌未找到、已过期或已撤销 |
| 400 | `REVOKED` |邀请被明确撤销 |
| 409 | `ALREADY_ACCEPTED` | 无法撤销已接受的邀请 |
| 422 | `EMAIL_MISMATCH` | 经过身份验证的用户的电子邮件与邀请电子邮件不匹配 |
| 422 | `MISSING_EMAIL` | 请求正文中缺少 `email` 字段 |

---

## 另请参阅

- [Workspace Members API](/docs/api-reference/members) - 直接管理花名册（需要事先成为组织会员）
- [工作区 API](/docs/api-reference/workspaces) - 工作区 CRUD
- [错误代码](/docs/api-reference/errors) - 完整错误参考
