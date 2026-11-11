---
slug: "security/api-keys"
title: "API 密钥"
description: "创建、限定范围、轮换和撤销程序使用的密钥，并了解平台强制执行的预设和限制。"
section: "Security & compliance"
order: 14001
---

密钥用于验证程序的身份。它带有一组作用域、一个可选的工作区绑定、一个可选的过期时间、一个可选的源 IP 允许列表和一个速率限制，并且只在创建时显示一次。

## 常用命令

```bash
miosa api-key list                      # revoked keys hidden unless --all
miosa api-key create pipeline \
  --preset ci --expires-in 90 --rate-limit 600 \
  --allowed-ip 203.0.113.0/24
miosa api-key get pipeline              # never prints the secret
miosa api-key rotate pipeline           # old secret stops working at once
miosa api-key rm pipeline               # revoke
miosa api-key events pipeline           # when it was used, what changed
```

`--quiet-key` 只打印密钥本身，供需要将其通过管道传到安全位置的脚本使用。密钥之后无法找回；请轮换以获得新密钥。

## 作用域

`--scope` 每个标志接受一个作用域，并且可以重复指定。以下几个值得了解：

| 作用域 | 允许的操作 |
|---|---|
| `projects:read` / `projects:write` | 项目、部署目标、服务、发布、预览设置 |
| `env:read` / `env:write` / `env:reveal` | 变量、写入变量，以及读回变量值 |
| `sandboxes:read` / `sandboxes:write` / `sandboxes:exec` | 沙箱生命周期和命令 |
| `databases:read` / `databases:write` | 托管数据库，包括公共端点 |
| `deployments:read` / `deployments:write` | Docker Host 部署 |
| `previews:read` / `previews:write` | 沙箱预览，而非拉取请求预览 |

拉取请求预览设置属于 `projects:read` / `projects:write`：沙箱的 `previews:*` 作用域从不授予这些设置。

## 预设

`miosa api-key presets` 列出服务器提供的预设；`--preset` 在创建时选择一个。具名预设包括 `read-only`、`ci`、`agent`、`cli`、`automation` 和 `full-access`。预设只是起点，不是上限：对于预设未包含的任何内容，都可以添加 `--scope`，而创建的密钥会准确列出其拥有的权限。

对于产品的一位最终用户，`miosa api-key scoped` 会签发一个绑定到该用户而不是组织的密钥。

> **注意：`env:reveal` 是独立的作用域**
>
> `env:reveal` 与 `env:read` 是刻意分开的：列出变量的密钥无法读取变量的值。不要在不需要它的密钥上启用它，也永远不要把密钥放进客户端包中。
