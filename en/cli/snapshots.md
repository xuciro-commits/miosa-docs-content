---
slug: "cli/snapshots"
title: "Snapshots, checkpoints and restore"
description: "Browse, name, fork and delete snapshots of sandboxes and computers, plus checkpoints and restore."
section: "CLI, SDKs & MCP"
order: 11005
---

## Snapshots

`miosa snapshot` works on sandboxes and computers alike. Every snapshot of a machine is its history. A named snapshot is a restore point you pin on purpose: it outlives the machine and never appears in history. Each organization keeps 10 named snapshots free.

```bash
miosa snapshot list my-box
miosa snapshot save my-box before-upgrade
miosa snapshot fork before-upgrade
miosa snapshot latest my-box
```

| Command | What it does |
|---|---|
| `miosa snapshot list [machine]` | History across machines, or for one; `--search`, `--workspace`, `--limit` |
| `miosa snapshot named` | Named snapshots and the free allowance |
| `miosa snapshot save <machine> <name>` | Pin a machine (running: captured now, stopped: its newest snapshot) under a name |
| `miosa snapshot save --snapshot <id> <name>` | Pin one existing snapshot |
| `miosa snapshot rm <name>` | Remove a name |
| `miosa snapshot latest <machine>` | Newest snapshot of a machine |
| `miosa snapshot info <id>` | One snapshot and what depends on it |
| `miosa snapshot tree <id> [path]` | List files inside a snapshot, machine stopped or gone |
| `miosa snapshot pull <id> <path> -d ./out` | Download a file, or a folder (extracted from a tar), without overwriting |
| `miosa snapshot fork <id>` | New machine from a snapshot |
| `miosa snapshot fork --name <name>` | New machine from a named snapshot |
| `miosa snapshot delete <id>... \| --all <machine>` | Delete what you name; anything still depended on is skipped and reported |

Deleting needs `--yes` when there is no terminal. Use `--computer` when the machine argument is a computer. `miosa snapshot save` captures a running machine now (waiting with `--wait`, on by default) or the newest snapshot of a stopped one.

## Checkpoints

Checkpoints are sandbox-local restore points.

```bash
miosa checkpoint create my-box --comment "before deps"
miosa checkpoint list my-box
miosa checkpoint info <checkpoint>
miosa checkpoint delete <checkpoint>
```

## Restore

```bash
miosa restore my-box <checkpoint>
```

Restores a sandbox from a checkpoint.

> **Tip**
>
> Snapshots are the portable, cross-machine history and the basis for forking. Checkpoints are the quick sandbox-local restore points. Use `miosa snapshot` when you want to move or fork a machine, and `miosa checkpoint` for a fast rollback in place.
