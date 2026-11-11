---
slug: "opencomputers/containers"
title: "主机上的容器"
description: "在一台主机上运行 Docker 或 Podman 容器，流式传输其日志和资源使用情况，并保持 docker-compose 项目运行。"
section: "Computers"
order: 3016
---

主机运行自己的容器运行时，MIOSA 管理其上的容器：创建、启动、停止、重新启动、删除以及流式传输日志和资源使用情况。
一切都由主机 ID 和容器 ID 寻址，并且每个操作都是租户范围内的。

MIOSA 不会为您构建形象。您的图像必须已可由主机的运行时拉取。

## 运行容器

**MIOSA CLI**

```bash
miosa host container run <host> --image nginx:alpine --name web --set PORT=8080
miosa host container list <host>
miosa host container get <host> <container>
```

`miosa host container run` 采用 `--image`、`--name`、`--cmd`（可重复）、`--set KEY=VALUE`（可重复）和 `--restart`。
别名：`miosa host containers`、`miosa host container create`。

**休息API**

```bash
curl --fail-with-body -sS \
  -X POST "https://api.miosa.ai/api/v1/opencomputers/hosts/$HOST_ID/containers" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name":"web","image":"nginx:alpine","env":{"PORT":"8080"},"ports":[{"host_port":8080,"container_port":8080}]}'
```

|领域|类型 |必填 |描述 |
|---|---|---|---|
| `name` |字符串|是的 | 1 到 63 个字符，以字母或数字开头 |
| `image` |字符串|是的 |主机运行时可以拉取的镜像参考|
| `command` |字符串[] |没有 |命令和参数；默认 `[]` （图像默认）|
| `env` |对象|没有 |环境变量作为映射；默认 `{}` |
| `ports` |对象[] |没有 | `{host_port, container_port, protocol}`; `protocol` 默认为 `tcp` |
| `volumes` |对象[] |没有 | `{source, target, readonly}`; `readonly` 默认为 `false` |
| `restart_policy` |字符串|没有 | `no`、`on-failure`、`always` 或 `unless-stopped`（默认） |
| `runtime` |字符串|没有 | `auto`（默认）、`docker` 或 `podman` |

响应为 `{"container": {...}}` 和 `id`、`name`、`image`、`command`、`env`、`ports`、`volumes`、`restart_policy`、`runtime`、 `state`、`runtime_id`、`runtime_name`、`exit_code`、`started_at`、`stopped_at`、`last_error`、`last_cpu_percent`、`last_mem_mb` 和 `last_stats_at`。

容器状态为 `provisioning`、`running`、`stopped`、`failed`、`removing` 和 `removed`。

> **注意：检查状态，不要假设**
>
> `create` 主机接受请求后立即返回。在依赖容器之前请阅读 `state`：`provisioning` 不是 `running`，而 `last_error` 是启动失败的地方。

## 日志

**MIOSA CLI**

```bash
miosa host container logs <host> <container>
miosa host container logs <host> <container> --json
```

一直流式传输，直到连接结束或按 Ctrl-C。对于 `--json`，每个事件都是每行一个 JSON 对象。

**休息API**

```bash
curl -N --fail-with-body -sS \
  "https://api.miosa.ai/api/v1/opencomputers/hosts/$HOST_ID/containers/$CONTAINER_ID/logs" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

`{stream, line, ts}` 帧的 SSE 流，其中 `stream` 是容器上的源流。

## 统计数据

**MIOSA CLI**

```bash
miosa host container stats <host> <container>
miosa host container stats <host> <container> --json
```

**休息API**

```bash
curl -N --fail-with-body -sS \
  "https://api.miosa.ai/api/v1/opencomputers/hosts/$HOST_ID/containers/$CONTAINER_ID/stats" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

一个 SSE 流，大约每 10 秒用 `{container_id, cpu_percent, mem_mb, stats_at}` 计时一次，并在容器停止时发送最终的 `{state, container_id}` 帧。

最新样本也以 `last_cpu_percent`、`last_mem_mb` 和 `last_stats_at` 形式保存在容器记录中。

## 生命周期

```bash
miosa host container start <host> <container>
miosa host container stop <host> <container>
miosa host container restart <host> <container>
miosa host container rm <host> <container> --force --yes
```

|行动|休息 |笔记|
|---|---|---|
|开始| `POST /opencomputers/hosts/{host_id}/containers/{id}/start` |仅来自 `stopped` 或 `failed`；否则返回 `409` |
|停止| `POST .../containers/{id}/stop` |仅来自 `running`；接受 `timeout_s` 宽限期，默认 10 |
|重新启动 | `POST .../containers/{id}/restart` |允许来自 `running`、`stopped` 或 `failed` |
|更新 | `PATCH .../containers/{id}` | `image`、`command`、`env`、`ports`、`volumes`、`restart_policy`、`runtime` |
|删除| `DELETE .../containers/{id}?force=true` | `force` 删除正在运行的容器；返回 `202` |

删除容器不会为您删除其卷，除非您要求将卷与容器一起删除。

## 编写项目

compose 项目是主机持续运行的 docker-compose 文件。

```bash
miosa host compose list <host>
miosa host compose up <host> app --build --pull
miosa host compose down <host> app --remove-volumes
miosa host compose rm <host> app --yes
```

|行动|休息 |
|---|---|
|列表 | `GET /opencomputers/hosts/{host_id}/compose-projects` |
|创建| `POST .../compose-projects` 与 `name`、`yaml`（必需）、`env`、`pull`、`build` |
|上 | `POST .../compose-projects/{id}/up` |
|向下| `POST .../compose-projects/{id}/down` 与 `remove_volumes` |
|删除| `DELETE .../compose-projects/{id}` |
|活动 | `GET .../compose-projects/{id}/events`（上海证券交易所）|

Compose 项目状态为 `creating`、`running`、`partial`、`stopped`、`failed` 和 `removing`。
事件流携带 `{type, payload}` 帧，其中 `type` 是 `progress`、`log` 或 `error`。`--remove-volumes` 破坏项目的卷。在将其用于有状态项目之前，请先确认您打算失去什么。

## 计划限制

租户可以使用的容器槽数量取决于计划，并且组合项目根据相同的预算来计算其服务。

|计划|容器和组合服务槽 |
|---|---|
| `free` | 2 |
| `starter` | 10 |
| `pro` | 50 |
| `team` | 250 |
| `enterprise` | 1000 |

超出限制将返回 `402` 和 `error: "plan_limit_reached"`，加上 `limit` 和 `current` 值。
请参阅[计费和限制](/docs/opencomputers/billing)了解其他计划范围的限制。

> **注意：主机上的容器不是沙箱**
>
> 容器共享主机的内核。它们不是 Firecracker microVM，并且不会为您提供[沙箱](/docs/develop/sandboxes) 的隔离或一次性生命周期。
> 当您希望在自己的机器上获得长期服务时，请使用容器。
