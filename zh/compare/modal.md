---
slug: "compare/modal"
title: "MIOSA 与 Modal"
description: "使用截至 2026 年 10 月的独立 ComputeSDK 基准和 Modal 公共文档对 MIOSA 和 Modal 进行事实比较。"
section: "Optional"
order: 17019
---

比较

Modal 和 MIOSA 都为 AI 产品运行计算。本页面在独立的 ComputeSDK 沙​​箱基准测试和 MIOSA 功能文档上对它们进行了比较。

## 什么是模态

Modal 是一个无服务器云平台，用于运行 AI 工作负载的计算，包括沙盒容器，其公共文档（[modal.com/docs](https://modal.com/docs)，截至 2026 年 10 月）对此进行了描述。

## 测量性能

独立的 ComputeSDK 基准测试针对每个提供商运行相同的工作负载。这是这里唯一可以比较的数字。

|公制|MIOSA |Modal|截至 |
|---|---|---|---|
|突发启动、中位交互时间 | **57.09 毫秒（第 1 条，共 32 条）** | 1070.58 毫秒 (#17) | 2026年10月23日 |
| DAX 完整构建，2026-08-28 | **39.3 秒（第 1 条，共 19 条）** | 97.7 秒 | 2026-08-28 |
| DAX 完整版本，2026 年 10 月 23 日 | **38.8 秒（第 3 条，共 22 条）** | 82.3 秒 (#16) | 2026年10月23日 |

来源：ComputeSDK，通过[我们的基准页面](/docs/benchmarks)。突发启动是指从创建请求到新沙箱内第一个成功命令（少于 100 个并发创建）之间的时间。如果未列出某个运行，则表示该提供程序未出现在该运行中。

> **注意：公平阅读**
>
> Modal 比沙箱平台更广泛：团队使用它的大部分用途（GPU 服务、计划作业）都不在此比较范围内。此页面仅比较基准测试测量的沙箱工作负载。

## 并排的功能

每个 Modal 答案都来自 Modal 自己的文档或定价页面，链接在最后一列中，于 2026 年 10 月 23 日获取。
每个 MIOSA 答案都链接了记录它的页面。
“未记录”意味着供应商的文档没有说明，而不是缺少该功能。

|能力|MIOSA |Modal|来源 |
|---|---|---|---|
|隔离| [默认情况下 Firecracker microVM](/docs/develop/sandboxes) | gVisor，或虚拟机运行时 | [模态文档](https://modal.com/docs/guide/sandboxes.md) |
|沙箱寿命 | 【空闲超时，默认1小时，最长24小时，可延长；计算机持续存在](/docs/develop/sandboxes/timeouts) |默认5分钟，最长24小时 | [模态文档](https://modal.com/docs/guide/sandboxes.md) |
|暂停、快照、分叉 | [是：暂停和恢复、快照、恢复和分叉](/docs/develop/snapshots) |快照；无需暂停和继续| [模态文档](https://modal.com/docs/guide/sandbox-snapshots) |
|完整的Linux桌面和计算机使用| [是的，计算机](/docs/computers/overview) |举个例子（VM 沙箱中的 VNC），不是产品 | [模态文档](https://modal.com/docs/examples/computer_use_vnc.md) |
|端口的公共预览 URL | [是](/docs/develop/previews) |是的，隧道| [模态文档](https://modal.com/docs/guide/sandbox-networking) |
|部署到生产 URL | [是的，服务和 Docker 主机，具有发布和回滚](/docs/services/overview) |不适用于沙箱；应用程序通过模态部署进行部署 | [模态文档](https://modal.com/docs/guide/sandboxes.md) |
|托管数据库| [Postgres、MySQL、Redis、MongoDB、Qdrant](/docs/data/overview) |没有记录| [模态文档](https://modal.com/llms.txt) |
| SDK | [TypeScript、Python、Go、Elixir、Java、Rust](/docs/sdks) | Python、JavaScript、Go | [模态文档](https://modal.com/docs/guide/sandboxes.md) |
|命令行 | [是](/docs/cli) |是的 | [模态文档](https://modal.com/llms.txt) |
| MCP服务器| [是，托管](/docs/mcp) |没有记录| [模态文档](https://modal.com/llms.txt) |
|自带云 | [是的，AWS 和 Google Cloud](/docs/infrastructure/byoc/overview) |没有记录| [模态文档](https://modal.com/pricing) |
|计费| [运行时每秒，每种尺寸一个价格](/docs/pricing) |每秒、每核心和 GiB | [模态文档](https://modal.com/pricing) |
|地区 | [美国东部、美国西部、美国米亚](/docs/regions) |很多，价格乘数；区域路由不适用于沙箱 | [模态文档](https://modal.com/docs/guide/region-selection) |

## MIOSA 的不同之处

本网站上记录了 MIOSA 的这些功能：

- **具有完整 Linux 桌面的持久计算机**，而不仅仅是无头沙箱。请参阅[计算机](/docs/computers/overview)。
- **部署和 Docker 主机**，因此机器成为版本化 URL 或容器化应用程序。请参阅[部署](/docs/docker-host/overview)。
- **白标平台**，让您的客户看到您的品牌和域名。请参阅[白标签设置](/docs/platform/white-label)。
- **适合所有人的一个界面**：MCP、CLI 和六个 SDK。请参阅 [MCP](/docs/mcp)、[CLI](/docs/cli)、[SDK](/docs/sdks)。
- **具有连接注入的托管数据**：Postgres、MySQL、Redis、MongoDB、Qdrant。请参阅[数据](/docs/data/overview)。
- **区域和 OpenComputers** 适用于您拥有的机器。请参阅[区域](/docs/regions)和[OpenComputers](/docs/opencomputers)。

## 何时选择每个- **选择 Modal**：如果您需要在沙箱之外获得无服务器 GPU 计算、模型服务或通用批处理基础设施。
- **如果您的产品是智能体和应用程序型的：沙箱、完整计算机、智能体运行、部署、托管数据和白标签表面，以及上面测量的启动数据，请选择 MIOSA**。

## 来源

- Modal 公开文档：https://modal.com/docs（截至 2026 年 10 月）。
- ComputeSDK 基准测试结果，来自[我们的基准测试页面](/docs/benchmarks)（2026 年 10 月 23 日）。
- MIOSA 功能，如本网站所述。

- [MIOSA 与 Vercel Sandbox](/docs/compare/vercel)：下一项比较。
- [所有比较](/docs/compare)：对每个提供商 MIOSA 进行衡量。
