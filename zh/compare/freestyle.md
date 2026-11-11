---
slug: "compare/freestyle"
title: "MIOSA VS 自由泳"
description: "使用截至 2026 年 10 月的独立 ComputeSDK 基准和 Freestyle 公共文档对 MIOSA 和 Freestyle 进行事实比较。"
section: "Optional"
order: 17022
---

比较

Freestyle 和 MIOSA 都为 AI 智能体提供了自己的 Linux 机器。本页在独立的 ComputeSDK 基准测试、MIOSA 文档的功能以及 Freestyle 所做但 MIOSA 不做的事情上对它们进行了比较。

## 什么是自由泳

Freestyle (freestyle.sh) 提供“用于 AI 智能体的完整 Linux 虚拟机”，针对智能体平台和应用程序构建者，其公共文档 ([freestyle.sh](https://freestyle.sh/)，截至 2026 年 10 月) 对此进行了描述。

## 测量性能

独立的 ComputeSDK 基准测试针对每个提供商运行相同的工作负载。这是这里唯一可以比较的数字。

|公制|MIOSA |自由泳 |截至 |
|---|---|---|---|
|突发启动、中位交互时间 | **57.09 毫秒（第 1 条，共 32 条）** | 3237.49 毫秒 (#28) | 2026年10月23日 |
| DAX 完整构建，2026-08-28 | **39.3 秒（第 1 条，共 19 条）** |未列出 | 2026-08-28 |
| DAX 完整版本，2026 年 10 月 23 日 | **38.8 秒（第 3 条，共 22 条）** |没有结果 | 2026年10月23日 |

来源：ComputeSDK，通过[我们的基准页面](/docs/benchmarks)。突发启动是指从创建请求到新沙箱内第一个成功命令（少于 100 个并发创建）之间的时间。如果未列出某个运行，则表示该提供程序未出现在该运行中。

> **注意：公平阅读**
>
> 负载下的启动时间是一项指标。 Freestyle 在 MIOSA 不提供的多项功能上处于领先地位（如下），并且其自己的“65 毫秒内就绪”数字是由 Freestyle 测量的，而不是由该基准测试的。比较是否适合您的产品，而不是单个数字。

## 并排的功能

每个 Freestyle 答案都来自 Freestyle 自己的文档或定价页面，链接在最后一列中，获取日期为 2026 年 10 月 23 日。
每个 MIOSA 答案都链接了记录它的页面。
“未记录”意味着供应商的文档没有说明，而不是缺少该功能。

|能力|MIOSA |自由泳 |来源 |
|---|---|---|---|
|隔离| [默认情况下 Firecracker microVM](/docs/develop/sandboxes) |拥有自己的内核的完整虚拟机 | [自由式文档](https://www.freestyle.sh/products/vms) |
|沙箱寿命 | 【空闲超时，默认1小时，最长24小时，可延长；计算机持续存在](/docs/develop/sandboxes/timeouts) |为每个虚拟机设置 TTL 和空闲策略 | [自由式文档](https://www.freestyle.sh/docs/vms/lifecycle) |
|暂停、快照、分叉 | [是：暂停和恢复、快照、恢复和分叉](/docs/develop/snapshots) |是的：使用内存、快照、实时分叉暂停 | [自由式文档](https://www.freestyle.sh/docs/vms/lifecycle) |
|完整的Linux桌面和计算机使用| [是的，计算机](/docs/computers/overview) |作为指南（GNOME 优于 noVNC）| [自由式文档](https://www.freestyle.sh/docs/guides/run-gnome-in-a-sandbox) |
|端口的公共预览 URL | [是](/docs/develop/previews) |是的，在 style.dev 或自定义域上 | [自由式文档](https://www.freestyle.sh/docs/vms/domains) |
|部署到生产 URL | [是的，服务和 Docker 主机，具有发布和回滚](/docs/services/overview) |具有域的持久虚拟机；没有单独的部署功能| [自由式文档](https://www.freestyle.sh/docs/vms/domains) |
|托管数据库| [Postgres、MySQL、Redis、MongoDB、Qdrant](/docs/data/overview) |没有记录| [自由式文档](https://www.freestyle.sh/docs/llms.txt) |
| SDK | [TypeScript、Python、Go、Elixir、Java、Rust](/docs/sdks) |打字稿 | [自由式文档](https://www.freestyle.sh/docs/quickstart) |
|命令行 | [是](/docs/cli) |是的 | [自由式文档](https://www.freestyle.sh/docs/cli) |
| MCP服务器| [是，托管](/docs/mcp) |没有记录| [自由式文档](https://www.freestyle.sh/docs/llms.txt) |
|自带云 | [是的，AWS 和 Google Cloud](/docs/infrastructure/byoc/overview) |没有记录| [自由式文档](https://www.freestyle.sh/docs/vms/pricing-and-limits) |
|计费| [运行时每秒，每种尺寸一个价格](/docs/pricing) |每秒、每个 vCPU 和 GiB | [自由式文档](https://www.freestyle.sh/docs/vms/pricing-and-limits) |
|地区 | [美国东部、美国西部、美国米亚](/docs/regions) |没有记录| [自由式文档](https://www.freestyle.sh/docs/vms/pricing-and-limits) |

## MIOSA 的不同之处

本网站上记录了 MIOSA 的这些功能：- **具有完整 Linux 桌面的持久计算机**，而不仅仅是无头沙箱。请参阅[计算机](/docs/computers/overview)。
- **部署和 Docker 主机**，因此机器成为版本化 URL 或容器化应用程序。请参阅[部署](/docs/docker-host/overview)。
- **白标平台**，让您的客户看到您的品牌和域名。请参阅[白标签设置](/docs/platform/white-label)。
- **适合所有人的一个界面**：MCP、CLI 和六个 SDK。请参阅 [MCP](/docs/mcp)、[CLI](/docs/cli)、[SDK](/docs/sdks)。
- **具有连接注入的托管数据**：Postgres、MySQL、Redis、MongoDB、Qdrant。请参阅[数据](/docs/data/overview)。
- **区域和 OpenComputers** 适用于您拥有的机器。请参阅[区域](/docs/regions)和[OpenComputers](/docs/opencomputers)。

## Freestyle 的领先地位

Freestyle 记录了 MIOSA 目前**未**记录的功能。这些是真正的差异：

- **嵌套虚拟化**：在机器内部运行 K3 或 VM。 MIOSA 在沙箱内运行 Docker（[Docker in sandboxes](/docs/develop/sandboxes/docker)），但没有记录嵌套虚拟化。
- **正在运行的虚拟机（包括其内存）的实时克隆。** MIOSA 对沙箱进行快照并从该状态分叉，但它不记录正在运行的计算机的实时内存克隆。
- **WireGuard 对专用网络的访问。** MIOSA 并未将 WireGuard 专用网络记录为公共功能。
- **带有 systemd 的完整 Debian 映像。** MIOSA 沙箱是独立的 Linux 工作区；它不会将基于 systemd 的映像记录为面向用户的功能。

## 何时选择每个

- **如果您特别需要嵌套虚拟化、实时内存克隆、WireGuard 专用网络或完整的 systemd Debian 映像，请选择 Freestyle**。
- **如果您希望在一个平台上实现快速沙箱启动、具有桌面的持久计算机、部署、托管数据以及白标签多租户界面，请选择 MIOSA**。如果您需要“Freestyle 领先之处”中的功能，Freestyle 更适合这些功能。

## 来源

- Freestyle 公共文档：https://freestyle.sh（截至 2026 年 10 月）。
- ComputeSDK 基准测试结果，来自[我们的基准测试页面](/docs/benchmarks)（2026 年 10 月 23 日）。
- MIOSA 功能，如本网站所述。

- Freestyle Launch HN 线程：https://news.ycombinator.com/item?id=47663147。
- Freestyle 定价：https://www.freestyle.sh/pricing（截至 2026 年 10 月）。

- [MIOSA vs Runloop](/docs/compare/runloop)：下一个比较。
- [所有比较](/docs/compare)：对每个提供商 MIOSA 进行衡量。
