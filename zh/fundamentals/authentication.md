---
slug: "fundamentals/authentication"
title: "验证"
description: "API 密钥、浏览器令牌和五种 MIOSA 令牌类型及其前缀、调用者和信任级别。"
section: "API reference"
order: 12002
---

每个 API 请求都带有一个标头：
```http
Authorization: Bearer <token>
```

该令牌是 **API 密钥** 或 **浏览器令牌**（作用域 JWT）。

## API 密钥

API 密钥具有前缀 `msk_` 后跟角色段：

|前缀 |角色 |使用 |
|---|---|---|
| `msk_u_...` |用户 |通用服务器端调用|
| `msk_a_...` |管理员 |操作员/平台工具|
| `msk_p_...` |平台|租户范围的自动化 |

**API 密钥仅在服务器端。** 切勿将 `msk_*` 密钥放入浏览器代码、移动应用程序或公共存储库中。

> **注意：关键曝光**
>
> 泄露的 `msk_*` 密钥授予完全工作区访问权限。立即在 `DELETE /api/v1/api-keys/{id}` 处撤销它并旋转。

## 浏览器令牌

对于浏览器端访问 - 嵌入预览、在 iframe 中打开终端 - 您的后端会创建一个短期作用域令牌并将其传递给客户端：

**Python**

```python
import miosa

client = miosa.Miosa(api_key="msk_u_...")

token = client.browser_tokens.create(
    scope="sandbox",
    resource_id="sbx_abc123",
    expires_in=3600,
)
# Pass token.value to the browser  -  never the msk_ key.
```

**打字稿**

```typescript
import { Miosa } from '@miosa/sdk';

const miosa = new Miosa({ apiKey: 'msk_u_...' });

const token = await miosa.browserTokens.create({
  scope: 'sandbox',
  resourceId: 'sbx_abc123',
  expiresIn: 3600,
});
// Pass token.value to the browser  -  never the msk_ key.
```

**curl**

```bash
curl -X POST https://api.miosa.ai/api/v1/browser-tokens \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{
    "scope": "sandbox",
    "resource_id": "sbx_abc123",
    "expires_in": 3600
  }'
```

浏览器令牌会过期、不可更新，并且仅限于单个资源。他们无法创造新资源。

有关完整参考，请参阅[身份验证](/docs/authentication)和[浏览器令牌](/docs/platform/browser-tokens)。

## 代币类型

MIOSA 发行五种不同的代币类型。每个覆盖不同的呼叫者和信任级别：

|代币|前缀 |来电者 | TTL |读/写|
|---|---|---|---|---|
| API 密钥 | `msk_*` |您的服务器 |直至撤销 |读+写|
|智威汤逊 | `ey...` |管理 UI/浏览器会话 | ~1 小时（刷新 7 天）|读+写|
|浏览器令牌 | （不透明）|您的用户的浏览器 |可配置（默认 1 小时）|只读、单一资源 |
|预览代币 | `mp_*` |浏览器 / `<MiosaPreview>` iframe |可配置（默认 1 小时）|只读、沙盒预览 |
|分享代币 | `ms_*` |任何人 - 无需帐户 |可配置|只读、沙盒预览 |

`msk_*` API 密钥是根凭证。所有其他令牌类型均由您的服务器在其下游铸造。切勿将 `msk_*` 键提供给浏览器或最终用户。

有关铸造流程、撤销和验证规则，请参阅[身份验证](/docs/authentication)和[预览令牌](/docs/develop/previews)。

- [身份验证](/docs/authentication)：API 密钥格式、浏览器令牌、SSE 票证和速率限制详细信息。
