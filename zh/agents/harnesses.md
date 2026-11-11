---
slug: "agents/harnesses"
title: "运行框架和授权"
description: "MIOSA 智能体可以在您的计算机上运行，​​每个智能体如何进行身份验证，以及 MIOSA 如何让每个智能体做好准备。"
section: "Agents"
order: 6008
---

运行框架是在机器上运行的编码智能体 MIOSA，以您自己的帐户运行。
MIOSA 永远不会退回到自己的钥匙。

## 运行框架

|运行框架| `runner` |
|---|---|
|克劳德·代码 | `claude-code` |
|法典| `codex` |
| OSA | `osa` |
|圆周率 | `pi` |
|开放代码 | `opencode` |
|首席智能体 | `prime` |
|基米代码| `kimi` |
|米斯特拉尔氛围 | `mistral` |
|爱马仕智能体| `hermes` |

还有`custom`用于网关模型，它运行在MIOSA的AI网关上。

## MIOSA 如何让他们做好准备

运行速度很快，因为机器很少在运行时安装任何东西。

|准备好 |运行框架|
|---|---|
|机器镜像中预烘焙 |克劳德代码，法典，OSA |
|首次使用时安装 | pi、OpenCode、Prime Agent、Kimi Code、Mistral Vibe |
|附加组件，在创建时安装 |爱马仕智能体|

Hermes Agent 是一个大型安装，大约 2.5 GB，因此它是在创建时添加的，并在每个主机上缓存。

## 哪个账户驱动哪个工具

您创建的连接一次后会被接受它的每个运行框架重复使用。
连接一次 ChatGPT 计划，Codex、pi、OpenCode、Prime Agent 和 Hermes Agent 都使用它。

|运行框架|计划| API 密钥 |
|---|---|---|
|克劳德·代码 |克劳德计划|人类、OpenRouter、DeepSeek；亚马逊基岩 |
|法典| ChatGPT 计划 | OpenAI、DeepSeek；亚马逊基岩 |
| OSA | |奥拉马云； Anthropic、OpenAI、OpenRouter |
|圆周率 |克劳德计划、ChatGPT 计划、Mistral Vibe 密钥 | Anthropic、OpenAI、OpenRouter、Mistral、LLM 网关 |
|开放代码 | ChatGPT 计划，Mistral Vibe 密钥 | Anthropic、OpenAI、OpenRouter、Mistral、LLM 网关 |
|首席智能体 |克劳德计划、ChatGPT 计划、Mistral Vibe 密钥 | Anthropic、OpenAI、OpenRouter、Mistral、LLM 网关 |
|基米代码| Kimi代码计划|登月计划 |
|米斯特拉尔氛围 |米斯特拉尔振动键 |米斯特拉尔|
|爱马仕智能体| ChatGPT 计划、Claude 计划（Max，额外使用）| Anthropic、OpenAI、OpenRouter |

需要了解的两个注意事项：

- **OpenCode 不能使用 Claude 计划。** Anthropic 不允许这样做，OpenCode 删除了这样做的插件。
- **Claude 计划的 Hermes Agent 需要具有额外使用积分的 Claude Max 计划。** Claude Pro 计划被拒绝。

## 爱马仕智能体

Hermes Agent 是运行 API 中的一流运行器：发送 `runner: "hermes"` 到 `POST /api/v1/runs`。
它尚未出现在 CLI `--harness` 列表中，因此请使用 API 或控制台，直到它出现为止。

MIOSA 运行该平台固定的 Hermes 版本，并且从不安装更新的版本。
没有帐户运行的 Hermes 会返回 `OWN_CREDENTIALS_REQUIRED`，与任何其他 Harness 相同。

## 作为 API 的运行框架列表

`GET /api/v1/agents/harnesses` 返回每个运行框架：

- `supported_auth`：可以为其提供支持的帐户，按照运行路径尝试它们的顺序。
  每个都有 `id`、`label`、`kind`、`vendor`、`injection`、`model`、`source` 和 `note`。
- `also_supports`：智能体自己的 CLI 采用 MIOSA 尚未连接的内容。
- `unsupported_auth`：人们可能期望工作但供应商拒绝的功能，使用 `reason` 和 `source`。

每个模型都带有 `served_by`，即可以运行它的帐户 ID。

刷新令牌永远不会到达机器。
控制平面是单一刷新所有者，并为机器提供一个在运行结束后仍存在的短期访问令牌。

## 相关

- [快速入门](/docs/quickstart)：在沙箱中运行智能体。
- [智能体、聊天和运行](/docs/cli/agents)：CLI 界面。
- [与机器上的智能体聊天](/docs/agents/chat-on-a-machine)：从聊天中选择一个运行框架。
