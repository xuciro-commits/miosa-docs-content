---
slug: "data/auth"
title: "身份验证即服务"
description: "选择沙箱或部署的最终用户注册、登录和 JWT 颁发。"
section: "Databases & storage"
order: 4008
---

> **提示：可用**
>
> 托管最终用户身份验证已上线。
> 身份验证是**选择加入**并默认禁用。
> 为沙箱或部署启用它，MIOSA 在链接的托管数据库中提供身份验证模式。
> 运行时接收 `AUTH_URL`、`AUTH_JWT_SECRET` 和 `AUTH_DATABASE_URL`。

## 选择加入模型

默认情况下禁用身份验证。
在您启用之前，沙箱或部署没有托管注册界面。
这可以防止身份验证端点的意外暴露。

**通过API启用：**
```bash
curl -X POST https://api.miosa.ai/api/v1/project-auth/enable \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "resource_type": "sandbox",
    "resource_id": "'"$SANDBOX_ID"'",
    "config": {
      "signup_enabled": true,
      "email_confirm_required": false,
      "token_expiry_sec": 3600
    }
  }'
```

如果链接数据库仍在配置，端点将返回 `202` 和 `state: "pending"`。
轮询状态端点，直到身份验证处于活动状态：

```bash
curl \
  "https://api.miosa.ai/api/v1/project-auth/status?resource_type=sandbox&resource_id=$SANDBOX_ID" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

对于运行沙箱，MIOSA 尝试将身份验证变量同步到当前运行时。
部署更改遵循部署环境和发布生命周期。

---

Auth 是选择加入的托管身份验证体验。
启用后，MIOSA 在链接数据库中配置身份验证表。
该应用程序接收：
```bash

AUTH_URL=https://api.miosa.ai/api/v1/app-auth/sandbox/<sandbox-id>
AUTH_JWT_SECRET          # 每个项目一个 64 位十六进制值
AUTH_DATABASE_URL=postgresql://...
```

生成的应用程序如下使用它们：
```typescript
// Sign up a new end user
const res = await fetch(`${process.env.AUTH_URL}/signup`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ email, password }),
})
const { access_token, user } = await res.json()

