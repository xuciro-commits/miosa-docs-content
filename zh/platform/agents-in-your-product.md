---
slug: "platform/agents-in-your-product"
title: "产品中的智能体"
description: "为每位用户分配一台运行智能体或 harness 的机器，并在你自己的 UI 中展示结果。"
section: "Platform builders"
order: 9003
---

平台会为每位用户创建一台机器，然后在其中放入一个智能体或 harness。
智能体会写入文件、运行命令并开放端口。
你的产品负责展示结果：预览 URL、终端或截图。

## 每位用户一台机器

每个最终用户对应一个沙箱或一台计算机，绝不在客户之间共享。

每台机器都有自己的文件系统、网络和进程树。
代码执行和预览选用沙箱；智能体需要桌面或浏览器时选用计算机：[智能体设备选择](/docs/guides/agent-device-selection)。

代码智能体使用沙箱，浏览智能体使用计算机。
两者都有各自独立的文件系统和网络。

## 让用户自带模型账户

智能体运行时使用其所有者自己的账户进行思考。
你的客户可以直接在你的产品中连接自己的 Claude 套餐、ChatGPT 套餐或模型密钥，无需访问控制台。
此后在他们机器上运行的每个任务都会优先使用该账户，其次才使用工作区的账户。

```typescript
const signin = await miosa.connections.connectSubscription({ provider: "claude", externalUserId: "cus_123" })
// show signin.verificationUrl in your UI, then: await signin.submitCode(pastedCode)
```

请参阅[连接智能体账户](/docs/guides/connect-agent-accounts)。

## 向用户展示结果

浏览器只能看到预览 URL 以及限定范围、会过期的令牌。
为预览和终端签发短期[浏览器令牌](/docs/platform/browser-tokens)。

要在产品中展示预览，请在服务端签发分享令牌，只将 URL 发送给浏览器：

```typescript
// GET /embed-url on your backend
const share = await sandbox.previews.share(preview.id, { expiresInSec: 3600 })
return { embedUrl: share.share_url }
```

令牌绑定到那一个预览，并且会过期。

绝不要把 `msk_*` 密钥放进浏览器。
应像对待 Stripe 密钥密钥一样对待它：仅在服务端使用，定期轮换并接受审计。

始终展示返回的 `public_url`，不要自行拼接主机名。

## 下一步

- [智能体操作指南](/docs/guides/agent-operating-guide)：针对沙箱或计算机运行 AI 智能体。
- [智能体运行](/docs/api-reference/agent-runs)：派发任务、流式接收事件并收集产物。
- [连接智能体账户](/docs/guides/connect-agent-accounts)：为工作区或每位最终用户配置密钥和订阅套餐。
- [浏览器令牌](/docs/platform/browser-tokens)：用于最终用户浏览器中预览和终端的短期令牌。
- [智能体设备选择](/docs/guides/agent-device-selection)：为每个智能体选择沙箱或计算机。
