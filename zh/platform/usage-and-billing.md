---
slug: "platform/usage-and-billing"
title: "用量与计费"
description: "按工作区、项目、外部客户和用户跟踪 MIOSA 用量，并向你的客户进行成本分摊。"
section: "Organization & billing"
order: 13005
---

如果你是一个转售 MIOSA 容量的平台，就需要将 MIOSA 成本归集到你自己的客户身上。MIOSA 会在每次可计费操作时发出带有[资源归属和外部归因](/docs/platform/attribution)信息的用量事件；你可以查询、分组，并按自己的货币/加价模型进行成本分摊。

> **注意：当前状态**
>
> 租户范围用量汇总、会话用量、摘要、工作区用量和审计日志端点均已上线。
> 响应中可用的具体维度取决于资源和计量来源。
> 缺失的测量值应视为不可用，而不是零。

## 计量内容

| 维度 | 单位 | 发出时机 |
|---|---|---|
| 沙箱运行时长 | 沙箱秒，按规格计价 | 运行时按秒 |
| 构建器运行时长 | 构建器秒 | 发布期间 |
| 运行时实例运行时长 | 运行时秒，按 CPU 和内存计量，上限为对应规格价格 | 提供服务期间按秒 |
| 静态产物存储 | GB-月 | 持续 |
| 静态产物带宽 | GB 出口流量 | 每次响应 |
| 对象存储（存储桶） | GB-月 + GB 出口流量 | 持续/每次响应 |
| 托管 Postgres | GB-月 + 连接数 | 持续 |
| 托管 Redis | GB-月 | 持续 |
| 归属与归因：每个事件都携带 `workspace_id`、`project_id`，以及源资源上存在的 `external_*` 字段 |

计算机（桌面产品）有单独的计量模式；该部分请参阅[计算机](/docs/computers/overview)。

## 用量报告 API
```http
GET /api/v1/usage?period_start=2026-05-01&period_end=2026-06-01
GET /api/v1/usage?workspace_id=550e8400-e29b-41d4-a716-446655440000
GET /api/v1/usage?external_workspace_id=acct_123&period=2026-05
```

**TypeScript**

```typescript
const report = await miosa.usage.report({
  period: { start: "2026-05-01", end: "2026-06-01" },
  groupBy: ["external_workspace_id"],
})

for (const row of report.data) {
  console.log(
    row.external_workspace_id,
    "$" + row.total_usd.toFixed(2),
    row.line_items
  )
}
```

**Python**

```python
from miosa import Miosa

miosa = Miosa(api_key="msk_u_...")

# Grouped chargeback report for a billing period
report = miosa.usage.report(
    period_start="2026-05-01",
    period_end="2026-06-01",
    group_by=["external_workspace_id"],
)

for row in report.get("data", []):
    print(row["external_workspace_id"], f"${row['total_usd']:.2f}")

# Single-workspace query for the current period
summary = miosa.usage.report(
    external_workspace_id="acct_123",
)
print(summary)

# Raw per-session events
sessions = miosa.usage.sessions(external_workspace_id="acct_123")
for s in sessions:
    print(s["dimension"], s["qty"])
```

响应结构：
```json
{
  "data": [
    {
      "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
      "project_id": "660e8400-e29b-41d4-a716-446655440001",
      "external_workspace_id": "acct_123",
      "external_user_id": null,
      "external_project_id": null,
      "total_usd": 1.89,
      "line_items": [
        { "dimension": "sandbox_seconds", "qty": 36000, "usd": 0.36 },
        { "dimension": "static_bandwidth_gb", "qty": 38.2, "usd": 1.53 }
      ]
    },
    {
      "external_workspace_id": "acct_456",
      "total_usd": 11.75,
      "line_items": [...]
    }
  ],
  "period": { "start": "2026-05-01", "end": "2026-06-01" },
  "currency": "usd"
}
```

使用 `groupBy` 按归属和外部 ID 的任意组合进行切分。常见的分组方式：

- `["workspace_id"]`——按客户工作区
- `["workspace_id", "project_id"]`——按客户内部的应用/项目
- `["external_workspace_id"]`——按客户进行成本分摊
- `["external_workspace_id", "external_project_id"]`——按客户内部的应用
- `["external_user_id"]`——按用户归因（例如用于公平使用限制）
- `[]`——整个 MIOSA 组织的总计

## 审计日志

每次变更还会发出一个带归因信息的审计事件。使用不同的表，读取端点也单独提供：
```http
GET /api/v1/audit-log?workspace_id=550e8400-e29b-41d4-a716-446655440000
GET /api/v1/audit-log?external_workspace_id=acct_123
GET /api/v1/audit-log?api_key_id=550e8400-e29b-41d4-a716-446655440000
GET /api/v1/audit-log?action=deployments.publish
```

审计用于安全/合规，用量用于计费。两者共享相同的归属与归因模型。

## 配额与限制

MIOSA 在组织和计划级别执行配额。你可以在自己的应用逻辑中执行按工作区、按项目或按用户的限制，通过查询用量来做出决策。

示例：Lumen 希望将每个客户账户的上限设为每月 $50。他们的后端会：

1. 查询 `/api/v1/usage?workspace_id=550e8400-e29b-41d4-a716-446655440000&period=current_month`。
2. 如果 `total_usd &gt; 50`，则在 Lumen 应用中拒绝创建新沙箱。
3. 可选地，调用 MIOSA 端点暂停该客户的沙箱（目前没有单一端点；可在客户端通过销毁其沙箱来实现）。

MIOSA 还在 `/api/v1/quotas/external/{external_user_id}` 提供外部用户配额端点。
当调用方在达到配置的限制后应被阻止创建更多资源时，请使用这些端点。

## 定价

每种规格只有一个价格，所有计划相同；你的计划只决定限制，绝不影响价格。费率请参阅[定价](/docs/pricing)，实时价格表请使用 `GET /api/v1/pricing`。

## 计费租户关系

重要：你的 MIOSA 账单只有一份。MIOSA 向组织所有者收取所有用量的费用，无论用量来自哪个工作区/项目或 `external_workspace_id`。归属与归因的存在是为了让你向自己的客户收费。MIOSA 不直接向你下游的客户计费。
```text

MIOSA -> Lumen:        $X / month, one invoice
Lumen -> acct_123:   $Y / month (markup, your pricing model)
Lumen -> acct_456:   $Z / month
```

## 另请参阅

- [归属与归因](/docs/platform/attribution)——流经用量事件的各个 ID
- [API 密钥](/docs/platform/api-keys)——用于用量查询的凭证（`usage:read` 作用域）
- [审计](/docs/api-reference/events)——用于安全日志查询
