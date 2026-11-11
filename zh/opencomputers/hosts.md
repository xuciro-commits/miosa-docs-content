---
slug: "opencomputers/hosts"
title: "主机和功能"
description: "检查注册的主机、读取其运行时和服务、在一台主机或一组主机上运行命令、管理文件、应用程序、编写项目、机密、SSH 密钥和集群。"
section: "Computers"
order: 3015
---

主机是一台注册的机器。
每个主机都通过其主机 ID 进行寻址，每个主机命令都将 `<host>` 作为其第一个参数。
主机ID稳定；计算机的 IP 地址和主机名不是 MIOSA 使用的身份。

```bash
miosa host list --tag studio
miosa host get <host>
```

`miosa host list` 采用 `--tag` 来缩小带有标签的主机。
`miosa host get <host>` 返回主机记录及其平台、区域、标签和报告的功能。
别名：`miosa hosts`、`miosa host`。

## 主机报告的内容

|命令 |它告诉你什么 |
|---|---|
| `miosa host get <host>` |主机记录：状态、平台、区域、标签、功能 |
| `miosa host metrics <host>` |主机CPU、内存、磁盘|
| `miosa host runtime <host>` |主机正在运行的容器运行时 |
| `miosa host services <host>` |主机上运行的服务 |
| `miosa host jobs <host>` |最近发送到主机的命令 |

仅注册既不能证明准备就绪，也不能证明功能平等。
将新的心跳加上已完成的操作视为主机可用于您所需功能的证据。

> **注意：检查功能，而不是接口**
>
> 通用 MIOSA 接口并不意味着每个主机都支持所有托管操作。
> 直接执行在主机用户的操作系统权限内运行；它不会自动将每个命令放置在隔离的 microVM 中。
> 注册不会将任意主机变成沙箱工作池或 Docker Host 主机。

## 运行命令

发送到主机的命令是作业。

```bash
miosa host job list <host>
miosa host job get <host> <job>
miosa host job cancel <host> <job>
```

`miosa host job` 具有别名 `jobs` 和 `exec`，因此 `miosa host exec <host> -- uname -a` 是同一个曲面。
取消作业并不能证明远程进程已停止；在重试有副作用的工作之前确认结果。

REST 等效项是 [exec 端点 ](/docs/api-reference/open-computers#run-a-job)，它默认流式传输 SSE，并在发送 `"stream": false` 时返回收集的输出。

### 在组中的每个主机上运行

组是一组命名的主机。在组上运行命令或安装应用程序会在每个成员上运行它。

```bash
miosa host group new ci --description "CI workers"
miosa host group add ci <host>
miosa host group exec ci --cmd npm --arg test --cwd /srv/app --timeout-ms 120000
miosa host group get ci
miosa host group remove ci <host>
miosa host group rm ci --yes
```

删除组会保留其主机。

## 主机上的文件

路径位于主机自己的文件系统上，而不是沙箱工作区上。

```bash
miosa host fs ls <host> --path /srv/app
miosa host fs stat <host> --path /srv/app/package.json
miosa host fs mkdir <host> --path /srv/app/logs
miosa host fs rm <host> --path /srv/app/old.log --yes
```

`miosa host fs ls` 默认为智能体用户的主目录。
通过 `/opencomputers/hosts/{id}/fs/*` 下的 REST 可以进行相同的操作 - 请参阅 [OpenComputers API](/docs/api-reference/open-computers#file-system)。

## 标签

为主机添加标签以进行选择和批量工作。

```bash
miosa host tag add <host> gpu
miosa host tag list
miosa host tag rm <host> gpu
```

`miosa host list --tag gpu` 然后选择主机，而无需您手动跟踪 ID。

## 秘密

主机机密是主机智能体可以在运行时使用的加密键值对。
列表调用永远不会返回值。

```bash
miosa host secret list <host>
miosa host secret set <host> GITHUB_TOKEN --description "CI token"
miosa host secret set <host> DEPLOY_KEY --key-stdin < deploy.key
miosa host secret rm <host> GITHUB_TOKEN --yes
```

`secret set` 提示输入值或使用 `--key-stdin` 从标准输入读取该值，因此该值不会出现在 shell 历史记录中。

## SSH 密钥

仅处理公钥。

```bash
miosa host ssh-key add <host> --name roberto --public-key "ssh-ed25519 AAAA..."
miosa host ssh-key import <host> <github-user>
miosa host ssh-key list <host>
miosa host ssh-key rm <host> <key> --yes
```

`ssh-key import` 复制 GitHub 用户已发布的公钥。

## 应用程序

目录列出了 MIOSA 可以安装的内容； install 将一个安装到主机上。

```bash
miosa host app catalog
miosa host app install <host> redis
miosa host app list <host>
miosa host app health <host> redis
miosa host app start <host> redis
miosa host app uninstall <host> redis --yes
```

安装后检查 `miosa host app health`，而不是假设应用程序已启动。

## 编写项目

compose 项目是主机持续运行的 docker-compose 文件。

```bash
miosa host compose list <host>
miosa host compose up <host> app --build --pull
miosa host compose down <host> app --remove-volumes
miosa host compose rm <host> app --yes
```

`--build` 首先构建图像，`--pull` 首先拉取它们。
`--remove-volumes` 会删除项目的卷，因此在有状态项目上使用它之前请确认您打算丢失什么。

## 容器

主机上的容器包含在 [Containers](/docs/opencomputers/containers) 中。

## 集群

集群是一组共同服务一个模型的主机，作为 OpenAI 兼容端点公开。

```bash
miosa host cluster list
miosa host cluster get <cluster>
miosa host cluster start <cluster>
miosa host cluster stop <cluster>
miosa host cluster rm <cluster> --yes
```

集群是通过 API 使用 `host_ids`、`name` 和 `model` 创建的；请参阅[推理集群](/docs/api-reference/open-computers#inference-clusters)。

## 删除主机

```bash
miosa host rm <host> --yes
```

删除主机会终止其身份验证能力。
它不会删除机器的文件，也不会证明其进程已停止。

- [Containers](/docs/opencomputers/containers)：在主机上运行图像并流式传输日志和统计信息。

- [安全模型](/docs/opencomputers/security)：主机智能体可以到达的内容以及MIOSA可以看到的内容。

- [OpenComputers API](/docs/api-reference/open-computers)：REST 上的主机、作业、文件、隧道、智能体、集群和机密。
