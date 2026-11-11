---
slug: "cli/devices"
title: "设备"
description: "miosa 设备是您可以在其上运行工作的所有运行时类型（沙箱、计算机等）的一个表面，用于列表、执行、文件、端口和生命周期。"
section: "CLI, SDKs & MCP"
order: 11013
---

设备是 MIOSA 可以在其上运行命令的任何运行时：沙箱、计算机或其他类型。 `miosa device` 是所有表面的一个表面，因此脚本不需要知道它正在与哪一种表面进行通信。 `<device>` 是一个 ID。

```bash
miosa device list
miosa device list --status running
miosa device capabilities <device>
miosa device exec <device> -- uname -a
```

别名：`devices`。

## 命令

|命令 |它有什么作用 |
|---|---|
| `miosa device list` |列出设备 (`-n, --limit`、`--name`、`--status`) |
| `miosa device get <device>` |显示一台设备（别名 `show`）|
| `miosa device capabilities <device>` |该设备可以做什么（别名 `caps`）|
| `miosa device exec <device>` |运行命令 (`-c, --command`, `--timeout-ms`) |
| `miosa device files <device>` |列出目录(`--path`，默认`/workspace`；别名`ls`) |
| `miosa device read <device>` |读取文件，base64(`--path`) |
| `miosa device write <device>` |写入文件（`--path`、`--content` 或 `--file`）|
| `miosa device expose <device> <port>` |发布端口 |
| `miosa device browser <device>` |显示设备的浏览器 URL（计算机） |
| `miosa device extend <device>` |替换生存时间（`--timeout`，例如 `30m` 或 `2h`）|
| `miosa device pause <device>` / `resume <device>` |暂停或恢复 |
| `miosa device stop <device>` |停止，保留磁盘|
| `miosa device rm <device> --yes` |永久销毁（别名 `delete`、`destroy`）|

## 执行

```bash
miosa device exec <device> -- uname -a
miosa device exec <device> -c "npm test" --timeout-ms 120000
```

`miosa device exec` 退出并显示远程命令的状态，如 `miosa exec`。

## 文件

```bash
miosa device files <device> --path /workspace
miosa device read <device> --path /workspace/out.json
miosa device write <device> --path /workspace/app.py --content "print(1)"
```

`miosa device read` 返回 base64 编码的文件内容。 `miosa device write` 从 `--content` 获取内容，或带有 `--file` 的 JSON 正文。

## 何时使用设备或键入的命令

当脚本应该跨运行时类型工作时，或者当您有一个 id 但不知道它是什么时，请使用 `miosa device`。当您了解类型并想要更完整的特定于类型的表面时，请使用 `miosa exec`、`miosa files` 或 `miosa computer`。

> **注意**
>
> 还可以通过键入命令中的名称对机器进行寻址 (`miosa exec my-box -- ...`)。 `miosa device` 需要一个 id。请参阅[机器和生命周期](/docs/cli/machines)。
