---
slug: "api-reference/egress"
title: "出口（安全）API"
description: "MIOSA Egress 凭证库、网络白名单、OAuth Connect 和审核日志的 API 参考。"
section: "API reference"
order: 12055
---

Egress API 为每台计算机和沙盒上的 **安全** 选项卡提供支持：加密凭证库、OAuth Connect、具有仅审核/强制模式的网络许可名单以及实时尾部审核日志。

基本路径：`/api/v1/egress`

> **注意**
>
> 所有端点都需要 `Authorization: Bearer &lt;msk_*&gt;`。请参阅[身份验证](/docs/authentication)。如需更高级别的访问，请使用 [SDKs](/docs/sdks/egress)。

有关架构概述、演练和白标签模式，请参阅[安全指南](/docs/security)。

---

## 秘密

### 创建一个秘密

**`POST /api/v1/egress/secrets`**

存储加密凭据并可选择将其作为环境变量绑定到资源。原始值使用 ChaCha20-Poly1305 加密，并且不会再次返回。

```bash
curl -X POST https://api.miosa.ai/api/v1/egress/secrets \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "openai_key",
    "value": "sk-proj-...",
    "type": "api_key",
    "scope": "user",
    "expose_as_env": "OPENAI_API_KEY"
  }'
```

必填正文字段：`name`、`value`、`type`、`scope`。

|领域|价值观 |笔记|
|---|---|---|
| `type` | `api_key`、`oauth_connect`、`oauth_machine` | OAuth 流通常通过 `/oauth/start` 启动。 |
| `scope` | `tenant`、`workspace`、`user`、`external_user`、`external_workspace` | 5 层范围界定模型。 |
| `expose_as_env` |外壳安全的大蛇形外壳 |如果设置，则自动创建绑定。 |

基于 `scope` 的可选作用域字段：`workspace_id`、`owner_user_id`、`external_user_id`、`external_workspace_id`。

返回 `201 Created` 和 `{data: <public_secret>, binding: <binding_or_null>}`。 `data.preview` 是值的前 6 + 后 4；永远不会返回完整值。

### 列出秘密

**`GET /api/v1/egress/secrets`**

查询参数：`scope`、`workspace_id`、`owner_user_id`、`external_user_id`、`external_workspace_id`。

仅返回元数据（无值）。

### 获取秘密

**`GET /api/v1/egress/secrets/:id`** - 一个秘密的元数据。

### 旋转

**`PATCH /api/v1/egress/secrets/:id`**

使用新值重新加密。绑定的 `placeholder_token` 不变；每个使用它的沙箱都会在下一个出站请求中获取新值。

```json
{ "value": "sk-proj-new-value", "expires_at": "2026-12-31T00:00:00Z" }
```

### 删除

**`DELETE /api/v1/egress/secrets/:id`**

---

## 绑定

绑定将秘密链接到计算机或沙箱，并将其显示为环境变量（或不透明标头令牌）。

### 创建绑定

**`POST /api/v1/egress/bindings`**

```json
{
  "secret_id": "<uuid>",
  "resource_id": "<computer-or-sandbox-id>",
  "resource_type": "sandbox",
  "exposure": "env_var",
  "expose_as_env": "OPENAI_API_KEY"
}
```

返回绑定，包括其 `placeholder_token` - 工作负载在 env 中看到的不透明 `miosa-tok-…` 值。

### 列出+删除

- `GET /api/v1/egress/bindings?resource_id=<id>&resource_type=sandbox`
- `DELETE /api/v1/egress/bindings/:id`

---

## OAuth 连接

### 列出提供商目录

**`GET /api/v1/egress/oauth/providers`** - 返回调用租户可见的提供程序（平台默认值与租户覆盖合并）。没有泄露客户秘密。

### 启动 OAuth 流程

**`POST /api/v1/egress/oauth/start`**

```json
{
  "provider": "github",
  "expose_as_env": "GITHUB_TOKEN",
  "scope": "user",
  "owner_user_id": "<uuid>"
}
```

返回 `{authorize_url, state}`。在浏览器中打开`authorize_url`；用户批准；上游重定向到 `/api/v1/egress/oauth/callback?code=&state=`。成功后，令牌将被加密为新的 `oauth_connect` 秘密 + 绑定。

### 状态

