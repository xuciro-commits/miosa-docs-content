---
slug: "cli/api"
title: "原始 API 访问"
description: "使用 miosa api，使用您存储的凭据从 CLI 调用任何 MIOSA API 端点。"
section: "CLI, SDKs & MCP"
order: 11010
---

`miosa api` 使用您存储的凭据向 MIOSA API 发送请求并打印响应正文。该路径相对于 API 基本 URL (`/api/v1`)。使用它来执行 CLI 还没有命令的任何操作。

```bash
miosa api [METHOD] <path> [flags]
```

当给出 `--data` 时，该方法默认为 GET 或 POST。

```bash
miosa api /credits/balance
miosa api GET /sandboxes/my-box/ports
miosa api POST /webhooks -d '{"url":"https://example.com/hook","events":["sandbox.ready"]}'
miosa api PATCH /sandboxes/my-box/tags -d @tags.json
echo '{"name":"x"}' | miosa api POST /workspaces -d -
```

|旗帜|意义|
|---|---|
| `-X, --method string` | HTTP 方法（默认 GET 或带有 `--data` 的 POST）|
| `-d, --data string` |请求正文：标准输入的 JSON、`@file` 或 `-` |
| `-H, --header stringArray` |额外标头，`'Name: value'`（可重复）|
| `-i, --include` |首先打印状态行和响应头 |

失败状态以非零值退出（请参阅退出代码），但仍打印正文。

## 何时使用原始 API

|需要|使用 |
|---|---|
|代码/构建/测试/运行时工作区 | `miosa create`、`miosa exec`、`miosa files`、`miosa services`、`miosa url` |
|完整的 GUI/浏览器计算机 | `miosa api /computers ...` 或 SDK (`client.computers`) |
| Docker 主机设备 | SDK 或 API Docker 主机端点；从沙盒发布到 Docker 主机 |
|产品/模板准备| `miosa catalog` |
|还有什么吗| `miosa api <METHOD> <path>` |

> **提示**
>
> `miosa api` 使用与其他命令相同的配置文件、密钥和组织。将其与 `--json` 和 `jq` 结合起来编写脚本。请参阅[JSON 输出和脚本编写](/docs/cli/scripting)。
