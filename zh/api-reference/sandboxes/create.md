---
slug: "api-reference/sandboxes/create"
title: "沙箱 API - 创建和运行"
description: "创建 MIOSA 沙箱，使用融合运行端点，启动模板应用程序，获取工件和自定义模板边界。"
section: "API reference"
order: 12015
---

基本路径：`/api/v1/sandboxes`

> **注意**
>
> 所有端点都需要 `Authorization: Bearer &lt;api_key&gt;`。

## 创建一个沙箱

**`POST /api/v1/sandboxes`**

生成一个新的沙箱虚拟机。 API接受创建请求后立即返回沙箱记录；轮询 `GET /api/v1/sandboxes/{id}` 或订阅事件，直到 `state` 为 `running` 且 `ready` 为 `true`。

生产别名 `miosa-sandbox` 解析为经过认证的不可变工件生成。

### 授权

需要不记名令牌。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `template_id` |字符串|没有 |创建时在沙箱映像内暂存的模板。对于没有任何内容的沙箱，请忽略它。请参阅下面的模板。 |
| `size` |字符串|没有 |规范的资源配置。默认为 `small`（`2 vCPU`、`4096 MiB`、`10240 MiB` 磁盘）。 |
| `cpu_count` |整数 |没有 |精确的 vCPU 兼容性输入。提供完整的匹配资源三元组。 |
| `memory_mb` |整数 |没有 |精确的 RAM 覆盖。它必须符合所选的规范尺寸。 |
| `disk_size_mb` |整数 |没有 |精确的磁盘覆盖。它必须符合所选的规范尺寸。 `disk_mb` 已弃用。 |
| `timeout_sec` |整数 |没有 |活动会话超时。默认为 `3600`；最小值 `1`，最大值 `86400`。 |
| `always_on` |布尔 |没有 |禁用超时和空闲超时强制，直到执行明确的生命周期操作。默认为 `false` 并仍受政策约束。 |
| `persistent` |布尔 |没有 |默认为 `true`。在停止/超时期间保留文件系统状态并允许恢复。仅针对一次性一次性作业设置 `false`。 |
| `idle_timeout_sec` |整数 |没有 |沙箱暂停前无活动的秒数。如果未设置，则对于 2026 年 10 月 11 日或之后创建的帐户，默认为 `0`（禁用）或 `1800`（30 分钟）；将 `0` 设置为禁用。 |
| `env` |对象|没有 |启动时注入的键值环境变量。 |
| `metadata` |对象|没有 |任意调用者提供的元数据存储在记录中。 |
| `agent_runtime_profile_id` | UUID/字符串 |没有 |强制此沙箱使用特定的智能体运行时配置文件。别名：`agent_profile_id`。 |
| `skip_agent_runtime_profile` |布尔 |没有 |如果 `true`，则不应用租户/工作区默认运行时配置文件。 |
| `auto_start` |布尔 |没有 |如果 `true`，MIOSA 在沙箱达到 `running` 后启动所选模板。生成应用程序平台通常保留此错误，先写入文件，然后调用`/template/start`。 |
| `workspace_id` | UUID |没有 |拥有沙箱的现有 MIOSA 工作区。默认为组织默认工作区。 |
| `workspace_slug` |字符串|没有 |现有或自动创建的工作区 slug。 |
| `workspace_name` |字符串|没有 |工作区显示名称（如果自动创建）。 |
| `project_id` | UUID |没有 |拥有沙箱的现有 MIOSA 项目。默认为工作区默认项目。 |
| `project_slug` |字符串|没有 |工作区中现有或自动创建的项目 slug。 |
| `project_name` |字符串|没有 |项目显示名称（如果自动创建）。 |
| `external_workspace_id` |字符串|没有 |您的客户/帐户/工作区 ID。 |
| `external_user_id` |字符串|没有 |您的最终用户 ID。 |
| `external_project_id` |字符串|没有 |您的项目/应用程序/文档 ID。 |

