---
slug: "soma/choosing-a-runtime"
title: "选择运行时"
description: "在使用 runtime 选项创建沙箱时选择 Firecracker、SOMA 或 auto。列出你的组织提供的运行时，并查看每个运行时当前声明支持的功能。"
section: "Sandboxes"
order: 2022
---

沙箱运行在两种引擎之一上：**Firecracker**（MIOSA 长期以来的默认引擎）或 **SOMA**（MIOSA 自研的开源 VMM）。
通过 `runtime` 选项选择，该选项在传输时对应 `runtime_profile` 字段。

> **注意：默认是 Firecracker**
>
> 如果你从不选择运行时，一切保持不变。你会得到 Firecracker（`standard`）。
> 以 SOMA 为默认值的组织和工作区运行时设置即将推出。
> SOMA 面向所有客户提供。

## 运行时

| 运行时 | `runtime_profile` | 说明 |
|---|---|---|
| Firecracker | `standard` | 完整机器。所有规格和模板的默认选项。 |
| SOMA | `soma` | 对受支持的沙箱镜像提供最快的启动速度，提供 `small`（2 vCPU / 4 GiB）、`medium`（4 / 8 GiB）和 `large`（8 / 16 GiB）规格，并支持 vCPU 和内存的在线扩缩容。 |
| 自动 | `auto` | 让平台为请求选择最佳运行时。 |

`runtime` 是 SDK 和 CLI 使用的名称；REST 字段是 `runtime_profile`。映射关系为 `firecracker` 对应 `standard`，`soma` 对应 `soma`，`auto` 对应 `auto`。
省略该字段完全等同于 `standard`。这三个值之外的值会被拒绝，并返回 `422 INVALID_SANDBOX_RUNTIME_PROFILE`，错误中会列出允许的集合。

## 列出你的组织提供的运行时

`GET /api/v1/sandboxes/runtimes` 报告你的组织可用的运行时，以及每个运行时当前声明支持的功能。

```http
GET /api/v1/sandboxes/runtimes
Authorization: Bearer msk_u_...
```

```json
{
  "data": [
    {
      "id": "firecracker",
      "runtime_profile": "standard",
      "label": "Firecracker",
      "description": "Full machine. The default for every size and template.",
      "default": true,
      "available": true,
      "unavailable_reason": null,
      "capabilities": {
        "browser": true,
        "network_egress": true,
        "docker": true,
        "persistence": true,
        "pause": true,
        "gpu": false,
        "max_shape": { "cpu_count": 16, "memory_mb": 32768, "disk_size_mb": 163840 }
      }
    },
    {
      "id": "soma",
      "runtime_profile": "soma",
      "label": "SOMA",
      "description": "Fastest start for supported sandbox images.",
      "default": false,
      "available": false,
      "unavailable_reason": "GLOBAL_GATE_CLOSED",
      "capabilities": {
        "browser": false,
        "network_egress": false,
        "persistence": false,
        "pause": true,
        "gpu": false,
        "max_shape": { "cpu_count": 8, "memory_mb": 16384, "disk_size_mb": 40960 }
      }
    },
    {
      "id": "auto",
      "runtime_profile": "auto",
      "label": "Auto",
      "description": "Let the platform pick the best runtime for the request.",
      "default": false,
      "available": true,
      "unavailable_reason": null,
      "capabilities": {
        "browser": true,
        "network_egress": true,
        "docker": true,
        "persistence": true,
        "pause": true,
        "gpu": false,
        "max_shape": { "cpu_count": 16, "memory_mb": 32768, "disk_size_mb": 163840 }
      }
    }
  ]
}
```

在提供某个运行时之前，请先读取 `available`。当它为 `false` 时，`unavailable_reason` 会说明原因：

- `GLOBAL_GATE_CLOSED` - 平台尚未为客户流量启用 SOMA。
- `ORG_RESTRICTED` - SOMA 因策略原因在你的组织中被关闭。
- `NOT_CONFIGURED` - SOMA 已开启，但尚未配置默认容量。

`capabilities` 标志说明某个运行时当前具备的能力。它们来自此端点，因此请读取你计划使用的运行时的条目，而不要根据名称推断能力。

## 创建时选择运行时

**REST**

```bash
# List the runtimes your organization can select
curl https://api.miosa.ai/api/v1/sandboxes/runtimes \
  -H "Authorization: Bearer $MIOSA_API_KEY"

# Create a SOMA sandbox
curl -X POST https://api.miosa.ai/api/v1/sandboxes \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "runtime_profile": "soma", "size": "small", "template_id": "miosa-sandbox" }'
```

**TypeScript**

```typescript
import { Miosa } from "@miosa/sdk";

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

const runtimes = await miosa.runtimes.list();
const sandbox = await miosa.sandboxes.create({ runtime: "soma", size: "small" });
```

**Python**

```python
import os
from miosa import Miosa

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

runtimes = client.runtimes.list()
sandbox = client.sandboxes.create(runtime="soma", size="small")
```

**CLI**

```bash
miosa runtimes list
miosa create --runtime soma --size small
```

