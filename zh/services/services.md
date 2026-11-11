---
slug: "services/services"
title: "服务与类型"
description: "Web 服务、Worker、私有服务、静态站点、Cron 任务和数据库服务的来源、设置、地址与限制。"
section: "Services & projects"
order: 1006
---

服务的 `kind` 是 API 存储的值。它可以是 `server`、`worker`、`cron`、`static` 或 `database`。产品类型映射如下：

| 产品类型 | `kind` | 公网地址 |
|---|---|---|
| Web 服务 | `server` | 有，启用后可用 |
| 私有服务 | `server` | 无 |
| Worker | `worker` | 无 |
| 静态站点 | `static` | 有，启用后可用 |
| Cron 任务 | `cron` | 无 |
| 数据库服务 | `database` | 无（使用 `connection`） |

不存在 `web` 或 `private` 这种 `kind`。CLI 和两个 SDK 都接受 `type: web` 或 `type: private`，并存储为 `kind: server`。私有服务是没有公网地址的 `server` 服务。

## 来源

服务从单一来源构建。

| `source.type` | 所需内容 | 说明 |
|---|---|---|
| `git` | `ref`（仓库 URL），可选的 `branch` 和 `root_dir` | 在服务构建节点上构建 |
| `image` | `ref`（OCI 镜像），可选的 `port`，可选的 `registry_secret` | 镜像按原样运行 |
| `upload` | `upload_id` | 先 PUT 一个 `.tar.gz`、`.tar.zst`、`.tar` 或 `.zip` 归档 |
| `forge` | `ref` | 由 MIOSA Forge 托管的仓库 |

Dockerfile 构建在服务构建节点上运行，并从来源中自动检测。构建节点关闭期间，Dockerfile 服务不会构建；请使用 Git 构建或镜像来源。

上传目录只需一条命令：

```bash
miosa service deploy shop API --dir . --wait
```

CLI 会打包目录（跳过 `.git`、`node_modules` 和 `.env*`），通过 `POST /uploads` 记录归档，将其 PUT 到返回的 URL，并使用 `{source: {type: "upload", upload_id}}` 部署。归档只能使用一次，并在一小时后过期。

## 设置

设置会严格校验：未知键会被拒绝，因此拼写错误不会悄悄保留。

| 设置 | 适用范围 | 含义 |
|---|---|---|
| `start_command` | 全部 | 副本启动时替换镜像入口点。 |
| `pre_deploy_command` | 全部 | 在新副本中、应用启动前运行一次。非零退出会导致本次发布失败。 |
| `healthcheck_path` | `server` | 新发布版本生效前探测的路径。 |
| `healthcheck_timeout_s` | `server` | 发布版本通过健康检查的最长等待时间。 |
| `restart_policy` | 全部 | `always`、`on_failure` 或 `never`。 |
| `max_retries` | 全部 | `on_failure` 副本被置于停止状态前的重启次数。 |
| `replicas` | 全部 | 1 到 50。可写卷会强制为 1。 |
| `region` | 全部 | `us-west`、`us-east` 或 `us-mia`。 |
| `limits` | 全部 | `{vcpu, memory_mb}`。超过 4 vCPU 或 8192 MB 会导致发布失败。 |
| `sleep` | 具有公网地址的服务 | `{enabled, idle_s}`。下一次请求会唤醒服务。 |
| `cron_schedule` | `cron` | 必填。五字段 cron，UTC。 |
| `overlap_s` | 全部 | 在停止上一个发布版本前额外等待的时间。 |
| `terminate_grace_s` | 全部 | 与 `overlap_s` 类似，在停止上一个发布版本前额外等待的时间。 |
| `volumes` | `server`、`worker` | 最多四个 `{volume, mount_path, read_only}`。 |

查看每项设置在你的平台上的实际行为：

```bash
miosa service settings shop
```

每个条目都是 `enforced`（已生效）、`stored`（已保存但尚未应用）或 `unsupported`。不要把 `stored` 设置当作已经生效来展示。

## 创建与变更

```bash
miosa service create shop API --type web --repo https://github.com/acme/api --replicas 2
miosa service update shop API --replicas 4
miosa service scale  shop API --replicas 4        # also queues a release
```

`update` 会合并设置：只更改你传入的标志。它会发送读取到的 `lock_version`；过期值会被以 `STALE_SERVICE` 拒绝，因此两位编辑者无法互相覆盖。`scale` 会更改副本数、区域或限制，并排队一个应用这些变更的发布版本。

## Cron 任务

```bash
miosa service create shop Nightly --type schedule \
  --repo https://github.com/acme/api \
  --start-command "node jobs/nightly.js" \
  --schedule "0 3 * * *"
miosa service runs shop Nightly --output
```

`runs` 会显示最新的 20 次运行，包括状态、退出码、时间和输出。单次运行最长五分钟。

## 数据库服务

数据库服务是绑定到项目的托管 Postgres 或 Redis。应用模板可创建服务并自动接好变量，也可以挂载现有的托管数据库。连接字符串只能通过 `connection` 获取，该操作会记录审计日志：

```bash
miosa service connection shop Postgres
```

## 限制与错误

| 错误 | 含义 |
|---|---|
| `SERVICE_NOT_FOUND` | 项目中不存在该 ID 的服务。API 仅通过 ID 解析服务。 |
| `STALE_SERVICE` | 你发送的 `lock_version` 比已存储的版本旧。请重新读取后重试。 |
| `SERVICE_CONFLICT` | 对已处于运行或启动中的服务执行 `start`。 |
| `SERVICES_NOT_STOPPED` | `stop` 未能停止某个副本；服务仍保持运行。 |
| `INGRESS_DISABLED` | 此平台已关闭公网地址；服务会运行，但没有可访问的地址。 |
| `PLAN_LIMIT_REACHED` | 已达到套餐的服务数量限制。 |
