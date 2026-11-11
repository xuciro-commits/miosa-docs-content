---
slug: "services/cli"
title: "服务 CLI"
description: "所有 miosa service 命令和标志，涵盖创建、部署、变量、域名、发布和回滚。"
section: "Services & projects"
order: 1018
---

`miosa service` 用于操作服务（Services）。
`miosa services` 和 `miosa svc` 是别名。
沙箱中具名的后台进程是 `miosa daemon`，属于另一项产品。

`<project>` 是项目名称、slug 或 ID。
`<service>` 是服务名称、slug 或 ID；CLI 会将名称解析为 API 所需的 ID。
`<release>` 是发布编号或 ID。
每个命令都接受 `--json`。

## 命令

| 命令 | 作用 |
|---|---|
| `service create <project> <name>` | 创建 Web 服务、worker、私有服务、静态站点、cron 作业或数据库服务 |
| `service list <project>` | 列出项目的服务 |
| `service get <project> <service>` | 显示单个服务 |
| `service update <project> <service>` | 更改名称、启动命令、副本数或区域 |
| `service deploy <project> <service>` | 构建并发布 |
| `service logs <project> <service>` | 显示某次发布的状态转换和失败原因 |
| `service scale <project> <service>` | 更改副本数、区域或限制 |
| `service start <project> <service>` | 启动已停止的服务 |
| `service stop <project> <service>` | 立即停止所有活动发布 |
| `service restart <project> <service>` | 排队一次重启服务的发布 |
| `service delete <project> <service>` | 删除服务并停止其副本；数据库服务还会删除其数据库 |
| `service releases <project> <service>` | 列出发布，按最新优先 |
| `service release <project> <service> <release>` | 显示单次发布 |
| `service rollback <project> <service> <release>` | 再次提供较早的发布 |
| `service promote <project> <service> <release>` | 与 rollback 相同 |
| `service cancel <project> <service> <release>` | 取消排队中或构建中的发布 |
| `service domains <project> <service>` | 显示私有地址和公共地址 |
| `service domains enable <project> <service>` | 为服务分配公共地址 |
| `service domains disable <project> <service>` | 移除公共地址 |
| `service vars list <project> <service>` | 列出变量，值会掩码 |
| `service vars set <project> <service> KEY=VALUE...` | 设置变量 |
| `service vars unset <project> <service> NAME[,NAME]` | 删除变量 |
| `service vars effective <project> <service>` | 显示部署时将收到的变量 |
| `service vars reveal <project> <service> NAME` | 打印单个值 |
| `service runs <project> <service>` | cron 服务最近的运行记录 |
| `service connection <project> <service>` | 数据库服务的 URL |
| `service settings <project>` | 每项设置当前的作用 |
| `service templates` | 列出模板目录 |
| `service templates apply <project> <slug>` | 应用模板 |
| `project vars list <project>` | 共享变量，值会掩码 |
| `project vars set <project> KEY=VALUE...` | 设置共享变量 |
| `project vars unset <project> NAME` | 删除共享变量 |
| `project target list <project>` | 列出部署目标 |
| `project target create <project> <name>` | 创建部署目标 |
| `project target duplicate <project> <target> --name <new>` | 复制部署目标的服务和变量 |
| `project target rm <project> <target>` | 删除部署目标 |

## `service create` 标志

| 标志 | 含义 |
|---|---|
| `--type` | `web`、`worker`、`private`、`static`、`schedule` 或 `database`（默认 `web`） |
| `--repo`、`--branch`、`--root-dir` | git 源 |
| `--image`、`--port` | 容器镜像源 |
| `--dockerfile` | git 源的 Dockerfile 路径（需要 Services 构建单元） |
| `--start-command` | 服务启动时使用的命令 |
| `--replicas` | 1 到 50 |
| `--region` | `us-west`、`us-east` 或 `us-mia` |
| `--schedule` | 五字段调度表达式，`--type schedule` 时必填 |
| `--env KEY=VALUE` | 变量；可重复，值可以是 `${{...}}` 引用 |
| `--no-public` | 不为 Web 服务或静态服务分配公共地址 |
| `--slug` | 主机标签（默认由名称生成） |
| `--deploy-target` | 部署目标名称或 ID（默认为项目的默认目标） |

除非传入 `--no-public`，Web 服务或静态服务在创建后会立即启用公共地址。
如果地址无法启用，服务仍会创建，CLI 会打印警告。

## `service deploy` 标志

| 标志 | 含义 |
|---|---|
| `--dir` | 将本地目录作为源上传 |
| `--repo`、`--branch`、`--root-dir` | 部署 git 源 |
| `--image`、`--port` | 部署容器镜像 |
| `--message` | 发布说明 |
| `--wait`、`--timeout` | 跟踪发布，直到其上线或失败 |
| `--deploy-target` | 部署目标名称或 ID |

不指定源标志时，会重新构建服务自身的源。

## `service get`

`service get` 会打印服务信息；如果写入变量导致正在运行的发布过期，还会显示 `Pending changes`。
该值为 `rebuild`（构建可见的变量发生变化）或 `redeploy`（其他任何变量），执行 `miosa service deploy` 可清除该状态。

## `service logs`

`logs` 会打印最新发布的状态转换及其失败原因。
传入 `--release <number|id>` 可查看较早的发布。
在 Services 日志流上线之前，`--follow` 会被拒绝；发布事件会携带发布失败的原因。

## 部署目标

每个项目都有一个默认部署目标，首次读取时会创建 `production`。
服务、变量和发布都位于同一个部署目标内；`--deploy-target` 用于选择目标，默认使用项目的默认目标。

```bash
miosa project target list shop
miosa project target duplicate shop production --name staging
miosa service list shop --deploy-target staging
```

复制操作会复制服务和变量；密钥会重新生成，不会复制任何数据或托管数据库。

## 确认

`stop`、`delete`、`rollback` 和 `promote` 在终端中会要求确认；在脚本中除非传入 `--yes`，否则会拒绝执行。

### 删除数据库服务

删除数据库服务也会删除其数据库及其中所有行。
此操作不可撤销，因此需要服务的准确名称，而不是 `--yes`：

- 在终端中，CLI 会要求你输入名称。
- 在脚本中，传入 `--confirm <exact name>`。
- 仅传 `--yes` 不够。
- 没有终端且没有 `--confirm` 的运行会以代码 2 退出，且不会发送任何操作。
- `--confirm` 与名称不完全匹配时也会以代码 2 退出。

```bash
miosa service delete shop Postgres --confirm Postgres
```

`--confirm` 需要 CLI 2.13.3 或更高版本。

## 自动化

每个命令配合 `--yes` 都可非交互运行（删除数据库服务需改用 `--confirm <name>`），配合 `--json` 会打印 JSON，API 出错时以非零状态退出。
API 的错误码会保留；例如过期的设置写入会以非零状态退出并返回 `STALE_SERVICE`。
