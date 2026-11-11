---
slug: "api-reference/tenant-branding"
title: "租户品牌 API"
description: "自定义沙盒 404 和 502 错误页面上显示的徽标、名称和颜色。"
section: "API reference"
order: 12051
---

品牌 API 控制在自定义预览域中提供的智能体生成的错误页面（404、502、网关超时）的外观。设置您的徽标、产品名称、支持 URL 和主要强调色，以便最终用户看到您的品牌，而不是 MIOSA 的品牌。

基本路径：`/api/v1/tenant/branding`

> **注意**
>
> 品牌更改会立即从缓存中提供。不存在传播延迟。

---

## 端点

|方法|路径|描述 |
|--------|------|-------------|
| `GET` | `/api/v1/tenant/branding` |获取当前品牌 |
| `PUT` | `/api/v1/tenant/branding` |设置或更换品牌 |
| `DELETE` | `/api/v1/tenant/branding` |重置为 MIOSA 默认值 |

---

## 获得品牌

**`GET /api/v1/tenant/branding`**

### 授权

```http
Authorization: Bearer msk_...
```

### 响应 - `200 OK`

```json
{
  "branding": {
    "logo_url": "https://cdn.example.com/logo.png",
    "name": "Lumen",
    "support_url": "https://example.com/support",
    "primary_color": "#0066FF"
  }
}
```

如果未设置自定义品牌，则返回 `{"branding": null}`。

---

## 设置品牌

**`PUT /api/v1/tenant/branding`**

替换所有品牌字段。省略的字段将被清除为其默认值。

### 请求正文

将字段包裹在 `"branding"` 键下。

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `logo_url` |字符串|没有 |徽标图像的 HTTPS URL |
| `name` |字符串|没有 |产品或公司名称|
| `support_url` |字符串|没有 |错误页面上显示的链接以获取用户支持 |
| `primary_color` |字符串|没有 | CTA 按钮的十六进制颜色（例如 `"#0066FF"`）|

```json
{
  "branding": {
    "logo_url": "https://cdn.example.com/logo.png",
    "name": "Lumen",
    "support_url": "https://example.com/support",
    "primary_color": "#0066FF"
  }
}
```

### 响应 - `200 OK`

```json
{
  "branding": {
    "logo_url": "https://cdn.example.com/logo.png",
    "name": "Lumen",
    "support_url": "https://example.com/support",
    "primary_color": "#0066FF"
  }
}
```

---

## 删除品牌

**`DELETE /api/v1/tenant/branding`**

将所有品牌重置为 MIOSA 默认值。错误页面将显示 MIOSA 徽标和配色方案。

### 响应 - `200 OK`

```json
{
  "branding": null
}
```

---

## 错误

|状态 |代码|原因 |
|--------|------|--------|
| 404 | `tenant not found`|租户不存在 |
| 422 | 验证错误|字段验证失败（例如无效 URL、无效十六进制颜色）|

---

## 示例

**curl**

```bash
# Set branding
curl -X PUT https://api.miosa.ai/api/v1/tenant/branding \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{
    "branding": {
      "logo_url": "https://cdn.example.com/logo.png",
      "name": "Lumen",
      "support_url": "https://example.com/support",
      "primary_color": "#0066FF"
    }
  }'

# Get
curl https://api.miosa.ai/api/v1/tenant/branding \
  -H "Authorization: Bearer msk_u_..."

# Reset
curl -X DELETE https://api.miosa.ai/api/v1/tenant/branding \
  -H "Authorization: Bearer msk_u_..."
```

**Python**

```python
import miosa

client = miosa.Miosa()

client.tenant.set_branding(
    logo_url="https://cdn.example.com/logo.png",
    name="Lumen",
    support_url="https://example.com/support",
    primary_color="#0066FF",
)

branding = client.tenant.get_branding()
print(branding.name)
```

**打字稿**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

await client.tenant.setBranding({
  logoUrl: 'https://cdn.example.com/logo.png',
  name: 'Lumen',
  supportUrl: 'https://example.com/support',
  primaryColor: '#0066FF',
});

const branding = await client.tenant.getBranding();
console.log(branding?.name);
```
