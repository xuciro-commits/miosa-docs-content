---
slug: "api-reference/sandboxes"
title: "沙箱API"
description: "用于在 MIOSA 沙箱（默认情况下持久隔离的 microVM 工作区）中创建、管理和执行代码的 API 参考。"
section: "API reference"
order: 12014
---

沙箱是默认情况下永久隔离的 microVM 工作区，可以从预先植入的快照中恢复、接受执行和文件操作、公开预览，并按计算/运行时使用情况计费。它们专为 AI 智能体代码执行工作负载、工件生​​成和应用程序预览而设计。

基本路径：`/api/v1/sandboxes`

> **注意**
>
> 所有端点都需要 `Authorization: Bearer &lt;api_key&gt;`。从 [MIOSA 仪表板 ](https://miosa.ai/settings/api-keys) 获取密钥。

对于更高级别的访问，请使用官方 SDK：[Python](/docs/sdks/python)、[TypeScript](/docs/sdks/typescript)、[Go](/docs/sdks/go)、[Java](/docs/sdks/java)、[Elixir](/docs/sdks/elixir)。

- [创建并运行](/docs/api-reference/sandboxes/create)：创建沙箱、融合运行端点、模板应用程序、工件和自定义模板。
- [管理](/docs/api-reference/sandboxes/manage)：列出、获取、销毁、暂停、恢复、分叉、延长超时和读取使用情况。

---

## 列出沙箱模板和大小

**`GET /api/v1/templates`**

这是沙箱模板 ID、模板默认值、每个模板大小支持、准备情况和精确版本化资源合同的规范目录。
客户端不需要辅助目录端点来发现或接受沙箱配置。
```bash
curl https://api.miosa.ai/api/v1/templates
```

### 响应 - `200 OK`
```json
{
  "data": [
    {
      "id": "miosa-sandbox",
      "name": "MIOSA Sandbox",
      "product": "sandbox",
      "default_size": "small",
      "primitive": "sandbox",
      "sdk_name": "miosa-sandbox",
      "cli_name": "miosa-sandbox",
      "readiness": "fast_ready",
      "sizes": [
        {
          "size": "small",
          "state": "fast_ready",
          "resource_contract": {
            "contract_id": "sandbox/small@v1",
            "contract_version": "v1",
            "product": "sandbox",
            "size": "small",
            "vcpus": 2,
            "memory_mb": 4096,
            "disk_size_mb": 10240
          }
        }
      ]
    }
  ],
  "templates": [
    {
      "id": "miosa-sandbox",
      "name": "MIOSA Sandbox",
      "product": "sandbox",
      "default_size": "small",
      "primitive": "sandbox",
      "sdk_name": "miosa-sandbox",
      "cli_name": "miosa-sandbox",
      "readiness": "fast_ready",
      "sizes": [
        {
          "size": "small",
          "state": "fast_ready"
        }
      ]
    }
  ],
  "shape_contracts": {
    "sandbox": [],
    "computer": []
  },
  "readiness_states": ["fast_ready", "cold_boot_only", "missing"]
}
```

`fast_ready` 尺寸可通过快速启动路径创建。
`cold_boot_only` 尺寸也可以创建，但启动速度较慢。
`missing` 表示不可用，不得提交。
当调用者没有更强的要求时，使用该模板的 `default_size`。
请勿将一个模板宣传的尺寸与另一模板相结合。

---

## 配置和请求参数

`GET /api/v1/templates` 返回平台配置：可用模板、默认大小、准备情况和版本化资源合同。
应用程序使用沙箱参数选择该配置；它们不发送内部主机、提供商、网络或放置字段。

对于创建请求，首选 `size`。
默认的 `small` 合约是 2 个 vCPU、4096 MiB RAM 和 10240 MiB 磁盘。
当目录报告可用时，可以选择 `xs`。
兼容性资源字段必须同时包含 `cpu_count`、`memory_mb` 和 `disk_size_mb`，并且与一份已发布的合同完全匹配。

响应包括 `resource_contract` 下已解决的合同。
合约 ID 是有版本的，例如 `sandbox/small@v1`；当使用或审计记录必须保持可重现时，请保留该标识。

---

## 沙箱对象

create、get 和 list 端点返回的完整沙箱对象：

| 领域 | 类型 | 描述 |
|--------|------|-------------|
| `id` |字符串|唯一的沙箱标识符。 |
| `tenant_id` |字符串|拥有租户 ID。 |
| `owner_id` |字符串|创建用户 ID。 |
| `template_id` |字符串|启动模板名称。 |
| `state` |字符串|当前生命周期状态。 |
| `size` |字符串|规范命名大小：`xs`、`small`、`medium`、`large` 或 `xl`。 |
| `resource_contract` |对象|版本化解决了包含 ID、产品、大小、vCPU、内存和磁盘的合同。 |
| `cpu_count` |整数 |分配的 vCPU。 |
| `memory_mb` |整数 |分配的 RAM（以 MB 为单位）。 |
| `disk_size_mb` |整数 |分配的根磁盘（以 MB 为单位）。 |
| `ready` |布尔 |指挥和准备检查是否正在接受工作。 |
| `ready_at` |字符串/空 | `ready` 变为 true 时的 ISO 时间戳。 |
| `preview_url` |字符串/空 |沙盒 slug 的租户感知基本预览 URL。对特定端口使用 `/expose`。 |
| `timeout_sec` |整数 |活动会话的最大秒数。持续超时暂停；非持久超时破坏。 |
| `timeout_remaining_ms` |整数\|空 |剩余活动会话时间；在启动之前或禁用超时强制时为 null。 |
| `always_on` |布尔 |是否禁用超时和空闲超时强制。 |
| `persistent` |布尔 |沙箱是否在停止/超时时保留文件系统状态。 |
| `idle_timeout_sec` |整数 |沙箱暂停前无活动的秒数。 `0` 禁用它；在 October 11, 2026 或之后创建的帐户在未设置时默认为 `1800`（30 分钟）。 |
| `total_runtime_sec` |整数\|空 |按秒计费。仅在破坏后设置。 |
| `metadata` |对象|调用者提供的元数据。 |
| `created_at` |字符串| ISO-8601 创建时间戳。 |
| `started_at` |字符串\|空 | VM 进入 `running` 时的 ISO-8601 时间戳。 |
| `destroyed_at` |字符串\|空 | VM 被销毁时的 ISO-8601 时间戳。 |

---

## 生命周期状态

创建就绪后即可运行；停止时运行变为停止，恢复时返回运行。

暂停时跑步会进入暂停状态，恢复时会回到跑步状态。

运行会被破坏。

任何状态在失败时都会出错（终端）。

|状态 |描述 |
|--------|-------------|
| `provisioning` |虚拟机正在启动。 Exec 和文件操作尚不可用。 |
| `running` |虚拟机可达。允许所有操作。 |
| `paused` |计算停止并保留文件系统状态。可以恢复。 |
| `destroyed` |终端。 VM 消失了，计费已结算。 |
| `error` |终端故障。销毁并创建一个新的沙箱。 |

---

## 错误响应格式

所有错误响应都使用此形状：
```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "sandbox not found",
    "details": null
  }
}
```

`x-request-id` 响应标头在每个请求上设置。将其包含在支持请求中。

---

## 代码解释器

**`POST /api/v1/sandboxes/{id}/code/run`**（别名：`POST /api/v1/sandboxes/{id}/code`）

在持久解释器中运行一个单元并返回其输出。
内核在第一次使用时会延迟启动，并且变量、导入和定义在同一上下文中的调用之间保持不变。
`language` 是 `python` （默认值）或 `javascript`；不支持的值返回 `400 UNKNOWN_LANGUAGE`。
`context` 命名一个独立的命名空间，`default` 除非你传递一个； `timeout` 的上限为 300 秒。

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/{id}/code/run \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "code": "total = sum(range(10))\ntotal", "language": "python" }'
```

```json
{
  "data": {
    "ok": true,
    "stdout": "",
    "stderr": "",
    "result": "45",
    "language": "python"
  }
}
```

|端点|它有什么作用 |
|---|---|
| `GET /api/v1/sandboxes/{id}/code` |报告解释器状态。从不启动内核。当沙箱未运行时返回 `sandbox_not_running`。 |
| `GET /api/v1/sandboxes/{id}/code/contexts?language=` |列出某种语言存在的上下文。 |
| `POST /api/v1/sandboxes/{id}/code/contexts` |创建或重用上下文。 |
| `DELETE /api/v1/sandboxes/{id}/code/contexts/{context}` |重置一个上下文。 |
| `DELETE /api/v1/sandboxes/{id}/code?language=` |重置语言的每个上下文。 |

读取(`GET`)需要`sandboxes:read`；运行和重置需要`sandboxes:exec`。
沙箱必须正在运行：解释器不会唤醒暂停的沙箱。

---

## 速率限制

| 限制 | 价值 |
|--------|--------|
|请求率|每个工作区和每个端点组强制执行；超限响应携带 `Retry-After` |
|每个租户的并发沙箱 |计划上限：10（开发者）、250（商业）、500（企业）|
| `timeout_sec` | 1-86400秒；默认3600 |
|默认资源形状 | `small`：2 个 vCPU、4 GiB RAM、10 GiB 磁盘 |
|命名资源模式| `xs` 到 `xl`；实际可用性来自`GET /templates`和租户政策 |
|最大执行超时 | 1-86400秒；默认3600 |
|最大文件上传大小 | 500 MB |

速率限制的响应返回 HTTP `429`，其中 `Retry-After` 标头指示等待的秒数。

---

## 另请参阅

- [Python SDK](/docs/sdks/python) - `miosa`
- [TypeScript SDK](/docs/sdks/typescript) - `@miosa/sdk`
- [Go SDK](/docs/sdks/go) - `github.com/miosa-ai/miosa-go`
- [Java SDK](/docs/sdks/java) - `ai.miosa:miosa-sdk`
- [Elixir SDK](/docs/sdks/elixir) - `:miosa`
- [事件 (SSE)](/docs/api-reference/events) - 其他 MIOSA 资源的 SSE 参考
- [错误代码](/docs/api-reference/errors) - 完整的错误代码目录，包括沙箱特定的代码
