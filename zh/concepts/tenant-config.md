---
slug: "concepts/tenant-config"
title: "租户配置"
description: "租户预览域和租户品牌 - 替换 MIOSA 后备预览域和最终用户看到的错误页面的 apex 主机名。"
section: "Optional"
order: 17010
---

## 租户预览域

**租户预览域** 是一个 apex 主机名，它替换最终用户看到的每个沙箱预览 URL 中的 MIOSA 后备预览域。它是按租户设置的。

当您设置并验证预览域（例如 `yourproduct.com`）时，租户的沙盒预览将在以下位置提供：

```text
https://<sandbox-slug>.yourproduct.com
```

设置是两步 DNS 验证：添加指向 `proxy.miosa.ai` 的通配符 CNAME (`*.yourproduct.com`)，然后调用 `POST /api/v1/tenant/preview-domain/verify`。 CNAME 解析后，MIOSA 会自动配置通配符 TLS 证书。仅接受顶级域 - 拒绝子域和协议前缀。

每个租户一个预览域。该域适用于该租户下的所有沙箱。

```bash
# Set
curl -X PUT https://api.miosa.ai/api/v1/tenant/preview-domain \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{"preview_domain": "yourproduct.com"}'

# Verify
curl -X POST https://api.miosa.ai/api/v1/tenant/preview-domain/verify \
  -H "Authorization: Bearer msk_u_..."
```

[租户预览域 API →](/docs/api-reference/tenant-preview-domain)

## 租户品牌

**租户品牌** 控制最终用户在 MIOSA 呈现的错误页面上看到的内容 - 当沙箱不可用、预览 URL 无效或自定义域尚未验证时显示的 404、502 和网关超时页面。

可配置字段：

|领域|描述 |
|--------|-------------|
| `logo_url` |徽标图像的 HTTPS URL |
| `name` |产品或公司名称|
| `support_url` |错误页面上显示用于最终用户支持的可选链接 |
| `primary_color` | CTA 按钮的十六进制颜色（例如 `#0066FF`）|

品牌通过 `PUT /api/v1/tenant/branding` 设置并立即从缓存中提供 - 没有传播延迟。删除它会将错误页面重置为 MIOSA 默认值。

```bash
# Set branding
curl -X PUT https://api.miosa.ai/api/v1/tenant/branding \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{
    "branding": {
      "logo_url": "https://cdn.example.com/logo.png",
      "name": "Lumen",
      "primary_color": "#0066FF"
    }
  }'
```

[租户品牌 API →](/docs/api-reference/tenant-branding)

## 下一步

- [预览和共享令牌](/docs/concepts/tokens)：浏览器访问沙箱预览。
- [租户预览域 API](/docs/api-reference/tenant-preview-domain)：完整的请求架构。
- [核心概念](/docs/concepts)：一屏概述。
