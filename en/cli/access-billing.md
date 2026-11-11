---
slug: "cli/access-billing"
title: "Access, org and billing"
description: "API keys, service accounts, webhooks, members, billing, audit log, alerts and notifications from the miosa CLI."
section: "CLI, SDKs & MCP"
order: 11009
---

## API keys

```bash
miosa api-key create my-key --preset cli
miosa api-key list
miosa api-key presets
miosa api-key rotate <key>
miosa api-key update <key> --name renamed --rate-limit 600
miosa api-key rm <key> --yes
```

`api-key create` takes `--preset` (`read-only`, `ci`, `agent` or `cli`), `--scope` (repeatable, for example `sandboxes:exec`), `--expires-in` (days), `--rate-limit`, `--allowed-ip` (repeatable), `--workspace-id` and `--quiet-key` (print only the secret).

Scoped keys for external users:

```bash
miosa api-key scoped --external-user-id <id> --workspace-id <id> --scope sandboxes:exec --expires-at 2027-01-01T00:00:00Z
```

## Service accounts

Non-human accounts that own API keys for pipelines and programs.

```bash
miosa service-account new deploy-pipeline
miosa service-account list
miosa service-account key new <account> --preset ci --expires-in 90 --quiet-key
miosa service-account keys <account>
miosa service-account update <account> --description "..." --name renamed
miosa service-account rm <account> --yes
```

`service-account key new` takes the same scoping flags as `api-key create`.

## Webhooks

Send events to your own HTTPS endpoints.

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

`webhook create` takes `--event` (repeatable, see `webhook events`), `--header NAME=VALUE` (repeatable), `--retries` (0 to 10) and `--workspace-id`.

## Members

```bash
miosa member list
miosa member add <email> --role member
miosa member role <member> --role admin
miosa member rm <member> --yes
```

Roles are `admin`, `member` or `viewer`.

## Billing

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

`miosa billing usage` shows model usage by day and takes no filters. To filter the product catalog by `--product`, `--size`, `--state` or `--template`, use `miosa catalog`.

## Audit log

```bash
miosa audit -n 50 --type sandbox.created --workspace-id <id>
```

## Alerts and notifications

```bash
miosa alerts list
miosa alerts resolve <alert>

miosa notifications list
miosa notifications unread
miosa notifications read <notification>
miosa notifications read-all
```

> **Note**
>
> Billing reads the organization new machines bill. Pin it with `miosa org bill <org>`, or override per command with `--org`. See [Account and environment](/docs/cli/account).
