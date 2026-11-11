---
slug: "develop/sandboxes/timeouts"
title: "超时和自动停止"
description: "沙箱活动会话超时、如何延长它以及如何测量和报告运行时、CPU 和网络使用情况。"
section: "Sandboxes"
order: 2007
---

`timeout_sec` 是空闲锚定超时：它从最后一个活动开始计数。
默认值为 3600 秒，接受 1 到 86400 之间的值。
`POST /api/v1/sandboxes/{id}/extend` 替换该超时；它不会在先前的值上添加秒数。
省略的扩展主体保留当前超时。
仅当策略允许沙箱运行直至执行显式生命周期操作时，才设置 `always_on: true`。

## 自动停止语义

`timeout_sec` 从上次活动开始计算，而不是从会话开始时开始计算。
Exec 调用、文件操作、预览 HTTP 流量和终端标准输入都会重置它。
到期时：

- **持久**沙箱（`persistent: true`，默认值）停止并移动到 `stopped`。磁盘已保留，因此恢复会将其带回，并且 `/workspace` 和已安装的依赖项完好无损。正在运行的进程无法在停止后继续存在。计费从计算费率下降到冷存储费率。
- **非持久性**沙箱 (`persistent: false`) 被破坏，磁盘被丢弃。

SOMA 沙箱尚不支持持久性，因此 SOMA 沙箱是一次性的，并在超时后被删除。

## 空闲自动停止并根据请求唤醒

`idle_timeout_sec` 在持久沙箱闲置后停止它。

新帐户（在 2026 年 10 月 11 日或之后创建）：不带 `idle_timeout_sec` 的沙箱在空闲 30 分钟后停止，并在下一个请求时唤醒。将 `idle_timeout_sec` 设置为 `0` 以禁用。现有帐户保持其当前行为（除非设置空闲超时）。

停止的持久沙箱**根据请求唤醒**：对其预览 URL 的请求将恢复它并最多保留 10 秒。如果此时沙箱尚未运行，则请求将返回 `503` 和 `Retry-After`，同时恢复在后台完成，因此重试会发现沙箱正在运行。在停止或暂停的持久性沙箱上执行、文件操作和部署首先恢复，然后运行。非持久性沙箱永远不会自动恢复：调用返回 `409`。首先检查授权，因此私有预览永远不会唤醒匿名流量，并且对未公开的端口的请求不会唤醒沙箱。

> **注意：延长，不要累积**
>
> `POST /api/v1/sandboxes/{id}/extend` 替换超时。它不会在之前的值上添加秒数，因此在扩展时计算新的绝对 TTL。省略正文将保留当前超时。请参阅[沙盒生命周期](/docs/develop/sandboxes/lifecycle)了解此触发的停止转换。

## 沙箱使用

`GET /api/v1/sandboxes/{id}/usage` 返回 `runtime_sec`、`provisioned_vcpu_ms`、`active_cpu_ms`、网络入口和出口字节、估计成本和超时可见性。
`runtime_sec` 和 `provisioned_vcpu_ms` 是分配测量。
在解释活动 CPU 或网络值之前使用 `measurement_status`：

|状态 |意义|客户行为|
|---|---|---|
| `measured` |该值是针对该沙箱测量的。 |正常显示和聚合。 |
| `unavailable` |没有可靠的测量可用。相关值为`null`。 |显示“不可用”；永远不要将其强制为零。 |
| `stale` |该值是最新保留的样本，但不是最新的。 |将其标记为陈旧并避免将其呈现为实时使用。 |

所配置的资源始终以 `measurement_status.provisioned_resources: "measured"` 进行报告。
`timeout_remaining_ms` 在开始之前或禁用超时强制时可以为 `null`。

> **注意：相关**
>
> [沙箱生命周期](/docs/develop/sandboxes/lifecycle)涵盖超时后持久沙箱的变化，[大小和限制](/docs/develop/sandboxes/sizing)涵盖`always_on`、`persistent`和`idle_timeout_sec`字段。
