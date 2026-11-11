---
slug: "api-reference/computers"
title: "计算机应用程序接口"
description: "用于创建、管理和控制 MIOSA 计算机的 API 参考 - 完整生命周期、桌面控制、文件、指标和环境变量。"
section: "API reference"
order: 12029
---

计算机是持久桌面虚拟机，具有 Linux 桌面、流式桌面访问以及用于执行、文件、应用程序、指标和桌面控制的 API。基本路径：`/api/v1/computers`。

支持的动词：**GET**（列表/获取/配置）、**POST**（创建/启动/停止/重新启动/调整大小）、**PATCH**（更新）、**PUT**（更新）、**DELETE**（删除）。

> **注意**
>
> 选定的突变端点（沙箱模板和迁移）接受 `Idempotency-Key` 标头。使用 UUID v4。 24 小时内重复相同的键将返回原始响应，而无需重新执行突变。

> **注意**
>
> 速率限制因密钥、租户策略和端点系列而异。超出限制将返回带有 `Retry-After` 标头的 HTTP 429；当你退缩时要尊重它。

---

## 创建一台计算机

**`POST /api/v1/computers`**

创建一台新计算机并开始配置。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `name` |字符串|是的 |显示名称 |
| `template_type` |字符串|是的 |模板（`"miosa-desktop"`）|
| `size` |字符串|没有 | `"small"` (4GB/1CPU)、`"medium"` (8GB/2CPU)、`"large"` (16GB/4CPU)。默认值：`"small"` |
| `selected_apps` |字符串[] |没有 |启动后安装的应用程序 |
| `workspace_id` | UUID |没有 |将此计算机分配到的工作空间。默认为租户的默认工作区。 |
| `workspace_slug` |字符串|没有 |现有或自动创建的工作区 slug。 |
| `workspace_name` |字符串|没有 |工作区显示名称（如果自动创建）。 |
| `project_id` | UUID |没有 |要将此计算机分配到的项目。默认为工作区默认项目。 |
| `project_slug` |字符串|没有 |现有或自动创建的项目 slug。 |
| `project_name` |字符串|没有 |项目显示名称（如果自动创建）。 |
| `external_workspace_id` |字符串|没有 |您自己的客户/帐户/工作区 ID。 |
| `external_user_id` |字符串|没有 |您自己的最终用户 ID。 |
| `external_project_id` |字符串|没有 |您自己的项目/应用程序/文档 ID。 |
| `agent_runtime_profile_id` | UUID/字符串 |没有 |强制该计算机使用特定的智能体运行时配置文件。别名：`agent_profile_id`。 |
| `skip_agent_runtime_profile` |布尔 |没有 |如果 `true`，则不应用租户/工作区默认运行时配置文件。 |

如果工作区或租户存在默认智能体运行时配置文件并且它
适用于计算机，MIOSA 自动应用它。配置文件环境变量是
存储在加密的计算机环境存储中，并在 OSA 启动后同步到虚拟机中
健康。配置文件连接器通过 MIOSA 出口绑定为不透明
`miosa-tok-*` 占位符标记。