如果工作区或租户存在默认智能体运行时配置文件，MIOSA
在创建过程中自动应用它。配置文件环境变量与请求合并
环境变量，请求环境在冲突时获胜，配置文件元数据记录在
`metadata.agent_runtime_profile`，和配置文件连接器通过连接
出口占位符令牌，而不是复制为明文机密。

**可用模板：**

| `template_id` |描述 |
|----------------|-------------|
| `miosa-sandbox` |默认沙箱，以及省略 `template_id` 时的默认沙箱。里面的一切：带有 `docker compose` 和 `docker buildx` 的 Docker Engine 29.9、虚拟显示器上带有 CDP 的实时浏览器、持久代码解释器、Node 和 Python。 |每个其他内置 id 都是 `miosa-sandbox` 的别名，保留下来以便现有代码继续工作； [Sandbox Templates API](/docs/api-reference/sandbox-templates) 列出了它们。
对于桌面，使用 `template_type: "miosa-desktop"` 创建一台计算机；请参阅[计算机](/docs/computers/overview)。

### 请求标头

|标题 |描述 |
|--------|-------------|
| `Idempotency-Key` |客户端生成的密钥（建议使用 UUID）。 24 小时内相同的密钥将返回现有沙箱，而不是创建新沙箱。 |

### 响应 - `201 Created`
```json
{
  "id": "sbx_01j9xr2t4fk8me3n5q",
  "tenant_id": "tnt_abc123",
  "owner_id": "usr_def456",
  "workspace_id": "550e8400-e29b-41d4-a716-446655440000",
  "project_id": "660e8400-e29b-41d4-a716-446655440001",
  "external_workspace_id": "acct_123",
  "external_user_id": "user-456",
  "external_project_id": "project_789",
  "template_id": "miosa-sandbox",
  "image_id": "miosa-sandbox",
  "state": "provisioning",
  "ready": false,
  "size": "small",
  "resource_contract": {
    "id": "sandbox/small@v1",
    "product": "sandbox",
    "size": "small",
    "version": "v1",
    "vcpus": 2,
    "memory_mb": 4096,
    "disk_size_mb": 10240
  },
  "cpu_count": 2,
  "memory_mb": 4096,
  "disk_size_mb": 10240,
  "preview_url": "https://sbx01j9x.sandbox.miosa.ai",
  "timeout_sec": 3600,
  "timeout_remaining_ms": null,
  "always_on": false,
  "persistent": true,
  "idle_timeout_sec": 0,
  "total_runtime_sec": null,
  "metadata": {},
  "created_at": "2026-04-25T10:00:00Z",
  "started_at": null,
  "ready_at": null,
  "destroyed_at": null
}
```

### 错误

|状态 |代码|原因 |
|--------|------|--------|
| 400 | `INVALID_TEMPLATE` | `template_id` 不是可识别的模板。 |
| 402 | `INSUFFICIENT_CREDITS`|积分不足，无法配置虚拟机。 |
| 429 | `SANDBOX_LIMIT_REACHED`|租户已达到其计划的并发沙箱限制（Developer 10、Business 250、Enterprise 500）。 |
| 422 | `VALIDATION_ERROR`|字段值无效（例如，超出租户计划上限的资源请求）。 |

---

## 创建并运行

**`POST /api/v1/sandboxes/run`**

创建一个沙箱，等待它变成`running`，执行第一个命令，并在一个请求中返回沙箱记录加上命令结果。将此用于智能体循环，其中第一个有用的操作是“创建沙箱并立即运行代码”。

对于需要沙箱和立即执行第一个命令的智能体来说，这是生产快速路径。

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/run \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: run-2026-06-09-001" \
  -d '{
    "template_id": "miosa-sandbox",
    "command": "python3 -c \"print(1+1)\"",
    "size": "small",
    "timeout_sec": 300,
    "wait_timeout_ms": 30000,
    "metadata": { "agent_run": "abc123" }
  }'
