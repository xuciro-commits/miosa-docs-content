---
slug: "platform/templates-and-sizing"
title: "模板与规格"
description: "为每位用户的机器选择模板和规格，限制并发，并设置空闲默认值，让闲置机器暂停。"
section: "Platform builders"
order: 9004
---

你为用户创建的每台机器都需要一个模板和一个规格。
本页介绍平台构建者需要设置的默认值，以及完整资源合同的所在位置。

## 模板

平台通常用同一个模板启动每位用户的机器。

**TypeScript**

```typescript
const sandbox = await miosa.sandboxes.create({
  templateId: "miosa-sandbox",
  workspaceId: workspace.id,
  externalUserId: user.id,
})
```

**Python**

```python
sandbox = client.sandboxes.create(
    template_id="miosa-sandbox",
    workspace_id=workspace.id,
    external_user_id=user.id,
)
```

## 规格与空闲默认值

规格、命名的资源合同和空闲自动暂停默认值对所有沙箱统一记录在以下页面：

- [规格与限制](/docs/develop/sandboxes/sizing)：规格名称、资源范围和空闲自动暂停默认值。
- [超时](/docs/develop/sandboxes/timeouts)：`timeout_sec`、`always_on` 和 `idle_timeout_sec`。
- [沙箱生命周期](/docs/develop/sandboxes/lifecycle)：暂停、恢复和持久化。

限制并发数和运行时长，并设置空闲超时，让闲置机器暂停。

## 配额与上限

每位最终用户一台机器。
设置每用户限制，避免单个账户耗尽组织容量：[配额](/docs/api-reference/quotas)。

## 成本与计费

每个用量事件都会携带工作区、项目以及你的 `external_*` ID。
按你用于计费的任一维度分组。

```typescript
const report = await miosa.usage.report({
  period: { start: "2026-05-01", end: "2026-06-01" },
  groupBy: ["external_workspace_id"],
})

for (const row of report.data) {
  await charge(row.external_workspace_id, row.total_usd)
}
```

MIOSA 会向组织开具一张包含所有项目的发票。
MIOSA 不向你的客户计费。
加价幅度由你决定，并在你自己的系统中向他们收费。
使用 `miosa.webhooks.create` 订阅经过签名的生命周期事件，并在处理投递前验证 `x-miosa-signature` HMAC（原始正文的 SHA-256）。
请参阅[用量与计费](/docs/platform/usage-and-billing)和 [Webhook](/docs/integrations/webhooks)。

首次创建时，将 `workspace_id` 和 `project_id` 与你自己的 ID 一起持久化保存。
每个周期查询用量，并执行你自己的支出上限。

## 下一步

- [规格与限制](/docs/develop/sandboxes/sizing)：命名的资源合同和空闲自动暂停默认值。
- [配额](/docs/api-reference/quotas)：每用户限制，避免单个账户耗尽组织容量。
- [用量与计费](/docs/platform/usage-and-billing)：在单张 MIOSA 账单上实现按客户计量和费用分摊。
