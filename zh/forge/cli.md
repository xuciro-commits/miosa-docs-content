---
slug: "forge/cli"
title: "Forge CLI 参考"
description: "每个 miosa forge 命令都有一个示例：身份验证、存储库、拉取请求、检查、分支策略、授权、分叉和发布。"
section: "Services & projects"
order: 1038
---

每个命令都按名称、`org/name` 或 id 获取存储库，但 `repo show`、`repo update` 和 `repo delete` 除外，它们采用 id。
将 `--json` 添加到任何命令以获得稳定的 JSON 信封。
更改某些内容的命令需要 `--idempotency-key`，因此重试永远不会执行两次。

> **注：版本**
>
> 这些示例使用 CLI 2.3.10。
> 运行 `miosa update` 获取最新版本。

## 授权

|命令 |示例|
|---|---|
| `auth setup-git` | `miosa forge auth setup-git` |

让 git 将您的 MIOSA 密钥用于 `https://forge.miosa.ai`。
`--host` 将其指向另一个 Forge 主机。

## 仓库

|命令 |示例|
|---|---|
| `repo create` | `miosa forge repo create hello-forge --visibility private --default-branch main` |
| `repo list` | `miosa forge repo list` |
| `repo show` | `miosa forge repo show <repository-id>` |
| `repo clone` | `miosa forge repo clone hello-forge` |
| `repo update` | `miosa forge repo update <repository-id> --name "Hello Forge"` |
| `repo delete` | `miosa forge repo delete <repository-id> --yes` |

`repo create` 也采用 `--slug` 和 `--project-id`。
`repo delete` 是永久的。

## pr

|命令 |示例|
|---|---|
| `pr new` | `miosa forge pr new hello-forge --head greeting --title "Change the greeting"` |
| `pr list` | `miosa forge pr list hello-forge --state all` |
| `pr show` | `miosa forge pr show hello-forge 1` |
| `pr diff` | `miosa forge pr diff hello-forge 1 --name-only` |
| `pr comment` | `miosa forge pr comment hello-forge 1 --body "Looks good to me."` |
| `pr comments` | `miosa forge pr comments hello-forge 1` |
| `pr review` | `miosa forge pr review hello-forge 1 --decision approved --body "Ship it."` |
| `pr merge` | `miosa forge pr merge hello-forge 1` |
| `pr check` | `miosa forge pr check hello-forge 1 --name lint --conclusion passed --head-sha <sha> --file check.json` |
| `pr checkout` | `miosa forge pr checkout hello-forge 2 --branch review-2` |
| `pr close` | `miosa forge pr close hello-forge 2` |
| `pr reopen` | `miosa forge pr reopen hello-forge 2` |

`pr new` 接受 `--base`、`--body`、`--draft` 和 `--head-repo` 来自 fork 的拉取请求。
`pr list` 使用 `--state open|closed|merged|all`、`--author` 和 `--base` 进行过滤。
`pr review --decision` 是 `approved`、`changes_requested` 或 `commented`。
`pr check` 报告对拉取请求的头部提交的检查，并且拉取请求自己的作者被拒绝。

## 检查

|命令 |示例|
|---|---|
| `check report` | `miosa forge check report hello-forge --commit <sha> --name unit-tests --conclusion passed --file check.json` |
| `check list` | `miosa forge check list hello-forge --commit <sha>` |
| `check suite` | `miosa forge check suite hello-forge <sha>` |
| `check show` | `miosa forge check show hello-forge <check-id>` |
| `check evidence` | `miosa forge check evidence hello-forge <check-id>` |

完成的检查需要 `--file` 中的 `{"evidence": {...}}`，最多 64 KB。
`--status queued` 或 `in_progress` 报告仍在运行的检查。

## policy

|命令 |示例|
|---|---|
| `policy show` | `miosa forge policy show hello-forge` |
| `policy set` | `miosa forge policy set hello-forge --approvals 1 --check unit-tests` |
| `policy owners` | `miosa forge policy owners hello-forge` |

请参阅[分支政策](/docs/forge/pull-requests#branch-policy)了解每个标志。

## 授予

|命令 |示例|
|---|---|
| `grant set` | `miosa forge grant set hello-forge <user-id> --role write` |

角色为 `read`、`write`、`maintain` 和 `admin`。

## 叉子

|命令 |示例|
|---|---|
| `fork` | `miosa forge fork hello-forge --name hello-forge-fork` |

## 发布

|命令 |示例|
|---|---|
| `release tag` | `miosa forge release tag hello-forge --name v0.1.0 --message "First tag"` |
| `release new` | `miosa forge release new hello-forge --tag v0.1.0 --name "First release" --body "Hello from Forge."` |
| `release upload` | `miosa forge release upload hello-forge v0.1.0 --source-archive` |
| `release list` | `miosa forge release list hello-forge` |
| `release show` | `miosa forge release show hello-forge v0.1.0` |

`release new` 如果标签不存在则创建该标签； `--commit` 选择它所指向的内容，并 `--prerelease` 对其进行标记。
`release upload` 附加文件，或使用 `--source-archive` 标记的提交的 `.tar.gz`。
