---
slug: "cli/environments"
title: "Environments"
description: "The template a new sandbox or computer starts from: repositories, variables, secret files and credentials, with immutable versions."
section: "CLI, SDKs & MCP"
order: 11006
---

An environment is the template a new sandbox or computer starts from. It decides which repositories are cloned, which variables and secret files are injected, and which of your credentials the machine may use. Every workspace has one default environment named `base`. A machine without `--env` uses the default of its workspace.

## Commands

```bash
miosa env list
miosa env new staging
miosa env info staging
miosa env rename staging stage
miosa env rm staging
miosa env default
```

Commands that edit one environment take `--env <name>`, and edit the default environment when it is omitted. `--workspace <slug>` scopes any `miosa env` command to one workspace. Without it the API uses the workspace of a workspace-bound key, and the organization level otherwise.

## Variables and secret files

```bash
miosa env set --env staging STRIPE_KEY       # value is asked for, input hidden
miosa env set --env staging --from-file .env
miosa env unset --env staging STRIPE_KEY
miosa env file put --env staging hello-world/backend/.env ./backend.env
miosa env info staging                        # masked
miosa env info staging --reveal               # print values; written to the audit log
```

A value is never accepted as an argument, because arguments end up in shell history. Give the `NAME` and enter the value when asked, pipe it on stdin, or load a `.env` file with `--from-file`.

`miosa env info` masks variable values and never prints secret file contents. `--reveal` prints variable values and writes each reveal to the audit log.

## Repositories

```bash
miosa env repo add --env staging octocat/hello-world --branch develop --setup-file ./repo-setup.sh --blocking
```

`--source` selects `github` or `forge` (default `github`). `--blocking` means the machine is not usable until the setup script finishes; `--non-blocking` runs it alongside everything else.

## Toggles and protection

```bash
miosa env toggle --env staging github off
miosa env protect --env staging on
```

The four toggles are `github`, `secrets`, `machine-key` and `agent-connections`. `protect on` passes nothing of yours, whatever the toggles say. `miosa create --no-env` gives the same guarantee to one sandbox. `miosa env new --safe-for-third-parties` gives it to the environment itself.

## Versions and upgrades

```bash
miosa env versions staging     # how many machines sit on each version
miosa info my-box              # shows "staging v1", and whether an upgrade is available
miosa env upgrade my-box       # move one machine to the latest version
miosa env upgrade --all staging
```

Every save that changes what a machine receives creates a new immutable version. A machine pins the latest version when it starts and keeps it. Saving never reaches into a running machine.

> **Note**
>
> An upgrade is not reversible on the machine's disk. A secret the new version withholds is deleted from the machine, so `miosa env upgrade` asks you to confirm unless you pass `--force`.

## Start a machine from an environment

```bash
miosa create my-box --env staging
miosa computer create --env staging --size small
```

## Setup scripts

`--setup-file ./setup.sh` runs a script (UTF-8, at most 64KB) once, in the background, after the machine is ready. It never delays `ready`. Watch it with `miosa info`: the setup status is `pending`, `running`, `done` or `failed`, and a failed run shows its error. The CLI checks the size and encoding before it sends anything.

```bash
miosa create my-box --env staging --setup-file ./setup.sh
miosa info my-box
miosa env repo add --env staging octocat/hello-world --setup-file ./repo-setup.sh --blocking
```
