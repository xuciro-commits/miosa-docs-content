---
slug: "develop/sandboxes/production-checklist"
title: "生产清单"
description: "在生产中依赖沙箱之前，默认设置沙箱：超时、持久性、幂等性、模板、秘密和事件订阅。"
section: "Sandboxes"
order: 2012
---

在生产中依赖沙箱之前：

- 设置显式的 `timeout_sec`。交互式应用程序构建工作流程通常应使用 `1h`；短的一次性命令可以使用更少。
- 保留智能体工作区的默认 `persistent=True`，以便超时保留 `/workspace` 和依赖项安装。
- 为面向用户的开发环境设置 `idle_timeout_sec`，以便废弃的会话停止计算并保留状态。
- 在智能体重试循环中传递 `idempotency_key` 上的 `sandboxes.create` 呼叫。该平台会在 24 小时窗口内对使用相同密钥的创建进行重复数据删除，并且该密钥还使不确定的放置结果可重试：API 会回答可重试的 `503 SANDBOX_PLACEMENT_RECONCILING`，而不是终端 `409`。
- 使用已发布的模板，而不是为每个沙箱重新安装相同的依赖项。
- 当您需要当前运行状态的独立分支时，在破坏性操作之前分叉。
- 订阅`sbx.events.stream()`以检测意外退出并触发重试。
- 切勿将长期存在的秘密存储在 `env` 中；每次执行时传递它们或使用 Secrets API。
- 按列表调用中的 `external_workspace_id` 进行过滤，以避免扫描整个租户的沙箱集。

> **注意：相关**
>
> [超时和自动停止](/docs/develop/sandboxes/timeouts)解释了`timeout_sec`和使用情况测量，[大小调整和限制](/docs/develop/sandboxes/sizing)解释了`persistent`和`always_on`，[沙盒事件](/docs/develop/sandboxes/events)涵盖了事件流。
