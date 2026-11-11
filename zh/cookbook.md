---
slug: "cookbook"
title: "食谱"
description: "每个 MIOSA 食谱都集中在一个地方，并按区域分组。沙盒程序和平台构建器演练，每个都有一个工作代码路径。"
section: "Sandboxes"
order: 2030
---

客户在 MIOSA 上构建的内容的实用演练。每个配方都以产品目标开始，以运行结果结束，并且每个配方都以当前的 SDK 和 API 为基础。

菜谱位于侧边栏中其所属产品旁边，位于 **沙箱 > 菜谱** 和 **平台构建器 > 菜谱** 下。此页面是所有这些内容的索引。

## 沙盒食谱

用于在沙箱内运行代码、智能体和浏览器。

- [智能体构建应用程序](/docs/cookbook/agent-builds-app)：完整的生命周期：用户提示文件、开发服务器、实时预览、反馈循环、发布和稳定的生产 URL。
- [具有秘密的智能体](/docs/cookbook/agent-with-secrets)：将 API 密钥和 OAuth 凭据注入沙箱中，而不将它们暴露在代码、日志或浏览器中。
- [流式代码执行](/docs/cookbook/streaming-code-execution)：实时运行代码并将标准输出流式传输到您的 UI：评分器、工作流构建器、数据作业和 AI 工具执行。
- [浏览器智能体](/docs/cookbook/browser-agent)：通过 CDP 上的 LLM 驱动沙箱浏览器：屏幕截图、决定、行动并实时观看同一会话。

## 平台构建秘诀

将 MIOSA 嵌入到您自己的产品中并向您的客户收费。

- [多租户 SaaS](/docs/cookbook/multi-tenant-saas)：使用您自己的工作区和用户 ID 标记每个沙箱，查询每个租户的使用情况，并向浏览器提供短期预览共享令牌。
- [每用户配额和计费](/docs/cookbook/per-user-quotas-billing)：使用 max_sandbox 和信用限额限制最终用户，提取由 external_user_id 键控的每月汇总，并将总计连接到 Stripe。
- [在 iframe 中嵌入沙箱](/docs/cookbook/embed-sandbox-iframe)：您的服务器会生成预览令牌；你的前端将其放入 iframe 中。 React、Vue、Svelte 和 vanilla 变体，带有 TTL 刷新。
- [实时沙箱事件](/docs/cookbook/sandbox-events-realtime)：通过 HMAC 签名的 webhooks 或 SSE 流，使用幂等模式对沙箱生命周期事件做出反应。
- [构建虚拟设备产品](/docs/cookbook/build-virtual-device-product)：用于向每个用户公开设备的产品蓝图：MIOSA 映射、架构、数据模型和作业。

## 开始之前

所有食谱都有相同的先决条件：

- MIOSA 工作区 API 密钥 (`msk_u_*`) - 请参阅 [API 密钥](/docs/platform/api-keys)
- 本地安装的 Node 22+ **或** Python 3.11+
- 为您的语言安装的 MIOSA SDK：

```bash
# Python
pip install miosa

# TypeScript / Node
npm install @miosa/sdk
```

在您的环境中设置一次密钥 - 每个食谱都从那里读取它：

```bash
export MIOSA_API_KEY="msk_u_..."
```
