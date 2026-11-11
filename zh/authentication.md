---
slug: "authentication"
title: "验证"
description: "如何使用 msk_ API 密钥或 JWT 令牌通过 MIOSA API 进行身份验证。"
section: "API reference"
order: 12000
---

每个 MIOSA API 请求都带有一个 `Authorization: Bearer &lt;token&gt;` 标头。
该令牌可以是 **API 密钥**（前缀 `msk_`）或来自 `POST /auth/login` 的 **JWT**。

一键覆盖全平台——电脑、桌面、文件、exec、CUA智能体、
OSA、学分和（具有正确角色的）管理员。 AI 没有单独的密钥，没有
每个产品的配置。

## 凭证层次结构

```mermaid
graph TB
    MSK[msk_* workspace key] -->|server-side only| API[MIOSA API calls]
    MSK -->|mints via POST /api/v1/browser-tokens| BT[Browser token - scoped + short-lived]
    BT -->|browser only| EP[Specific sandbox endpoint]
    MSK -->|mints via POST /api/v1/auth/sse-ticket| ST[SSE ticket - single use]
    ST -->|EventSource| SS[SSE stream]
    MSK -->|mints via POST /api/v1/sandboxes/:id/preview-token| MP[mp_ Preview token]
    MP -->|iframe embed - no msk_ exposed| Preview[MiosaPreview component]
    MSK -->|mints via POST /api/v1/sandboxes/:id/shares| MS[ms_ Share token]
    MS -->|URL param - public read-only| ShareURL[Public share URL]
```

## API 密钥

### 密钥格式
```text

msk_<role>_<random>
```

|前缀 |角色 |能力|
|--------|----------|----------------------------------------------------------------|
| `msk_u_...` |用户 |计算、桌面、文件、CUA、OSA、积分 |
| `msk_a_...` |管理员 |以上所有内容 ****** `/api/v1/admin/*` |
| `msk_p_...` |平台| MIOSA平台发布租户范围自动化 |

公共自助服务端点创建普通的 API 用途密钥。
专业操作员或情报凭证使用单独的受控流。

### 创建 API 密钥
```bash
curl -X POST https://api.miosa.ai/api/v1/api-keys \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Production SDK Key",
    "key_type": "user"
  }'
```
```json
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "key": "msk_u_a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6",
    "key_prefix": "msk_u_a1b2c3",
    "name": "Production SDK Key",
    "key_type": "user",
    "key_purpose": "api",
    "rate_limit_rpm": 600,
    "status": "active",
    "created_at": "2026-04-17T00:00:00Z"
  }
}
```

> **注意**
>
> 原始密钥在创建时**仅返回一次**。安全存放。如果丢失了，
> 撤销它并创建一个新的 - 原始的无法恢复。

用户可以创建个人 `user` 密钥。
租户所有者和管理员还可以为后端自动化创建 `platform` 密钥。
管理密钥保留在操作员内部，并且不是通过公共自助服务端点创建的。

### 使用 API 密钥

在 `Authorization` 标头中传递密钥：
```bash
curl https://api.miosa.ai/api/v1/computers \
  -H "Authorization: Bearer msk_u_a1b2c3d4e5f6..."
```

默认情况下，所有五个 SDK 从环境中读取 `MIOSA_API_KEY`：

**Python**

```python
  from miosa import Miosa

  # Explicit key
  client = Miosa(api_key="msk_u_...")

  # Or from environment: export MIOSA_API_KEY="msk_u_..."
  client = Miosa()
```

**打字稿**

```ts
  import { Miosa } from '@miosa/sdk';

  // Explicit key
  const miosa = new Miosa({ apiKey: 'msk_u_...' });
```

**去**

```go
  import "github.com/miosa-ai/miosa-go"

  // Explicit key
  client := miosa.NewClient("msk_u_...")

  // Or from environment
  client := miosa.NewClient(os.Getenv("MIOSA_API_KEY"))
```

**长生不老药**

```elixir
  # Explicit key
  client = Miosa.client("msk_u_...")

  # Or from environment
  client = Miosa.client(System.fetch_env!("MIOSA_API_KEY"))
```

**Java**

```java
  import ai.miosa.sdk.MiosaClient;

  MiosaClient miosa = new MiosaClient("msk_u_...");
```

### 管理密钥

|行动|端点|
|--------|----------|
|列出你的钥匙 | `GET /api/v1/api-keys` |
|创建密钥 | `POST /api/v1/api-keys` |
|撤销密钥 | `DELETE /api/v1/api-keys/{id}` |

撤销密钥会将其状态翻转为 `revoked`。更改被缓存并且
在几秒钟内强制执行 - 后续请求返回 `401 Unauthorized`。
撤销是不可逆转的；如果您需要恢复访问权限，请创建一个新密钥。

### 管理员密钥

`msk_a_*` 键解锁操作员管理员端点。
`msk_p_*` 密钥是受信任的租户自动化凭据，但它们不会自动使调用者成为 MIOSA 操作员。

