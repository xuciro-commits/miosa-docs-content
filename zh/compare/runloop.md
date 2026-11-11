---
slug: "compare/runloop"
title: "MIOSA 与 Runloop"
description: "使用截至 2026 年 10 月的独立 ComputeSDK 基准测试和 Runloop 的公共文档对 MIOSA 和 Runloop 进行事实比较。"
section: "Optional"
order: 17023
---

比较

Runloop 和 MIOSA 都在托管机器中运行 AI 智能体的代码。本页在独立的 ComputeSDK 基准测试和 MIOSA 功能文档上对它们进行了比较。

## 什么是 Runloop

Runloop（“您的 AI 智能体加速器”）为 AI 编码智能体提供开发盒和沙箱，其公共文档（[runloop.ai](https://runloop.ai/)，截至 2026 年 10 月）对此进行了描述。

## 测量性能

独立的 ComputeSDK 基准测试针对每个提供商运行相同的工作负载。这是这里唯一可以比较的数字。

|公制|MIOSA |运行循环|截至 |
|---|---|---|---|
|突发启动、中位交互时间 | **57.09 毫秒（第 1 条，共 32 条）** |未列出 | 2026年10月23日 |
| DAX 完整构建，2026-08-28 | **39.3 秒（第 1 条，共 19 条）** | 114.4 秒 (#16) | 2026-08-28 |
| DAX 完整版本，2026 年 10 月 23 日 | **38.8 秒（第 3 条，共 22 条）** |未列出 | 2026年10月23日 |

来源：ComputeSDK，通过[我们的基准页面](/docs/benchmarks)。突发启动是指从创建请求到新沙箱内第一个成功命令（少于 100 个并发创建）之间的时间。如果未列出某个运行，则表示该提供程序未出现在该运行中。

> **注意：公平阅读**
>
> Runloop没有出现在最新的突发运行中，所以这里没有类似的启动编号。比较对您重要的工作负载和功能。

## 并排的功能

每个 Runloop 答案都来自 Runloop 自己的文档或定价页面，链接在最后一列中，于 2026 年 10 月 23 日获取。
每个 MIOSA 答案都链接了记录它的页面。
“未记录”意味着供应商的文档没有说明，而不是缺少该功能。

|能力|MIOSA |运行循环|来源 |
|---|---|---|---|
|隔离| [默认情况下 Firecracker microVM](/docs/develop/sandboxes) |虚拟机| [Runloop 文档](https://docs.runloop.ai/docs/devboxes/overview.md) |
|沙箱寿命 | 【空闲超时，默认1小时，最长24小时，可延长；计算机持续存在](/docs/develop/sandboxes/timeouts) |默认1小时，可配置| [Runloop 文档](https://docs.runloop.ai/docs/devboxes/start-stop.md) |
|暂停、快照、分叉 | [是：暂停和恢复、快照、恢复和分叉](/docs/develop/snapshots) |是：快照和挂起（仅限磁盘）| [Runloop 文档](https://docs.runloop.ai/docs/devboxes/snapshots.md) |
|完整的Linux桌面和计算机使用| [是的，计算机](/docs/computers/overview) |是的，计算机能力 | [Runloop 文档](https://docs.runloop.ai/llms-full.txt) |
|端口的公共预览 URL | [是](/docs/develop/previews) |是的，隧道| [Runloop 文档](https://docs.runloop.ai/docs/devboxes/tunnels.md) |
|部署到生产 URL | [是的，服务和 Docker 主机，具有发布和回滚](/docs/services/overview) |没有记录| [Runloop 文档](https://docs.runloop.ai/llms.txt) |
|托管数据库| [Postgres、MySQL、Redis、MongoDB、Qdrant](/docs/data/overview) |没有记录| [Runloop 文档](https://docs.runloop.ai/llms.txt) |
| SDK | [TypeScript、Python、Go、Elixir、Java、Rust](/docs/sdks) | Python、TypeScript | [Runloop 文档](https://docs.runloop.ai/docs/tools/sdks.md) |
|命令行 | [是](/docs/cli) |是的 | [Runloop 文档](https://docs.runloop.ai/docs/tools/rl-cli.md) |
| MCP服务器| [是，托管](/docs/mcp) |是的，在 CLI 中 | [Runloop 文档](https://docs.runloop.ai/llms-full.txt) |
|自带云 | [是的，AWS 和 Google Cloud](/docs/infrastructure/byoc/overview) |企业VPC部署| [Runloop 文档](https://runloop.ai/pricing) |
|计费| [运行时每秒，每种尺寸一个价格](/docs/pricing) |每秒、每 CPU 和 GB | [Runloop 文档](https://runloop.ai/pricing) |
|地区 | [美国东部、美国西部、美国米亚](/docs/regions) |没有记录| [Runloop 文档](https://runloop.ai/pricing) |

## MIOSA 的不同之处

本网站上记录了 MIOSA 的这些功能：

- **具有完整 Linux 桌面的持久计算机**，而不仅仅是无头沙箱。请参阅[计算机](/docs/computers/overview)。
- **部署和 Docker 主机**，因此机器成为版本化 URL 或容器化应用程序。请参阅[部署](/docs/docker-host/overview)。
- **白标平台**，让您的客户看到您的品牌和域名。请参阅[白标签设置](/docs/platform/white-label)。
- **适合所有人的一个界面**：MCP、CLI 和六个 SDK。请参阅 [MCP](/docs/mcp)、[CLI](/docs/cli)、[SDK](/docs/sdks)。
- **具有连接注入的托管数据**：Postgres、MySQL、Redis、MongoDB、Qdrant。请参阅[数据](/docs/data/overview)。
- **区域和 OpenComputers** 适用于您拥有的机器。请参阅[区域](/docs/regions)和[OpenComputers](/docs/opencomputers)。

## 何时选择每个- **如果您希望针对编码智能体和专注的智能体基础设施产品调整托管开发盒，请选择 Runloop**。
- **如果您需要沙箱、带有桌面的完整计算机、智能体运行、部署、托管数据和一个平台中的白标签表面（上面有测量数据），请选择 MIOSA**。

## 来源

- Runloop 公共文档：https://runloop.ai（截至 2026 年 10 月）。
- ComputeSDK 基准测试结果，来自[我们的基准测试页面](/docs/benchmarks)（2026 年 10 月 23 日）。
- MIOSA 功能，如本网站所述。

- [MIOSA vs Blaxel](/docs/compare/blaxel)：下一个比较。
- [所有比较](/docs/compare)：对每个提供商 MIOSA 进行衡量。
