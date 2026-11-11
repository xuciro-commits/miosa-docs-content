---
slug: "api-reference/sandboxes/manage"
title: "沙箱 API - 管理"
description: "列出、获取、销毁、暂停、恢复、重新启动、分叉、延长超时以及读取 MIOSA 沙箱的使用情况。"
section: "API reference"
order: 12016
---

基本路径：`/api/v1/sandboxes`

> **注意**
>
> 所有端点都需要 `Authorization: Bearer &lt;api_key&gt;`。

## 列出沙箱

**`GET /api/v1/sandboxes`**

返回属于经过身份验证的租户的所有沙箱。

### 查询参数

|参数|类型 |描述 |
|---|---|---|
| `workspace_id` | UUID |筛选到一个 MIOSA 工作区 |
| `project_id` | UUID |筛选到一个 MIOSA 项目 |
| `external_workspace_id` |字符串|按您的客户/帐户 ID 过滤 |
| `external_user_id` |字符串|按您的最终用户 ID 过滤 |
| `external_project_id` |字符串|按您的项目/应用程序/文档 ID 过滤 |
| `state` |字符串|按生命周期状态过滤：`provisioning`、`running`、`paused`、`destroyed`、`error`。 |

### 授权

需要不记名令牌。

### 响应 - `200 OK`
```json
{
  "data": [
    {
      "id": "sbx_01j9xr2t4fk8me3n5q",
      "template_id": "miosa-sandbox",
      "state": "running",
      "cpu_count": 2,
      "memory_mb": 4096,
      "created_at": "2026-04-25T10:00:00Z"
    }
  ]
}
```
```bash
curl "https://api.miosa.ai/api/v1/sandboxes?state=running" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 获取沙盒

**`GET /api/v1/sandboxes/{id}`**

### 授权

需要不记名令牌。

### 路径参数

|参数|类型 |描述 |
|------------|------|-------------|
| `id` |字符串|沙箱 ID。 |

### 响应 - `200 OK`

完整沙箱对象（与当前 `state` 的创建响应形状相同）。

### 错误

|状态 |代码|原因 |
|--------|------|--------|
| 404 | `NOT_FOUND`|沙箱不存在或属于不同的租户。 |
```bash
curl https://api.miosa.ai/api/v1/sandboxes/sbx_01j9xr2t4fk8me3n5q \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 销毁沙箱

**`DELETE /api/v1/sandboxes/{id}`**

永久删除沙箱，删除保存的状态并结算计费。当用户暂时完成但可能稍后恢复时使用 `POST /pause`。

### 授权

需要不记名令牌。

### 路径参数

|参数|类型 |描述 |
|------------|------|-------------|
| `id` |字符串|沙箱 ID。 |

### 响应 - `200 OK`
```json
{
  "id": "sbx_01j9xr2t4fk8me3n5q",
  "state": "destroyed",
  "total_runtime_sec": 42
}
```

### 错误

|状态 |代码|原因 |
|--------|------|--------|
| 404 | `NOT_FOUND`|沙箱不存在或属于不同的租户。 |
```bash
curl -X DELETE https://api.miosa.ai/api/v1/sandboxes/sbx_01j9xr2t4fk8me3n5q \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 暂停和恢复

**`POST /api/v1/sandboxes/{id}/pause`** 暂停正在运行的持久沙箱的来宾 vCPU 执行，同时保留其工作空间。
如果沙箱未运行，则返回 `409`。

**`POST /api/v1/sandboxes/{id}/resume`** 将暂停的沙箱转换回 `running`，如果未暂停则返回 `409`。
命令和文件操作可以在调度之前自动恢复暂停的持久沙箱。

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/{id}/pause \
  -H "Authorization: Bearer $MIOSA_API_KEY"

curl -X POST https://api.miosa.ai/api/v1/sandboxes/{id}/resume \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Idempotency-Key: resume-session-001"
```

---

## 重新启动

**`POST /api/v1/sandboxes/{id}/restart`** 在同一 ID 下重新启动正在运行的沙箱。
它会停止沙箱保留其磁盘，然后再次启动该磁盘。
文件、包和启用的服务能够保存下来；正在运行的进程不会。
当沙箱停止应答时使用它。

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/{id}/restart \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

|状态 |代码|意义|
|---|---|---|
| `200` | |重新启动：`{"id", "state": "running", "restarted": true}`。 |
| `202` | |仍在启动：`state`是`resuming`或`starting`；民意调查`GET /sandboxes/{id}`。 |
| `409` | `SANDBOX_NOT_RUNNING` |只有正在运行的沙箱才会重新启动；使用`POST /resume`。 |
| `409` | `RESTART_NOT_POWER_CYCLED` |它的主机通过内存检查点停止了它，因此它从内存中恢复：再次运行，进程没有重新启动。 |
| `502` | `RESTART_BOOT_FAILED` |它停止并保留其磁盘，但没有启动； `POST /resume` 再次启动它。 |
| `503` | `RESTART_STOP_IN_PROGRESS` |它仍然在停止；一旦 `stopped` 就恢复它。 |

---

## 分叉一个沙盒

**`POST /api/v1/sandboxes/{id}/fork`** 从正在运行的沙箱的写时复制快照创建一个新沙箱。
可选主体接受新沙箱的 `timeout_sec` 和 `template_id`。

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/{id}/fork \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: fork-experiment-001" \
  -d '{"timeout_sec":3600}'
```

Canonical public V1 公开了独立的快照创建、列表、检查、删除和恢复操作。
当客户需要当前运行状态的独立分支而不直接管理中间快照时，请使用 fork。

---

## 延长超时时间

**`POST /api/v1/sandboxes/{id}/extend`**

将活动会话超时替换为从 1 到 86400 的 `timeout_sec`。
该操作不会将持续时间添加到旧值。
省略正文将保留当前超时。

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/{id}/extend \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"timeout_sec":7200}'
```

---

## 获取沙盒使用情况

**`GET /api/v1/sandboxes/{id}/usage`**

返回测量的运行时间、配置的 vCPU 时间、活动 CPU、网络流量、估计成本和超时可见性。

```json
{
  "data": {
    "sandbox_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "state": "running",
    "runtime_sec": 47,
    "provisioned_vcpu_ms": 94000,
    "active_cpu_ms": 12000,
    "network_ingress_bytes": 2048,
    "network_egress_bytes": 4096,
    "measurement_status": {
      "active_cpu": "measured",
      "network": "measured",
      "provisioned_resources": "measured"
    },
    "estimated_cost_cents": 1,
    "timeout_sec": 3600,
    "timeout_remaining_ms": 3553000
  }
}
```通过 `measurement_status` 解释可为空的测量值。
`unavailable` 表示不存在可信值，并且相关字段为 `null`，而不是零。
`stale` 表示保留值不是当前值，必须如此标记。
`timeout_remaining_ms` 在开始之前或禁用超时强制时可以是 `null`。
