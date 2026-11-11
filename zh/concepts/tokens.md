---
slug: "concepts/tokens"
title: "预览与分享令牌"
description: "短期预览令牌（mp_*）与可撤销的分享令牌（ms_*），让浏览器可以加载沙箱预览而无需暴露工作区密钥。"
section: "Optional"
order: 17012
---

## 预览代币

**预览令牌**（前缀 `mp_*`）是一种短暂的签名令牌，它授权浏览器在不接收工作区 API 密钥的情况下加载沙盒预览。

典型流程：

1. 您的后端使用沙箱 ID 和 TTL（最长 24 小时）调用 `POST /api/v1/sandboxes/{id}/preview-token`。
2. MIOSA 返回嵌入了令牌的 iframe 就绪 URL。
3. 浏览器加载该 URL，例如 `https://<slug>.sandbox.miosa.ai?mt=mp_...`。
4. MIOSA 的边缘验证令牌服务器端并将请求智能体到沙箱端口。令牌值永远不会授予更广泛的访问权限。

预览代币不可更新且不可转让。当 TTL 过期时，URL 将返回 `401`。您的后端创建一个新令牌来恢复访问。

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SBX/previews/$PREVIEW/share \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -d '{"ttl_seconds": 3600}'
```

[浏览器令牌参考→](/docs/platform/browser-tokens)

## 分享代币

**共享令牌**（前缀 `ms_*`）是用于沙箱预览的可撤销公共共享 URL - 类似于文档编辑器中的“共享链接”。

与预览令牌的主要区别：

| |预览标记 (`mp_*`) |分享代币(`ms_*`) |
|--|------------------------------------|----------------------|
|预期收件人 |您的经过身份验证的用户 |任何知道链接的人 |
|铸造者 |您的后端 |您的后端 |
|可撤销|是（删除令牌）|是（删除令牌）|
|密码保护 |没有 |可选|
|到期 |所需 TTL |可选|

共享令牌对于收件人在您的系统中没有用户帐户的演示、客户评论或协作预览非常有用。该链接可以作为纯 URL 嵌入 - 不需要 `Authorization` 标头。

[预览参考→](/docs/develop/previews)

## 下一步

- [租户配置](/docs/concepts/tenant-config)：预览域和错误页面品牌。
- [浏览器令牌参考](/docs/platform/browser-tokens)：铸造、范围和撤销。
- [核心概念](/docs/concepts)：一屏概述。