`miosa runtimes list` 会打印每个运行时的可用性及原因，因此你可以在选择之前确认 SOMA 是否已向你的组织提供。

如果某个运行时无法满足请求的规格，创建操作会在任何机器被准入之前被拒绝，并返回 `422 SOMA_RUNTIME_UNSUPPORTED_REQUEST`，同时指明该运行时的最大规格。
沙箱响应带有 `runtime`：`"firecracker"` 或 `"soma"`，即其实际运行的引擎，因此回退的 `auto` 请求会报告 `"firecracker"`。

## 请求 SOMA 时会发生什么

在开始任何宿主机工作之前，`soma` 和 `auto` 都会根据你的组织的可信运行时策略进行检查。
如果请求无法满足，创建会失败关闭并返回类型化错误，而不会静默地改在另一个引擎上运行。绝不会发生静默回退。

组织启用后，SOMA 创建需要满足两项条件才能成功：

1. **已启用的运行时策略**，适用于你的组织并包含 SOMA 要求。
2. **合格的容量。** 一个已批准、具有宿主机真实状态的最新观测，并持有针对所请求规格的认证 Generation 的宿主机配置。

### 选择它

使用 `runtime_profile: "soma"` 创建沙箱，或使用 `"auto"` 让平台选择。
请先读取 `GET /api/v1/sandboxes/runtimes` 中的 `available`；当它为 `false` 时，`unavailable_reason` 会说明原因，而如果所请求的规格没有对应的认证 Generation，会在任何机器被准入之前被拒绝。

> **注意：不要与 SOMA 单跳运行器混淆**
>
> 不要将此与 **SOMA 单跳运行器**混淆，SDK 通过 `config.runner.enabled`（或 `MIOSA_RUNNER_MODE=1` 环境变量）提供该功能。
> 该选项是传输快速路径，而不是运行时选择器：它会将 `create`、`exec` 和 `destroy` 直接发送到运行器集群，而不是先经过控制平面。它与沙箱运行在 SOMA 还是 Firecracker 上无关，无论哪种情况，其他所有客户端路径都不受影响。
> [尺寸和限制](/docs/develop/sandboxes/sizing) 页面完整记录了该运行器路径，包括其请求形态和 `Soma-Runner-Url` 请求头。

## 功能一致性与差距

> **警告：切换前先了解差距**
>
> 只有在确认工作负载不依赖下列任何一项差距后，才选择 SOMA。

两种引擎上相同的内容：

- 沙箱 API 契约、响应结构和错误码。
- 命名资源契约。SOMA 提供 `small`、`medium` 和 `large`；Firecracker 提供从 `xs` 到 `xl` 的所有规格。
- `exec`、流式输出以及文件 API：读取、写入、列出、创建目录和删除。
- 创建、延长和销毁。暂停、持久化、快照和分叉目前仅限 Firecracker。
- 定价和计费，无论使用哪种引擎，都按对应规格的价格按秒累计。

切换前需要确认的事项：

- **尚无实时浏览器查看器。** SOMA 沙箱附带 Docker 和 Chromium，但不提供实时查看器所渲染的虚拟显示，因此实时查看器目前无法在 SOMA 上使用。
- **尚不支持暂停、快照或分叉。** SOMA 沙箱无法暂停、创建快照或分叉。
  这些功能将在各自的版本中推出。
- **SOMA 上的长命令。** 可执行文件及每个参数均限制为 4096 字节，更长的 shell 命令目前会失败并返回 `502 AGENT_UNAVAILABLE`，看起来像是可以重试。
  请改为以文件或标准输入发送。

- **SOMA 沙箱尚不具备持久性。** 状态在停止或超时后不会保留，因此 SOMA 创建默认 `persistent: false`，而在 SOMA 上使用 `persistent: true` 会被拒绝并返回 `422`。超时过期的一次性 SOMA 沙箱会被删除。请将持久状态保存在数据服务中，或在需要持久沙箱时使用 Firecracker。
- **只有沙箱接受运行时选项。** 桌面计算机没有运行时选择，它们运行在 Firecracker 上。
- **检查能力标志。** 从 `GET /api/v1/sandboxes/runtimes` 读取运行时的 `capabilities`，并在切换前确认工作负载所需的功能已启用。
- **规格上限确实存在。** 超过运行时 `max_shape` 的请求会被拒绝并返回 `422 SOMA_RUNTIME_UNSUPPORTED_REQUEST`，因此请保持在公布的规格范围内。
- **SOMA 容量有限。** SOMA 创建需要针对你请求的确切规格获得认证 Generation，且宿主机必须已批准并经过最新观测。
- **SOMA 是 alpha 软件。** 其自己的仓库将引擎标记为 `1.0.0-alpha.1`，并维护一份声明账本，记录哪些已被证实、哪些仍仅停留在设计阶段。

## 另请参阅

- [SOMA](/docs/soma)：了解该引擎是什么以及如何工作。
- [基准测试](/docs/benchmarks)：了解 MIOSA 的独立突发性能数据。
- [尺寸和限制](/docs/develop/sandboxes/sizing)：了解资源契约和单跳运行器。
