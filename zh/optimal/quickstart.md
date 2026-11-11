---
slug: "optimal/quickstart"
title: "最佳快速入门"
description: "向 Optimal 提出问题，在沙盒上构建并预览一个小型应用程序，然后通过 Optimal API 发送您的第一条消息。"
section: "Optimal"
order: 7001
---

本演练大约需要五分钟。
您向 Optimal 提出一个问题，让它在沙箱上构建一个页面，然后从您自己的代码发送相同类型的消息。

## 开始之前

- 具有工作区的 MIOSA 帐户。
- 钱包积分或付费计划。
  在没有剩余积分的免费计划中，最佳大脑将被禁用，直到您添加积分。

## 1.问一个问题

1. 在控制台侧边栏中打开 **Optimal**。
2. 在工作区切换器中选择一个工作区。
3. 将模式保留为**询问**，然后选择启动项“此工作区需要我注意什么？”，或输入您自己的问题。
4. 按**发送消息**。

Optimal 会在聊天中读取您的实时工作区和答案。
询问模式永远不会接触机器。

## 2. 构建一些东西

1. 打开 Composer 中的模式菜单并选择 **Build**。
2. 输入“构建登陆页面并发布”，然后发送。
3. Optimal 询问在哪里构建。
   在计算机菜单中，选择 **新沙箱**，或选择您已有的计算机。
4. 侧面板打开。
   观看步骤的**进度**，并在提供页面时打开**预览**。

更改工作空间的步骤需要等待您的批准。
按作曲家上方卡片上的“**批准**”，或“允许此聊天**”以停止要求在此聊天中执行此类步骤。

当页面看起来正确时，按预览卡上的 **发布** 将其放在公共 URL 上。
请参阅[在您的计算机上构建](/docs/optimal/machines)。

## 3. 通过代码发送消息

Optimal 在 `https://optimal.miosa.ai/api/v1` 有一个 HTTP API。
它需要一个短暂的 Optimal 令牌，由您的 API 密钥铸造。

使用 **与 Optimal 聊天并开始运行** 权限 (`osa:write`) 创建 API 密钥，然后：

```bash
export MIOSA_API_KEY=msk_your_key
export WORKSPACE=your_workspace_id

# 1. Mint a 10-minute Optimal token for one workspace.
TOKEN=$(curl -s https://api.miosa.ai/api/v1/optimal/workspace-token \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"workspace_id\": \"$WORKSPACE\"}" | jq -r .token)

# 2. Start a chat.
CHAT=$(curl -s https://optimal.miosa.ai/api/v1/sessions \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"resume": false}' | jq -r .id)

# 3. Open the stream in the background, so you see the turn from its start.
curl -sN "https://optimal.miosa.ai/api/v1/sessions/$CHAT/stream" \
  -H "Authorization: Bearer $TOKEN" &

# 4. Send a message. This call returns at once with 202; the reply arrives on the stream.
curl -s https://optimal.miosa.ai/api/v1/sessions/$CHAT/message \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"message": "Summarize what changed in the last day", "mode": "ask"}'
```

该流在写入回复时发送 `streaming_token` 事件，并发送带有完整回复的 `agent_response` 事件。
`GET /sessions/$CHAT/messages` 随时返回完成的记录。
该聊天还会显示在控制台的**历史记录**中。
请参阅每个端点的[最佳 API](/docs/optimal/api)。

> **注意：令牌在 10 分钟后过期**
>
> 当呼叫应答 `401` 时，铸造一个新的最佳令牌。
> 聊天继续进行；只有令牌是新的。

## 下一步

- [聊天和模式](/docs/optimal/chats)：附件、排队消息和上下文。
- [Approvals](/docs/optimal/approvals)：选择 Optimal 之前询问的内容。
- [定价和使用](/docs/optimal/pricing)：聊天费用以及在哪里可以看到它。
