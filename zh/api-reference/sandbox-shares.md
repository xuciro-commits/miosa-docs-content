---
slug: "api-reference/sandbox-shares"
title: "沙盒共享 API"
description: "创建和管理沙箱的公共共享 URL。"
section: "API reference"
order: 12027
---

沙箱共享是可公开访问的 URL，提供对正在运行的沙箱的只读访问权限。与预览令牌不同，共享 URL 的范围不限于 iframe，并且不需要接收者持有任何凭据。

共享令牌使用前缀 `ms_` 后跟 base64url 编码的有效负载。公共 URL 模式是：

```text
https://{sandbox-id}.sandboxes.miosa.ai?ms=ms_<token>
```

---

## 端点

|方法|路径|描述 |
|--------|------|-------------|
| `POST` | `/api/v1/sandboxes/{id}/shares` |创建分享网址 |
| `GET` | `/api/v1/sandboxes/{id}/shares` |列出活跃股票 |
| `DELETE` | `/api/v1/sandboxes/{id}/shares/{share_id}` |撤销分享 |

---

## 创建共享

**`POST /api/v1/sandboxes/{id}/shares`**

创建公共共享链接。 `share_url` 字段可以安全地分发 - 它包含令牌作为查询参数。

### 授权

```http
Authorization: Bearer msk_...
```

### 请求正文

|领域|类型 |必填 |默认 |描述 |
|--------|------|----------|---------|-------------|
| `expires_in` |整数 |没有 | `3600` | TTL（以秒为单位）|
| `scope` |字符串|没有 | `"read"` |必须是 `"read"`（仅受支持的值）|

```json
{
  "expires_in": 86400
}
```

### 响应 - `201 Created`

```json
{
  "share_id": "shr_01hx9m7n3k...",
  "share_url": "https://sbx-abc123.sandboxes.miosa.ai?ms=ms_eyJ...",
  "expires_at": "2026-05-27T12:00:00Z",
  "scope": "read"
}
```

|领域|类型 |描述 |
|--------|------|-------------|
| `share_id` |字符串|用于撤销的不透明共享标识符 |
| `share_url` |字符串|嵌入令牌的可公开访问的 URL |
| `expires_at` |字符串| ISO 8601 过期，或 `null` 无过期 |
| `scope` |字符串|始终`"read"` |

---

## 列出股票

**`GET /api/v1/sandboxes/{id}/shares`**

返回沙箱的所有活动（未撤销、未过期）共享。

### 响应 - `200 OK`

```json
{
  "data": [
    {
      "share_id": "shr_01hx9m7n3k...",
      "share_url": "https://sbx-abc123.sandboxes.miosa.ai?ms=ms_eyJ...",
      "expires_at": "2026-05-27T12:00:00Z",
      "scope": "read"
    }
  ]
}
```

---

## 撤销共享

**`DELETE /api/v1/sandboxes/{id}/shares/{share_id}`**

立即使共享失效。任何使用共享 URL 的访问者在撤销后都将收到 403。

### 响应 - `204 No Content`

无响应体。

---

## 错误

|状态 |代码|原因 |
|--------|------|--------|
| 404 | `sandbox_not_found`|沙盒不存在或租户错误 |
| 403 | `forbidden`|租户不匹配|
| 404 | `not_found`|共享 ID 不存在（删除时）|
| 422 | `invalid_scope`| `scope` 值不是 `"read"` |

---

## 示例

**curl**

```bash
# Create
curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/shares \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{"expires_in": 86400}'

# List
curl https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/shares \
  -H "Authorization: Bearer msk_u_..."

# Revoke
curl -X DELETE https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/shares/shr_01hx9m7n3k \
  -H "Authorization: Bearer msk_u_..."
```

**Python**

```python
import miosa

client = miosa.Miosa()

# Create
share = client.sandboxes.shares.create("sbx_abc123", expires_in=86400)
print(share.share_url)

# List
shares = client.sandboxes.shares.list("sbx_abc123")

# Revoke
client.sandboxes.shares.delete("sbx_abc123", share.share_id)
```

**打字稿**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// Create
const share = await client.sandboxes.shares.create('sbx_abc123', {
  expiresIn: 86400,
});
console.log(share.shareUrl);

// List
const shares = await client.sandboxes.shares.list('sbx_abc123');

// Revoke
await client.sandboxes.shares.delete('sbx_abc123', share.shareId);
```
