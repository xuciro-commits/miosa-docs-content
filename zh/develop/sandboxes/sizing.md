---
slug: "develop/sandboxes/sizing"
title: "尺寸和限制"
description: "命名沙箱资源合同、原始资源覆盖、沙箱使用如何计费以及选择加入运行程序快速路径。"
section: "Sandboxes"
order: 2006
---

每个沙箱都在命名资源合约上运行。
按名称选择 `size`；原始 CPU、内存和磁盘字段是兼容性输入，必须与已发布的合同完全匹配。

## 尺码参考

|尺码名称 | vCPU |内存 |磁盘 |笔记|
|---|---|---|---|---|
| `xs` | 1 | 2GB| 10GB|轻量级脚本 |
| `small` | 2 | 4GB| 10GB| **智能体和构建工作负载的默认值** |
| `medium` | 4 | 8GB| 20GB|更大的构建和并行测试|
| `large` | 8 | 16GB| 40GB|重型测试套件 |
| `xl` | 16 | 32GB| 80GB|密集并行|

使用默认的 `small` 合约进行 Claude Code、Codex、包安装、普通应用程序构建和智能体执行。
仅当测量的 CPU、内存或磁盘要求超过 `small` 时，才选择 `medium`。
原始资源字段是兼容性输入，必须准确标识已发布的命名合约。

**原始资源覆盖**（仍然接受）：

|领域|默认 |验证范围|
|---|---|---|
| `cpu_count` | 2 | 1-16 |
| `memory_mb` | 4096 | 2048-32768 |
| `disk_size_mb` | 10240 | 必须与选定的命名合约匹配|
| `timeout_sec` | 3600 | 1-86400|
| `idle_timeout_sec` | 0（2026 年 10 月 11 日或之后创建的帐户为 1800）| 0-86400 |
| `always_on` | `false` |大多数智能体工作不需要 |
| `persistent` | `true` |在停止和恢复期间保留磁盘（停止后仍可正常运行，而不是主机丢失）|

设置 `always_on=True` 以完全禁用 `timeout_sec` 强制。对于长期开发环境或托管 IDE 很有用。设置 `idle_timeout_sec` 以在一段时间不活动后停止已放弃的持久会话。仅针对一次性作业设置 `persistent=False`，其中文件系统应在停止或超时时被丢弃。

**默认空闲自动停止。** 新帐户（在 2026 年 10 月 11 日或之后创建）：不带 `idle_timeout_sec` 的沙箱在空闲 30 分钟后停止，并在下一个请求时唤醒。将 `idle_timeout_sec` 设置为 `0` 以禁用。现有帐户保持其当前行为（除非设置空闲超时）。

## 配置与每个沙箱参数

平台配置定义了可以选择的内容：模板别名、不可变图像生成、命名资源合同、准备情况、放置策略和租户限制。
客户使用 `GET /api/v1/templates` 检查该配置；应用程序不发送内部布局、主机或提供者字段。

每个沙箱参数选择一个沙箱的行为。
它们包括 `template_id`、`size`、`timeout_sec`、`always_on`、`persistent` 和 `idle_timeout_sec`。
优先选择命名的 `size` 而不是原始 CPU、内存和磁盘字段。
如果使用兼容性字段，则 `cpu_count`、`memory_mb` 和 `disk_size_mb` 必须一起提供，并且与一份已发布的合同完全匹配。

默认值为 `small`：2 个 vCPU、4096 MiB RAM 和 10240 MiB 磁盘。
当模板目录报告所选模板的情况并且当前准备情况允许准入时，`xs` 可用于较轻松的工作。
每个沙箱响应都包含版本化的 `resource_contract`，例如 `sandbox/small@v1`，因此客户端可以记录已解析的资源，而不是假设大小名称永远不会演变。

```json
{
  "template_id": "miosa-sandbox",
  "size": "small",
  "timeout_sec": 3600,
  "persistent": true,
  "idle_timeout_sec": 0
}
```

## 调整正在运行的沙箱的大小

正在运行的 Firecracker (`standard`) 沙箱可以就地调整大小：vCPU、内存或两者，而无需销毁或重新创建它。
`/workspace` 下的文件系统被保留。
大小被命名为合约，因此调整大小会在已发布的合约之间移动沙箱，而不是移动到任意值。

