---
slug: "cli/agents"
title: "智能体、聊天和运行"
description: "在沙箱或计算机上运行智能体，管理保存的智能体定义、聊天和持久运行，并查看运行流的情况。"
section: "CLI, SDKs & MCP"
order: 11007
---

## 带上你自己的模型

智能体运行使用您自己的模型提供者连接，而不是 MIOSA 平台密钥。使用 `miosa api-key create NAME --preset cli` 创建一个可以执行此操作的密钥。添加与 `miosa connections add models` 的连接（密钥是从提示符或标准输入中读取的，而不是从参数中读取的）。没有可用连接的运行会失败，并显示一条指定该命令的消息。

```bash
miosa api-key create agent-key --preset cli
miosa connections add models
```

## 提示

在沙箱或计算机上运行智能体并遵循它。

```bash
miosa prompt --sandbox my-box "fix the failing test"
miosa prompt --new-machine --harness codex "write a README"
miosa prompt --sandbox my-box --continue
```

|旗帜|意义|
|---|---|
| `-s, --sandbox string` |要运行的沙箱（默认：当前沙箱）|
| `--computer string` |计算机运行 |
| `--agent string` |运行已保存的智能体（名称或 ID）而不是临时智能体 |
| `--harness string` | `claude-code`、`codex` 或 `osa`（默认：智能体的或 `claude-code`）|
| `--model string` |运行框架模型（默认：运行框架默认）|
| `--new-machine` |为此运行配置一台机器 |
| `--machine string` |新机器类型：`sandbox` 或 `computer` |
| `--size string` |新机尺寸|
| `--template string` |新机器模板|
| `--chat string` |聊天ID；如果聊天已运行，则继续聊天 |
| `--new-chat` |开始聊天（打印新的聊天 ID）|
| `--reuse string` |新机器寿命：`run`（每次运行一次）或`chat`（每次`--chat`一次，文件保留）|
| `--after-run string` |运行结束时新机器会发生什么：`destroy`、`pause` 或 `keep` |
| `--max-time int` |这么多秒后停止运行 |
| `--no-follow` |打印运行 ID 并返回，无需等待 |
| `--notify-email` |跑步结束后给您发电子邮件 |
| `--notify-webhook string` |运行完成时通知的 Webhook id |

## 已保存的智能体

智能体是一个保存的、版本化的定义：运行框架、模型、指令、它运行的机器以及它可以做什么。 `<agent>` 是名称或 ID。

```bash
miosa agent new reviewer --harness claude-code --instructions-file review.md
miosa agent publish reviewer --file config.yaml
miosa agent list
miosa agent diff reviewer 2 3
miosa agent run reviewer "review PR 42" --sandbox my-box
miosa agent harnesses
```

`miosa agent new` 需要 `--harness`（`claude-code`、`codex`、`osa` 或 `custom`）、`--model`、`--instructions` 或 `--instructions-file`、`--overdrive`（未经批准运行运行框架提示）、`--timeout-sec`、`--description` 和完整的 `--file`（YAML 或 JSON）。

其他子命令：`agent get`、`agent versions`、`agent version`、`agent update`、`agent duplicate`、`agent rm`、`agent runs`、`agent templates`、`agent triggers`、`agent approvals`、 `agent authority`、`agent defaults`、`agent harness-versions`。

## 运行

列出、跟踪和停止智能体运行。

```bash
miosa run list --status running
miosa run get <id>
miosa run follow <id>
miosa run events <id>
miosa run logs <id>
miosa run messages <id>
miosa run outputs <id>
miosa run files <id>
miosa run diagnostics <id>
miosa run download <id> -O out.tar
miosa run steer --sandbox my-box "also add tests"   # queue the next turn for the running agent
miosa run interrupt --sandbox my-box                # stop the agent attached to a machine
miosa run stop <id>
miosa run usage --group-by model
```

`miosa run list` 使用 `--agent-id`、`--chat`、`--harness`、`--source`、`--status`、`--target`、`--workspace-id` 和 `--limit` 进行过滤。

## 聊天

聊天是有记忆的智能体对话。

```bash
miosa chat list
miosa chat start --sandbox my-box --harness claude-code
miosa chat show <chat-id>
miosa chat context <chat-id>     # how full the context is
miosa chat compact <chat-id>     # compact it so the chat can go on
```

## 连接

您的智能体使用的模型提供商、应用程序和工具服务器。

```bash
miosa connections add models
miosa connections list --kind models
miosa connections test <name>
miosa connections share <name>
miosa connections rm <name>
```

`miosa connections add` 采用 `--key-stdin`（绝不是命令行上的键）、`--label`、`--workspace-id`、`--url`（对于 MCP 工具服务器）和 `--header-env HEADER=ENV_VAR`（可重复）。

> **注意**
>
> 运行是规范的智能体执行合约。 `miosa prompt`、`miosa agent run` 和 `miosa chat` 都创建运行； `miosa run` 读取并控制它们。
