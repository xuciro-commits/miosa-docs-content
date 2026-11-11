---
slug: "cli"
title: "CLI 概述"
description: "miosa 命令行工具。从终端安装、登录和驱动沙箱、计算机、智能体、部署和每个平台资源。"
section: "CLI, SDKs & MCP"
order: 11001
---

CLI 和 MCP

为您和您的智能体控制所有 MIOSA 的最快方式。控制台所做的一切都可以编写脚本，并带有 JSON 输出。

`npm i -g @miosa/cli`

macOS、Linux 和 Windows 上的 miosa CLI。容器或 CI 作业内最快的路径。

- [安装并验证](/docs/cli/install)：npm、Homebrew、脚本或发布存档，然后登录。
- [从智能体使用](/docs/cli#use-the-cli-from-agents)：让 OSA 或任何带有 shell 的智能体驱动平台。
- [命令参考](/docs/cli#command-groups)：每个命令组、标志和退出代码。
- [CLI 或 MCP](/docs/cli/cli-vs-mcp)：向智能体提供哪一个，以及为什么两者都有效。

`miosa` 涵盖整个平台：沙箱和计算机、文件和预览、智能体及其运行、部署、数据库、存储、API 密钥、Webhooks、计费和您所在的组织。

## 快速开始

```bash
npm i -g @miosa/cli               # or Homebrew, the install script, or a release archive
miosa --version                   # prints 2.3.1
miosa login                       # browser device-code sign in
miosa whoami                      # user, organization, plan and scopes
miosa create my-box --wait        # provision a sandbox
miosa exec my-box -- echo hello   # run a command
miosa destroy my-box --yes
```

- [安装和验证](/docs/cli/install)：npm、Homebrew、安装脚本、版本、登录和 API 密钥。
- [机器和生命周期](/docs/cli/machines)：创建、列出、使用、停止、分叉、恢复、销毁。
- [运行命令并移动文件](/docs/cli/work-inside)：exec、ssh、控制台、文件、服务、预览、端口。
- [设备](/docs/cli/devices)：您可以运行工作的任何运行时：沙箱、计算机等。
- [JSON 输出和脚本](/docs/cli/scripting)：--json、流输出、退出代码、CI 使用。

## 使用智能体的 CLI

CLI 不仅适用于人类。 [OSA](/docs/mcp#osa)，MIOSA 的开源智能体，运行 shell 命令，因此它可以从您的计算机或 MIOSA 计算机驱动 `miosa`。 `osa run "..."` 无头运行，无需 TUI。任何具有 shell 访问权限的智能体都可以执行相同的操作，并使用 `miosa` 驱动整个平台：它涵盖 API 和 SDK 公开的所有资源，而不仅仅是工具表面，并且它由智能体已有的 shell 和工具组成。

- **安装在沙箱或 CI 内。** `npm i -g @miosa/cli` 是容器或 CI 作业中最快的路径。沙盒映像附带 Node，因此一台新的智能体机器在一个命令中包含 CLI。
- **使用密钥进行身份验证。** 设置 `MIOSA_API_KEY`，每个命令都无需浏览器进行身份验证。切勿将密钥作为参数传递。
- **获取 JSON 和退出代码。** 添加 `--json` 以获取机器可读的输出并在退出代码上进行分支。 `exec` 和 `ssh` 退出并显示远程命令的状态。

```bash
export MIOSA_API_KEY="msk_u_..."
npm i -g @miosa/cli

# An agent provisioning, using and cleaning up a machine, all scriptable
ID=$(miosa create agent-box --size small --ttl 1h --wait --json | jq -r .id)
miosa exec "$ID" -- npm ci
miosa preview create "$ID" 3000 --visibility public --json | jq -r .url
miosa destroy "$ID" --yes
```

在具有 shell 访问权限的 Claude Code 或 Codex 中运行的智能体完全使用这些命令：配置沙箱、运行构建、发布预览、读取 JSON、清理。无需 MCP 设置。

### 当 CLI 击败 MCP 时

当智能体已有 shell 时，请使用 **CLI**。它到达整个平台而不是一个工具表面，其输出是可编写脚本的，并由 `jq`、管道和文件组成，并且 `miosa api <METHOD> <path>` 覆盖高级命令未覆盖的任何端点。当客户端没有 shell 时，或者当您希望模型选择工具而不是编写命令时，请使用 **MCP**。请参阅[CLI 或 MCP：使用哪个](/docs/cli/cli-vs-mcp)。

## 命令组

每个顶级命令都按照 CLI 的组织方式进行分组。

| 分组 | 命令 |
|---|---|
| 账户 | `login`、`logout`、`whoami`、`org`、`limits`、`profile`、`config`、`doctor`、`open`、`update`、`version` |
| 生命周期 | `create`、`list`、`info`、`use`、`destroy`、`stop`、`pause`、`resume`、`fork`、`recover`、`extend`、`tag`、`wait`、`usage`、`computer`、`device` |
| 内部操作 | `exec`、`ssh`、`console`、`files`、`process` (`ps`)、`services`、`logs`、`events`、`ports`、`preview`、`url`、`proxy`、`desktop` |
| 状态 | `snapshot`、`checkpoint`、`restore`、`env`、`policy` |
| 沙箱资源 | `sandbox` (`resources`, `resize`) |
| 智能体 | `prompt`、`run`、`chat`、`agent`、`connections`、`mcp` |
| 平台 | `deploy`、`batch`、`host`、`db`、`storage`、`volume`、`workflow`、`schedule`、`function`、`template`、`project`、`workspace`、`forge`、`catalog`、`regions`、`egress`、`cloud`、`gateway` |
| 访问与计费 | `api-key`、`service-account`、`webhook`、`member`、`billing`、`audit`、`alerts`、`notifications` |
| 原始 | `api <METHOD> <path>`，配合 `-d`（`@file` 或 `-`）和 `-H` |

## 机器参数

在任何需要沙箱的地方，您都可以给出其名称、ID、`current`（使用 `miosa use` 设置）或 `self`（在沙箱内）。在计算机上执行的命令采用 `--computer`。

```bash
miosa exec my-box -- npm test      # by name
miosa exec sbox_abc123 -- npm test # by id
miosa use my-box                   # then omit the name
miosa exec -- npm test
```

## 全局标志

这些适用于每个命令。

```text
--api-key string      API key (overrides MIOSA_API_KEY and config)
--api-url string      API base URL (overrides MIOSA_BASE_URL and config)
--profile string      Config profile to use (MIOSA_PROFILE)
--org string          Organization to bill for this command (MIOSA_ORG)
--json                Machine-readable JSON (same as -o json)
-o, --output string   text (default) or json (MIOSA_OUTPUT)
-q, --quiet           Suppress informational output
--retries int         Retries on 429 and transient 5xx (default 3, MIOSA_RETRIES)
--timeout int         Request timeout in seconds; for exec and ssh the command timeout
--no-update-notice    Silence the new-version notice
```

## 退出代码

|代码|意义|
|---|---|
| 0 |成功|
| 1 |错误 |
| 2 |用途 |
| 3 |未登录或被禁止 |
| 4 |没有找到|
| 5 |冲突或无效输入 |
| 6 |速率受限（重试后）|
| 7 |服务器或网络错误 |

`miosa exec` 和 `miosa ssh` 退出并显示远程命令的状态。
