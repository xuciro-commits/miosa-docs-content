---
slug: "data/volumes"
title: "卷"
description: "附加到运行时实例的持久块存储。不适合 Postgres 的状态。"
section: "Databases & storage"
order: 4007
---

> **提示：可用**
>
> 卷已上线。
> 创建持久块设备，将其连接到同一主机上的兼容计算机，并独立于计算机生命周期保留写入。
> 与替换计算机的连接是明确的。
> 大多数工作负载应使用 [Postgres](/docs/data/postgres) 或 [Storage](/docs/data/storage)，因为本地卷是主机绑定的。

**卷**是连接到兼容计算机的持久块设备。
它可以在普通计算机重新启动后继续存在，而主机本地卷仍保持正常运行。
它不会自动跨主机移动。
有用的案例包括：

- 需要运行时本地的 SQLite 数据库。
- 重启时您无法承受丢失的缓存。
- LLM 模型权重下载一次并在实例之间共享。
- 合法需要文件系统状态（不仅仅是键值）的工作负载。

卷对于以下情况没有用：

- 有状态数据库 - 使用 [Postgres](/docs/data/postgres) 代替。它可以更好地扩展，自动备份，并且可以在主机故障时幸存下来。
- 用户上传的文件 - 使用[存储/存储桶](/docs/data/storage)。更便宜、可通过浏览器上传、可地理复制。
- 跨多个副本共享状态 - 卷通常附加到一个运行时实例。

## 创建

**打字稿**

```typescript
await miosa.projects.dataServices.create(projectId, {
  type: "volume",
  name: "model-weights",
  sizeGb: 50,
})
```

**Python**

```python
from miosa import Miosa

miosa = Miosa(api_key="msk_u_...")

# Create a 50 GB volume
vol = miosa.volumes.create(name="model-weights", size_gb=50)
print(vol["id"])  # vol_...

# List all volumes
for v in miosa.volumes.list():
    print(v["name"], v["size_gb"], "GB")

# Delete a volume
miosa.volumes.delete(vol["id"])
```

## 按照您的方式使用：API、CLI 或 SDK

无需 SDK。

**原始 API (curl)：** 原始 API 采用 `size_bytes` 和 `backend`（使用 `local`）。
```bash
# Create a 50 GB volume
curl -X POST https://api.miosa.ai/api/v1/volumes \
  -H "Authorization: Bearer $MIOSA_API_KEY" -H "Content-Type: application/json" \
  -d '{"name": "model-weights", "size_bytes": 53687091200, "backend": "local"}'

# Attach it to a running computer at a mount path
curl -X POST https://api.miosa.ai/api/v1/computers/COMPUTER_ID/volumes \
  -H "Authorization: Bearer $MIOSA_API_KEY" -H "Content-Type: application/json" \
  -d '{"volume_id": "VOL_ID", "mount_path": "/mnt/data"}'

# Detach (by attachment id)
curl -X DELETE https://api.miosa.ai/api/v1/computers/COMPUTER_ID/volumes/ATTACHMENT_ID \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

**命令行界面：**
```bash
miosa volume create model-weights --size-gb 50
miosa volume list
```

连接和分离是针对每台计算机的；用 `miosa api` 驱动它们：

```bash
# Attach the volume to a running computer at a mount path
miosa api POST /computers/<computer-id>/volumes \
  -d '{"volume_id": "<volume-id>", "mount_path": "/mnt/data"}'

# Detach by attachment id
miosa api DELETE /computers/<computer-id>/volumes/<attachment-id>
```

删除卷（必须先将其分离）：

```bash
miosa volume rm <volume-id>
```

> 本地后端卷位于一台主机上，因此它连接到同一主机上的计算机。跨主机连接返回一个明确的错误，而不是默默地失败。

## 挂载

卷被安装到运行时实例的文件系统中的以下路径：
```typescript
await miosa.services.update(serviceId, {
  volumes: [
    { dataServiceId: volumeId, mountPath: "/var/models" },
  ],
})
```

启动时，`/var/models` 是卷的内容。写入持续存在。

## 副本和卷

大多数卷附加到单个运行时实例。如果将服务扩展到 N 个副本，则默认情况下，如果每个副本标记为 `shared: true`，则每个副本都会获得**相同的卷安装**，如果标记为 `shared: false`，则每个副本都会获得自己的卷。

共享卷具有所有常见的并发警告（文件系统锁定、多个写入者接触同一文件时的文件损坏风险）。使用 Postgres 来共享状态，除非您真的知道自己在做什么。

## 限制

|限制|默认 |
|---|---|
|最大体积大小 | 500GB|
|每个项目的最大数量 | 10 |
|文件系统 | ext4 |
|主机安置|必须匹配电脑主机|

## 何时使用卷

现实的候选名单：

1. 您正在发布一个由 SQLite 支持的自托管应用程序，以保持部署简单。
2. 您正在运行 LLM 推理服务，并且不希望在每次实例启动时重新下载模型权重。
3. 您有一个可写的文件系统缓存，重建起来成本很高。

除此之外，请使用 Postgres 或 Storage。

## 另请参阅

- [概述](/docs/data/overview) - 更广泛的数据平面
- [Postgres](/docs/data/postgres) - 用于关系数据（几乎总是正确的答案）
- [存储](/docs/data/storage) - 用于用户上传的文件
