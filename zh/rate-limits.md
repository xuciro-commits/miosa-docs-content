---
slug: "rate-limits"
title: "速率限制"
description: "按工作区和按端点划分的 API 请求限制，如何读取速率限制响应头，以及遇到 429 时如何退避。"
section: "API reference"
order: 12007
---

MIOSA 按 API 密钥、工作区和端点组对 API 请求进行限流。超出限制时，API 会返回 HTTP `429`，并通过 `Retry-After` 响应头告知需要等待多少秒再重试。

本页只介绍请求限流。与请求速率无关的硬性上限——并发沙箱、exec 与上传限制、webhook 重试、浏览器令牌 TTL——以及所有计费相关内容，请参阅[定价与限制](/docs/pricing)。

## 请求限制

| 范围 | 限制 |
|---|---|
| 每个 API 密钥 | 每分钟 60 到 30,000 次请求，取决于密钥的套餐层级 |
| 每个工作区 | 套餐的公共 API 限制，单位为每分钟请求数：Developer 2,500，Business 25,000，Enterprise 100,000 |
| 每个端点组 | 每个端点系列固定的每分钟上限（例如公共 API 为 30,000，事件流为 3,000） |
| 所有 `/egress/*` 端点 | 当前限制通过 `X-RateLimit-*` 响应头返回 |

限制会因密钥和策略而异。每个响应都会通过 `X-RateLimit-*` 响应头携带当前数值，因此请读取这些响应头，不要硬编码上表中的值。

> **提示：可按需申请更高限制**
>
> 如果集成流量较大，可联系支持团队提高请求限制。用量条款按套餐协商；请参阅[定价页面](https://miosa.ai/pricing)。

## 处理 429

```python
import time, requests

def call(url, **kwargs):
    while True:
        res = requests.get(url, **kwargs)
        if res.status_code != 429:
            return res
        time.sleep(int(res.headers.get("Retry-After", "1")))
```

按照 `Retry-After` 中的秒数退避，然后重试。不要在紧密循环中反复重试；`429` 表示应放慢速度，而非加大尝试力度。

在选择启用的 runner 端点上，有两种不同的 `429`：`rate_limited` 表示按密钥或按租户的限制，`runtime_busy`（带有 `Retry-After: 0`）表示 runner 已达到准入上限，应重试请求。

## 另请参阅

- [定价与限制](/docs/pricing)：计费状态、机器规格、硬性上限以及每个最终用户的配额。
- [错误](/docs/api-reference/errors)：错误码和响应封装，包括速率限制错误。
- [配额 API](/docs/api-reference/quotas)：为单个最终用户设置上限，而不影响租户限制。
