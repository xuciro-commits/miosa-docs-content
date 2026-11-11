---
slug: "cli/config"
title: "配置和配置文件"
description: "~/.miosa/config.toml 中的命名登录和默认设置，以及配置命令、选项和环境变量。"
section: "CLI, SDKs & MCP"
order: 11011
---

设置位于 `~/.miosa/config.toml`（模式 0600）中。 `config` 永远不会打印 API 密钥；使用 `miosa login` 登录。

## 配置命令

```bash
miosa config list            # effective settings of the current profile (alias ls)
miosa config get <key>       # print one setting
miosa config set <key> <value>
miosa config path            # print the config file path
```

键：`api_url`、`current_sandbox`、`default_workspace`。

```toml
api_url           = "https://api.miosa.ai/api/v1"
api_key           = "msk_u_..."
default_workspace = "default"
current_sandbox   = "my-box"
```

## 个人资料

配置文件是具有自己的 API URL、组织和默认值的命名登录。多个配置文件可让您将帐户或环境分开。

```bash
miosa login --profile work
miosa profile list
miosa profile use work
miosa --profile default list
```

|命令 |它有什么作用 |
|---|---|
| `miosa profile add <name>` |创建一个空配置文件（然后`miosa login --profile <name>`）|
| `miosa profile list` |列出配置文件（别名 `ls`）|
| `miosa profile show [name]` |显示配置文件的设置（密钥被屏蔽）|
| `miosa profile use <name>` |将配置文件设置为新命令的默认配置文件 |
| `miosa profile rm <name>` |删除配置文件及其存储的密钥 |

`miosa profile add` 采用 `--api-url` 和 `--workspace`。

### 哪个配置文件有效

1. `--profile`
2. `MIOSA_PROFILE`
3. 一组带有`miosa profile use`
4. `default`

## 凭证解析

1. `--api-key`
2. `MIOSA_API_KEY`
3. `~/.miosa/config.toml` 中的活动配置文件

## 环境变量

```text
MIOSA_API_KEY          API key
MIOSA_BASE_URL         API base URL
MIOSA_PROFILE          Profile to use
MIOSA_ORG              Organization to bill
MIOSA_OUTPUT           Output format (text or json)
MIOSA_RETRIES          Retries on 429 and transient 5xx
MIOSA_NO_UPDATE_NOTICE Set to 1 to silence the new-version notice
```

## 退出

```bash
miosa logout                # remove the stored key of the active profile
miosa logout --all          # delete the config file (all profiles)
```

> **注意**
>
> `miosa doctor` 检查配置、连接和身份验证，并打印每个故障的修复。
