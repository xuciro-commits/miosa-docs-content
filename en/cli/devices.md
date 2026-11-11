---
slug: "cli/devices"
title: "Devices"
description: "miosa device is one surface over every runtime kind you can run work on - sandboxes, computers and more - for listing, exec, files, ports and lifecycle."
section: "CLI, SDKs & MCP"
order: 11013
---

A device is any runtime MIOSA can run a command on: a sandbox, a computer or another kind. `miosa device` is one surface over all of them, so a script does not need to know which kind it is talking to. `<device>` is an id.

```bash
miosa device list
miosa device list --status running
miosa device capabilities <device>
miosa device exec <device> -- uname -a
```

Aliases: `devices`.

## Commands

| Command | What it does |
|---|---|
| `miosa device list` | List devices (`-n, --limit`, `--name`, `--status`) |
| `miosa device get <device>` | Show one device (alias `show`) |
| `miosa device capabilities <device>` | What this device can do (alias `caps`) |
| `miosa device exec <device>` | Run a command (`-c, --command`, `--timeout-ms`) |
| `miosa device files <device>` | List a directory (`--path`, default `/workspace`; alias `ls`) |
| `miosa device read <device>` | Read a file, base64 (`--path`) |
| `miosa device write <device>` | Write a file (`--path`, `--content`, or `--file`) |
| `miosa device expose <device> <port>` | Publish a port |
| `miosa device browser <device>` | Show a device's browser URL (computers) |
| `miosa device extend <device>` | Replace the time to live (`--timeout`, for example `30m` or `2h`) |
| `miosa device pause <device>` / `resume <device>` | Pause or resume |
| `miosa device stop <device>` | Stop, keeping the disk |
| `miosa device rm <device> --yes` | Destroy permanently (aliases `delete`, `destroy`) |

## exec

```bash
miosa device exec <device> -- uname -a
miosa device exec <device> -c "npm test" --timeout-ms 120000
```

`miosa device exec` exits with the remote command's status, like `miosa exec`.

## Files

```bash
miosa device files <device> --path /workspace
miosa device read <device> --path /workspace/out.json
miosa device write <device> --path /workspace/app.py --content "print(1)"
```

`miosa device read` returns the file contents base64-encoded. `miosa device write` takes the contents from `--content`, or a JSON body with `--file`.

## When to use device or the typed command

Use `miosa device` when a script should work across runtime kinds, or when you have an id and do not know what it is. Use `miosa exec`, `miosa files` or `miosa computer` when you know the kind and want the fuller, kind-specific surface.

> **Note**
>
> Machines are also addressable by name in the typed commands (`miosa exec my-box -- ...`). `miosa device` takes an id. See [Machines and lifecycle](/docs/cli/machines).
