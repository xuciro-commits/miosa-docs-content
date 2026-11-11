---
slug: "services/metrics"
title: "服务指标"
description: "每个服务的请求、错误和延迟，以及这些序列不衡量哪些内容。"
section: "Services & projects"
order: 1017
---

每个服务都会报告自己的请求序列：请求数、错误数、错误率，以及 p50 和 p95 响应延迟。

```bash
miosa service metrics lumen API
miosa service metrics lumen API --range 24h --json
```

同样的序列也可通过 `GET /api/v1/projects/:id/services/:sid/metrics/series?range=` 获取，`range` 支持 `15m`、`1h`、`6h`、`24h` 或 `7d`，所需作用域为 `projects:read`。

## 如实解读

服务端会对序列分桶，并报告 `step_seconds`：`15m` 和 `1h` 为 60，`6h` 为 300，`24h` 为 900，`7d` 为 3600。即使窗口内没有活动，也会返回完整时间线；没有发生任何事情的位置字段为 null。`requests` 为 null 时该数据点为空，CLI 会显示短横线而非零。

`available` 指明实际采集的指标。对于服务，只有 `requests` 和 `latency`；**服务不会采集 CPU、内存和网络指标**，因此 CLI 会在表格下方说明这一点，而不是打印空列。

> **注意：Web 性能指标**
>
> 对于包含信标的静态服务和 Web 服务，会采集真实用户 Web 性能指标（LCP、INP、FCP、TTFB、CLS），并通过 `web_vitals` 和 `available.web_vitals` 返回。这些指标单独报告，因为它们衡量的是浏览器而非服务器。