实时了解沙箱的功能以及它可以更改的内容，而无需更改任何内容。
每个沙箱都会报告自己的功能，因此客户端根据答案采取行动，而不是假设：

**休息**

```http
GET /api/v1/sandboxes/:id/resources

{ "id": "...",
  "resources": { "cpu": 2, "memory_mb": 4096 },
  "capabilities": { "live_cpu": true, "live_cpu_max": 2,
                    "live_memory": false, "live_memory_max_mb": null,
                    "cpu_mechanism": "cgroup_quota", "runtime_engine": "firecracker" } }
```

**打字稿**

```typescript
const r = await sandbox.resources();
```

**Python**

```python
r = sandbox.resources()
```

更改它，仅发送您要更改的字段：

**休息**

```http
PATCH /api/v1/sandboxes/:id/resources
{ "cpu_count": 4, "memory_mb": 8192 }
```

**打字稿**

```typescript
await sandbox.resize({ cpuCount: 4, memoryMb: 8192 });
```

**Python**

```python
sandbox.resize(cpu_count=4, memory_mb=8192)
```

**命令行界面**

```bash
miosa sandbox resources my-box
miosa sandbox resize my-box --cpu 4 --memory 8192
```

今天直播什么：- **vCPU** 在正在运行的 Firecracker 沙箱上运行，最多可达沙箱启动时使用的计数，并且仅在启用了 vCPU 控制器的主机上。
  来宾保留其启动时所用的核心；主机通过 cgroup 配额限制了它们可以使用的时间，因此较低的 vCPU 数量会限制而不是删除核心。
- **内存**位于运行中的 Firecracker 沙箱上，该沙箱通过可热插拔区域 (virtio-mem) 启动：增大将内存插入来宾中，收缩则将其拔出，并且内存永远不会低于启动大小。
  `live_memory_max_mb` 是该沙箱的上限，默认为 16384 MiB。
  实时内存正在整个热池中滚动：来自旧热槽的沙箱报告 `live_memory: false` 直到该槽被重建，从快照恢复的沙箱也是如此。
- **SOMA** 沙箱在自己的主机上应用 vCPU 和内存，上下（vCPU 按 cgroup 配额，内存按 virtio-mem），平台已为该主机启用它。

在调整大小之前读取 `capabilities`，然后从调整大小响应中再次读取。
`live_cpu`、`live_memory`，其最大值是每个沙箱、每个主机和每个映像生成； `*_reason` 字段说明更改未生效的原因。

报告无法实时应用的更改 `requires_restart`：请求回答 `409 RESIZE_REQUIRES_RESTART`，没有应用任何内容，并且没有任何内容存储为待处理。
超出您的计划上限的请求会得到答复 `422 PLAN_CAP_EXCEEDED`； SOMA 沙箱或没有控制器的主机回答 `409 RESIZE_UNSUPPORTED`。

发送 `"restart": true` 以新大小引导同一磁盘。
这是一个重新创建，而不是就地更改：文件系统保留下来，新沙箱有新的 ID，旧沙箱的进程消失，源保持暂停状态并计费为暂停沙箱，直到您销毁它。
响应为 `202` 以及 `restarted: true`、`new_sandbox` 和 `source_sandbox`。
SOMA 沙箱没有可启动的主机磁盘，因此不会以这种方式重新创建它们。

## 账单备注

账单从 `state = "running"` 开始累积，直至 `state = "stopped"`、`state = "paused"` 或 `state = "destroyed"`。

- **运行**：按其大小的价格每秒计费，每个计划以及无论机器空闲还是繁忙都相同（例如 `small` 每小时 0.0893 美元）
- **已停止**：按冷存储费率计费（磁盘 GiB 秒）
- **暂停**：仅按存储费率计费（磁盘 GiB 秒）
- **销毁**：计费立即停止

模板准备情况和可用大小由目录报告，并可能因环境而异。

## 跑步者端点（选择加入预览）

