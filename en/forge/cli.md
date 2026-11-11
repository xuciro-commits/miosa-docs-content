---
slug: "forge/cli"
title: "Forge CLI reference"
description: "Every miosa forge command with an example: auth, repositories, pull requests, checks, branch policy, grants, forks and releases."
section: "Services & projects"
order: 1038
---

Every command takes a repository by name, `org/name` or id, except `repo show`, `repo update` and `repo delete`, which take the id.
Add `--json` to any command for a stable JSON envelope.
Commands that change something take `--idempotency-key`, so a retry never acts twice.

> **Note: Version**
>
> These examples use CLI 2.3.10.
> Run `miosa update` to get the latest.

## auth

| Command | Example |
|---|---|
| `auth setup-git` | `miosa forge auth setup-git` |

Makes git use your MIOSA key for `https://forge.miosa.ai`.
`--host` points it at another Forge host.

## repo

| Command | Example |
|---|---|
| `repo create` | `miosa forge repo create hello-forge --visibility private --default-branch main` |
| `repo list` | `miosa forge repo list` |
| `repo show` | `miosa forge repo show <repository-id>` |
| `repo clone` | `miosa forge repo clone hello-forge` |
| `repo update` | `miosa forge repo update <repository-id> --name "Hello Forge"` |
| `repo delete` | `miosa forge repo delete <repository-id> --yes` |

`repo create` also takes `--slug` and `--project-id`.
`repo delete` is permanent.

## pr

| Command | Example |
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

`pr new` takes `--base`, `--body`, `--draft` and `--head-repo` for a pull request from a fork.
`pr list` filters with `--state open|closed|merged|all`, `--author` and `--base`.
`pr review --decision` is `approved`, `changes_requested` or `commented`.
`pr check` reports a check on the pull request's head commit and is refused for the pull request's own author.

## check

| Command | Example |
|---|---|
| `check report` | `miosa forge check report hello-forge --commit <sha> --name unit-tests --conclusion passed --file check.json` |
| `check list` | `miosa forge check list hello-forge --commit <sha>` |
| `check suite` | `miosa forge check suite hello-forge <sha>` |
| `check show` | `miosa forge check show hello-forge <check-id>` |
| `check evidence` | `miosa forge check evidence hello-forge <check-id>` |

A completed check needs `{"evidence": {...}}` in `--file`, up to 64 KB.
`--status queued` or `in_progress` reports a check that is still running.

## policy

| Command | Example |
|---|---|
| `policy show` | `miosa forge policy show hello-forge` |
| `policy set` | `miosa forge policy set hello-forge --approvals 1 --check unit-tests` |
| `policy owners` | `miosa forge policy owners hello-forge` |

See [branch policy](/docs/forge/pull-requests#branch-policy) for every flag.

## grant

| Command | Example |
|---|---|
| `grant set` | `miosa forge grant set hello-forge <user-id> --role write` |

Roles are `read`, `write`, `maintain` and `admin`.

## fork

| Command | Example |
|---|---|
| `fork` | `miosa forge fork hello-forge --name hello-forge-fork` |

## release

| Command | Example |
|---|---|
| `release tag` | `miosa forge release tag hello-forge --name v0.1.0 --message "First tag"` |
| `release new` | `miosa forge release new hello-forge --tag v0.1.0 --name "First release" --body "Hello from Forge."` |
| `release upload` | `miosa forge release upload hello-forge v0.1.0 --source-archive` |
| `release list` | `miosa forge release list hello-forge` |
| `release show` | `miosa forge release show hello-forge v0.1.0` |

`release new` creates the tag if it does not exist; `--commit` picks what it points at and `--prerelease` marks it.
`release upload` attaches files, or a `.tar.gz` of the tagged commit with `--source-archive`.
