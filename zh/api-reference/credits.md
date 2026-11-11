---
slug: "api-reference/credits"
title: "积分API"
description: "用于检查信用余额、交易历史记录和使用数据的 API 参考。"
section: "API reference"
order: 12046
---

MIOSA 使用基于信用的计费系统。积分用于计算时间和 AI API 调用。

基本路径：`/api/v1/credits`

---

## 获得平衡

**`GET /api/v1/credits/balance`**

返回租户的当前信用余额。

### 响应 - `200 OK`
```json
{
  "tenant_id": "550e8400-e29b-41d4-a716-446655440000",
  "balance_credits": 850,
  "lifetime_earned": 1000,
  "lifetime_spent": 150,
  "credit_expiry_at": "2026-10-08T00:00:00Z",
  "updated_at": "2026-04-11T15:00:00Z"
}
```

### 响应字段

| 字段 | 类型 | 描述 |
|--------|------|-------------|
| `tenant_id` | UUID |您的租户 ID |
| `balance_credits` | 整数 | 当前可用积分 |
| `lifetime_earned` |整数 |获得的总积分（购买+促销）|
| `lifetime_spent` | 整数 | 累计已花费积分 |
| `credit_expiry_at` | ISO 8601 | 最近的积分批次到期时间，或在没有任何已发放积分即将到期时为 `null`。购买的积分不会过期；见[积分到期](/docs/api-reference/credits#credit-expiry) |
| `updated_at` | ISO 8601 | 上次余额变化 |

如果尚不存在贷方余额记录，则返回零：
```json
{
  "tenant_id": "...",
  "balance_credits": 0,
  "lifetime_earned": 0,
  "lifetime_spent": 0,
  "credit_expiry_at": null,
  "updated_at": null
}
```
```bash
curl https://api.miosa.ai/api/v1/credits/balance \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 获取交易

**`GET /api/v1/credits/transactions`**

返回分页的信用交易历史记录。

### 查询参数

|参数|类型 |默认 |描述 |
|------------|------|---------|-------------|
| `page` |整数 | 1 |页码（从 1 开始）|
| `page_size` |整数 | 20 |每页项目（最多：100）|

### 响应 - `200 OK`
```json
{
  "transactions": [
    {
      "id": "uuid",
      "tenant_id": "uuid",
      "user_id": "uuid",
      "computer_id": "uuid",
      "type": "spend",
      "amount_credits": -5,
      "description": "LLM API call",
      "provider": "ollama",
      "model": "nemotron-3-super",
      "input_tokens": 1500,
      "output_tokens": 500,
      "cost_usd_micros": 250,
      "inserted_at": "2026-04-11T15:00:00Z"
    },
    {
      "id": "uuid",
      "tenant_id": "uuid",
      "user_id": "uuid",
      "computer_id": null,
      "type": "earn",
      "amount_credits": 1000,
      "description": "Business plan monthly credits",
      "provider": null,
      "model": null,
      "input_tokens": null,
      "output_tokens": null,
      "cost_usd_micros": null,
      "inserted_at": "2026-04-01T00:00:00Z"
    }
  ],
  "total": 47,
  "page": 1,
  "page_size": 20
}
```

### 交易字段

| 字段 | 类型 | 描述 |
|--------|------|-------------|
| `id` | UUID |交易ID |
| `tenant_id` | UUID |租户 ID |
| `user_id` | UUID |触发交易的用户 |
| `computer_id` | UUID |关联的计算机（如果适用）|
| `type` |字符串| `"earn"` 或 `"spend"` |
| `amount_credits` |整数 |积分（正=赚取，负=花费）|
| `description` |字符串|人类可读的描述 |
| `provider` | 字符串 | LLM 提供商（用于 AI 交易）|
| `model` | 字符串 | 模型名称（用于 AI 交易）|
| `input_tokens` | 整数 | 使用的输入 token 数（用于 AI 交易）|
| `output_tokens` | 整数 | 生成的输出 token 数（用于 AI 交易）|
| `cost_usd_micros` | 整数 | 以微美元计的成本（1/1,000,000 USD）|
| `inserted_at` | ISO 8601 | 交易时间戳 |
```bash
curl "https://api.miosa.ai/api/v1/credits/transactions?page=1&page_size=50" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 获取使用摘要

**`GET /api/v1/credits/usage`**

返回按提供商和型号分组的每日使用情况汇总。

### 查询参数

|参数|类型 |默认 |描述 |
|------------|------|---------|-------------|
| `from` | ISO 8601 | 30 天前 | 日期范围的开始 |
| `to` | ISO 8601 | 现在 | 日期范围的结束 |

### 响应 - `200 OK`
```json
{
  "usage": [
    {
      "day": "2026-04-11T00:00:00Z",
      "provider": "ollama",
      "model": "nemotron-3-super",
      "credits_spent": 45,
      "input_tokens": 15000,
      "output_tokens": 5000,
      "cost_usd_micros": 2250,
      "call_count": 12
    },
    {
      "day": "2026-04-10T00:00:00Z",
      "provider": "ollama",
      "model": "nemotron-3-super",
      "credits_spent": 30,
      "input_tokens": 10000,
      "output_tokens": 3000,
      "cost_usd_micros": 1500,
      "call_count": 8
    }
  ],
  "from": "2026-03-12T00:00:00Z",
  "to": "2026-04-11T15:00:00Z"
}
```

### 用法输入字段

| 字段 | 类型 | 描述 |
|--------|------|-------------|
| `day` | ISO 8601 | 日期（截断到日）|
| `provider` | 字符串 | LLM 提供商 |
| `model` |字符串|型号名称 |
| `credits_spent` | 整数 | 当天花费的总积分 |
| `input_tokens` | 整数 | 输入 token 总数 |
| `output_tokens` | 整数 | 总输出 token 数 |
| `cost_usd_micros` | 整数 | 以微美元计的总成本 |
| `call_count` |整数 | API 调用次数 |

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 400 | `invalid 'from' timestamp` |无效的 RFC 3339 |
| 400 | `invalid 'to' timestamp` |无效的 RFC 3339 |
| 400 | `'from' must be before 'to'` |日期范围无效 |
```bash
curl "https://api.miosa.ai/api/v1/credits/usage?from=2026-04-01T00:00:00Z&to=2026-04-11T23:59:59Z" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 信用定价

MIOSA 按大小对沙箱和计算机进行定价：每种大小都有一个小时价格，在机器运行时按秒计费，并且价格不会随着机器的繁忙程度而变化。应用程序和数据库根据它们使用的 CPU 和内存进行计费，并且绝不会超过其大小的价格。一个积分始终等于一美分。没有每种产品的价格，也没有计划折扣。

### 尺寸价格

| 尺寸 | vCPU | 内存 | 每小时价格 | 始终开启（730 小时） |
|------|------|-----|--------------|-------------------|
| `micro` | 1 | 0.5 GiB | $0.0316 | $23.05 |
| `tiny` | 1 | 1 GiB | $0.0359 | $26.22 |
| `xs` | 1 | 2 GiB | $0.0446 | $32.58 |
| `small` | 2 | 4 GiB | $0.0893 | $65.16 |
| `quad` | 4 | 4 GiB | $0.1437 | $104.90 |
| `medium` | 4 | 8 GiB | $0.1785 | $130.32 |
| `large` | 8 | 16 GiB | $0.3570 | $260.65 |
| `xl` | 16 | 32 GiB | $0.7141 | $521.29 |

大小的价格为每 vCPU 小时 0.027216 美元，加上每 GiB 小时 RAM 0.0087075 美元，因此表中未包含的形状（例如 2 个 vCPU / 8 GiB）的成本相同。 `GET /api/v1/pricing` 返回带有 `usd_per_hour` 和 `usd_per_month` 的每个尺寸。

### 其他费率

|资源 |评分 |
|----------|------|
|磁盘（正在运行；也包括卷）| 0.09 美元/GiB 月（约 0.0123 积分/GiB 小时）|
|静态数据（暂停的磁盘、快照、备份、运行工件）| 0.018 美元/GiB 月（约 0.00247 积分/GiB 小时）|
|对象存储（存储桶、静态站点）| 0.0162 美元/GiB 月（约 0.00222 积分/GiB 小时）|
| 网络出口 | 5.4 积分 / GB（无免费额度）|
|构建 |免费|
| 最优 token | $1.90 / 1M 输入，$8.00 / 1M 输出 |

**计费基础。** 沙箱或计算机按其大小的价格计费，每秒计量，最少 1 秒，并根据您的信用余额在 5 分钟内结算。停止或暂停的计算机不会产生计算费用。内存和磁盘为 GiB（基于 1024）。

### 计划

| 计划 | 价格 | 包含积分 |
|------|--------|------------------|
| Free | $0/月 | 每月 $1 的用量（100 积分）|
| Developer | $20/月 | 2,000 积分/月 |
| Business | $500/月 | 50,000 积分/月 |
| Enterprise | $2,000/月 | 200,000 积分/月 |

每个计划为尺寸支付相同的价格。计划仅在上限和并发数上有所不同，而在价格上没有差异。免费包括每月 1 美元的使用费；随时添加学分。只需为您使用的内容付费。请参阅[定价页面](https://miosa.ai/pricing)了解实时数字和成本计算器。

### 信用到期

有效期取决于积分的授予方式：

- **您添加的积分**不会过期。
- **定期计划积分** 在授予积分的 30 天计费期后的 UTC 日开始时到期。
- **促销积分**（一次性欢迎赠款和促销代码兑换）在授予后 180 天到期。

最近的到期时间报告为 `GET /api/v1/credits/balance` 上的 `credit_expiry_at`。请参阅[定价和限制](/docs/pricing)了解如何计量和计费。

---

## 另请参阅

- [API 参考概述](/docs/api-reference) - 基本 URL 和身份验证
- [错误代码](/docs/api-reference/errors) - `INSUFFICIENT_CREDITS`错误处理
- [工作空间](/docs/api-reference/workspaces) - 工作空间和租户管理
