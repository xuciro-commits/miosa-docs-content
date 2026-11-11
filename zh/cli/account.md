---
slug: "cli/account"
title: "账号及环境"
description: "检查您是谁、哪个组织付费、该组织的成员和限制以及 CLI 版本。"
section: "CLI, SDKs & MCP"
order: 11002
---

## 你是谁

```bash
miosa whoami
```

显示用户、组织、工作空间、计划、信用和密钥所持有的范围。

## 组织

该组织是支付新机器费用的人。 `miosa org` 显示它并管理其成员。

|命令 |它有什么作用 |
|---|---|
| `miosa org` |该组织的新机器账单、成员和限制|
| `miosa org list` |您可以计费的组织 |
| `miosa org bill [org]` |固定组织新机器账单 |
| `miosa org bill --clear` |停止固定组织 |
| `miosa org members` |组织成员|
| `miosa org invite <email> --role member` |邀请某人（`owner`、`admin` 或 `member`）|
| `miosa org caps` |显示会员上限 |

任何命令上的 `--org` 都会覆盖该命令的固定组织：

```bash
miosa --org acme list
miosa --org acme create build-box
```

会员上限限制一个人可以花费或运行的金额：

```bash
miosa org caps set dana --spend 25            # dollars per billing period
miosa org caps set dana --concurrent 10       # most machines at once
miosa org caps set dana --no-spend-cap
miosa org caps set dana --no-concurrency-cap
```

## 限制

```bash
miosa limits
```

显示新机器计费的组织限制。

## 打开控制台

```bash
miosa open            # open the web console in a browser
miosa open --print    # print the URL instead
```

## 版本和更新

```bash
miosa version
miosa update          # download and replace the binary
miosa update --check  # report only
```

## 医生

```bash
miosa doctor
```

检查配置、连接和身份验证，并打印每个故障的修复程序。

> **注意**
>
> 成员及其角色位于 `miosa member` 下进行日常更改，并位于组织下进行邀请和上限。 API 密钥和服务帐户位于各自的页面下：[访问、组织和计费](/docs/cli/access-billing)。
