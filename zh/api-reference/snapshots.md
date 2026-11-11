---
slug: "api-reference/snapshots"
title: "快照API"
description: "用于创建、列出和恢复 microVM 快照（检查点）的 API 参考。"
section: "API reference"
order: 12025
---

快照捕获正在运行的计算机的完整内存状态（CPU 状态、RAM 和文件系统）并将其持久存储。恢复快照以在几秒钟内启动同一台计算机。

> **注意：正在寻找沙箱快照？**
>
> 本页仅记录桌面计算机快照。
> Canonical 沙箱 V1 不公开直接快照 CRUD；使用[持久性、暂停和分叉](/docs/develop/snapshots)和[分叉端点](/docs/api-reference/sandboxes#fork-a-sandbox)。

基本路径：`/api/v1/computers/{id}/snapshots`

> **注意**
>
> 快照要求计算机**运行**。创建快照不会停止计算机。

---

## 快速入门
```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// Create a snapshot
const snap = await client.snapshots.create(computerId, {
  comment: 'before-deploy-v2',
});

// Poll until ready
let status = snap.status;
while (status !== 'ready') {
  await new Promise(r => setTimeout(r, 3000));
  const updated = await client.snapshots.get(computerId, snap.id);
  status = updated.status;
}

// Restore  -  creates a new computer from the snapshot
const restored = await client.snapshots.restore(computerId, snap.id);
console.log(`Restored computer: ${restored.id}`);
```
```bash
# Create a snapshot
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/snapshots \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"comment": "before-deploy-v2"}'
```

---

## 端点

|方法|路径|描述 |
|--------|------|-------------|
| `POST` | `/computers/{id}/snapshots` |创建快照 |
| `GET` | `/computers/{id}/snapshots` |列出计算机的快照 |
| `GET` | `/computers/{id}/snapshots/{snap_id}` |获取快照 |
| `DELETE` | `/computers/{id}/snapshots/{snap_id}` |删除快照 |
| `POST` | `/computers/{id}/restore/{snap_id}` |从快照恢复|
| `GET` | `/computers/{id}/snapshots/{snap_id}/events` |SSE进展 |

---

## 创建快照

**`POST /api/v1/computers/{id}/snapshots`**

启动异步快照。返回的状态将为`"creating"`。轮询 `GET /snapshots/{snap_id}` 或订阅 SSE 事件以获取进度。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `comment` |字符串|没有 |人类可读的标签（最多 500 个字符）|

### 响应 - `201 Created`
```json
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "computer_id": "...",
    "tenant_id": "...",
    "comment": "before-deploy-v2",
    "status": "creating",
    "state_size_bytes": null,
    "memory_size_bytes": null,
    "rootfs_size_bytes": null,
    "compressed_size_bytes": null,
    "s3_bucket": null,
    "s3_prefix": null,
    "error": null,
    "created_at": "2026-04-11T00:00:00Z",
    "updated_at": "2026-04-11T00:00:00Z"
  }
}
```

### 状态状态机

创建进入上传；上传准备就绪。

删除时准备就绪转为删除。

任何状态都会因失败而失败。

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 404 | `computer not found`|计算机不存在或租户错误 |
| 409 | `computer is not running`|快照需要运行VM |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/snapshots \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"comment": "checkpoint-before-risky-op"}'
```

---

## 列出快照

**`GET /api/v1/computers/{id}/snapshots`**

### 响应 - `200 OK`
```json
{
  "data": [
    {
      "id": "...",
      "comment": "before-deploy-v2",
      "status": "ready",
      "compressed_size_bytes": 524288000,
      "created_at": "2026-04-11T00:00:00Z"
    }
  ]
}
```
```bash
curl https://api.miosa.ai/api/v1/computers/{id}/snapshots \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 获取快照

**`GET /api/v1/computers/{id}/snapshots/{snap_id}`**

### 路径参数

|参数|类型 |描述 |
|------------|------|-------------|
| `id` | UUID |计算机 ID |
| `snap_id` | UUID |快照 ID |

### 响应 - `200 OK`

完整快照对象（与创建响应形状相同）。

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 403 | `forbidden`|快照属于不同的租户 |
| 404 | `snapshot not found`|不存在|
```bash
curl https://api.miosa.ai/api/v1/computers/{id}/snapshots/{snap_id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 删除快照

**`DELETE /api/v1/computers/{id}/snapshots/{snap_id}`**

软删除快照（转换到 `"deleted"` 状态）。 S3 对象是异步清理的。

### 响应 - `200 OK`
```json
{
  "data": {
    "id": "...",
    "status": "deleted"
  }
}
```

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 404 | `snapshot not found`|不存在|
| 409 | `snapshot is not in a deletable state`|快照正在恢复 |
```bash
curl -X DELETE https://api.miosa.ai/api/v1/computers/{id}/snapshots/{snap_id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 从快照恢复

**`POST /api/v1/computers/{id}/restore/{snap_id}`**

使用快照中的状态创建一台新计算机。原来的电脑没变。

### 响应 - `201 Created`
```json
{
  "data": {
    "id": "new-computer-uuid",
    "name": "my-computer-restored",
    "status": "provisioning",
    "size": "small"
  },
  "snapshot": {
    "id": "snap-uuid",
    "status": "restoring"
  }
}
```

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 404 | `snapshot not found`|不存在|
| 409 | `snapshot is not ready`|快照必须处于 `ready` 状态 |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/restore/{snap_id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## SSE进展情况

**`GET /api/v1/computers/{id}/snapshots/{snap_id}/events`**

订阅实时快照进度。需要短期票证而不是承载令牌（浏览器无法在 EventSource 连接上设置授权）。

### 事件类型

|活动 |描述 |
|--------|-------------|
| `snapshot_creating` | VM将内存和VM状态写入磁盘|
| `snapshot_uploading` |压缩并上传到对象存储|
| `snapshot_ready` |上传完成；快照可用|
| `snapshot_failed` |不可恢复的错误 - 请参阅 `error` 字段 |

---

## 常见食谱

### 危险操作前的自动检查点
```python
from miosa import Miosa
import time

client = Miosa()

snap = client.snapshots.create(computer_id, comment="pre-migration")
while snap.status not in ("ready", "failed"):
    time.sleep(2)
    snap = client.snapshots.get(computer_id, snap.id)

if snap.status == "failed":
    raise RuntimeError(f"Snapshot failed: {snap.error}")

# Proceed with risky operation knowing you can restore
do_migration(computer_id)
```

### 基于快照的并行性（克隆基线）
```typescript
// Build a baseline environment once, clone it N times for parallel jobs
const baseline = await client.snapshots.create(setupComputerId, {
  comment: 'baseline-env',
});

// Each restore creates an independent computer from the same state
const workers = await Promise.all(
  Array.from({ length: 5 }, () =>
    client.snapshots.restore(setupComputerId, baseline.id)
  )
);
```

---

## 另请参阅

- [计算机 API](/docs/api-reference/computers) - 计算机生命周期
- [Events (SSE)](/docs/api-reference/events) - 流快照进度事件
- [错误代码](/docs/api-reference/errors) - 快照特定的错误代码
