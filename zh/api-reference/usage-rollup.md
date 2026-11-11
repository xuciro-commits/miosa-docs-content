---
slug: "api-reference/usage-rollup"
title: "使用情况汇总 API"
description: "按外部用户、项目或工作区汇总沙箱计算使用情况和支出。"
section: "API reference"
order: 12048
---

使用情况汇总端点聚合沙箱和计算机之间的计量使用情况，并按您选择的标识符进行分组。使用它来构建每用户计费仪表板、执行支出策略或将成本归因于项目。

基本路径：`/api/v1/usage`

> **注意**
>
> 每个请求的结果限制为 1000 行。对于高基数租户，缩小期限或使用更具体的 `group_by`。

---

## 端点

|方法|路径|描述 |
|--------|------|-------------|
| `GET` | `/api/v1/usage` |一段时间内的总使用量 |

---

## 获取使用情况汇总

**`GET /api/v1/usage`**

### 授权

```http
Authorization: Bearer msk_...
```

### 查询参数

|参数|类型 |必填 |默认 |描述 |
|------------|------|----------|---------|-------------|
| `group_by` |字符串|没有 | `external_user_id` |聚合维度 |
| `period` |字符串|没有 | `30d` |预定义时间段或 `custom` |
| `start` |字符串|没有 |  - | ISO 8601 启动（`period=custom` 时需要）|
| `end` |字符串|没有 |  - | ISO 8601 结束（`period=custom` 时需要）|

**`group_by` 值：**

|价值|描述 |
|--------|-------------|
| `external_user_id` |按您在沙箱上设置的 `external_user_id` 分组 |
| `external_project_id` |按 `external_project_id` | 分组
| `workspace_id` |按 MIOSA 工作区分组 |

**`period` 值：**

|价值|描述 |
|--------|-------------|
| `7d` |过去 7 天 |
| `30d` |过去 30 天 |
| `mtd` |本月至今（日历月，1 日重置）|
| `custom` |任意范围 - 需要 `start` 和 `end` |

### 响应 - `200 OK`

```json
{
  "period_start": "2026-04-26T00:00:00Z",
  "period_end": "2026-05-26T00:00:00Z",
  "results": [
    {
      "external_user_id": "user_42",
      "sandbox_seconds": 14400,
      "computer_seconds": 0,
      "storage_gb_hours": 2.5,
      "credit_cents": 312
    },
    {
      "external_user_id": "user_99",
      "sandbox_seconds": 3600,
      "computer_seconds": 7200,
      "storage_gb_hours": 0.0,
      "credit_cents": 108
    }
  ]
}
```

|领域|类型 |描述 |
|--------|------|-------------|
| `period_start` |字符串| ISO 8601 聚合窗口开始 |
| `period_end` |字符串| ISO 8601 聚合窗口结束 |
| `results[].{group_by}` |字符串|分组维度的值 |
| `results[].sandbox_seconds` |数量 |沙箱使用的 vCPU 秒数 |
| `results[].computer_seconds` |数量 |计算机使用的 vCPU 秒数 |
| `results[].storage_gb_hours` |数量 | GiB 小时的附加存储 |
| `results[].credit_cents` |数量 |总成本（以平台信用分计）|

结果按 `credit_cents` 降序排列。

---

## 错误

|状态 |代码|原因 |
|--------|------|--------|
| 422 | `invalid group_by`| `group_by` 不是可接受的值之一 |
| 422 | `invalid period`| `period` 不是可接受的值之一 |
| 422 | `invalid_custom_range` | 当 `period=custom` 时，`start` 或 `end` 缺失或不是合法的 ISO 8601 |

---

## 示例

**curl**

```bash
# 30-day rollup by user
curl "https://api.miosa.ai/api/v1/usage?group_by=external_user_id&period=30d" \
  -H "Authorization: Bearer msk_u_..."

# Custom range
curl "https://api.miosa.ai/api/v1/usage?group_by=external_user_id&period=custom&start=2026-05-01T00:00:00Z&end=2026-05-26T00:00:00Z" \
  -H "Authorization: Bearer msk_u_..."

# Group by project
curl "https://api.miosa.ai/api/v1/usage?group_by=external_project_id&period=mtd" \
  -H "Authorization: Bearer msk_u_..."
```

**Python**

```python
import miosa

client = miosa.Miosa()

# Monthly rollup per user
results = client.usage.rollup(
    group_by="external_user_id",
    period="30d",
)

for row in results:
    print(f"{row.external_user_id}: {row.credit_cents / 100:.2f} USD")
```

**打字稿**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

const { results } = await client.usage.rollup({
  groupBy: 'external_user_id',
  period: '30d',
});

for (const row of results) {
  console.log(`${row.externalUserId}: $${(row.creditCents / 100).toFixed(2)}`);
}
```