### 响应  -  `201 Created`
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "tenant_id": "...",
  "owner_user_id": "...",
  "name": "my-computer",
  "slug": "my-computer",
  "template_type": "miosa-desktop",
  "size": "small",
  "vcpus": 1,
  "memory_mb": 4096,
  "status": "provisioning",
  "vm_id": null,
  "workspace_id": "660e8400-e29b-41d4-a716-446655440001",
  "project_id": "770e8400-e29b-41d4-a716-446655440002",
  "external_workspace_id": null,
  "external_user_id": null,
  "external_project_id": null,
  "sandbox_url": "https://my-computer.sandbox.miosa.ai",
  "desktop_url": "https://my-computer.computer.miosa.ai/desktop/index.html",
  "selected_apps": [],
  "settings": {},
  "ai_config": {},
  "agent_session_id": null,
  "agent_status": null,
  "resolution": "1280x720",
  "auto_stop": 0,
  "created_at": "2026-04-11T00:00:00Z",
  "updated_at": "2026-04-11T00:00:00Z"
}
```

`sandbox_url` 是计算机响应中的旧版预览字段。使用 `desktop_url` 作为桌面流，使用 `/computers/{id}/urls` 端点作为短期桌面和终端 URL。

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 400 | `name is required` |缺少名称或 template_type |
| 402 | `INSUFFICIENT_CREDITS`|没有足够的积分来创建 |
| 404 | `AGENT_RUNTIME_PROFILE_NOT_FOUND`|此租户中不存在显式配置文件 ID |
| 422 | `TENANT_RESOLUTION_FAILED`|无法确定用户的租户 |
| 422 | `AGENT_RUNTIME_PROFILE_NOT_APPLICABLE`|显式配置文件不适用于计算机 |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "my-computer",
    "template_type": "miosa-desktop",
    "size": "small",
    "workspace_slug": "acme-corp",
    "workspace_name": "Acme Corp",
    "project_slug": "records-portal",
    "project_name": "Records Portal",
    "external_workspace_id": "acct_123",
    "external_user_id": "user-456",
    "external_project_id": "project_789",
    "agent_runtime_profile_id": "9f4b3d8e-0000-0000-0000-000000000000"
  }'
```

---

## 列出计算机

**`GET /api/v1/computers`**

返回属于经过身份验证的用户的租户的所有计算机。

### 查询参数

|参数|类型 |描述 |
|------------|------|-------------|
| `workspace_id` | UUID |将计算机过滤到特定工作区。省略返回所有工作区中的计算机。 |
| `project_id` | UUID |将计算机过滤到特定项目。 |
| `external_workspace_id` |字符串|按您的客户/帐户/工作区 ID 进行过滤。 |
| `external_user_id` |字符串|按您的最终用户 ID 进行过滤。 |
| `external_project_id` |字符串|按您的项目/应用程序/文档 ID 进行过滤。 |

### 响应  -  `200 OK`
```json
{
  "computers": [
    {
      "id": "...",
      "name": "my-computer",
      "status": "running",
      "size": "small",
      "created_at": "2026-04-11T00:00:00Z"
    }
  ],
  "total": 1
}
```
```bash
# All computers
curl https://api.miosa.ai/api/v1/computers \
  -H "Authorization: Bearer $MIOSA_API_KEY"

# Filtered to a workspace
curl "https://api.miosa.ai/api/v1/computers?workspace_id=660e8400-e29b-41d4-a716-446655440001" \
  -H "Authorization: Bearer $MIOSA_API_KEY"

# Filtered to a project
curl "https://api.miosa.ai/api/v1/computers?project_id=770e8400-e29b-41d4-a716-446655440002" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 获取一台电脑

**`GET /api/v1/computers/{id}`**

### 路径参数

|参数|类型 |描述 |
|------------|------|-------------|
| `id` | UUID |计算机 ID |

### 响应  -  `200 OK`

完整的计算机对象（与创建响应相同，具有当前状态）。

### 错误

| 状态 | 错误 | 原因 |
|--------|--------|--------|
| 400 | `invalid computer id` |不是有效的 UUID |
| 404 | `computer not found`|不存在或属于不同的租户 |
```bash
curl https://api.miosa.ai/api/v1/computers/{id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 更新计算机

**`PATCH /api/v1/computers/{id}`**

目前仅支持更新`agent_session_id`。

### 请求正文

|领域|类型 |描述 |
|--------|------|-------------|
| `agent_session_id` |字符串|链接 AI 智能体会话 |

### 响应  -  `200 OK`

更新的计算机对象。
```bash
curl -X PATCH https://api.miosa.ai/api/v1/computers/{id} \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"agent_session_id": "session-uuid"}'
```

---

## 删除计算机

**`DELETE /api/v1/computers/{id}`**

永久销毁虚拟机并删除计算机记录。

### 响应  -  `202 Accepted`

