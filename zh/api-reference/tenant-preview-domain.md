---
slug: "api-reference/tenant-preview-domain"
title: "租户预览域 API"
description: "为沙箱预览 URL（白标签）配置自定义 apex 域。"
section: "API reference"
order: 12052
---

预览域 API 允许您将自己的顶级域（例如 `example.com`）映射到沙箱预览 URL。配置后，可以通过 `{sandbox-slug}.{your-domain}` 访问在租户下创建的沙箱，而不是默认的 `*.sandbox.miosa.ai`。

> **注意**
>
> **授权说明：** 这些端点接受 `msk_*` API 密钥和 JWT 令牌。这对于白标签集成很重要，您可以通过后端服务以编程方式管理域。

> **注意**
>
> 仅接受顶级域（例如 `example.com`，而不是 `app.example.com` 或 `https://example.com`）。通配符和子域输入将被 `422 invalid_domain` 拒绝。

---

## 端点

|方法|路径|描述 |
|--------|------|-------------|
| `GET` | `/api/v1/tenant/preview-domain` |获取当前域名和验证状态 |
| `PUT` | `/api/v1/tenant/preview-domain` |设置或替换预览域 |
| `POST` | `/api/v1/tenant/preview-domain/verify` |重新检查 DNS 验证 |
| `DELETE` | `/api/v1/tenant/preview-domain` |删除自定义域 |

---

## 获取预览域名

**`GET /api/v1/tenant/preview-domain`**

返回配置的域及其 DNS 验证状态。

### 授权

```http
Authorization: Bearer msk_...
```

### 响应 - `200 OK`（已配置域）

```json
{
  "preview_domain": "example.com",
  "status": "configured",
  "dns_instructions": {
    "type": "CNAME",
    "name": "*.example.com",
    "value": "proxy.miosa.ai"
  }
}
```

### 响应 - `200 OK`（未配置）

```json
{
  "preview_domain": null,
  "default_domain": "miosa.ai",
  "status": "not_configured",
  "dns_instructions": null
}
```

|领域|类型 |描述 |
|--------|------|-------------|
| `preview_domain` |字符串\|空|配置的顶级域，或 `null` |
| `default_domain` |字符串|未设置自定义域时使用的后备域 |
| `status` |字符串| `"not_configured"`、`"pending_dns"` 或 `"configured"` |
| `dns_instructions` |对象\|空 |要在 DNS 注册商处添加的 CNAME 记录 |

---

## 设置预览域

**`PUT /api/v1/tenant/preview-domain`**

设置或替换自定义预览域。 DNS 验证是异步的 - 一旦 CNAME 正确解析，状态就会从 `pending_dns` 转换为 `configured`。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `preview_domain` |字符串|是的 |顶点域（例如 `"example.com"`）|

```json
{
  "preview_domain": "example.com"
}
```

### 响应 - `200 OK`

```json
{
  "preview_domain": "example.com",
  "status": "pending_dns",
  "dns_status": "pending",
  "dns_instructions": {
    "type": "CNAME",
    "name": "*.example.com",
    "value": "proxy.miosa.ai"
  }
}
```

### DNS 设置

在您的注册商处添加通配符 CNAME：

|类型 |名称 |价值|
|------|------|--------|
| `CNAME` | `*`（或`*.example.com`）| `proxy.miosa.ai` |

---

## 验证 DNS

**`POST /api/v1/tenant/preview-domain/verify`**

立即触发 DNS 重新检查。返回当前验证状态，无需等待后台轮询器。

### 响应 - `200 OK`

```json
{
  "preview_domain": "example.com",
  "status": "configured",
  "dns_status": "verified"
}
```

---

## 删除预览域

**`DELETE /api/v1/tenant/preview-domain`**

删除自定义域。沙盒预览 URL 恢复为 `*.sandbox.miosa.ai`。

### 响应 - `200 OK`

```json
{
  "preview_domain": null,
  "status": "removed"
}
```

---

## 错误

|状态 |代码|原因 |
|--------|------|--------|
| 400 | `preview_domain is required` |失踪的身体领域|
| 422 | `invalid_domain`|检测到协议前缀、通配符、路径或子域 |
| 404 | `tenant not found`|租户不存在 |

---

## 示例

**curl**

```bash
# Set
curl -X PUT https://api.miosa.ai/api/v1/tenant/preview-domain \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{"preview_domain": "example.com"}'

# Verify
curl -X POST https://api.miosa.ai/api/v1/tenant/preview-domain/verify \
  -H "Authorization: Bearer msk_u_..."

# Get
curl https://api.miosa.ai/api/v1/tenant/preview-domain \
  -H "Authorization: Bearer msk_u_..."

# Delete
curl -X DELETE https://api.miosa.ai/api/v1/tenant/preview-domain \
  -H "Authorization: Bearer msk_u_..."
```

**Python**

```python
import miosa

client = miosa.Miosa()

# Set
result = client.tenant.set_preview_domain("example.com")
print(result.dns_instructions)

# Poll until verified
import time
while True:
    domain = client.tenant.get_preview_domain()
    if domain.status == "configured":
        break
    time.sleep(30)
    client.tenant.verify_preview_domain()
```

**打字稿**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// Set
const result = await client.tenant.setPreviewDomain('example.com');
console.log(result.dnsInstructions);

// Get
const domain = await client.tenant.getPreviewDomain();
console.log(domain.status); // "pending_dns" | "configured"

// Force re-check
await client.tenant.verifyPreviewDomain();

// Remove
await client.tenant.deletePreviewDomain();
```
