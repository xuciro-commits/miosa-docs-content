---
slug: "compare/boat"
title: "MIOSA VS 船"
description: "使用截至 2026 年 10 月 23 日的 Boat 公共文档，对 MIOSA 和 Boat (boat.dev)（AI 智能体的云沙箱）进行事实比较。"
section: "Optional"
order: 17026
---

比较

Boat 和 MIOSA 都为 AI 智能体提供了自己的 Linux 机器。此页面比较了每个文档的内容，包括船的前方位置。

下面的每条船声明均来自船自己的公共网站和文档（[boat.dev](https://boat.dev/)、[docs.boat.dev](https://docs.boat.dev/)），请于 2026 年 10 月 23 日阅读。
MIOSA 声明链接到记录这些声明的页面。

## 什么是船

Boat 是一款用于 AI 智能体的云沙箱产品：带有 SSH 的持久 Ubuntu VM、内部 Docker、桌面以及来自快照的磁盘级分支 ([boat.dev](https://boat.dev/))。
2026年9月15日之前它被称为Box；旧的 `box` CLI 和端点在 2026 年 10 月 31 日之前仍然有效（[从 Box](https://docs.boat.dev/migrating-from-box.md) 迁移）。
它是一个沙箱产品，而不是部署平台：没有部署、自定义域、回滚或托管数据库命令（[始终开启的服务](https://docs.boat.dev/long-running-tasks/always-on-services.md)、[主机 CLI](https://docs.boat.dev/hosting/host-cli.md)）。

船没有出现在 ComputeSDK 运行的[基准页面](/docs/benchmarks)报告中，因此该页面没有测量比较。

## 并排的功能

| 能力 | MIOSA | boat | boat source |
|---|---|---|---|
|持久 Linux 机器 | [是的，沙箱和计算机](/docs/sandboxes-or-computers) |是的，支持 SSH 的 Ubuntu VM | [boat.dev](https://boat.dev/) |
|机器内部的Docker | [是](/docs/develop/sandboxes/docker) |是的 | [boat.dev](https://boat.dev/) |
|完整桌面| [是的，计算机](/docs/computers/overview) |是的 | [boat.dev](https://boat.dev/) |
|快照和分叉 | [是：快照、恢复、分叉、暂停和恢复](/docs/develop/snapshots) |是：停止对磁盘进行快照，恢复并从中分叉 | [CLI 参考](https://docs.boat.dev/cli-reference.md) |
|从一张快照开始扇出 | [是的，`miosa batch new --count`](/docs/cli/platform) | `boat new --from <snapshot> --count` 最多 1000 | [CLI 参考](https://docs.boat.dev/cli-reference.md) |
|调整跑步机的大小 | [是的，运行时到位](/docs/develop/sandboxes/sizing) |先停止，然后以新尺寸继续 | [CLI 参考](https://docs.boat.dev/cli-reference.md) |
|端口的公共 URL | [是的，预览](/docs/develop/previews) |是的，`<sandbox>-<port>.on.boat.dev`，默认是令牌门控 | [主机 CLI](https://docs.boat.dev/hosting/host-cli.md) |
|部署到生产 URL | [是的，有发布和回滚](/docs/services/overview) |不;永远在线意味着您编写的 systemd 单元 | [永远在线的服务](https://docs.boat.dev/long-running-tasks/always-on-services.md) |
|自定义域 | [是](/docs/services/domains) |没有 | [主机 CLI](https://docs.boat.dev/hosting/host-cli.md) |
|托管数据库| [Postgres、MySQL、Redis、MongoDB、Qdrant](/docs/data/overview) |不在文档索引中 | [llms.txt](https://docs.boat.dev/llms.txt) |
| SDK | [TypeScript、Python、Go、Elixir、Java、Rust](/docs/sdks) | Python、TypeScript | [快速入门](https://docs.boat.dev/quickstart.md) |
|命令行 | [是](/docs/cli) |是的 | [CLI 参考](https://docs.boat.dev/cli-reference.md) |
| 计费 | [运行时每秒，每种尺寸一个价格](/docs/pricing) | 运行时每秒；每月 20 至 2000 美元的计划将作为计算回来 | [定价](https://docs.boat.dev/pricing) |
| 地区 | [us-east、us-west、us-mia](/docs/regions) | 仅限欧盟 | [定价](https://docs.boat.dev/pricing) |

## 船通向何处

从船的文档来看，这些是真正的差异：

- **安装让您登录。** 安装脚本一步打开登录和结账 ([quickstart](https://docs.boat.dev/quickstart.md))； MIOSA 安装 CLI，然后 `miosa login` 是第二步。
- **智能体自行注册是一个已记录的协议**，智能体采用分开的启动然后等待登录方式（[智能体身份验证](https://docs.boat.dev/agent-auth.md)）。
- **长命令。** `boat exec --timeout 0` 运行无限制，并且可以分离和重新附加。
- **当输出通过管道传输时打开 JSON 输出**（[CLI 参考](https://docs.boat.dev/cli-reference.md)）； MIOSA 需要 `--json`。
- **从快照扇出**，`--count` 最多 1000，以及创建时的 **每个沙箱变量** (`-e KEY=VALUE`)。
- **命令历史记录**带输出，每个沙箱（`boat history`）。

## MIOSA 的不同之处

- **部署是一个产品**，而不是一个配方：从存储库、目录或沙箱进行部署，并进行发布、升级和回滚。请参阅[服务](/docs/services/overview)。
- **自定义域**，具有 DNS 验证。请参阅[域](/docs/services/domains)。
- **托管数据库**，具有备份和连接字符串。请参阅[数据库](/docs/data/overview)。
- 正在运行的沙箱的**实时调整大小**，以及**检查点和恢复**。请参阅[调整大小](/docs/develop/sandboxes/sizing)和[快照](/docs/develop/snapshots)。
- **适合所有人的一个界面**：MCP、CLI 和六个 SDK。请参阅 [MCP](/docs/mcp)、[CLI](/docs/cli)、[SDK](/docs/sdks)。
- **美国地区**，并在 AWS 和 Google Cloud 上自带云。请参阅[区域](/docs/regions)和[BYOC](/docs/infrastructure/byoc/overview)。

## 何时选择每个- **如果您想要具有 SSH、无限长度命令和一步智能体注册功能的欧盟托管沙箱，并且不需要来自同一产品的部署、域或数据库，请选择 Boat**。
- **如果您希望在美国地区的一个平台上获得沙箱、完整计算机、部署、域和托管数据，请选择 MIOSA**。

> **注：船自己的比较**
>
> Boat 的比较页面将 MIOSA 列出一次，作为“具有 /dev/kvm 且无 24 小时超时的完整虚拟机”([boat.dev/compare](https://boat.dev/compare))。
> Boat 还构建了 [battleships.dev](https://battleships.dev/)，一个看起来像第三方的比较网站；请参阅[比较概述](/docs/compare#independent-comparisons)。

## 来源

- 船网站和文档：https://boat.dev 和 https://docs.boat.dev（于 2026 年 10 月 23 日阅读）。
- MIOSA 功能，如本网站所述。

- [MIOSA vs CodeSandbox](/docs/compare/codesandbox)：下一个比较。
- [所有比较](/docs/compare)：对每个提供商 MIOSA 进行衡量。