**`GET /api/v1/egress/oauth/status?state=<state>`** - 在用户完成流程时进行轮询。返回 `pending`、`complete` 或 `failed`。

### 管理员：租户覆盖提供商（白标）

- `POST /api/v1/egress/oauth/admin/providers` - 注册您自己的 OAuth 应用程序凭据，以便最终用户在同意屏幕上看到您的品牌。
- `DELETE /api/v1/egress/oauth/admin/providers/:id`

需要管理员 (`msk_a_*`) 凭据。

---

## 网络策略 + 白名单

### 政策

- `GET /api/v1/egress/policies` - 列表（可以选择按 `resource_id` 过滤）
- `POST /api/v1/egress/policies` - 创建
- `GET /api/v1/egress/policies/:id`
- `PATCH /api/v1/egress/policies/:id` - 设置 `mode` (`audit_only` / `enforce`)、`default_action` (`allow` / `deny`)

每台计算机或沙箱按以下顺序解析策略：资源覆盖 → 租户默认。租户默认以 `audit_only` 模式启动。

### 白名单规则

- `GET /api/v1/egress/allowlist?policy_id=<uuid>`
- `POST /api/v1/egress/allowlist`
- `DELETE /api/v1/egress/allowlist/:id`

```json
{
  "policy_id": "<uuid>",
  "pattern": "*.openai.com",
  "kind": "wildcard",
  "method": "POST",
  "path_glob": "/v1/*",
  "action": "allow",
  "warn_only": false,
  "priority": 100
}
```

模式种类：`exact`（文字主机）、`wildcard`（`*.example.com`）、`cidr`（仅限 IP 文字目标）。

---

## 审核日志

### 查询

**`GET /api/v1/egress/audit`**

查询参数: `resource_id`, `host`, `action` (`allow`/`reject`/`stub`/`error`), `since` (相对如 `1h` / `7d` 或 ISO 时间戳）、`until`、`limit`（默认 100，最大 1000）、`external_user_id`、`external_workspace_id`。每行包含完整身份链（`tenant_id`、`workspace_id`、`owner_user_id`、`external_user_id`、`external_workspace_id`）、请求元数据（`host`、`method`、`path`、`sni`、 `mode`、`remote_addr`)、结果 (`action`、`rejected_by`、`status_code`、`duration_ms`、`bytes_in`、`bytes_out`、`error_message`) 和转换跟踪 JSONB (`request_transforms`、`response_transforms`、`mcp`)。

### 获取单个事件

**`GET /api/v1/egress/audit/:id`**

### 允许名单建议

**`GET /api/v1/egress/audit/suggestions?resource_id=<id>`** - 返回资源最近联系过的主机，按频率排名。一键式“锁定并选择为白名单”UX 的基础。

### Live tail (WebSocket)

**`wss://api.miosa.ai/api/v1/egress/audit/resource/:resource_id?token=<jwt>`**

子协议：`miosa-egress-audit-v1`。旨在在记录出站请求时实时推送 `{"type":"audit_event","data":{...}}` 帧。当前版本中尚未启用出口审核捕获，因此尚未发出任何帧。

租户范围的变体（仅限管理员）：`wss://api.miosa.ai/api/v1/egress/audit/tenant/:tenant_id?token=<jwt>`。

---

## 速率限制 + 保留

|范围 |限制|
|---|---|
|所有 `/egress/*` 端点 |当前限制通过 `X-RateLimit-*` 标头返回 |
|审计保留|每个租户 90 天；更关注企业|
|审计实时尾部（WebSocket）|无限;不计入 REST 配额 |

---

## 错误

与 API 其余部分相同的信封 - `{"error": "...", "details": ...}` 以及相应的状态代码。

|状态 |当 |
|---|---|
| `400` |无效的主体、未知的范围、格式错误的环境变量名称 |
| `401` |凭证缺失/无效 |
| `403` |角色不允许该操作（例如，以非管理员身份创建租户范围的机密）|
| `404` |在此租户中未找到秘密/绑定/策略/事件 |
| `409` |范围内的秘密名称冲突（唯一部分索引）|
| `422` |范围 ↔ 身份列不匹配（例如，范围=用户，没有owner_user_id）|

---

## SDK快捷方式

上述所有端点均由 [Egress SDK 参考](/docs/sdks/egress) 封装，并使用 Python、TypeScript、Go、Java 和 Elixir 中的惯用方法。