点击管理端点的用户角色调用者会收到 `403 Forbidden`。

管理端点是运营商内部的，不公开记录。

## 预览代币

预览令牌 (`mp_<base64url>`) 是短暂的签名令牌，用于将沙箱预览嵌入到 `<iframe>` 中，而不会将 `msk_*` 键暴露给浏览器。

您的后端在服务器端生成预览令牌并将其传递给 `<MiosaPreview>` UI 组件。该组件自动包含它 - 浏览器代码中不需要 `Authorization` 标头。

### 铸造预览代币

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/preview-token \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{"expires_in": 3600}'
```
```json
{
  "data": {
    "token": "mp_eyJhbGciOiJFZERTQSIs...",
    "expires_at": "2026-05-25T13:00:00Z",
    "sandbox_id": "sbx_abc123"
  }
}
```

**Python**

```python
from miosa import Miosa

client = Miosa(api_key="msk_u_...")

preview = client.sandboxes.create_preview_token(
    sandbox_id="sbx_abc123",
    expires_in=3600,
)
# Pass preview.token to the browser  -  never the msk_ key.
```

**打字稿**

```typescript
import { Miosa } from '@miosa/sdk';

const miosa = new Miosa({ apiKey: 'msk_u_...' });

const preview = await miosa.sandboxes.createPreviewToken('sbx_abc123', {
  expiresIn: 3600,
});
// Pass preview.token to the browser  -  never the msk_ key.
```

在浏览器中，将令牌传递给 `<MiosaPreview>` 组件：
```html
<MiosaPreview token="mp_eyJhbGciOiJFZERTQSIs..." sandboxId="sbx_abc123" />
```

请参阅 [`<MiosaPreview>` 和其他 UI 组件 ](/docs/ui-components) 以及 [预览令牌 API 参考 ](/docs/api-reference/preview-tokens) 了解完整的组件 API 和令牌验证规范。

- **格式：** `mp_<base64url>` - 签名，不透明；不要尝试解码。
- **TTL：** 默认 1 小时；可通过 `expires_in` 配置（最长 24 小时）。
- **范围：** 对特定沙箱的预览端口的只读访问权限。无法创建或改变资源。
- **撤销：** 预览令牌不能单独撤销。销毁沙箱会使其所有未完成的代币失效。

---

## 分享代币

共享令牌 (`ms_<base64url>`) 将沙箱预览转换为公共只读 URL，任何知道该链接的人都可以访问 - 无需 MIOSA 帐户。

### 创建共享 URL

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/shares \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{"expires_in": 86400}'
```
```json
{
  "data": {
    "token": "ms_eyJhbGciOiJFZERTQSIs...",
    "url": "https://sbx-abc123.sandboxes.miosa.ai?ms=ms_eyJhbGciOiJFZERTQSIs...",
    "expires_at": "2026-05-26T00:00:00Z",
    "sandbox_id": "sbx_abc123"
  }
}
```

**Python**

```python
from miosa import Miosa

client = Miosa(api_key="msk_u_...")

share = client.sandboxes.create_share(
    sandbox_id="sbx_abc123",
    expires_in=86400,
)
print(share.url)  # Hand this URL to the end user.
```

**打字稿**

```typescript
import { Miosa } from '@miosa/sdk';

const miosa = new Miosa({ apiKey: 'msk_u_...' });

const share = await miosa.sandboxes.createShare('sbx_abc123', {
  expiresIn: 86400,
});
console.log(share.url); // Hand this URL to the end user.
```

- **格式：** `ms_<base64url>` - 令牌是 URL 查询参数 (`?ms=ms_...`)。
- **访问：** 公开、只读、未经身份验证 - 任何知道 URL 的人都可以查看预览。
- **TTL:** 通过 `expires_in` 以秒为单位设置。没有强制执行的最高限额；对于未到期的共享，请省略。
- **撤销：** `DELETE /api/v1/sandboxes/:id/shares/:share_id` 立即撤销。

请参阅 [Sandbox Shares API 参考 ](/docs/api-reference/sandbox-shares) 和 [集成：预览令牌 ](/docs/develop/previews)。

---

## 代币比较

|代币|范围 | TTL |谁使用它 |
|---|---|---|---|
| `msk_*` API 密钥 |完整的工作空间 |直至撤销 |服务器|
|智威汤逊 |用户会话| ~1 小时（刷新 7 天）|服务器/浏览器（UI） |
|浏览器令牌 |单沙箱 |可配置（默认 1 小时）|浏览器 |
| `mp_` 预览令牌 |单沙盒预览 |可配置（默认 1 小时）|浏览器/ iframe |
| `ms_` 分享代币 |单沙盒预览 |可配置|任何人（未经身份验证）|

