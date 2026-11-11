---
slug: "cli/access-billing"
title: "访问、组织和计费"
description: "来自 miosa CLI 的 API 密钥、服务帐户、Webhook、成员、计费、审核日志、警报和通知。"
section: "CLI, SDKs & MCP"
order: 11009
---

## API 密钥

```bash
miosa api-key create my-key --preset cli
miosa api-key list
miosa api-key presets
miosa api-key rotate <key>
miosa api-key update <key> --name renamed --rate-limit 600
miosa api-key rm <key> --yes
```

`api-key create` 需要 `--preset`（`read-only`、`ci`、`agent` 或 `cli`）、`--scope`（可重复，例如 `sandboxes:exec`）、`--expires-in`（天）， `--rate-limit`、`--allowed-ip`（可重复）、`--workspace-id` 和 `--quiet-key`（仅打印机密）。

外部用户的作用域键：

```bash
miosa api-key scoped --external-user-id <id> --workspace-id <id> --scope sandboxes:exec --expires-at 2027-01-01T00:00:00Z
```

## 服务帐户

拥有管道和程序的 API 密钥的非人类帐户。

```bash
miosa service-account new deploy-pipeline
miosa service-account list
miosa service-account key new <account> --preset ci --expires-in 90 --quiet-key
miosa service-account keys <account>
miosa service-account update <account> --description "..." --name renamed
miosa service-account rm <account> --yes
```

`service-account key new` 采用与 `api-key create` 相同的作用域标志。

## 网络钩子

将事件发送到您自己的 HTTPS 端点。

```bash
miosa webhook create https://example.com/hook --name deploys --event sandbox.created
miosa webhook list
miosa webhook get <webhook>
miosa webhook events
miosa webhook deliveries <webhook>
miosa webhook test <webhook>
miosa webhook replay <webhook> <delivery>
miosa webhook update <webhook> --active --event sandbox.created --retries 5
miosa webhook rm <webhook> --yes
```

`webhook create` 采用 `--event`（可重复，请参阅 `webhook events`）、`--header NAME=VALUE`（可重复）、`--retries`（0 到 10）和 `--workspace-id`。

## 成员

```bash
miosa member list
miosa member add <email> --role member
miosa member role <member> --role admin
miosa member rm <member> --yes
```

角色为 `admin`、`member` 或 `viewer`。

## 计费

```bash
miosa billing plan
miosa billing balance
miosa billing spend
miosa billing usage
miosa billing breakdown
miosa billing invoices
miosa billing upcoming
miosa billing transactions -n 50 --page 1
```

`miosa billing usage` 按天显示模型使用情况，并且不使用过滤器。要按 `--product`、`--size`、`--state` 或 `--template` 过滤产品目录，请使用 `miosa catalog`。

## 审核日志

```bash
miosa audit -n 50 --type sandbox.created --workspace-id <id>
```

## 警报和通知

```bash
miosa alerts list
miosa alerts resolve <alert>

miosa notifications list
miosa notifications unread
miosa notifications read <notification>
miosa notifications read-all
```

> **注意**
>
> 账单读取组织新机器账单。使用 `miosa org bill <org>` 固定它，或使用 `--org` 覆盖每个命令。请参阅[帐户和环境](/docs/cli/account)。
