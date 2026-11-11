---
slug: "cli/machines"
title: "Machines and lifecycle"
description: "Create, list, inspect, stop, fork, recover and destroy sandboxes and computers with the miosa CLI."
section: "CLI, SDKs & MCP"
order: 11003
---

A machine is either a sandbox (code and headless work) or a computer (a full GUI desktop). Lifecycle commands work on both.

## Create a sandbox

```bash
miosa create [name] [flags]
```

```bash
miosa create my-box --wait
miosa create ci-box --size small --template miosa-sandbox --ttl 2h --wait
miosa create my-box --env staging --setup-file ./setup.sh
```

| Flag | Meaning |
|---|---|
| `--size string` | `micro`, `tiny`, `xs`, `small`, `quad`, `medium`, `large`, `xl` (default `small`) |
| `--template string` | Template staged inside the sandbox image, for example `nextjs` or `static-html` (default `miosa-sandbox`) |
| `--env string` | Environment the sandbox starts from (default: your default environment) |
| `--no-env` | Pass nothing of yours to the sandbox (safe to hand to third parties) |
| `--setup-file string` | Shell script (UTF-8, max 64KB) run in the background once the sandbox is ready |
| `--ttl string` | Time to live: seconds, `90m`, `2h` or `2d` (default: the plan's) |
| `--idle-timeout string` | Pause after this much inactivity: seconds, `15m`, `1h` |
| `--meta stringArray` | Metadata, `KEY=VALUE` (repeatable) |
| `--workspace string` | Workspace slug to assign the sandbox to |
| `--wait` | Wait until the sandbox is ready for commands |
| `--wait-timeout duration` | How long `--wait` may take (default `3m0s`) |

## List and inspect

```bash
miosa list --state running
miosa list --json | jq -r '.data[].name'
miosa info my-box            # state, environment and setup status
miosa usage my-box           # runtime, time left and estimated cost
```

`miosa list` takes `--state`, `--tag`, `--template`, `--search`, `--workspace`, `--workspace-id`, `--limit`, `--page`, `--ids` (print only ids) and `--errored` (include sandboxes in the error state).

`miosa info` shows the pinned environment version (for example `staging v1`) and whether an upgrade is available.

## Current sandbox

```bash
miosa use my-box     # set the current sandbox
miosa use            # show it
miosa exec -- npm test
```

Anywhere a sandbox is expected you may give its name, its id, `current` or `self` (inside a sandbox).

## Start, stop, pause, resume, restart

```bash
miosa stop my-box     # keep the disk
miosa pause my-box    # keep disk and memory state
miosa resume my-box
miosa restart my-box --wait   # power-cycle in place: same id, same disk
```

`miosa restart` stops the sandbox keeping its disk and boots that disk again, so files, packages and enabled services survive and running processes start fresh.
Use it when a sandbox stops answering.
If its host stopped it with a memory checkpoint instead, the sandbox comes back from memory and the command says so, with the next step.

## Extend, tag and wait

```bash
miosa extend my-box --for 2h    # replace the time to live
miosa tag my-box --set team=ci
miosa wait my-box --for running
```

`miosa extend --for` accepts seconds, `90m`, `2h` or `2d`, and replaces the old value rather than adding to it.

## Fork and recover

```bash
miosa fork my-box copy-box      # a running copy of a sandbox
miosa recover my-box            # recover the disk into a NEW sandbox
```

## Destroy

```bash
miosa destroy my-box --yes
miosa destroy my-box other-box
```

`--yes` is required when there is no terminal. Destroy is permanent.

## Computers

Computers are the GUI machines. Every command that acts on one takes `--computer` where a machine argument appears.

| Command | What it does |
|---|---|
| `miosa computer create --size small` | Create a computer (`--size`, `--template`, `--env`, `--setup-file`) |
| `miosa computer list` | List computers |
| `miosa computer get <id>` | Details and status |
| `miosa computer start` / `stop` / `restart` | Power states |
| `miosa computer logs` | Recent VM-level logs |
| `miosa computer ports` / `urls` | Exposed ports and public URLs |
| `miosa computer auto-stop` | Auto-stop settings |
| `miosa computer rm --yes` | Destroy permanently |

> **Tip**
>
> To drive the desktop (screenshot, click, type), use an SDK or the MCP tools. The CLI exposes computer lifecycle, logs, ports and URLs. See [sdks overview](/docs/sdks).
