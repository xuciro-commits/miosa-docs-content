---
slug: "faq/security-and-compliance"
title: "安全性和合规性"
description: "SOC 2、HIPAA、数据所在位置、机密存储方式、保留和可发布密钥。"
section: "Help"
order: 16008
---

关于认证、数据位置、秘密和保留的简短回答。

## 您是否符合 SOC 2 标准？

还没有。我们的 SOC 2 审核正在进行中，尚未完成，因此今天没有可分享的报告。
在发布之前，我们可以根据 NDA 共享我们的安全调查问卷和控制文档。
请参阅 [Trust](https://miosa.ai/trust) 了解当前状态。

## 您支持 HIPAA 吗？

MIOSA 为医疗保健工作负载运行符合 HIPAA 的基础设施，并且业务伙伴协议 (BAA) 可供企业客户使用。
请参阅[信任](https://miosa.ai/trust)。

## 我的数据存放在哪里？

在美国的数据中心，我们直接提供、联网和管理； MIOSA 不是超大规模提供商的经销商。
三个活动区域均位于美国，资源创建后无法在它们之间移动。
请参阅[区域](/docs/regions)和[信任](https://miosa.ai/trust)。

## 秘密是如何存储的？

值使用 ChaCha20-Poly1305 加密。
沙箱看到一个不透明的 `miosa-tok-<hex>` 占位符，该占位符在出口处交换，因此任何人（包括 MIOSA 工程师）都可以重新读取存储的值。
请参阅[秘密](/docs/security/secrets)。

## 数据保留多长时间？

出口审核事件保留 90 天。
使用记录和信用记录分开保存。
没有公开的零保留模式。
请参阅[数据保留](/docs/data-retention)。

## 浏览器有可发布的密钥吗？

不。没有 `pk_*` 可发布的密钥：API 密钥仅适用于服务器。
浏览器和嵌入访问使用短期作用域令牌（`mp_` 预览、`ms_` 共享和浏览器令牌）。
请参阅[API 密钥](/docs/platform/api-keys) 和[浏览器令牌](/docs/platform/browser-tokens)。
