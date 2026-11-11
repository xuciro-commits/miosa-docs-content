---
slug: "cli/environments"
title: "环境"
description: "新沙箱或计算机的模板始于：存储库、变量、秘密文件和凭证，具有不可变的版本。"
section: "CLI, SDKs & MCP"
order: 11006
---

环境是新沙箱或计算机启动的模板。它决定克隆哪些存储库、注入哪些变量和秘密文件以及机器可以使用您的哪些凭据。每个工作区都有一个名为 `base` 的默认环境。没有 `--env` 的计算机使用其工作空间的默认值。

## 命令

```bash
miosa env list
miosa env new staging
miosa env info staging
miosa env rename staging stage
miosa env rm staging
miosa env default
```

编辑一个环境的命令采用 `--env <name>`，当省略它时编辑默认环境。 `--workspace <slug>` 将任何 `miosa env` 命令的范围限定为一个工作区。如果没有它，API 将使用工作区绑定密钥的工作区，否则使用组织级别。

## 变量和秘密文件

```bash
miosa env set --env staging STRIPE_KEY       # value is asked for, input hidden
miosa env set --env staging --from-file .env
miosa env unset --env staging STRIPE_KEY
miosa env file put --env staging hello-world/backend/.env ./backend.env
miosa env info staging                        # masked
miosa env info staging --reveal               # print values; written to the audit log
```

值永远不会被接受为参数，因为参数最终会出现在 shell 历史记录中。给出 `NAME` 并在询问时输入值，通过管道将其传输到标准输入，或使用 `--from-file` 加载 `.env` 文件。

`miosa env info` 屏蔽变量值并且从不打印秘密文件内容。 `--reveal` 打印变量值并将每个显示写入审核日志。

## 存储库

```bash
miosa env repo add --env staging octocat/hello-world --branch develop --setup-file ./repo-setup.sh --blocking
```

`--source` 选择 `github` 或 `forge`（默认 `github`）。 `--blocking` 表示在安装脚本完成之前机器无法使用； `--non-blocking` 将其与其他所有内容一起运行。

## 切换和保护

```bash
miosa env toggle --env staging github off
miosa env protect --env staging on
```

四个切换开关是 `github`、`secrets`、`machine-key` 和 `agent-connections`。 `protect on` 不会传递任何你的内容，无论切换开关说什么。 `miosa create --no-env` 为一个沙箱提供相同的保证。 `miosa env new --safe-for-third-parties` 将其赋予环境本身。

## 版本和升级

```bash
miosa env versions staging     # how many machines sit on each version
miosa info my-box              # shows "staging v1", and whether an upgrade is available
miosa env upgrade my-box       # move one machine to the latest version
miosa env upgrade --all staging
```

每次保存更改机器接收的内容都会创建一个新的不可变版本。机器在启动时会固定最新版本并保留它。储蓄永远不会影响到正在运行的机器。

> **注意**
>
> 机器磁盘上的升级是不可逆的。新版本保留的秘密将从机器中删除，因此`miosa env upgrade`要求您确认，除非您通过`--force`。

## 从环境启动机器

```bash
miosa create my-box --env staging
miosa computer create --env staging --size small
```

## 设置脚本

`--setup-file ./setup.sh` 机器准备就绪后，在后台运行一次脚本（UTF-8，最多 64KB）。它从不延迟`ready`。用`miosa info`观察：设置状态为`pending`、`running`、`done`或`failed`，运行失败显示其错误。 CLI 在发送任何内容之前都会检查大小和编码。

```bash
miosa create my-box --env staging --setup-file ./setup.sh
miosa info my-box
miosa env repo add --env staging octocat/hello-world --setup-file ./repo-setup.sh --blocking
```
