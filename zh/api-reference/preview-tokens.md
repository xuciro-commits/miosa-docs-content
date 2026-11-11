---
slug: "api-reference/preview-tokens"
title: "预览令牌 API"
description: "为 iframe 嵌入生成短期签名令牌，而无需暴露 API 密钥。"
section: "API reference"
order: 12028
---

预览令牌 (`mp_*`) 是短暂的签名凭据，允许您从第三方前端将正在运行的沙箱嵌入到 `<iframe>` 中，而不会将您的 `msk_*` API 密钥暴露给浏览器。

基本路径：`/api/v1/sandboxes/{id}/preview-token`

> **注意**
>
> 预览令牌是单一资源。每个代币的范围仅限于其铸造的沙箱。令牌最多 24 小时后过期。

---

## 端点

|方法|路径|描述 |
|--------|------|-------------|
| `POST` | `/api/v1/sandboxes/{id}/preview-token` |铸造预览令牌 |

---

## 铸造预览代币

**`POST /api/v1/sandboxes/{id}/preview-token`**

生成与沙箱绑定的签名预览令牌。返回的 `url` 可以用作 iframe `src`。

### 授权

需要具有 `previews:write` 范围的 API 密钥或 JWT。

```http
Authorization: Bearer msk_...
```

### 请求正文

|领域|类型 |必填 |默认 |描述 |
|--------|------|----------|---------|-------------|
| `expires_in` |整数 |没有 | `3600` |令牌 TTL（以秒为单位）（最大 `86400`）|
| `scope` |字符串|没有 | `"read"` | `"read"` 或 `"interact"` |

```json
{
  "expires_in": 3600,
  "scope": "read"
}
```

### 响应 - `200 OK`

```json
{
  "token": "mp_eyJhbGciOiJIUzI1NiJ9...",
  "url": "https://sbx-abc123.sandbox.miosa.ai?mt=mp_eyJ...",
  "expires_at": "2026-05-26T04:00:00Z",
  "scope": "read"
}
```

|领域|类型 |描述 |
|--------|------|-------------|
| `token` |字符串|已签名的预览令牌 (`mp_<base64url>`) |
| `url` |字符串| iframe 就绪 URL，其令牌预嵌入为 `?mt=` 查询参数 |
| `expires_at` |字符串| ISO 8601 到期时间戳 |
| `scope` |字符串| `"read"` 或 `"interact"` |

### 范围

|范围 |描述 |
|--------|-------------|
| `read` |仅供查看。鼠标和键盘输入被阻止。 |
| `interact` |完整的鼠标/键盘输入转发到沙箱中。 |

### 错误

|状态 |代码|原因 |
|--------|------|--------|
| 404 | `sandbox_not_found`|沙箱不存在或属于不同的租户 |
| 403 | `forbidden`|租户不匹配|
| 422 | `invalid_scope`| `scope` 不是 `"read"` 或 `"interact"` |

---

## 示例

**curl**

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/preview-token \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{"expires_in": 3600, "scope": "read"}'
```

**Python**

```python
import miosa

client = miosa.Miosa()

token = client.sandboxes.preview_token(
    "sbx_abc123",
    expires_in=3600,
    scope="read",
)

print(token.url)  # embed this in your frontend
```

**打字稿**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

const token = await client.sandboxes.previewToken('sbx_abc123', {
  expiresIn: 3600,
  scope: 'read',
});

// Pass token.url to your frontend  -  safe to expose in the browser
console.log(token.url);
```

---

## 令牌格式

预览令牌使用前缀 `mp_` 后跟 base64url 编码的签名有效负载。当作为 `?mt=` 查询参数传递时，沙箱智能体会消耗该令牌。不要解析令牌主体 - 将其视为不透明。

---

## Iframe 嵌入

```html
<!-- Read-only embed  -  no API key in the browser -->
<iframe
  src="https://sbx-abc123.sandbox.miosa.ai?mt=mp_eyJ..."
  width="1280"
  height="800"
  allow="clipboard-read; clipboard-write"
></iframe>
```

在令牌的生命周期内，沙箱智能体会针对每个请求验证令牌。一旦过期，iframe 将显示一个拒绝访问的页面。
