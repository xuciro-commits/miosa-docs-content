---
slug: "build"
title: "构建一个应用程序，例如..."
description: "平台构建者指南。在 MIOSA 上运行开源智能体和框架，或构建熟悉的 AI 产品，例如 Lovable、Manus 和 Perplexity - 应用程序构建器、编码智能体、自主和计算机使用智能体、聊天和同伴、应答引擎、多智能体、语音、设计、视频、垂直应用程序和内部业务操作系统。"
section: "Build"
order: 10000
---

构建

人们已经知道的产品形状被映射到 MIOSA 基元上。在同一架构上为您自己的公司构建其中任何一个或作为您的客户的产品。每个指南只有一页：它是什么、您需要的资源、架构图、您可以运行的六个步骤以及编码智能体的智能体提示。

[比较 MIOSA](/docs/compare)[平台构建指南](/docs/platform/overview)[沙箱](/docs/develop/sandboxes)[部署和 Docker 主机](/docs/docker-host/overview)

每个指南都是以下两种类型之一：

- **在 MIOSA 上运行 X。** 您可以实际安装和使用的开源智能体、框架和工具（OpenClaw、Hermes Agent、OpenHands、Claude Code、Codex、Cursor's CLI 智能体、CrewAI、AutoGen、LangGraph、Pipecat、Browser Use、Stagehand、Steel）。该指南及其智能体提示从其官方文档中安装真正的工具，在 MIOSA 沙箱或计算机上运行它，然后为您的团队或客户包装它。
- **构建像 X 这样的应用程序** 您无法自己运行的封闭产品（Lovable、Bolt、v0、Manus、Perplexity、Character.AI 等）。该指南为产品命名，将其每项功能映射到 MIOSA 原语，并构建您自己的版本。

任何一种都可以在同一架构上**以两种方式设置**：

- **对于您自己的公司（内部）。** 一个组织、每个团队一个工作区、访问控制和 SSO 以及一个内部域。不计费。
- **作为产品（针对您的客户）。** 多租户：白标品牌、每个客户的组织或工作区、自定义域和使用计量。

每个指南都会附带一个**针对您正在构建的内容量身定制的智能体提示** - 选择形状和名称，然后将摘要交给 OSA、Claude Code、Codex 或 Cursor。浏览下面的应用程序和平台。

MIOSA 是人工智能产品下面的执行层。核心思想贯穿于每一个指南：
**构建由 Claude Code、Codex、Hermes、OpenClaw、OSA 或您自己的模型支持的产品**。
该运行框架在 MIOSA 沙箱或计算机内运行，并且您的产品将其包裹起来。

有两条路径，并且它们不是唯一的：

- [为您的团队按原样运行智能体](/docs/build/run-agents)：后台编码智能体、每个沙箱的并行会话以及每个开发人员的持久工作区。无需构建产品：只需在云中运行 Claude Code、Codex、Cursor 或您自己的工具即可观看工作。
- [在顶部构建品牌产品](/docs/build/coding-agent-product)：在您自己的 UI、模型和计费中包装一个运行框架：Lovable 风格的构建器、内部工作区或面向客户的产品，使用 Claude Code、Hermes、OpenClaw 或您自己的模型。

## 构建想法

- [您自己的克劳德桌面风格应用程序](/docs/build/claude-cowork)：一种品牌桌面/网络应用程序，您的用户可以通过浏览器和文件与在其自己的云计算机中工作的智能体聊天。
- [带有链接的工件](/docs/build/lovable)：智能体构建某些内容（站点、文档、仪表板或应用程序），在预览 URL 上共享它，然后将其提升到您自己的品牌域上的部署。
- [内部 AI 工作区 ](/docs/build/business-os)：Acme Corp 风格的内部平台：在您自己的登录后面放置您团队的智能体、应用程序和数据的地方。
- [面向客户的产品](/docs/platform/white-label)：相同的形状、流明风格和白标：您的品牌、您的域名和您的最终用户，背后有 MIOSA 运行。
- [在云中运行编码智能体](/docs/build/run-agents)：将 Claude Code、Codex 或 Cursor 智能体放入沙箱中：后台工作、并行会话和持久工作区。
- [多智能体团队](/docs/build/openswarm)：粉丝在自己的机器上为子智能体制定并加入结果的编排器。

### 人工智能应用程序构建者

提示运行、可预览、可部署的应用程序。- [构建一个Lovable 风格的应用程序提示一个全栈应用程序，实时预览它，发布它。](/docs/build/lovable)
- [在真实的开发服务器上根据提示构建 Bolt 式应用程序全栈应用程序。](/docs/build/bolt)
- [构建 v0 风格的应用程序 根据提示生成 UI 并发布它。](/docs/build/v0)
- [构建 Replit-Agent 风格的应用程序 云工作区，其中提示成为正在运行的应用程序。](/docs/build/replit)
- [构建一个 Base44 风格的应用程序 描述一个应用程序，获取一个工作的、预览过的应用程序。](/docs/build/base44)
- [构建一个Emergent风格的应用程序提示成为具有真实基础设施的运行软件。](/docs/build/emergent)
- [构建 Rork 风格的应用程序 提示您可以预览的移动应用程序。](/docs/build/rork)
- [构建任何类型的应用程序 提示工作网络或移动应用程序。](/docs/build/anything)
- [构建相同风格的应用程序从屏幕截图或 URL 克隆并重建 UI。](/docs/build/same-new)