返回计算机对象；它的 `state` 反映了正在进行的破坏。

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 403 | `not a member of this computer`|无法访问此计算机 |
| 404 | `computer not found`|不存在|
```bash
curl -X DELETE https://api.miosa.ai/api/v1/computers/{id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 启动计算机

**`POST /api/v1/computers/{id}/start`**

恢复停止或暂停的计算机。

### 响应  -  `200 OK`

使用 `status: "running"` 更新了计算机对象。

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 402 | `INSUFFICIENT_CREDITS`|学分不足 |
| 409 | `computer cannot be started from its current status`|未处于可启动状态|
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/start \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 停止计算机

**`POST /api/v1/computers/{id}/stop`**

暂停正在运行的计算机。稍后可以重新启动。

### 响应  -  `200 OK`

使用 `status: "stopped"` 更新了计算机对象。

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 409 | `computer cannot be stopped from its current status`|没有运行 |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/stop \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 重新启动计算机

**`POST /api/v1/computers/{id}/restart`**

停止并立即重新启动正在运行的计算机。

### 响应  -  `200 OK`

更新的计算机对象。

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 409 | `computer must be running to restart`|没有运行 |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/restart \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 获取自动停止配置

**`GET /api/v1/computers/{id}/auto-stop`**

### 响应  -  `200 OK`
```json
{
  "auto_stop_seconds": 3600,
  "enabled": true
}
```
```bash
curl https://api.miosa.ai/api/v1/computers/{id}/auto-stop \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 更新自动停止配置

**`PATCH /api/v1/computers/{id}/auto-stop`**

### 请求正文

|领域|类型 |描述 |
|--------|------|-------------|
| `seconds` |整数 |自动停止前不活动的秒数。 `0` 禁用。 |

### 响应  -  `200 OK`

更新了自动停止配置。
```bash
curl -X PATCH https://api.miosa.ai/api/v1/computers/{id}/auto-stop \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"seconds": 3600}'
```

---

## 获取桌面流凭证

**`GET /api/v1/computers/{id}/vnc-credentials`**

返回直接桌面流访问的凭据。

### 响应  -  `200 OK`
```json
{
  "desktop_url": "https://my-computer.computer.miosa.ai/desktop/index.html",
  "ws_url": "wss://my-computer.computer.miosa.ai/ws/vnc/550e8400-e29b-41d4-a716-446655440000?auth=stream_token",
  "token": "stream_auth_token",
  "expires_at": 1712700060,
  "computer_id": "550e8400-e29b-41d4-a716-446655440000",
  "slug": "my-computer"
}
```
```bash
curl https://api.miosa.ai/api/v1/computers/{id}/vnc-credentials \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 获取流令牌

**`POST /api/v1/computers/{id}/stream-token`**

返回桌面和终端 WebSocket 连接的有时间限制的身份验证令牌。

### 响应  -  `200 OK`
```json
{
  "token": "stream_auth_token",
  "expires_at": 1712700060
}
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/stream-token \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 获取计算机 URL

**`GET /api/v1/computers/{id}/urls`**

返回计算机的所有连接 URL。

### 响应  -  `200 OK`
```json
{
  "desktop_url": "https://my-computer.computer.miosa.ai/desktop/index.html",
  "stream_url": "https://my-computer.computer.miosa.ai/desktop/index.html",
  "terminal_url": "wss://my-computer.computer.miosa.ai/ws/terminal/550e8400-e29b-41d4-a716-446655440000",
  "computer_id": "550e8400-e29b-41d4-a716-446655440000"
}
```
```bash
curl https://api.miosa.ai/api/v1/computers/{id}/urls \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 列出已安装的应用程序

**`GET /api/v1/computers/{id}/apps`**

返回计算机上安装的应用程序列表。

### 响应  -  `200 OK`

包含已安装应用程序列表的响应。
```bash
curl https://api.miosa.ai/api/v1/computers/{id}/apps \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 调整计算机大小

**`POST /api/v1/computers/{id}/resize`**

更改 vCPU、内存和磁盘分配。正在运行的计算机已就地调整大小。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `cpu` |整数 |没有 |目标 vCPU 数量 |
| `memory_mb` |整数 |没有 |目标内存（以 MiB 为单位）|
| `disk_gb` |整数 |没有 |目标磁盘（以 GiB 为单位）|

