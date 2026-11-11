---
slug: "services/releases"
title: "服务发布"
description: "部署服务、跟踪发布状态机、读取失败原因、提升或回滚，以及取消发布。"
section: "Services & projects"
order: 1012
---

发布是服务的一次构建并运行尝试。
它是不可变的：会冻结其所使用的源代码、设置和构建出的镜像。

## 部署

```bash
miosa service deploy shop API                        # rebuild the service's own source
miosa service deploy shop API --dir . --wait         # upload a directory
miosa service deploy shop API --repo https://github.com/acme/api --branch main
miosa service deploy shop API --image ghcr.io/acme/api:1.4 --port 8080
```

每次发布都会接受一个 `Idempotency-Key`；CLI 会发送一个，因此重试的命令会返回第一次的发布，而不会启动第二次发布。

`--wait` 会跟踪发布进度，如果失败则以非零状态退出，并打印原因。

## 状态机

| 状态 | 含义 |
|---|---|
| `queued` | 已接受，等待工作进程。 |
| `building` | 正在构建源代码。 |
| `built` | 镜像已存在；发布即将启动副本。 |
| `pre_deploy` | `pre_deploy_command` 正在新副本中运行。 |
| `starting` | 副本正在启动。 |
| `health_check` | 正在探测健康检查路径。 |
| `active` | 正在提供服务。 |
| `sleeping` | 空闲休眠已停止副本；下一次请求会将其唤醒。 |
| `draining` | 较早的发布正在停止。 |
| `crashed` | 副本崩溃，且重启策略未能恢复。 |
| `failed` | 最终失败，并附带原因。 |
| `canceled` | 已被用户取消。 |
| `stopped` | 已通过 `stop` 停止。 |

未在截止时间前离开某个状态的发布，会被清理器以 `deadline_exceeded` 结算。

## 查看失败原因

```bash
miosa service releases shop API
miosa service logs shop API
miosa service logs shop API --release 6
```

`logs` 会打印每次状态转换以及最新发布（或你指定发布）的失败原因。
发布事件会携带发布失败的原因。
构建日志和每个副本的运行时输出位于控制台的日志标签页；请参阅[日志](/docs/services/logs)。CLI 不会流式传输这些内容。

运行中的副本打印的内容需要通过单独的读取操作获取：

```text
GET /projects/:id/services/:sid/logs                    # the active release
GET /projects/:id/services/:sid/deployments/:did/logs   # one release
```

每个端点都会返回每个副本输出的最后几行，并遮盖你的机密值。
`tail` 设置行数（1 到 1000，默认 200），`replica` 可将范围缩小到单个副本。
无法读取的副本会在自己的条目中说明，且不会隐藏其他副本。
这是快照，而不是流。

常见原因代码：

| 代码 | 处理方式 |
|---|---|
| `build_failed` | 查看构建输出；命令以非零状态退出。 |
| `clone_failed` | 仓库或分支无法访问，或者凭证有误。 |
| `image_pull_failed` | 镜像引用有误，或者缺少镜像仓库密钥。 |
| `healthcheck_timeout` | 应用未及时响应其健康检查路径。请检查 `start_command` 和端口。 |
| `pre_deploy_failed` | `pre_deploy_command` 以非零状态退出；没有服务在运行。 |
| `no_capacity` | 该区域没有主机能容纳此规格。请尝试其他区域或更小的 `limits`。 |
| `reference_missing` | 某个变量引用了不存在的内容。`miosa service vars effective` 会列出它们。 |
| `reference_cycle` | 两个变量相互引用。 |
| `deadline_exceeded` | 发布在某个状态停留过久，清理器已将其结算。 |
| `quota_exceeded` | 已达到计划限制。 |

## 提升与回滚

回滚会使用较早发布构建的镜像及其冻结的源代码和设置来提供服务。
不会重新构建任何内容，因此速度很快。

```bash
miosa service rollback shop API 3 --yes
miosa service promote  shop API 3 --yes   # the same call
```

服务 API 对此只有一个操作，称为 rollback。
CLI 接受 `promote` 作为“再次提供此发布”的名称；没有单独的 promote 路由。

## 取消与删除

```bash
miosa service cancel shop API 7     # a queued or building release
miosa service release shop API 3    # one release
```

已完成的发布可以从历史记录中删除；仍在运行的发布则不能。

## 停止、启动、重启

```bash
miosa service stop    shop API --yes   # stop every active release now
miosa service start   shop API         # start a stopped service
miosa service restart shop API         # queue a release that restarts it
```

如果某个副本无法停止，`stop` 会以 `SERVICES_NOT_STOPPED` 失败，因此服务绝不会在仍在运行时被报告为已停止。
对已处于运行或启动中状态的服务执行 `start` 会返回 `SERVICE_CONFLICT`。

## 多个服务的发布与回滚

每个服务独立发布。
对于需要协调的变更，请应用模板或变更集，它会暂存每个服务的变更并将它们一起部署。