### 运行编码智能体

在 MIOSA 上安装 Claude Code、Codex、Cursor、Cline 和 OpenHands，或在其上构建产品。

- [在云端运行编码智能体 Claude Code、Codex、Cursor：云端后台智能体。](/docs/build/run-agents)
- [在 MIOSA 上运行 Claude Code 将其安装在沙箱中；流式传输并收集其作品。](/docs/build/claude-code)
- [在 MIOSA 上运行 Codex 在包含流事件的沙箱中安装 Codex CLI。](/docs/build/codex)
- [在 MIOSA 上运行 Cursor 智能体安装 Cursor CLI 智能体并在隔离的计算机上无头运行它。](/docs/build/cursor)
- [构建 Windsurf 风格的智能体 IDE 带有智能体的编辑器，可以跨文件编辑并运行命令。](/docs/build/windsurf)
- [构建 GitHub Copilot 风格的编码智能体 一个可以处理问题、工作并打开 PR 的智能体。](/docs/build/github-copilot)
- 【在MIOSA上运行OpenHands安装开源软件智能体；沙箱是它的工作空间。](/docs/build/openhands)
- [在 MIOSA 上运行 Cline 最常安装的开源编码智能体，在沙箱中无头。](/docs/build/cline)
- [构建编码智能体产品将运行框架包裹在您自己的品牌产品中。](/docs/build/coding-agent-product)

### 自主智能体

智能体拥有自己的机器来计划和完成工作。

- [构建一个Manus风格的智能体一个带有桌面、浏览器和文件的通用智能体。](/docs/build/manus)
- [构建 Genspark 风格的智能体 一个计划并生成完成工作的超级智能体。](/docs/build/genspark)
- [构建 Devin 风格的 SWE 智能体 一个拥有 shell、IDE 和浏览器的自主工程师。](/docs/build/devin)
- [构建 Cowork 风格的桌面智能体 一个在桌面上与您一起工作的智能体。](/docs/build/claude-cowork)

### 浏览器和计算机使用智能体

在 MIOSA 上构建您自己的 AI 浏览器、平台或工具，或运行开源浏览器智能体。

- [构建您自己的浏览器智能体您的模型和循环，驱动沙箱浏览器或完整的计算机。](/docs/build/chatgpt-agent)
- [构建一个Polar风格的AI浏览器 一个运行长时间任务和预定工作流程的AI浏览器。](/docs/build/polar)
- [在MIOSA上运行浏览器使用安装开源库并让智能体驱动真正的浏览器。](/docs/build/browser-use)
- [在 MIOSA 上运行 Stagehand 安装框架并在计算机上运行可靠的浏览器自动化。](/docs/build/stagehand)

### AI聊天和同伴

助手和永远在线的同伴。

- 【打造点式伴侣永远在线的伴侣，每个伴侣都有自己的云电脑。](/docs/build/dots)
- [构建 Grok Bot 风格的聊天应用程序 一组共享一台云计算机的命名机器人。](/docs/build/grok-bot)
- [打造 Muse 风格的伴侣 跨应用程序代表您行事的个人伴侣。](/docs/build/meta-muse)
- [建立一个角色风格的同伴角色，陪伴用户随着时间的推移进行聊天。](/docs/build/character-ai)
- [打造Replika式伴侣 具有长期记忆的个人伴侣。](/docs/build/replika)
- [打造 Pi 式伴侣 热情、健谈的私人助理。](/docs/build/pi)
- [在 MIOSA 上运行 OpenClaw 在永不休眠的持久计算机上安装开源助手。](/docs/build/openclaw)

### 答案引擎

研究可搜索、阅读和回答来源的产品。- [构建一个 Perplexity 风格的答案引擎 通过引用进行搜索、阅读和回答。](/docs/build/perplexity)
- [构建彗星风格的智能体浏览器 就地导航和应答的浏览器。](/docs/build/comet)
- [构建 You.com 风格的搜索和应答 API 您的智能体调用以搜索和应答的 API，并带有来源。](/docs/build/you-com)
- [构建 Phind 风格的答案引擎 将技术答案呈现为交互式迷你应用程序。](/docs/build/phind)
- [构建深度研究型智能体 长期运行的研究智能体生成报告。](/docs/build/deep-research)

### 多智能体

一个请求，多个智能体并行工作。