### 响应  -  `200 OK`

使用新分配更新了计算机对象。

```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/resize \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"cpu": 2, "memory_mb": 8192, "disk_gb": 32}'
```

---

## 移动计算机

**`POST /api/v1/computers/{id}/move`**

将计算机移动到同一组织中的不同工作区。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `workspace_id` |字符串|是的 |目标工作区 ID |

### 响应  -  `200 OK`

使用新的 `workspace_id` 更新了计算机对象。

```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/move \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"workspace_id": "550e8400-e29b-41d4-a716-446655440001"}'
```

---

## 获取指标

**`GET /api/v1/computers/{id}/metrics`**

返回时间序列 CPU、RAM 和信用消耗指标。对于仪表板和自动停止逻辑很有用。

### 查询参数

|参数|类型 |描述 |
|------------|------|-------------|
| `window` |字符串|时间窗口：`"1h"`（默认）、`"24h"`、`"7d"` |

### 响应  -  `200 OK`

```json
{
  "cpu_percent": [
    {"ts": "2026-05-17T10:00:00Z", "value": 12.4},
    {"ts": "2026-05-17T10:01:00Z", "value": 8.1}
  ],
  "memory_percent": [
    {"ts": "2026-05-17T10:00:00Z", "value": 45.2}
  ],
  "credits_per_hour": 3
}
```

```bash
curl "https://api.miosa.ai/api/v1/computers/{id}/metrics?window=1h" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 环境变量

加密的每台计算机环境变量。值在启动时解密并注入到虚拟机环境中。

### 列表

**`GET /api/v1/computers/{id}/env`**

```bash
curl https://api.miosa.ai/api/v1/computers/{id}/env \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

### 创建

**`POST /api/v1/computers/{id}/env`**

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `name` |字符串|是的 |变量名称（例如 `DATABASE_URL`）|
| `value` |字符串|是的 |明文值 - 静态加密 |

```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/env \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "DATABASE_URL", "value": "postgres://user:pass@host/db"}'
```

### 更新

**`PATCH /api/v1/computers/{id}/env/{name}`**

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `value` |字符串|是的 |新的明文值 |

### 删除

**`DELETE /api/v1/computers/{id}/env/{name}`**

返回 `200 OK`。

---

## 端口暴露

控制计算机内部运行的服务的每个端口的可见性。

### 列出端口

**`GET /api/v1/computers/{id}/ports`**

### 暴露端口

**`POST /api/v1/computers/{id}/ports`**

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `port` |整数 |是的 |端口号 (1-65535) |
| `visibility` |字符串|是的 | `"public"`、`"private"` 或 `"protected"` |

### 更新端口可见性

**`PATCH /api/v1/computers/{id}/ports/{port}`**

### 删除端口

**`DELETE /api/v1/computers/{id}/ports/{port}`**

```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/ports \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"port": 3000, "visibility": "public"}'
```

---

## 常见错误

|状态 |代码|原因 |
|--------|------|--------|
| 400 | `invalid computer id` |路径参数不是有效的 UUID |
| 402 | `INSUFFICIENT_CREDITS`|没有足够的积分来启动或创建 |
| 403 | `not a member of this computer`|无法访问此计算机 |
| 404 | `computer not found`|不存在或属于不同的租户 |
| 409 | `COMPUTER_NOT_RUNNING`|操作需要运行电脑|
| 502 | `AGENT_UNAVAILABLE`|计算机命令服务不可用 |

---

## 另请参阅

- [桌面 API](/docs/api-reference/desktop) - 屏幕截图、单击、键入、滚动、拖动、窗口
- [文件 API](/docs/api-reference/files) - 读取、写入、统计、mkdir、重命名、复制、chmod、下载
- [Exec API](/docs/api-reference/exec) - bash 和 Python 执行
- [快照 API](/docs/api-reference/snapshots) - 创建、恢复和删除检查点
- [事件 (SSE)](/docs/api-reference/events) - 计算机生命周期事件流
- [Regions](/docs/api-reference/regions) - 用于计算机放置的可用区域
