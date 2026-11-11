---
slug: "build/sora"
title: "构建文本转视频应用程序"
description: "Sora：通过视频模型 API 提示视频。使用确切的资源、步骤和命令在 MIOSA 上构建它。"
section: "Build"
order: 10052
---

`~25 min` `TypeScript` `Python`

> **您正在构建的内容**：通过视频模型 API 提示视频。
>
> **您将使用的原语**：沙箱、智能体和利用、部署（Docker 主机）

## 你正在构建什么

一款文本转视频应用程序，可调用视频模型 API 并为用户提供 OpenAI 在 Sora 上建模的创作源。

文本提示会发送到您带来的视频模型 API；该应用程序显示进度、存储剪辑并提供创作源以进行重新混合。

您将使用的基元：沙箱·智能体和利用·部署（App Engine）

- 通过模型 API 将文本转为视频
- 可供重新混合的创作源
- 渲染时的作业状态
- 短格式输出

来源：[openai.com/sora](https://openai.com/sora)。 Sora 是 OpenAI 的商标；本指南不隶属于他们，也不受他们认可。

## MIOSA 需要什么

每一行都是在开始之前创建的一件事。该数字与使用它的步骤相匹配。

1. 组织和 API 密钥 - 确定每次调用的范围；工作人员只需要一把工作区钥匙即可。 - `miosa api-key create app-key --preset agent` [文档](/docs/platform/api-keys)
2. 沙箱 - 智能体在其中编写代码和运行命令的隔离 Linux 工作区。 - `miosa create app-box --template nextjs --wait` [Docs](/docs/develop/sandboxes)
3. 每个客户一个工作区 - 隔离每个客户的机器、部署和数据。 - `miosa workspace create customer-1` [文档](/docs/platform/white-label)
4. 品牌和白标 - 您的姓名和预览、部署和桌面上的 slug；顾客永远不会看到 MIOSA。 - `miosa org` [文档](/docs/platform/white-label)
5. 使用计量和计费 - 每个客户的使用情况，以及哪个帐户为新机器付费。 - `miosa org bill` [文档](/docs/platform/usage-and-billing)
6. 智能体和工具 - 将提示转化为工作：选择运行使用的工具和模型。 - `miosa agent harnesses` [文档](/docs/api-reference/agent-runs)
7. 部署 (App Engine) - 将应用程序发布到不可变的版本化 URL，并进行回滚。 - `miosa deploy create --from-sandbox app-box --name app --wait` [文档](/docs/docker-host/overview)

## 架构

```mermaid
sequenceDiagram
  participant U as User
  participant P as Your app
  participant Q as Job queue
  participant G as Generation API
  participant S as Render sandbox

  U->>P: submit a prompt or a script
  P->>Q: create a job
  Q->>G: call the model provider you bring
  G-->>Q: clip or voice track
  Q->>S: compose and encode with ffmpeg
  S-->>P: finished media file
  P-->>U: progress, then the result
  U->>P: publish or download
```

## 它如何映射到 MIOSA

提示资产 -> 智能体运行 (`miosa prompt`, client.runs) 流事件，调用您的生成提供程序并将结果写入沙箱

繁重的渲染和编码 -> MIOSA 沙箱作业（ffmpeg 或无头渲染），大小为 `--size`

项目和资产 -> 用于作业元数据的托管 Postgres (`DATABASE_URL`)；媒体文件和卷

导出前预览 -> 沙箱预览 URL (`miosa preview create`)

## 数据和存储

作业（状态、输入、输出路径）是元数据；媒体本身很大，因此将其作为文件保存在渲染沙箱和卷中，而不是保存在 Postgres 中。

资产通过哈希进行内容寻址，因此永远不会重新生成相同的输入。

## 授权和租赁

您的组织就是平台；客户永远不会获得 MIOSA 帐户，他们会登录您的产品。每个客户在入职时创建一个工作区，隔离他们的机器、运行和数据。

```bash
miosa workspace create sora-customer-1
```

标记每台机器并使用 `metadata` 中的客户 ID 运行，并且切勿跨客户查询。使用 GET /api/v1/usage 计量每个客户的使用情况，并使用 PUT /api/v1/bill-to (/docs/platform/usage-and-billing) 设置付款人。

## 智能体循环

运行框架：OSA 是 MIOSA 自己的运行框架，可与您连接的任何模型提供商配合使用，包括您自己的模型。

模型：人类（克劳德）。呼叫使用我自己的提供商密钥（`miosa connections add models`）； MIOSA 平台密钥从未被使用过。

会话：每个项目或对话一次聊天，因此智能体可以保留其上下文。沙箱上的第一个 `miosa prompt` 使用 `--new-chat` （打印聊天 ID）；之后的每个回合都会经过 `--chat <chat-id>`。 `--reuse chat` 每个聊天保留一台新计算机，以便文件保留。

流式传输：使用 `miosa run follow <run-id>` 或 client.runs.streamEvents(run.id) 跟踪运行，并使用 `miosa run steer` 和 `miosa run interrupt` 引导或停止运行。

生成模型是你带来的一个提供者；该工具在沙箱中编排工作并运行 ffmpeg 以实现所有确定性。将生成和渲染作为单独的、可重试的步骤进行。

## 步骤1. 创建渲染工作器：一个更大的沙箱，用于编码和合成。
   ```bash
   miosa create sora-box --size large --wait
   ```
   检查：沙箱正在运行。
2、通过提示或者图片调用你自带的生成模型（你提供商的API）；存储返回的剪辑。
   检查：提供商返回剪辑或脚本，您存储其作业 ID。
3. 在沙箱中进行后处理：修剪、缝合、添加字幕和音频，并使用 ffmpeg 进行编码（将其安装在模板中一次，而不是每个作业）。
   ```bash
   miosa prompt --sandbox sora-box --harness osa --model <anthropic-model-id> --chat <chat-id> "Assemble the final video from /workspace/jobs/<id> with ffmpeg and write it to /workspace/out"
   ```
   检查：输出播放，具有正确的持续时间，并且作业行指向它。
4. 显示进度：将运行事件流式传输到 UI，以便长时间渲染不会看起来挂起。
   ```bash
   miosa run follow <run-id>
   ```
   检查：UI 显示作业运行时的进度以及完成后的最终文件。
5. 预览和导出：提供输出以进行播放，并提供下载。
   ```bash
   miosa preview create sora-box 3000 --name web
   ```
   勾选：预览播放剪辑；下载的大小和格式正确。
6. 发布接受作业并播放结果的应用程序。
   ```bash
   miosa deploy create --from-sandbox sora-box --name sora --dir /workspace --port 3000 --run-command "npm start" --wait
   ```
   检查：public_url 接受作业并播放完成的结果。

## 限制和成本

渲染是成本：根据作业调整沙箱大小，将 `timeout_sec` 设置为最长渲染，并在队列为空时暂停或销毁工作线程。

提供商的使用（发电、语音）由这些提供商计费；存储您自己的工作级别成本。

原型：将其保留在默认大小的一台计算机上，跳过不需要的副本和自定义主机名，并在完成后删除所有内容。

## 验收检查

作业从输入运行到可播放的输出文件。

失败的提供者调用使作业可以重试，而不是卡住。

作业运行时进度可见。

每个客户都被隔离在自己的工作空间中，并且根据他们的使用情况进行计量。

它创建的所有内容都可以被删除，而不会留下任何运行的内容。

## 后续步骤

- [智能体提示构建器选择产品形状并将简介交给您的智能体。](/docs/build)
- [运行编码智能体 在云中运行 Claude Code、Codex 或 Cursor，或在其上构建产品。](/docs/build/run-agents)
- [比较 MIOSA MIOSA 与其他沙箱和智能体平台的比较。](/docs/compare)
- [您可以构建什么 MIOSA 支持的每个产品模式。](/docs/use-cases)