- [构建 OpenSwarm 风格的多智能体应用程序 围绕并行运行多个智能体而构建的 AI 桌面。](/docs/build/openswarm)
- [在 MIOSA 上运行 Hermes Agent 安装开源智能体工具并运行许多使用工具的智能体。](/docs/build/hermes-agent)
- [构建一个协调器，委托一个智能体进行计划、委托给子智能体并加入结果。](/docs/build/gemini-agent)

### 语音智能体

座席会根据电话说话并采取行动。

- [构建 ElevenLabs 风格的语音智能体 一个会说话并采取实际行动的语音智能体。](/docs/build/elevenlabs)
- [构建 Vapi 风格的语音智能体 用于电话和网络的语音智能体。](/docs/build/vapi)
- [构建 Retell 风格的语音智能体 处理真实呼叫的电话智能体。](/docs/build/retell)
- [大规模构建平淡风格的电话智能体可编程电话智能体。](/docs/build/bland)
- [在 MIOSA 上运行 Pipecat 安装从语音到文本到文本到语音的开源语音管道。](/docs/build/pipecat)
- [在 MIOSA 上运行 LiveKit Agents 领先的开源实时语音智能体框架。](/docs/build/livekit-agents)

### 人工智能设计工具

提示设计好的 UI 或资产。

- [构建一个提示到 UI 设计工具提示（和一个设计文件）到一个工作的、预览的 UI。](/docs/build/figma-make)
- [构建草图到模型工具草图或屏幕截图成为可编辑的 UI 代码。](/docs/build/uizard)
- [构建一个 Magic-Patterns 风格的工具，根据提示生成 UI 组件。](/docs/build/magic-patterns)

### AI视频与创意

生成、编辑和制作媒体。

- [构建视频生成应用程序 在视频模型 API 之上生成和编辑视频。](/docs/build/runway)
- [构建文本转视频应用程序通过视频模型 API 提示视频。](/docs/build/sora)
- [使用 TTS 和唇形同步 API 构建一个头像视频应用程序脚本，用于说话的头像视频。](/docs/build/heygen)
- [建立一个希格斯菲尔德风格的视频工作室一个多模型工作室：简介、操纵相机和观看、生成。](/docs/build/higgsfield)
- [构建 Gamma 风格的套牌应用程序提示或记录到精美的、可共享的套牌或页面。](/docs/build/gamma)
- [构建描述式编辑器通过编辑脚本来编辑视频。](/docs/build/descript)

### 垂直人工智能应用

特定行业工作流程的智能体。

- [构建 Fin 式支持智能体 解决客户票证的 AI 支持智能体。](/docs/build/intercom-fin)
- [构建 Decagon 风格的支持智能体，通过聊天和电子邮件解决问题，并进行交接。](/docs/build/decagon)
- [建立粘土式销售智能体大规模研究并丰富潜在客户。](/docs/build/clay)
- [构建通话后分析智能体将通话记录转化为来源见解和后续步骤。](/docs/build/gong)
- [构建 OpenEvidence 风格的证据引擎 在可信语料库上引用答案。](/docs/build/openevidence)

### 公司内部应用程序

为您自己的公司构建这些：一个组织，每个团队一个工作区。

- [构建内部 Business OS MIOSA 自己的 BusinessOS 模板，Apache 2.0。](/docs/build/business-os)
- [构建内部 AI 工作空间，为您的团队的智能体、应用程序和数据提供一个地方。](/docs/build/team-ai-workspace)
- [在人工批准的情况下构建操作并支持副驾驶分类票据并运行操作手册。](/docs/build/ops-support-copilot)
- [构建内部工具构建器提示您域后面的已发货仪表板。](/docs/build/internal-tools-builder)
- [构建知识库智能体 来自公司文档的答案并附有引文。](/docs/build/knowledge-base-agent)
- [建立员工入职智能体，引导新员工度过第一周。](/docs/build/employee-onboarding-agent)

商标属于其所有者；这些指南展示了如何在 MIOSA 上构建类似的产品。不暗示任何隶属关系或认可。> **注意：相同的模板，每个指南**
>
> 每个指南都有相同的形状：**提示构建器**、**您正在构建的内容**、
> **您在 MIOSA** 清单上需要什么，其中包含每个资源的创建命令、**架构
> 图表**，六个编号步骤（**创建帐户和 API 密钥 → 创建资源 → 连线
> 智能体循环 → 预览 → 部署 → 自定义域**)、**成本和限制**以及**后续步骤**。

## 从平台开始

- [平台构建者指南](/docs/platform/overview)：组织、工作区、白标、租户和计费。
- [智能体构建应用程序](/docs/cookbook/agent-builds-app)：每个应用程序构建器指南都基于端到端配方。
- [智能体编排](/docs/guides/agent-orchestration)：协调沙箱和计算机智能体、工件和部署。
- [比较 MIOSA](/docs/compare)：MIOSA 与其他沙箱和智能体平台的比较。