```

### 请求正文

`/run` 接受与 `POST /api/v1/sandboxes` 相同的沙箱创建字段，此外：

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `command` |字符串|是的 |沙箱到达 `running` 后执行的第一个命令。 |
| `cwd` |字符串|没有 |命令的工作目录。默认为 `/workspace`。 |
| `timeout` |整数 |没有 |命令超时（以毫秒为单位）。 |
| `wait_timeout_ms` |整数 |没有 |返回 `504 SANDBOX_READY_TIMEOUT` 之前等待沙箱准备就绪的最长时间。 |

### 响应 - `201 Created`

```json
{
  "data": {
    "id": "sbx_01j9xr2t4fk8me3n5q",
    "state": "running",
    "ready": true,
    "template_id": "miosa-sandbox",
    "cpu_count": 2,
    "memory_mb": 4096
  },
  "exec": {
    "stdout": "2\n",
    "stderr": "",
    "exit_code": 0
  },
  "timings": {
    "server_wait_and_exec_ms": 512
  }
}
```

### 错误

|状态 |代码|原因 |
|--------|------|--------|
| 400 | `MISSING_PARAM` |缺少 `command`。 |
| 400 | `INVALID_TEMPLATE` | `template_id` 无法识别。 |
| 402 | `INSUFFICIENT_CREDITS`|积分不足，无法配置虚拟机。 |
| 429 | `SANDBOX_LIMIT_REACHED`|租户已达到并发沙箱限制。 |
| 409 | `SANDBOX_NOT_RUNNING`|沙箱无法达到 exec 的运行状态。 |
| 502 | `SANDBOX_BOOT_FAILED`|在命令运行之前 VM 启动失败。 |
| 502 | `AGENT_UNAVAILABLE`| exec 无法访问沙箱智能体。 |
| 504 | `SANDBOX_READY_TIMEOUT`|沙箱在 `wait_timeout_ms` 之前尚未准备就绪。 |

---

## 启动模板应用程序

**`POST /api/v1/sandboxes/{id}/template/start`**

从具有启动命令的模板创建的沙箱不会在创建时等待应用程序：尚未启动任何内容，因此 `wait: true` 创建会在沙箱运行后立即返回，而不是超时。
当您调用 `/template/start`; 时，将应用模板自己的就绪探测。您传递创建的探针总是获胜。

当您的平台将生成的文件写入`/workspace`后，调用此端点来运行选定的模板生命周期。 MIOSA 运行模板安装命令，在后台启动启动命令，存储 PID/日志路径，并返回预览 URL。
```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/{id}/template/start \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"install":true}'
```

回应：
```json
{
  "data": {
    "status": "started",
    "template_id": "nextjs",
    "workdir": "/workspace",
    "preview_port": 3000,
    "preview_url": "https://abc12345.sandbox.miosa.ai",
    "logs_path": "/tmp/miosa-run/template.log",
    "pid_path": "/tmp/miosa-run/template.pid",
    "artifact_paths": ["/workspace"]
  }
}
```

当生成的项目需要自定义命令时，您可以覆盖请求正文中的 `install_command`、`start_command`、`port` 或 `workdir`。

---

## 获取文物

**`GET /api/v1/sandboxes/{id}/artifacts`**

返回模板工件合约和当前生命周期元数据。
```bash
curl https://api.miosa.ai/api/v1/sandboxes/{id}/artifacts \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

响应包括 `preview.url`、`preview.port`、工件路径和模板生命周期日志/PID 路径。

具有白标签属性和环境的创建请求：

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "template_id": "miosa-sandbox",
    "workspace_slug": "acme-corp",
    "workspace_name": "Acme Corp",
    "project_slug": "lead-magnet",
    "project_name": "Lead Magnet",
    "external_workspace_id": "acct_123",
    "external_user_id": "user-456",
    "external_project_id": "project_789",
    "size": "small",
    "env": {"MY_VAR": "hello"}
  }'
```

---

## 自定义模板

规范的公共 V1 合约仅公开 `GET /api/v1/templates` 返回的升级模板。
独立的自定义模板构建、验证、日志、重试和取消端点不是该合同的一部分。
不要使公共集成依赖于这些操作。
