---
slug: "develop/sandboxes/events"
title: "沙盒活动"
description: "通过 SSE 订阅沙箱生命周期和活动事件，以监控智能体运行并检测意外退出。"
section: "Sandboxes"
order: 2008
---

`events.stream()` 端点发出生命周期和活动 SSE 事件
沙箱，对于监控智能体运行很有用。

**Python**

```python
for event in sbx.events.stream():
    print(event["type"], event.get("data"))
    # types: "sandbox.started", "sandbox.destroyed", "exec.started", "exec.completed", ...
```

**打字稿**

```typescript
for await (const event of sbx.events.stream()) {
  console.log(event.type, event.data);
}
```

**curl**

```bash
curl https://api.miosa.ai/api/v1/sandboxes/$SBX/events \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Accept: text/event-stream"
```

## 日志、指标和导出

每个沙箱都会暴露三个可观察表面：

|表面|端点|
|---|---|
|日志 | `GET /api/v1/sandboxes/{id}/logs` |
|每个沙箱指标 | `GET /api/v1/sandboxes/{id}/metrics` |
|生命周期事件流 | `GET /api/v1/sandboxes/{id}/events` - 此页面上的流 |

停止的沙箱磁盘可以导出为存档，而无需恢复。
`GET /api/v1/sandboxes/{id}/exports` 读取保存的磁盘，因此当沙箱为 `stopped` 时它可以工作并且不会启动计算。

> **注意：相关**
>
> [事件 API 参考](/docs/api-reference/events) 记录了完整的事件目录和负载形状，[生产清单](/docs/develop/sandboxes/production-checklist) 展示了如何将流连接到重试逻辑。
