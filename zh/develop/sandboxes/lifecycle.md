---
slug: "develop/sandboxes/lifecycle"
title: "沙箱生命周期"
description: "沙箱状态机从创建到销毁，每个状态意味着什么，停止保留什么，重置空闲时钟什么，以及如何停止、暂停、恢复和重新启动。"
section: "Sandboxes"
order: 2004
---

沙箱会在一小部分状态之间移动。
此页面是地图：每个状态的含义、触发转换的因素以及如何在不丢弃磁盘的情况下停止和重新启动计算。

## 生命周期

create() 创建创建；创建进入准备就绪状态。

运行在 stop() 上停止，并在resume() 上恢复运行。

运行在暂停（）时进入暂停状态，并在恢复（）时恢复运行。

运行、停止或暂停都会在 destroy() 上销毁。

创建或运行失败时出错；清理时错误会被破坏。

|状态 |意义|
|---|---|
| `creating` | VM 正在被认领、恢复并准备接受命令 |
| `running` |命令就绪：执行、文件、预览、终端和端口公开工作 |
| `stopped` |机器释放，磁盘保存；文件可以生存，进程则不能|
| `paused` |机器悬挂在原地，内存和进程完好无损；仅在通告暂停的运行时 |
| `destroyed` |终端：资源已释放，ID 不可用，已保存状态已删除 |
| `error` |启动或运行时失败；检查 `sbx.data["metadata"]` 中的 `last_error` |

`paused` 仅存在于通告暂停的运行时上。
鞭炮今天给它做广告。
SOMA 没有：它没有暂停，并且在停止时也不保持状态，因此 SOMA 沙箱永远不会 `paused`，并且一次性 SOMA 沙箱在超时到期时将被删除。
快照是一种单独的资源，而不是沙箱状态。

重置空闲时钟的活动：exec 调用、文件操作、预览 HTTP 流量、终端标准输入。
超时是空闲锚定的：它从最后一个活动开始计算。
一旦 `timeout_sec` 过去且没有任何活动，持久沙箱就会停止。
非持久性沙箱 (`persistent: false`) 被破坏，其磁盘被丢弃。

`idle_timeout_sec` 在一段时间不活动后停止持久沙箱。
新帐户（2026 年 10 月 11 日或之后创建）：不带 `idle_timeout_sec` 的沙箱在空闲 30 分钟后停止，并在下一个请求时唤醒。
将 `idle_timeout_sec` 设置为 `0` 以禁用。
现有帐户保持其当前行为（除非设置空闲超时）。

`resume()` 不是退出 `stopped` 或 `paused` 的唯一方法：持久沙箱 **根据请求唤醒**。
在停止或暂停（例如，到达其预览 URL）时到达的请求将恢复该请求并保留最多 10 秒，然后如果尚未运行，则使用 `Retry-After` 应答 `503`。
在停止或暂停的持久性沙箱上执行、文件操作和部署首先恢复，然后运行。
非持久性沙箱永远不会自动恢复：调用返回 `409`。

## 停止保留什么

Stop保存磁盘并释放机器，并将沙箱移至`stopped`。
文件、已安装的软件包、Docker 数据和启用的服务在停止后仍然存在。
在任何运行时，正在运行的进程永远不会停止。
停止后恢复的行为类似于重新启动的服务器：再次启动进程，或将它们声明为服务。

暂停是保持进程的明确动词。
它仅适用于 Firecracker。
暂停的沙箱最多可驻留 30 分钟，且内存和进程完好无损。
30 分钟后，它的行为就像停止一样：磁盘保留，进程不保留。
内存检查点是一个加速器，但事实并非如此：如果无法恢复，则恢复会回退到磁盘。

磁盘大小是在创建时选择的固定类别（10、20 或 40 GiB）。
它只能作为 `resume` 或 `fork` 的参数来增长。
收缩请求被 `409` 拒绝，并且沙箱保持不变。
磁盘增长尚未在每个运行时都可用。

持久意味着沙箱在停止后仍然存在。
这并不意味着沙箱在失去主机后仍能幸存。
沙箱记录中的 `durability` 字段指示有多少主机保持已保存的状态：`local_only`、`replicating` 或 `replicated`。
每个 Firecracker 沙箱都会暴露 `durability`。
它是`local_only`，直到已停止的沙箱副本在主机外得到验证，然后是`replicated`。
脱离主机复制首先在 QA 金丝雀上推出，因此预计今天大多数沙箱都会使用 `local_only`。有关完整的暂停/恢复/分叉流程，请参阅[持久性、暂停和分叉](/docs/develop/snapshots)。

## 准备情况和计费

沙箱仅在 `running` 状态下才可执行命令。有两个信号告诉您它何时到达：

- 沙盒记录上的 `ready` 布尔值。 `true` 表示 VM 会话已准备好接受执行、文件、预览和终端调用。
- `state` 字段。轮询 `GET /api/v1/sandboxes/{id}` 直到 `state` 离开 `creating` 并变为 `running`（或曲面 `error`）。

融合的 [run 端点 ](/docs/develop/sandboxes/create) 在服务器上拥有此等待，因此创建和执行的智能体不需要客户端轮询循环。

哪些州的费用是多少：

|状态 |计费|
|---|---|
| `creating` |准备资源；尚未准备好命令|
| `running` |按其大小的价格按秒计费（每个计划都相同）|
| `stopped` |无需计算；仅冷存储速率（磁盘 GiB 秒）|
| `paused` |无需计算；仅存储速率（磁盘 GiB 秒）|
| `destroyed` |计费立即停止 |

请参阅[大小和限制](/docs/develop/sandboxes/sizing)了解完整的帐单说明，并参阅[超时和自动停止](/docs/develop/sandboxes/timeouts)了解截止日期时发生的情况。

## 暂停和恢复

`POST /stop` 和 `resume()` 释放机器并将其带回到同一磁盘上。
`pause()` 和 `resume()` 在通告暂停的运行时（仅限 Firecracker，最多 30 分钟，然后表现为停止）将机器挂起并保持其进程。

**Python**

```python
sbx.pause()
# Firecracker only: state is "paused" and processes are kept for up to 30 minutes

sbx.resume()
# state returns to "running"
```

**打字稿**

```typescript
// Firecracker only: keeps processes for up to 30 minutes
await sbx.pause();
await sbx.resume();
```

**curl**

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SBX/stop \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'

curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SBX/resume \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## 重新启动

`POST /sandboxes/{id}/restart` (`miosa restart`) 是一个停止，然后是同一磁盘、同一 ID 下的恢复。
文件、包、Docker 数据和启用的服务得以保存；正在运行的进程不会，并且来宾会在新内核上启动。
当正在运行的沙箱停止响应命令时，可以使用它。
未运行的沙箱会回答 `409 SANDBOX_NOT_RUNNING`：恢复它。

> **注意：相关**
>
> [超时和自动停止](/docs/develop/sandboxes/timeouts)涵盖了`timeout_sec`过去时发生的情况，[快照和持久性](/docs/develop/snapshots)涵盖了快照和分叉，[大小调整和限制](/docs/develop/sandboxes/sizing)涵盖了每个状态的成本。
