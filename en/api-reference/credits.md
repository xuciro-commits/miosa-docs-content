---
slug: "api-reference/credits"
title: "Credits API"
description: "API reference for checking credit balance, transaction history, and usage data."
section: "API reference"
order: 12046
---

MIOSA uses a credit-based billing system. Credits are consumed for compute time and AI API calls.

Base path: `/api/v1/credits`

---

## Get Balance

**`GET /api/v1/credits/balance`**

Returns the current credit balance for your tenant.

### Response  -  `200 OK`
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

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `tenant_id` | UUID | Your tenant ID |
| `balance_credits` | integer | Current available credits |
| `lifetime_earned` | integer | Total credits ever earned (purchases + promos) |
| `lifetime_spent` | integer | Total credits ever spent |
| `credit_expiry_at` | ISO 8601 | Nearest credit-lot expiry, or `null` when no granted credits are due to expire. Purchased credits do not expire; see [Credit expiry](/docs/api-reference/credits#credit-expiry) |
| `updated_at` | ISO 8601 | Last balance change |

If no credit balance record exists yet, returns zeros:
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

## Get Transactions

**`GET /api/v1/credits/transactions`**

Returns paginated credit transaction history.

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Page number (1-based) |
| `page_size` | integer | 20 | Items per page (max: 100) |

### Response  -  `200 OK`
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

### Transaction Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Transaction ID |
| `tenant_id` | UUID | Tenant ID |
| `user_id` | UUID | User who triggered the transaction |
| `computer_id` | UUID | Associated computer (if applicable) |
| `type` | string | `"earn"` or `"spend"` |
| `amount_credits` | integer | Credits (positive = earned, negative = spent) |
| `description` | string | Human-readable description |
| `provider` | string | LLM provider (for AI transactions) |
| `model` | string | Model name (for AI transactions) |
| `input_tokens` | integer | Input tokens used (for AI transactions) |
| `output_tokens` | integer | Output tokens generated (for AI transactions) |
| `cost_usd_micros` | integer | Cost in microdollars (1/1,000,000 USD) |
| `inserted_at` | ISO 8601 | Transaction timestamp |
```bash
curl "https://api.miosa.ai/api/v1/credits/transactions?page=1&page_size=50" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Get Usage Summary

**`GET /api/v1/credits/usage`**

Returns daily usage rollups grouped by provider and model.

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `from` | ISO 8601 | 30 days ago | Start of date range |
| `to` | ISO 8601 | Now | End of date range |

### Response  -  `200 OK`
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

### Usage Entry Fields

| Field | Type | Description |
|-------|------|-------------|
| `day` | ISO 8601 | Date (truncated to day) |
| `provider` | string | LLM provider |
| `model` | string | Model name |
| `credits_spent` | integer | Total credits spent that day |
| `input_tokens` | integer | Total input tokens |
| `output_tokens` | integer | Total output tokens |
| `cost_usd_micros` | integer | Total cost in microdollars |
| `call_count` | integer | Number of API calls |

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `invalid 'from' timestamp` | Not valid RFC 3339 |
| 400 | `invalid 'to' timestamp` | Not valid RFC 3339 |
| 400 | `'from' must be before 'to'` | Invalid date range |
```bash
curl "https://api.miosa.ai/api/v1/credits/usage?from=2026-04-01T00:00:00Z&to=2026-04-11T23:59:59Z" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Credit Pricing

MIOSA prices sandboxes and computers per size: every size has one hourly price, billed per second while the machine runs, and the price does not change with how busy the machine is. Apps and databases are billed on the CPU and memory they use, and never more than the price of their size. One credit always equals one US cent. There are no per-product prices and no plan discounts.

### Size prices

| Size | vCPU | RAM | Price / hour | Always on (730 h) |
|------|------|-----|--------------|-------------------|
| `micro` | 1 | 0.5 GiB | $0.0316 | $23.05 |
| `tiny` | 1 | 1 GiB | $0.0359 | $26.22 |
| `xs` | 1 | 2 GiB | $0.0446 | $32.58 |
| `small` | 2 | 4 GiB | $0.0893 | $65.16 |
| `quad` | 4 | 4 GiB | $0.1437 | $104.90 |
| `medium` | 4 | 8 GiB | $0.1785 | $130.32 |
| `large` | 8 | 16 GiB | $0.3570 | $260.65 |
| `xl` | 16 | 32 GiB | $0.7141 | $521.29 |

A size's price is $0.027216 per vCPU-hour plus $0.0087075 per GiB-hour of RAM, so a shape that is not in the table (for example 2 vCPU / 8 GiB) costs the same formula. `GET /api/v1/pricing` returns every size with `usd_per_hour` and `usd_per_month`.

### Other rates

| Resource | Rate |
|----------|------|
| Disk (running; also volumes) | $0.09 / GiB-month (about 0.0123 credits / GiB-hour) |
| Data at rest (paused disk, snapshots, backups, run artifacts) | $0.018 / GiB-month (about 0.00247 credits / GiB-hour) |
| Object storage (buckets, static sites) | $0.0162 / GiB-month (about 0.00222 credits / GiB-hour) |
| Network egress | 5.4 credits / GB (no free allowance) |
| Builds | Free |
| Optimal tokens | $1.90 / 1M input, $8.00 / 1M output |

**Billing basis.** A sandbox or computer is billed the price of its size, metered per second with a 1-second minimum, and settled against your credit balance on a 5-minute tick. A stopped or paused machine is not billed for compute. Memory and disk are GiB (1024-based).

### Plans

| Plan | Price | Included credits |
|------|-------|------------------|
| Free | $0/month | $1 of usage each month (100 credits) |
| Developer | $20/month | 2,000 credits/month |
| Business | $500/month | 50,000 credits/month |
| Enterprise | $2,000/month | 200,000 credits/month |

Every plan pays the same price for a size. Plans differ only in caps and concurrency, never in price. Free includes $1 of usage each month; add credits any time. Pay only for what you use. See the [pricing page](https://miosa.ai/pricing) for the live numbers and the cost calculator.

### Credit expiry

Expiry depends on how the credits were granted:

- **Credits you add** do not expire.
- **Recurring plan credits** expire at the start of the UTC day after the 30-day billing period in which they were granted.
- **Promotional credits** (the one-time welcome grant and promo-code redemptions) expire 180 days after they are granted.

The nearest expiry is reported as `credit_expiry_at` on `GET /api/v1/credits/balance`. See [Pricing and limits](/docs/pricing) for how usage is metered and billed.

---

## See also

- [API Reference overview](/docs/api-reference)  -  base URL and authentication
- [Error Codes](/docs/api-reference/errors)  -  `INSUFFICIENT_CREDITS` error handling
- [Workspaces](/docs/api-reference/workspaces)  -  workspace and tenant management
