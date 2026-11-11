---
slug: "platform/api-keys"
title: "API 密钥"
description: "用于个人工具、后端自动化和限定范围 MIOSA API 访问的用户密钥与平台密钥。"
section: "Organization & billing"
order: 13002
---

**API 密钥**用于向 MIOSA 验证个人或后端服务的身份。
用户密钥是个人凭证。
平台密钥是租户自动化凭证，可以选择限制到一个工作区。

## 创建密钥

从控制台操作：**工作区 → 设置 → API 密钥 → 新建密钥**。或通过 API：

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

明文 `msk_*` 值在创建时**仅返回一次**。
此后控制台只显示前缀和后四位字符。
服务器只存储 SHA-256 哈希。

如果密钥丢失，请轮换——创建新密钥、替换旧密钥、再删除旧密钥。

## 密钥系列

```text

msk_<type>_<random>
```

- `msk_u_*` - 用于个人 CLI 和 SDK 的用户绑定密钥。
- `msk_p_*` - 用于服务端自动化的租户平台密钥。
- `msk_a_*` - 运维管理员密钥，不通过公开的自助服务 API 创建。

带环境标签的密钥前缀不属于当前的自助服务约定。
专用的测试模式计费隔离已列入计划。

对于服务端平台集成，请使用由租户所有者或管理员创建的 `platform` 密钥。
绝不要将其暴露给浏览器代码。

## 作用域

每个密钥都有一组作用域。MIOSA 会在每次变更操作时强制执行作用域。读取通常需要匹配的 `:read` 作用域；写入需要 `:write`。

常见作用域：

| 作用域 | 允许的操作 |
|---|---|
| `sandboxes:read` | 列出、获取沙盒 |
| `sandboxes:write` | 创建、执行、files.write、销毁 |
| `deployments:read` | 列出、获取部署、版本、域名 |
| `deployments:write` | 创建、发布、回滚、绑定域名 |
| `domains:write` | 创建、验证、删除自定义域名 |
| `databases:write` | 预置并操作托管数据库 |
| `storage:write` | 预置并操作对象存储 |
| `billing:read` | 读取计费和额度信息 |

对于服务端平台集成，请从后端所需的具体资源作用域开始。
创建时未指定作用域列表的用户密钥会获得标准产品作用域。
平台密钥是受信任的租户自动化凭证，因此只有租户所有者和管理员才能创建。

## 权限预设

预设是平台维护的一组具名作用域，因此你只需选择名称，不必自己维护列表。
`GET /api/v1/api-keys/presets` 会返回每个预设及其作用域。
控制台提供**只读**、**自动化**、**CI**、**完全访问**和**自定义**（自行选择每个作用域）。

```http
POST /api/v1/api-keys
{ "name": "Deploy pipeline", "preset": "ci" }
```

发送 `preset` 或 `scopes` 中的一项，不要同时发送两者。
平台会在创建密钥时将预设解析为当前作用域，此后密钥会保留这些作用域。

**完全访问**包含所有权限。
仅将其用于受信任的服务端工具，并优先使用服务账户。
控制台只向组织所有者和管理员提供该选项。

在生产环境中，请为服务账户创建密钥。
服务账户密钥不依赖某个人的成员身份，删除该账户会撤销其所有密钥。

## 轮换

按计划轮换密钥（建议每 90 天一次，或发生安全事件时轮换）。流程如下：

1. 使用相同作用域创建新密钥。
2. 将新密钥推广到你的后端（部署 + 重启）。
3. 确认新密钥可用（最近的活动日志中记录的是新密钥的 `key_id`）。
### 4. 删除旧密钥

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

删除密钥会立即使其失效。任何正在使用旧密钥的进行中请求都会开始收到 401。

## 速率限制

每个密钥都有一个由服务器提供的每分钟请求数限制。
当前默认值为用户密钥 600 rpm、平台密钥 30,000 rpm，但租户和端点系列的限制可能更严格。

- 密钥 ID（该密钥自己的配额桶）
- 租户或组织
- 端点（发布操作有独立的配额桶，与列表操作分开）

限制会随套餐扩展。每个响应中的响应头都会告诉你当前状态：

```http

X-RateLimit-Limit: 600
X-RateLimit-Remaining: 547
X-RateLimit-Reset: 1715712060
```

429 响应包含 `Retry-After` 响应头。SDK 会自动退避。

## 审计

每个由密钥认证的操作都会发出一个审计事件，包含：

- `api_key_id`（让你知道哪个密钥做了什么）
- `actor`（组织，加上密钥名称）
- `workspace_id`、`project_id`、`external_workspace_id`、`external_user_id`（如果在请求中传入或解析得到）
- 操作（例如 `deployments.publish`、`domains.create`）
- 时间戳 + 结果

通过以下方式读取审计日志：

```http
GET /api/v1/audit-log?api_key_id=550e8400-e29b-41d4-a716-446655440000
GET /api/v1/audit-log?workspace_id=550e8400-e29b-41d4-a716-446655440000
```

## 浏览器密钥（没有这种东西）

不存在浏览器安全的 `msk_*`。如果你需要让浏览器调用类似 MIOSA 的功能，请在服务端签发限定范围的浏览器令牌。请参阅[浏览器令牌](/docs/platform/browser-tokens)。

> **注意：公开密钥与私密密钥**
>
> 与 Stripe 不同，MIOSA 不签发单独的 `pk_*` 可公开密钥。每个 MIOSA 密钥都是服务端机密。如果第三方 SDK 或文档告诉你“直接在浏览器中使用可公开密钥”，那不是 MIOSA 的模式——它们很可能使用的是限定范围的浏览器令牌。

## 另请参阅

- [归属](/docs/platform/attribution) - 你的 API 密钥在每次调用中携带的 ID
- [浏览器令牌](/docs/platform/browser-tokens) - 用于面向最终用户的访问
- [身份验证](/docs/authentication) - 更完整的认证模型（会话使用 JWT 等）
