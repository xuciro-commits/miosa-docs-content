---
slug: "opencomputers/gha-runners"
title: "GitHub Actions 跑步者"
description: "在您自己的主机上为一个存储库注册自托管 GitHub Actions 运行器，并从 MIOSA 流式传输其活动。"
section: "Computers"
order: 3018
---

运行器是主机为一个存储库注册的 GitHub Actions 运行器。
MIOSA 在您的计算机上安装并监督运行程序进程，以便工作流作业在您控制的硬件上运行。

当构建需要 GPU、大型本地缓存、仅存在于您的计算机上的软件或 GitHub 托管的运行程序无法访问的网络路径时，请使用此选项。

## 注册一个跑步者

**MIOSA CLI**

```bash
miosa host gha-runner list <host>
miosa host gha-runner new <host> --repo https://github.com/acme/app --token <token>
miosa host gha-runner new <host> --repo https://github.com/acme/app --token-secret GH_TOKEN --ephemeral
miosa host gha-runner get <host> <runner>
```

别名：`miosa host gha-runners`、`miosa host runner`、`miosa host runners` 和 `create` 或 `new` 的 `add`。

**休息API**

```bash
curl --fail-with-body -sS \
  -X POST "https://api.miosa.ai/api/v1/opencomputers/hosts/$HOST_ID/gha-runners" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"repo_url":"https://github.com/acme/app","name":"gpu-builder","labels":["gpu","linux"],"pat_secret_id":"<secret-id>"}'
```

|领域|类型 |必填 |描述 |
|---|---|---|---|
| `repo_url` |字符串|是的 |存储库网址；必须位于 `github.com` 或 `*.github.com` 主机上 |
| `name` |字符串|是的 |跑步者名称，1到128个字符；每个主机都是唯一的|
| `labels` |字符串[] |没有 |跑步者标签；默认 `[]` |
| `ephemeral` |布尔 |没有 |完成一项工作后取消注册跑步者；默认 `false` |
| `pat_secret_id` |字符串|之一 |持有 GitHub 令牌的现有主机密钥的 ID |
| `pat_value` |字符串|之一 |内联令牌； MIOSA 将其存储为您的主机机密 |

CLI 标志名称与 API 字段名称不同：`--repo` 映射到 `repo_url`，`--token` 映射到 `pat_value`（发送到主机而不存储），`--token-secret` 映射到 `pat_secret_id`。
`--label` 是可重复的，`--name` 默认为生成的名称。

> **警告：MIOSA 永远不会返回令牌**
>
> GitHub 令牌不以可读形式存储，并且任何读取都不会返回。
> 首选 `pat_secret_id` 或 `--token-secret`，因此该值位于 [主机密钥](/docs/opencomputers/hosts#secrets) 中，而不是位于您可能粘贴到其他地方的命令中。

> **注意：不支持 GitHub Enterprise Server**
>
> `repo_url` 必须指向 `github.com`。 GHES URL 将被拒绝并显示 `422`。

## 跑步者状态

|状态 |意义|
|---|---|
| `provisioning` | MIOSA 正在安装并启动运行程序 |
| `idle` |已注册并等待工作 |
| `running_job` |执行工作流程作业 |
| `offline` |已注册但无法访问 |
| `failed` |安装或注册失败 |
| `removed` |注销 |

`current_job` 在运行程序执行时设置，并且 `last_activity_at` 告诉您它上次执行任何操作的时间。

## 观看跑步者

**MIOSA CLI**

```bash
miosa host gha-runner events <host> <runner>
miosa host gha-runner events <host> <runner> --json
```

当跑步者拿起并完成作业时流式传输事件。 Ctrl-C 停止流。

**休息API**

```bash
curl -N --fail-with-body -sS \
  "https://api.miosa.ai/api/v1/opencomputers/hosts/$HOST_ID/gha-runners/$RUNNER_ID/events" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

命名的 SSE 事件：`progress`、`ready`、`status`、`error` 和 `removed`。

## 修复和删除

```bash
miosa host gha-runner refresh <host> <runner>
miosa host gha-runner rm <host> <runner> --yes
```

|行动|休息 |笔记|
|---|---|---|
|刷新 | `POST .../gha-runners/{runner_id}/refresh` |重新安装并重新启动运行器；返回 `{"ok": true}` |
|删除 | `DELETE .../gha-runners/{runner_id}` |停止运行器并从 GitHub 取消注册；返回 `204` |

`refresh` 是针对升级后陷入 `failed` 或 `offline` 的跑步者的修复，而不是删除并重新注册。
如果 GitHub API 对注册调用进行速率限制，则这两个操作都可以返回 `429`。

## 在工作流程中引用它

将工作流作业指向运行者的标签：

```yaml
jobs:
  build:
    runs-on: [self-hosted, gpu, linux]
```

仅当 Host Agent 连接且运行程序为 `idle` 时，作业才会在您的计算机上运行。
如果主机在工作中离线，则 GitHub 端的工作流程将失败；重试之前检查 `miosa host gha-runner get` 和 [跑步者事件 ](/docs/opencomputers/gha-runners#watch-a-runner)。