// Verify a JWT in the app's own API
import jwt from "jsonwebtoken"
const claims = jwt.verify(token, process.env.AUTH_JWT_SECRET!)
const userId = claims.sub
```

这就是智能体在零日的体验：智能体在零日编写注册流程，永远不需要要求用户配置身份验证。

## 提供什么

|特色 |状态 |
|---|---|
|邮箱+密码注册/登录|可用 |
| JWT 访问令牌 |可用 |
|令牌验证 |可用 |
|邮箱验证 |可用（需要配置电子邮件传送）|
|密码重置 |可用（需要配置电子邮件传送）|
|组织单点登录 (SAML) |可用（按组织配置；对成员强制 SSO）|

今天不提供：刷新令牌、魔术链接、OAuth 提供商（例如 Google 和 GitHub）、MFA 和 TOTP，以及密钥和 WebAuthn。规划上述流程，并在访问令牌之外保留您自己的会话生命周期。有关完整列表，请参阅[已知限制](/docs/known-limitations)。

## 端点
`AUTH_URL` 已指向特定于资源的基址 (`.../app-auth/<type>/<id>`)。
附加操作路径：
```http
POST   {AUTH_URL}/signup                   create an end-user account → JWT
POST   {AUTH_URL}/login                    authenticate → JWT
POST   {AUTH_URL}/verify                   verify a JWT, returns the claims
POST   {AUTH_URL}/password-reset           request a reset link (sends email*)
POST   {AUTH_URL}/password-reset/confirm   complete the reset with the token
POST   {AUTH_URL}/email/confirm            confirm an email with the token
```
\* `/password-reset` 和 `/email/confirm` 仅在电子邮件送达后才发送邮件
配置；见下文。不管怎样，端点都会成功；电子邮件是一部分
需要设置。

## 配置电子邮件发送

密码重置和电子邮件验证生成令牌并调用发送，但是
电子邮件本身只有在电子邮件提供商连接后才会发出。 **在那之前，
这些端点返回成功，但没有发送电子邮件**（平台日志
`email_not_configured` 并跳过它；它永远不会使身份验证流程崩溃）。

MIOSA 目前通过运营商管理的交付提供商发送内置身份验证电子邮件。
请参阅[事务性电子邮件](/docs/data/email)了解验证清单和白标签发件人边界。
当前公共合约中没有针对每个资源的自助服务发送者凭证。

> 如今，电子邮件从单一平台范围的地址发送。每个项目和
> 自定义域发件人，因此每个白标品牌的电子邮件都来自自己的
> 域名，今天不提供。

## 最终用户不是 MIOSA 用户

重要区别：

- **MIOSA 平台用户** = 开发者/平台构建者。使用 `msk_*` API 密钥或 MIOSA 颁发的 JWT 会话进行身份验证。可以访问 MIOSA 仪表板。
- **生成的应用程序的最终用户** = 您的平台构建的应用程序的访问者（例如，登录到您的平台构建的应用程序的客户）。针对每个项目 `AUTH_URL` 进行身份验证。没有 MIOSA 帐户。

这些是完全独立的身份验证系统。同一个电子邮件地址可以是 MIOSA 平台用户和某人应用程序的最终用户；它们是不同的记录，具有不同的密码哈希值。

## 每个项目的隔离

每个启用的资源都会收到一个隔离的身份验证范围和 JWT 密钥。
为一种资源颁发的令牌无法针对不同的资源进行身份验证。

这使得“最终用户注册生成的应用程序”模式即使在同一个 MIOSA 租户托管数千个独立的最终用户应用程序时也是安全的。

## 行级安全性（Postgres 模式）

当与托管 [Postgres](/docs/data/postgres) 结合使用时，身份验证 JWT 携带用户 ID（`sub` 声明）。您的运行时代码使用它进行行级过滤：
```typescript
const claims = jwt.verify(token, process.env.AUTH_JWT_SECRET!)

const orders = await pool.query(
  "SELECT * FROM orders WHERE customer_id = $1",
  [claims.sub]
)
```

或者使用 Postgres RLS 策略（如果您的 Postgres 启用了它们）：
```sql
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
CREATE POLICY order_owner ON orders
  USING (customer_id = current_setting('jwt.claims.sub')::uuid);
```

## 带上你自己的授权？是的。跳过托管身份验证，设置您自己的 `AUTH_URL` 和 `AUTH_JWT_SECRET` 指向 Clerk / Auth0 / 您自己的服务。 MIOSA 的运行时会注入您设置的任何内容。

## Python SDK

**Python**

```python
from miosa import Miosa

miosa = Miosa(api_key="msk_u_...")

# Get current managed-auth status and config
status = miosa.project_auth.status(
    resource_type="sandbox",
    resource_id=sandbox_id,
)
print(status)  # {"enabled": false, ...}

# Enable project auth (with optional config kwargs)
miosa.project_auth.enable(
    resource_type="sandbox",
    resource_id=sandbox_id,
    signup_enabled=True,
)

# Update config after enabling
miosa.project_auth.update(
    resource_type="sandbox",
    resource_id=sandbox_id,
    signup_enabled=False,
)

# Disable when no longer needed
miosa.project_auth.disable(
    resource_type="sandbox",
    resource_id=sandbox_id,
)
```

## 另请参阅

- [概述](/docs/data/overview) - 更广泛的数据平面
- [Postgres](/docs/data/postgres) - 与 RLS 身份验证自然配对
- [平台/浏览器令牌](/docs/platform/browser-tokens) - 用于 MIOSA 级浏览器访问；与您的应用程序的最终用户身份验证不同
