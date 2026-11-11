---
slug: "computers/lifecycle"
title: "生命周期和规模"
description: "计算机快速启动、桌面状态机、资源大小以及每个租户归因的工作区范围创建。"
section: "Computers"
order: 3005
---

计算机是一台长寿的机器。本页介绍了它的启动速度、它移动的状态、它运行的大小以及如何将其范围限定到工作区。

## 快速启动

MIOSA 在托管机群上保留就绪的桌面容量，以便可以分配新计算机，而不是从头开始冷启动。
生产验收测试测量从创建到第一个可用桌面框架的完整用户可见路径。

经过验证的 2026 年 8 月 6 日金丝雀完成了 10 次热启动中的 10 次，并在中值 **740 毫秒**内到达了浏览器就绪桌面：

|相|时间 |
|---|---:|
|创建响应中位数 | `348 ms` |
|创建对浏览器就绪桌面中位的请求 | **`740 ms`** |
|创建对浏览器就绪桌面 p90 的请求 | **`809 ms`** |
|最慢的样本 | `942 ms` |

这是一个过时的热路径金丝雀，不能保证每个区域和容量条件都将在采样范围内完成。
有关准备合同、测量边界和故障排除指南，请参阅[快速启动和准备情况](/docs/computers/fast-start)。

## 生命周期

创建创建配置；配置准备就绪后即可运行；停止时运行变为停止，启动时返回运行。

运行用于恢复主机维护；恢复后将继续在恢复后运行。

恢复失败时恢复会出错。

运行或停止将在删除时删除。

|状态 |描述 |
|---|---|
| `provisioning` |正在声明、恢复和准备虚拟机 |
| `running` |桌面已启动、可流式传输并接受 API 操作 |
| `stopped` |计算停止并保留文件系统状态 |
| `recovering` | MIOSA 正在主机维护后恢复所需运行的计算机 |
| `error` |恢复未完成；保留的磁盘或快照仍可用于重试 |
| `deleted` |永久删除；不可逆|

停止运行的计算机不会产生计算费用；当其停止时，其磁盘的收费为每月每 GiB 0.018 美元。重新启动将从快照恢复。

在主机维护期间，MIOSA 会保留计算机身份、磁盘状态、快照、元数据和所需状态。
在主机返回活动服务之前，正在运行的计算机将被恢复和验证。
故意停止 计算机保持停止状态。

如果本地恢复失败，MIOSA 会保留磁盘和快照，而不是用空白计算机替换计算机。
仅当应永久删除状态时才使用 `delete`。

## 尺码

|尺寸| vCPU |内存 |磁盘 |典型用途|
|---|---|---|---|---|
| `xs` | 1 | 2GB| 10GB|轻量级无头任务 |
| `small` | 2 | 4GB| 20GB| **默认**单个智能体和基本浏览 |
| `medium` | 4 | 8GB| 50GB|桌面应用程序更重，页面加载更快 |
| `large` | 8 | 16GB| 100GB|多应用、大环境 AI 工作负载 |
| `xl` | 16 | 32GB| 200GB|密集并行、大模型权重 |

GPU 未产品化：公共 API 目前不调度 GPU 支持的桌面。

> **注：OSA 智能体**
>
> 计算机出厂时已安装 OSA 智能体，但默认情况下处于禁用状态。
> 激活它，给你的电脑一个桌面AI助手，可以自主运行多步桌面任务。
> OSA 通过记录的 MIOSA API 进行控制。

## 工作区范围内的创建

可以在命名工作区中创建计算机，以便隔离资源并单独计费。传递 `external_workspace_id` 将计算机归属于您平台中的租户。

**Python**

```python
import miosa, os

client = miosa.Miosa(api_key=os.environ["MIOSA_API_KEY"])

computer = client.computers.create(
    name="agent-1",
    template_type="miosa-desktop",
    size="small",
    external_workspace_id="acme-corp",      # your tenant identifier
    metadata={"agent_run": "run-2026-05-17"},
)
computer.start()
```

**打字稿**

```typescript
import { Miosa } from "@miosa/sdk"

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! })

const computer = await miosa.computers.create({
  name: "agent-1",
  template_type: "miosa-desktop",
  size: "small",
  externalWorkspaceId: "acme-corp",
  metadata: { agentRun: "run-2026-05-17" },
})
await computer.start()
```

`external_workspace_id` 是您控制的自由格式字符串。
使用您自己的租户、组织、客户或工作区标识符。
在计费仪表板中跟踪每个工作区的使用情况，以便您可以将计算成本归因于各个客户。