SOMA 单跳运行器是 `create`、`exec` 和 `destroy` 的可选快速路径：您的 SDK 直接与运行器对话，而不是首先通过控制平面。无论您是否选择加入，此处记录的所有内容都将保持不变。- `https://run-us.miosa.ai` 适用于每个呼叫。选择在那里发送请求而不是 `https://api.miosa.ai`；欧盟已在路线图上，但钥匙尚未包含区域。此端点与沙箱的放置 [region](/docs/api-reference/regions) (`us-west`, `us-east`, `us-mia`) 无关。
- 对位于运行者队列内不同主机上的沙箱的请求在内部转发；你几乎总是直接从 `run-us.miosa.ai` 得到答案（如果内部转发失败，`421` 是一个罕见的后备方案，将正确的 URL 命名为 `runner_url`）。为了降低延迟，每个 `create` 响应还会将该沙箱自己的运行程序 URL 命名为 `Soma-Runner-Url` 响应标头 (`https://<tag>.run-us.miosa.ai`)；官方 SDK 会读取它并从那时起直接调用该 URL。这是自动优化，不需要您自己做。
- 运行器上的 `create` 仅提供默认模板、`xs` 大小、非持久沙箱。超出该范围的请求（自定义模板、快照、自定义 `cwd`、`persistent: true` 或更大尺寸）会回答 `400 RUNNER_UNSUPPORTED_REQUEST`；官方 SDK 会自动将创建的内容发送到 `https://api.miosa.ai`。
- 运行器仅创建非持久性沙箱，因此空闲自动停止默认值在此不适用：空闲超时保持关闭状态，除非您设置 `idle_timeout_sec`（0-86400 秒）。 `timeout_sec`（默认 3600，最大 86400）限制生命周期，而 `always_on=True` 完全禁用长期沙箱和托管 IDE 的超时。
- `exec` 默认为 30 秒超时，与控制平面相同。 `POST /api/v1/sandboxes/{id}/exec/stream` 将 stdout/stderr 作为服务器发送的事件进行流式传输，并以退出事件结束。
- `destroy` 响应包括 `cpu_ms` 和 `lifetime_ms` 用于此沙箱的使用。
- `429 runtime_busy`（带有`Retry-After: 0`）表示该跑步者已达到入场限制；重试该请求。 `429 rate_limited` 是每个键或每个租户的速率限制，与控制平面的语义相同。 `503 feed_stale` 表示运行者的密钥和策略数据太旧，无法接受新创建的数据；其现有的沙箱继续工作。
- 运行程序首先提供 HTTP/3 (QUIC)，如果 QUIC 在您的网络上被阻止，则自动回退到 HTTP/2。官方 SDK 会为您处理后备问题。

原始 HTTP 客户端可以使用区域 URL 或 `Soma-Runner-Url` 标头中指定的每个沙箱 URL：

```bash
curl -i -X POST https://run-us.miosa.ai/api/v1/sandboxes \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"template_id": "miosa-sandbox"}'
# Soma-Runner-Url: https://3.run-us.miosa.ai

curl https://3.run-us.miosa.ai/api/v1/sandboxes/3a1b2c3d-1111-2222-3333-444444444444 \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

每个跑步者响应都会添加一个 `Server-Timing` 标头，将 `auth`、`pool` 和 `exec` 时间缩短（以毫秒为单位）。

### 特定于运行路径的错误

|状态 |代码|原因 |
|--------|------|--------|
| 401 | `unauthorized`|该运行程序的内存表中的密钥未知或已被撤销。 |
| 403 | `forbidden`|租户未启用此路径、已暂停或不允许项目。 |
| 400 | `RUNNER_UNSUPPORTED_REQUEST` |跑步者不提供此创建形状（见上文）。官方 SDK 针对 `https://api.miosa.ai` 重试。 |
| 429 | `rate_limited`|每个密钥或每个租户的速率限制，与控制平面相同的语义。 |
| 429 | `runtime_busy`|该跑步者已达到入场限制 (`Retry-After: 0`)。重试请求； `run-us.miosa.ai` 解析到该区域中的每个跑步者，因此重试可以落在不同的跑步者上。 |
| 503 | `feed_stale`|此运行程序的密钥和策略数据早于其新鲜度限制 (`Retry-After: 0`)。创建被拒绝；其现有的沙箱继续工作。 |
| 421 | `misdirected`|罕见：沙箱自身主机内部转发失败。正文再次将正确的 URL 命名为 `runner_url`。关注一次；不要重试原始 URL。 |

完整的端点契约（请求和响应主体）不会因运行者模式而改变；只有您发送给他们的主机会发生变化。有关控制平面参考，请参阅 [Sandboxes API](/docs/api-reference/sandboxes)。

> **注意**
>
> 跑步者模式是由租户和主机逐步推出的选择加入预览。在您选择加入之前，每个请求都会按照上面记录的方式继续通过控制平面。
