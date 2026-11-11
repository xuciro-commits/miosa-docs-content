---
slug: "compare/cloudflare"
title: "MIOSA 与 Cloudflare 沙盒"
description: "使用截至 2026 年 10 月的独立 ComputeSDK 基准和 Cloudflare Sandboxes 公共文档对 MIOSA 和 Cloudflare Sandboxes 进行事实比较。"
section: "Optional"
order: 17021
---

比较

Cloudflare 的 Sandbox SDK 和 MIOSA 都为 AI 智能体运行不受信任的代码。本页在独立的 ComputeSDK 基准测试和 MIOSA 功能文档上对它们进行了比较。

## Cloudflare 沙箱是什么

Cloudflare 的 Sandbox SDK 在 Cloudflare 的边缘运行沙盒代码执行，其公共文档中对此进行了描述（[developers.cloudflare.com/sandbox](https://developers.cloudflare.com/sandbox/)，截至 2026 年 10 月）。

## 测量性能

独立的 ComputeSDK 基准测试针对每个提供商运行相同的工作负载。这是这里唯一可以比较的数字。

|公制|MIOSA | Cloudflare 沙箱 |截至 |
|---|---|---|---|
|突发启动、中位交互时间 | **57.09 毫秒（第 1 条，共 32 条）** | 682.13 毫秒 (#12) | 2026年10月23日 |
| DAX 完整构建，2026-08-28 | **39.3 秒（第 1 条，共 19 条）** |没有结果 | 2026-08-28 |
| DAX 完整版本，2026 年 10 月 23 日 | **38.8 秒（第 3 条，共 22 条）** | 155.9 秒 (#21) | 2026年10月23日 |

来源：ComputeSDK，通过[我们的基准页面](/docs/benchmarks)。突发启动是指从创建请求到新沙箱内第一个成功命令（少于 100 个并发创建）之间的时间。如果未列出某个运行，则表示该提供程序未出现在该运行中。

> **注意：公平阅读**
>
> Cloudflare 的优势在于边缘网络及其与 Workers 的集成。如果您的架构已经是边缘优先，那么这种集成可以超过启动时间差距。

## 并排的功能

每个 Cloudflare Sandboxes 答案都来自 Cloudflare Sandboxes 自己的文档或定价页面，链接在最后一列中，于 2026 年 10 月 23 日获取。
每个 MIOSA 答案都链接了记录它的页面。
“未记录”意味着供应商的文档没有说明，而不是缺少该功能。

|能力|MIOSA | Cloudflare 沙箱 |来源 |
|---|---|---|---|
|隔离| [默认情况下 Firecracker microVM](/docs/develop/sandboxes) | Firecracker microVM 中的容器 | [Cloudflare 沙盒文档](https://developers.cloudflare.com/sandbox/concepts/) |
|沙箱寿命 | 【空闲超时，默认1小时，最长24小时，可延长；计算机持续存在](/docs/develop/sandboxes/timeouts) |不活动后停止；不活动超时长达 6 小时 | [Cloudflare 沙盒文档](https://developers.cloudflare.com/sandbox/concepts/lifetime/) |
|暂停、快照、分叉 | [是：暂停和恢复、快照、恢复和分叉](/docs/develop/snapshots) |文件系统快照（测试版）；无需暂停和继续| [Cloudflare 沙盒文档](https://developers.cloudflare.com/sandbox/concepts/lifetime/) |
|完整的Linux桌面和计算机使用| [是的，计算机](/docs/computers/overview) |没有记录| [Cloudflare 沙盒文档](https://developers.cloudflare.com/sandbox/llms.txt) |
|端口的公共预览 URL | [是](/docs/develop/previews) |是的，通过您自己的 Worker | [Cloudflare 沙盒文档](https://developers.cloudflare.com/sandbox/previews/) |
|部署到生产 URL | [是的，服务和 Docker 主机，具有发布和回滚](/docs/services/overview) |是的，与 wrangler 一起部署为 Worker | [Cloudflare 沙箱文档](https://developers.cloudflare.com/sandbox/sdk/guides/deploy/) |
|托管数据库| [Postgres、MySQL、Redis、MongoDB、Qdrant](/docs/data/overview) |没有沙箱记录| [Cloudflare 沙盒文档](https://developers.cloudflare.com/sandbox/llms.txt) |
| SDK | [TypeScript、Python、Go、Elixir、Java、Rust](/docs/sdks) |打字稿 | [Cloudflare 沙箱文档](https://developers.cloudflare.com/sandbox/) |
|命令行 | [是](/docs/cli) |牧马人；没有特定于沙箱的 CLI | [Cloudflare 沙盒文档](https://developers.cloudflare.com/sandbox/sdk/guides/deploy/) |
| MCP服务器| [是，托管](/docs/mcp) |没有记录| [Cloudflare 沙盒文档](https://developers.cloudflare.com/sandbox/llms.txt) |
|自带云 | [是的，AWS 和 Google Cloud](/docs/infrastructure/byoc/overview) |没有记录；在您的 Cloudflare 帐户中运行 | [Cloudflare 沙盒文档](https://developers.cloudflare.com/sandbox/) |
|计费| [运行时每秒，每种尺寸一个价格](/docs/pricing) |容器定价，以 10 毫秒为步长 | [Cloudflare 沙盒文档](https://developers.cloudflare.com/containers/platform/pricing/) |
|地区 | [美国东部、美国西部、美国米亚](/docs/regions) |没有记录| [Cloudflare 沙盒文档](https://developers.cloudflare.com/containers/platform/limits/) |

## MIOSA 的不同之处

本网站上记录了 MIOSA 的这些功能：- **具有完整 Linux 桌面的持久计算机**，而不仅仅是无头沙箱。请参阅[计算机](/docs/computers/overview)。
- **部署和 Docker 主机**，因此机器成为版本化 URL 或容器化应用程序。请参阅[部署](/docs/docker-host/overview)。
- **白标平台**，让您的客户看到您的品牌和域名。请参阅[白标签设置](/docs/platform/white-label)。
- **适合所有人的一个界面**：MCP、CLI 和六个 SDK。请参阅 [MCP](/docs/mcp)、[CLI](/docs/cli)、[SDK](/docs/sdks)。
- **具有连接注入的托管数据**：Postgres、MySQL、Redis、MongoDB、Qdrant。请参阅[数据](/docs/data/overview)。
- **区域和 OpenComputers** 适用于您拥有的机器。请参阅[区域](/docs/regions)和[OpenComputers](/docs/opencomputers)。

## 何时选择每个

- **如果您基于 Cloudflare Workers 构建并希望代码执行接近边缘计算和数据，请选择 Cloudflare Sandboxes**。
- **选择 MIOSA**：如果您想要一个运行时，在一处覆盖沙箱、完整计算机、智能体运行、部署和托管数据，并带有品牌化的租户界面以及上文实测的启动数据。

## 来源

- Cloudflare Sandboxes 公共文档：https://developers.cloudflare.com/sandbox/（截至 2026 年 10 月）。
- ComputeSDK 基准测试结果，来自[我们的基准测试页面](/docs/benchmarks)（2026 年 10 月 23 日）。
- MIOSA 功能，如本网站所述。

- [MIOSA vs Freestyle](/docs/compare/freestyle)：下一个比较。
- [所有比较](/docs/compare)：对每个提供商 MIOSA 进行衡量。
