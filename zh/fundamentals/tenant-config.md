---
slug: "fundamentals/tenant-config"
title: "租户配置"
description: "白标集成商的租户级 API 界面 - Webhook 订阅和每最终用户配额。"
section: "API reference"
order: 12006
---

白标集成商通过下面的 API 界面配置租户行为。所有路由都接受 `msk_a_*` 管理密钥或管理 JWT。

预览域和品牌是产品模型的一部分，而不是本集成商指南的一部分：[核心概念](/docs/concepts#tenant-preview-domain)解释了它们是什么以及何时使用它们，[租户预览域](/docs/api-reference/tenant-preview-domain)和[租户品牌](/docs/api-reference/tenant-branding)参考给出了请求架构。

## 网络钩子

订阅作为 HTTPS POST 请求发送到端点的沙箱和部署生命周期事件。 MIOSA 在 `X-Miosa-Signature` 标头中使用 HMAC-SHA256 签名对每个请求进行签名：

```bash
POST /api/v1/tenant/webhooks
{ "url": "https://yourapp.com/webhooks/miosa", "events": ["sandbox.*", "deployment.*"] }
```

有关事件目录和签名验证，请参阅 [Webhooks](/docs/api-reference/webhooks) 和 [Integrations](/docs/integrations)。

## 每最终用户配额

设置每个 `external_user_id` 的最大资源计数，以防止单个下游用户失控使用。配额在资源创建时进行检查，并在达到时返回 `429` 和代码 `QUOTA_EXCEEDED`：

```bash
PUT /api/v1/tenant/quotas
{ "max_sandboxes_per_user": 5, "max_computers_per_user": 2 }
```

请参阅[配额](/docs/api-reference/quotas)。
