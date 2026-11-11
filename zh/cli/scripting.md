---
slug: "cli/scripting"
title: "JSON 输出和脚本"
description: "miosa CLI 的机器可读输出、流式传输、错误形状、退出代码和 CI 模式。"
section: "CLI, SDKs & MCP"
order: 11012
---

## 机器可读的输出

`--json`（或 `-o json` 或 `MIOSA_OUTPUT=json`）在标准输出上打印机器可读的 JSON。

```bash
miosa list --json | jq -r '.data[].name'
miosa create ci-box --wait --json | jq -r .id
```

通过管道传输时，输出永远不会自动切换为 JSON。流式打印每行一个 JSON 对象的命令。

## 错误

错误将通过 API 错误代码发送至 stderr。对于 `--json`，它们是一个 `{"error":{...}}` 对象。

```bash
if ! miosa exec my-box --json -- ./flaky.sh > out.json 2> err.json; then
  jq -r '.error.code' err.json
fi
```

当没有终端时，破坏性命令需要 `--yes`。

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

`miosa exec` 和 `miosa ssh` 以远程命令的状态退出，因此它们直接进入管道：

```bash
miosa exec ci-box -- npm test || echo "tests failed with $?"
```

## 重试和超时

`--retries`（默认 3）在 429 和瞬态 5xx 上重试。 `--timeout` 以秒为单位限制请求，对于 `exec` 和 `ssh`，它还限制命令（1 到 300）。

## CI 模式

```bash
export MIOSA_API_KEY="$MIOSA_CI_KEY"
miosa create "ci-${CI_PIPELINE_ID}" --size small --ttl 1h --wait --json | jq -r .id
miosa exec "ci-${CI_PIPELINE_ID}" -- ./build.sh
status=$?
miosa destroy "ci-${CI_PIPELINE_ID}" --yes
exit $status
```

## 来自脚本的原始调用

`miosa api` 使用相同的凭据并打印正文：

```bash
miosa api GET /sandboxes/ci-box/ports --json | jq -r '.data[].port'
miosa api POST /webhooks -d @hook.json
```

> **提示**
>
> 对于非交互式使用，请设置 `MIOSA_API_KEY` 并完全避免浏览器登录。有关配置文件和凭证顺序，请参阅[配置和配置文件](/docs/cli/config)。