|代币|如何铸造 |可以撤销|创建资源 |
|---|---|---|---|
| `msk_*` API 密钥 |仪表板或 `POST /api/v1/api-keys` |是的 |是的 |
|智威汤逊 | `POST /api/v1/auth/login` |否（短 TTL）|是的 |
|浏览器令牌 | `POST /api/v1/browser-tokens` |否（短 TTL）|没有 |
| `mp_` 预览令牌 | `POST /api/v1/sandboxes/:id/preview-token` |没有 |没有 |
| `ms_` 分享代币 | `POST /api/v1/sandboxes/:id/shares` |是的 |没有 |

---

## 白标授权说明

`tenant.preview_domain.*` 和 `tenant.branding.*` 配置端点接受 **两者** `msk_*` API 密钥和 JWT。这些是唯一可以使用 API 密钥访问的租户级配置路由 - 所有其他租户管理路由都需要来自管理会话的 JWT。

```bash
# Setting a preview domain with an API key (allowed)
curl -X PUT https://api.miosa.ai/api/v1/tenant/preview-domain \
  -H "Authorization: Bearer msk_a_..." \
  -H "Content-Type: application/json" \
  -d '{"domain": "preview.yourdomain.com"}'

# Fetching tenant branding with an API key (allowed)
curl https://api.miosa.ai/api/v1/tenant/branding \
  -H "Authorization: Bearer msk_a_..."
```

有关完整配置规范，请参阅 [Tenant Preview Domain](/docs/api-reference/tenant-preview-domain) 和 [Tenant Branding](/docs/api-reference/tenant-branding) API 参考。

---

## JWT 代币

JWT 令牌由 MIOSA Web UI 和 SSE 票证流程使用。他们被发行
登录时携带用户身份+租户上下文。

### 获取令牌
```bash
curl -X POST https://api.miosa.ai/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "user@example.com", "password": "your_password"}'
```
```json
{
  "token": "eyJhbGciOiJIUzUxMiIs...",
  "refresh_token": "eyJhbGciOiJIUzUxMiIs...",
  "user": { "id": "...", "email": "user@example.com" }
}
```

### 令牌生命周期

|代币|终身|刷新 |
|--------|----------|---------|
|访问令牌|约 1 小时 |使用刷新令牌 |
|刷新令牌 | 7 天 |需要重新登录 |
```bash
curl -X POST https://api.miosa.ai/api/v1/auth/refresh \
  -H "Content-Type: application/json" \
  -d '{"refresh_token": "eyJhbGciOiJIUzUxMiIs..."}'
```

> **注意**
>
> 对于编程访问，请使用 **API 密钥**。 JWT 是为浏览器设计的
> 会话频繁且过期。

## SSE 流

服务器发送的事件端点（CUA 会话事件）不能使用标准
`Authorization` 标头，因为 `EventSource` 不支持自定义标头。
相反，首先铸造一张一次性票：
```bash
# 1. Ask for a ticket (Bearer-authed  -  API key or JWT both work)
curl -X POST \
  https://api.miosa.ai/api/v1/computers/{id}/cua/sessions/{session_id}/sse-ticket \
  -H "Authorization: Bearer msk_u_..."

# {"ticket":"sset_abc123","expires_in":3600,"session_id":"..."}

# 2. Open the stream with the ticket as a query param
curl https://api.miosa.ai/api/v1/computers/{id}/cua/sessions/{session_id}/events?ticket=sset_abc123
```

该门票是一次性的，一小时后失效。

## 速率限制

|范围 |当前默认|笔记|
|--------|--------|--------|
|用户 API 密钥 |每分钟 600 个请求 |租户和端点系列策略可以降低它|
|平台 API 密钥 |每分钟 30,000 个请求 |仅限租户业主和管理员 |
|公共身份验证端点 |每个 IP 每分钟 60 个请求 |单独的身份验证操作可能有单独的控制|

当速率受限时，API 返回 `429 Too Many Requests` 并带有
`Retry-After` 标头。

每个响应中都会显示速率限制状态：
```http

X-RateLimit-Limit: 600
X-RateLimit-Remaining: 587
X-RateLimit-Reset: 1712700060
```

## 错误代码

|状态 |代码|意义|
|--------|------|---------|
| 401 | `UNAUTHORIZED`|凭证缺失/无效/吊销 |
| 403 | `FORBIDDEN`|凭证有效但角色不足 |
| 429 | `RATE_LIMITED`|请求过多 |

## 安全最佳实践

1. **切勿将 API 密钥**提交给版本控制。
2. **使用环境变量**或机密管理器。
3. **曝光时旋转密钥** - 撤销并创建一个新密钥。
4. **按角色划分范围** - 将 `msk_u_*` 键分发给大多数呼叫者；储备
   `msk_a_*` / `msk_p_*` 用于操作工具。
5. **开发和生产的单独密钥** - 更容易撤销爆炸半径。
6. **通过 IP 限制**（管理密钥） - 通过 `allowed_ips` 铸造时通过
   `POST /admin/api-keys` 拒绝来自意外来源的流量。
