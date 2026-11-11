---
slug: "cli/machines"
title: "机器和生命周期"
description: "使用 miosa CLI 创建、列出、检查、停止、分叉、恢复和销毁沙箱和计算机。"
section: "CLI, SDKs & MCP"
order: 11003
---

机器要么是沙箱（代码和无头工作），要么是计算机（完整的 GUI 桌面）。生命周期命令对两者都有效。

## 创建沙箱

```bash
miosa create [name] [flags]
```

```bash
miosa create my-box --wait
miosa create ci-box --size small --template miosa-sandbox --ttl 2h --wait
miosa create my-box --env staging --setup-file ./setup.sh
```

|旗帜|意义|
|---|---|
| `--size string` | `micro`、`tiny`、`xs`、`small`、`quad`、`medium`、`large`、`xl`（默认 `small`）|
| `--template string` |模板在沙盒映像内暂存，例如 `nextjs` 或 `static-html`（默认 `miosa-sandbox`） |
| `--env string` |沙箱启动的环境（默认：您的默认环境）|
| `--no-env` |将您的任何内容都不传递到沙箱（可以安全地交给第三方）|
| `--setup-file string` |一旦沙箱准备就绪，Shell 脚本（UTF-8，最大 64KB）就会在后台运行 |
| `--ttl string` |生存时间：秒、`90m`、`2h` 或 `2d`（默认：计划的）|
| `--idle-timeout string` |如此长时间不活动后暂停：秒，`15m`，`1h` |
| `--meta stringArray` |元数据，`KEY=VALUE`（可重复）|
| `--workspace string` |将沙箱分配给的工作区 slug |
| `--wait` |等待沙盒准备好执行命令 |
| `--wait-timeout duration` | `--wait` 可能需要多长时间（默认 `3m0s`）|

## 列出并检查

```bash
miosa list --state running
miosa list --json | jq -r '.data[].name'
miosa info my-box            # state, environment and setup status
miosa usage my-box           # runtime, time left and estimated cost
```

`miosa list` 采用 `--state`、`--tag`、`--template`、`--search`、`--workspace`、`--workspace-id`、`--limit`、`--page`、`--ids`（仅打印 ID）和`--errored`（包括处于错误状态的沙箱）。

`miosa info` 显示固定的环境版本（例如 `staging v1`）以及是否可以升级。

## 当前沙箱

```bash
miosa use my-box     # set the current sandbox
miosa use            # show it
miosa exec -- npm test
```

在任何需要沙箱的地方，您都可以给出其名称、ID、`current` 或 `self`（在沙箱内）。

## 开始、停止、暂停、恢复、重新启动

```bash
miosa stop my-box     # keep the disk
miosa pause my-box    # keep disk and memory state
miosa resume my-box
miosa restart my-box --wait   # power-cycle in place: same id, same disk
```

`miosa restart` 停止沙箱保留其磁盘并再次启动该磁盘，因此文件、包和启用的服务可以生存，并且正在运行的进程可以重新开始。
当沙箱停止应答时使用它。
如果其主机使用内存检查点来停止它，则沙箱会从内存中返回，并且命令会如此进行下一步。

## 扩展、标记并等待

```bash
miosa extend my-box --for 2h    # replace the time to live
miosa tag my-box --set team=ci
miosa wait my-box --for running
```

`miosa extend --for` 接受秒、`90m`、`2h` 或 `2d`，并替换旧值而不是添加旧值。

## 分叉并恢复

```bash
miosa fork my-box copy-box      # a running copy of a sandbox
miosa recover my-box            # recover the disk into a NEW sandbox
```

## 销毁

```bash
miosa destroy my-box --yes
miosa destroy my-box other-box
```

当没有终端时需要`--yes`。破坏是永久性的。

## 计算机

计算机是 GUI 机器。每个作用于一个命令的命令都采用`--computer`，其中机器参数出现。

|命令 |它有什么作用 |
|---|---|
| `miosa computer create --size small` |创建计算机 (`--size`、`--template`、`--env`、`--setup-file`) |
| `miosa computer list` |列出计算机 |
| `miosa computer get <id>` |详细信息和状态 |
| `miosa computer start` / `stop` / `restart` |电源状态 |
| `miosa computer logs` |最近的虚拟机级别日志 |
| `miosa computer ports` / `urls` |暴露的端口和公共 URL |
| `miosa computer auto-stop` |自动停止设置 |
| `miosa computer rm --yes` |永久摧毁|

> **提示**
>
> 要驱动桌面（屏幕截图、单击、键入），请使用 SDK 或 MCP 工具。 CLI 公开计算机生命周期、日志、端口和 URL。请参阅[sdks 概述](/docs/sdks)。
