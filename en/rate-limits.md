---
slug: "rate-limits"
title: "Rate limits"
description: "The per-workspace and per-endpoint API request limits, how to read the rate-limit headers, and how to back off on 429."
section: "API reference"
order: 12007
---

MIOSA rate-limits API requests per API key, per workspace, and per endpoint group. When you exceed a limit, the API returns HTTP `429` with a `Retry-After` header telling you how many seconds to wait before retrying.

This page covers request throttling only. Hard caps that are not about request rate - concurrent sandboxes, exec and upload limits, webhook retries, browser token TTL - and everything billable live on [Pricing and limits](/docs/pricing).

## Request limits

| Scope | Limit |
|---|---|
| Per API key | 60 to 30,000 requests per minute, depending on the key's tier |
| Per workspace | Your plan's public-API limit, in requests per minute: Developer 2,500, Business 25,000, Enterprise 100,000 |
| Per endpoint group | Static per-minute caps per endpoint family (for example 30,000 on the public API and 3,000 on event streams) |
| All `/egress/*` endpoints | Current limit returned in `X-RateLimit-*` headers |

Limits can vary by key and policy. Every response carries the current numbers in `X-RateLimit-*` headers, so read them rather than hardcoding the values above.

> **Tip: Higher limits on request**
>
> Contact support to raise a request limit for a heavy integration. Volume terms are negotiated per plan; see the [pricing page](https://miosa.ai/pricing).

## Handle a 429

```python
import time, requests

def call(url, **kwargs):
    while True:
        res = requests.get(url, **kwargs)
        if res.status_code != 429:
            return res
        time.sleep(int(res.headers.get("Retry-After", "1")))
```

Back off for the number of seconds in `Retry-After`, then retry. Do not retry in a tight loop; `429` is a signal to slow down, not to try harder.

On the opt-in runner endpoint there are two distinct `429`s: `rate_limited` is the per-key or per-tenant limit, and `runtime_busy` (with `Retry-After: 0`) means the runner is at its admission limit and you should retry the request.

## See also

- [Pricing and limits](/docs/pricing): Billable states, machine sizes, hard caps, and per-end-user quotas.
- [Errors](/docs/api-reference/errors): Error codes and the response envelope, including rate-limit errors.
- [Quotas API](/docs/api-reference/quotas): Cap a single end-user without touching the tenant limit.
