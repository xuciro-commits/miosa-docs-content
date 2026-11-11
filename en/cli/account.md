---
slug: "cli/account"
title: "Account and environment"
description: "Check who you are, which organization pays, the organization's members and limits, and the CLI version."
section: "CLI, SDKs & MCP"
order: 11002
---

## Who you are

```bash
miosa whoami
```

Shows the user, organization, workspace, plan, credits and the scopes the key holds.

## Organization

The organization is who pays for new machines. `miosa org` shows it and manages its members.

| Command | What it does |
|---|---|
| `miosa org` | The organization new machines bill, with members and limits |
| `miosa org list` | Organizations you can bill |
| `miosa org bill [org]` | Pin the organization new machines bill |
| `miosa org bill --clear` | Stop pinning an organization |
| `miosa org members` | Members of the organization |
| `miosa org invite <email> --role member` | Invite a person (`owner`, `admin` or `member`) |
| `miosa org caps` | Show member caps |

`--org` on any command overrides the pinned organization for that command:

```bash
miosa --org acme list
miosa --org acme create build-box
```

Member caps limit what one person can spend or run:

```bash
miosa org caps set dana --spend 25            # dollars per billing period
miosa org caps set dana --concurrent 10       # most machines at once
miosa org caps set dana --no-spend-cap
miosa org caps set dana --no-concurrency-cap
```

## Limits

```bash
miosa limits
```

Shows the limits of the organization a new machine would bill.

## Open the console

```bash
miosa open            # open the web console in a browser
miosa open --print    # print the URL instead
```

## Version and updates

```bash
miosa version
miosa update          # download and replace the binary
miosa update --check  # report only
```

## Doctor

```bash
miosa doctor
```

Checks configuration, connectivity and authentication, and prints a fix for each failure.

> **Note**
>
> Members and their roles live under `miosa member` for day to day changes, and under the organization for invites and caps. API keys and service accounts live under their own pages: [Access, org and billing](/docs/cli/access-billing).
