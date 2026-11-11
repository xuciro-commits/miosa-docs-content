---
slug: "faq/pricing-and-limits"
title: "定价和限制"
description: "您需要支付的费用、暂停费用、指定大小、硬上限、费率限制和积分。"
section: "Help"
order: 16009
---

关于计费内容、大小和上限以及积分如何运作的简短回答。

## 我需要支付什么费用？

按其大小的每小时价格运行沙箱和计算机，按其使用的内容运行应用程序和数据库（绝不超过其大小的价格）、暂停存储、存储和出口以及构建。
完整列表位于[定价和限制](/docs/pricing)。

```bash
miosa usage my-box           # runtime, time left and estimated cost
```

## 暂停的沙箱会产生任何费用吗？

仅存储。当沙箱离开 `running` 时，计算计费将停止。
请参阅[大小调整和限制](/docs/develop/sandboxes/sizing)。

## 命名尺寸是多少？

`xs`（1 vCPU、2 GiB、10 GiB）到 `xl`（16 vCPU、32 GiB、80 GiB），其中 `small` 作为默认值。
请参阅[大小和限制](/docs/develop/sandboxes/sizing)。

## 什么是硬上限？

每个计划的并发沙箱（开发人员 10、商业 250、企业 500；通过支持升级或提高）、`timeout_sec` 1 到 86400、86,400 秒最大执行超时、500 MB 最大上传和 1 小时浏览器令牌 TTL。
请参阅[定价和限制](/docs/pricing)。

## 速率限制是多少？

按层每个 API 密钥每分钟 60 到 30,000 个请求，开发人员/商业/企业上每个工作区每分钟 2,500/25,000/100,000 个请求，`429` 和 `Retry-After` 出现溢出。
请参阅[速率限制](/docs/rate-limits)。

## 如何限制一位最终用户的支出上限？

[Quotas API](/docs/api-reference/quotas) 根据 `external_user_id` 设置 `max_sandboxes`、`max_concurrent`、`max_storage_gb` 和 `max_credit_cents`，在创建时强制执行。

## 我如何向我自己的客户收费？

MIOSA 会向您收取一次费用；您使用归因向客户收费。
[使用情况汇总 API](/docs/api-reference/usage-rollup) 按 `external_user_id`、`external_project_id` 或 `workspace_id` 分组。
请参阅[使用情况和计费](/docs/platform/usage-and-billing)。

## 积分会过期吗？

这取决于补助金。
您添加的积分不会过期。
定期计划积分在 30 天计费周期后的 UTC 日开始时到期。
促销积分（一次性欢迎赠款和促销代码兑换）在授予后 180 天到期。
最近的到期时间报告为 `GET /api/v1/credits/balance` 上的 `credit_expiry_at`。
请参阅[定价和限制](/docs/pricing)。
