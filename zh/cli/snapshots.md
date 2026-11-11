---
slug: "cli/snapshots"
title: "快照、检查点和恢复"
description: "浏览、命名、分叉和删除沙箱和计算机的快照，以及检查点和恢复。"
section: "CLI, SDKs & MCP"
order: 11005
---

## 快照

`miosa snapshot` 适用于沙箱和计算机。机器的每一个快照都是它的历史。命名快照是您有意固定的还原点：它的寿命比机器长，并且永远不会出现在历史记录中。每个组织免费保留 10 个命名快照。

```bash
miosa snapshot list my-box
miosa snapshot save my-box before-upgrade
miosa snapshot fork before-upgrade
miosa snapshot latest my-box
```

|命令 |它有什么作用 |
|---|---|
| `miosa snapshot list [machine]` |跨机器或一台机器的历史； `--search`、`--workspace`、`--limit` |
| `miosa snapshot named` |命名快照和免费津贴|
| `miosa snapshot save <machine> <name>` |将机器（正在运行：立即捕获，已停止：其最新快照）固定在名称下 |
| `miosa snapshot save --snapshot <id> <name>` |固定一张现有快照 |
| `miosa snapshot rm <name>` |删除姓名 |
| `miosa snapshot latest <machine>` |机器的最新快照 |
| `miosa snapshot info <id>` |一张快照及其依赖的内容 |
| `miosa snapshot tree <id> [path]` |列出快照内的文件、机器停止或消失 |
| `miosa snapshot pull <id> <path> -d ./out` |下载文件或文件夹（从 tar 中提取），而不覆盖 |
| `miosa snapshot fork <id>` |新机器快照 |
| `miosa snapshot fork --name <name>` |来自命名快照的新机器|
| `miosa snapshot delete <id>... \| --all <machine>` |删除您指定的内容；仍然依赖的任何内容都会被跳过并报告 |

无终端时删除需要`--yes`。当机器参数是计算机时，使用 `--computer`。 `miosa snapshot save` 现在捕获正在运行的机器（等待 `--wait`，默认情况下）或已停止机器的最新快照。

## 检查点

检查点是沙箱本地还原点。

```bash
miosa checkpoint create my-box --comment "before deps"
miosa checkpoint list my-box
miosa checkpoint info <checkpoint>
miosa checkpoint delete <checkpoint>
```

## 恢复

```bash
miosa restore my-box <checkpoint>
```

从检查点恢复沙箱。

> **提示**
>
> 快照是可移植的、跨机器的历史记录，也是分叉的基础。检查点是快速沙箱本地还原点。当您想要移动或分叉机器时，请使用 `miosa snapshot`，而 `miosa checkpoint` 则用于快速回滚到位。
