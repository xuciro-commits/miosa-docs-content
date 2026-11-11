---
slug: "cli/work-inside"
title: "运行命令并移动文件"
description: "沙箱中的 exec、ssh、控制台、文件、后台进程、服务、日志、端口和公共预览。"
section: "CLI, SDKs & MCP"
order: 11004
---

## 执行

```bash
miosa exec [name|id] -- <cmd> [args...] [flags]
```

在沙箱内运行命令并打印其输出。 `miosa` 使用远程命令的退出代码退出，因此它直接进入脚本和 CI。命令之前需要 `--` 分隔符。

```bash
miosa exec my-box -- echo hello
miosa exec -- ls -la /home                              # uses the current sandbox
miosa exec my-box --cwd my-repo -- npm run build
miosa exec my-box -- bash -c 'for i in 1 2 3; do echo $i; done'
```

`--cwd` 设置相对于沙箱主目录（或绝对）的工作目录。
`exec` 流输出直到命令完成。
对于运行时间较长的工作，请在后台启动它，以便在断开连接后它继续运行。

仍在启动的沙箱拒绝命令。 `exec` 进行后退重试，直至准备就绪，最多 90 秒。

## SSH

```bash
miosa ssh my-box
miosa ssh my-box -- bash -lc "cd app && npm test"
miosa ssh my-box -- bash -s < ./setup.sh
```

`miosa ssh` 使用一次性密钥和短期证书（15 分钟，仅对该沙箱有效）通过经过身份验证的隧道运行系统 `ssh`。如果服务器没有证书颁发机构，它会在沙箱中授权一次性密钥。密钥、证书和主机文件位于私有临时目录中，当 ssh 结束时该目录将被删除。

标准输入通过管道传输，最大可达 80KB。 `--api` 通过 MIOSA API 运行命令，没有 `ssh` 的机器也是如此。目前还没有交互式登录 shell。

直接针对沙箱的 `ssh user@host` 和 `scp` 不起作用：没有公共 SSH 端点。

## 控制台

```bash
miosa console my-box
```

沙箱中的交互式 shell，使用 miosa-terminal-v1 WebSocket 中继，具有真正的 PTY、调整大小处理和远程退出代码。

## 文件

```bash
miosa files [command]
```

远程路径写为 `<sandbox>:<path>`。沙箱可以是名称或id，对于当前沙箱可以省略。相对远程路径以 `/home/user` 开始。

```bash
miosa files ls my-box:/workspace
miosa files cp ./app.py my-box:/workspace/app.py
miosa files cp -r ./src my-box:/workspace/
miosa files cp my-box:/workspace/out.json ./out.json
miosa files cat my-box:/workspace/out.json
cat config.yaml | miosa files write my-box:/workspace/config.yaml
```

|命令 |它有什么作用 |
|---|---|
| `files ls [-l] <path>` |列出目录（`-l` 显示修改时间）|
| `files cp [-r] <src> <dst>` |复制本地到远程、远程到本地或远程到远程|
| `files cat <path>` |打印文件 |
| `files write <path>` |将 stdin 写入文件，创建父目录 |
| `files mkdir <path>` |创建目录及其父目录 |
| `files mv <from> <to>` |移动或重命名（别名 `rename`）|
| `files rm [-r] [-f] <path>...` |删除文件或目录 |
| `files chmod <mode> <path>` |更改权限（八进制，例如 `755`）|
| `files stat <path>` |类型、尺寸、方式及修改时间 |

目录需要 `-r`。除非给出 `-f`，否则不存在的路径是错误的。

## 后台进程

```bash
miosa process start --name build --cwd /workspace/app -- npm run build
miosa process list
miosa ps
miosa process logs <pid> --tail 50
miosa process kill <pid>
miosa process get <pid>
```

`miosa ps` 与 `miosa process list` 相同。

## 守护进程

沙箱守护进程是一个命名的长期进程：用名称启动它，然后读取
它的输出，重新启动它或以该名称停止它。

```bash
miosa daemon create my-box --name web --command "npm run dev" --cwd /workspace/app
miosa daemon list my-box
miosa daemon logs my-box web
miosa daemon restart my-box web
miosa daemon stop my-box web
```

命名服务属于计算机，由 systemd 拥有； `miosa services`
是项目服务命令，不采用沙箱标志。

## 日志和事件

```bash
miosa logs my-box -n 100 -f     # system log, follow
miosa events my-box             # stream lifecycle events
```

## 端口、预览和 URL

```bash
miosa ports my-box
miosa preview create my-box 3000 --visibility public
miosa preview list
miosa url my-box
```

`miosa preview` 在公共 URL 上发布沙箱端口，并且是可见性的所在
决定：`--visibility private|public`、`--ttl`、`--name`。 `miosa url` 打印
沙箱端点应答的 URL，并且 `miosa preview unshare` 撤销
公共链接。

## 智能体

```bash
miosa proxy my-box 5432:5432
```

通过 WebSocket 隧道将本地端口转发到沙箱端口（任何 TCP 服务；密钥需要 `sandboxes:exec`）。

## 桌面

```bash
miosa desktop start my-box
miosa desktop url my-box
```

> **注意**
>
> 通过 API 和 ssh 运行命令具有相同的退出代码行为：均以远程命令的状态退出。
