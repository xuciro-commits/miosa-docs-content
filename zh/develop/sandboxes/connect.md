---
slug: "develop/sandboxes/connect"
title: "连接到正在运行的沙箱"
description: "进入实时沙箱的四种方式：SDK、CLI、通过经过身份验证的隧道的 SSH 以及预览 URL。每种方法都有什么用，什么没用。"
section: "Sandboxes"
order: 2009
---

除了您公开的端口之外，沙箱没有公共 SSH 端点，也没有开放的入站端口。每条通道都经过 MIOSA API 和经过身份验证的隧道。一共有四个，他们擅长不同的事情。

> **注意：广告中提及，而非假设**
>
> SSH 和 TCP 隧道是公开的功能：在依赖它们之前，请在您使用 `GET /api/v1/runtimes` 定位的运行时上确认它们。访客无法访问平台内网；请参阅[互联网访问和出口](/docs/develop/sandboxes/internet-access)。

|方式 |最适合 |
|---|---|
| SDK |应用程序代码：运行命令、流输出、移动文件 |
|命令行 |从您的终端进行交互式和脚本化使用 |
| SSH |真正的 `ssh` 会话，适合需要的工具 |
|预览网址 |让浏览器访问沙箱中运行的服务 |

## 1. SDK

SDK 是与沙箱对话的代码的路径。运行命令并读取其输出，或在运行时对其进行流式传输。

**Python**

```python
from miosa import Miosa

client = Miosa(api_key="msk_u_...")
sandbox = client.sandboxes.get("sbx_123")

result = sandbox.exec("uname -a")
print(result.stdout, result.exit_code)

for event in sandbox.exec_stream("pnpm install"):
    print(event.line, end="")
```

**打字稿**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });
const sandbox = await client.sandboxes.get('sbx_123');

const result = await sandbox.exec.run('uname -a');
console.log(result.stdout, result.exitCode);
```

[文件和 exec](/docs/develop/files-and-exec) 中完整介绍了执行和文件操作。暂停的持久沙箱上的 `exec` 在调度之前恢复它。

## 2. CLI

CLI 是从终端查看正在运行的沙箱的最快方法。

```bash
miosa exec my-box -- ls -la /workspace        # run one command, get its exit code
miosa console my-box                          # interactive PTY shell
miosa files cp ./app.py my-box:/workspace/    # move files in and out
```

- **`miosa exec`** 运行命令并使用远程命令的退出代码退出，因此它会放入脚本和 CI 中。仍在启动的沙箱拒绝命令； `exec` 进行退避重试，直到准备就绪。
- **`miosa console`** 通过 `miosa-terminal-v1` WebSocket 打开全双工终端：真正的 PTY、工作窗口大小调整以及 Ctrl-C 到达远程进程。在 `--` 之后使用命令来运行程序而不是登录 shell。
- **`miosa files`** 使用 `<sandbox>:<path>` 语法复制、列出和检查文件。
- **`miosa proxy my-box 5432:5432`** 通过任何 TCP 服务的 WebSocket 隧道将本地端口转发到沙箱端口。

有关完整命令集，请参阅[运行命令和移动文件](/docs/cli/work-inside)。

## 3.SSH

`miosa ssh` 为您提供真正的 SSH 会话，无需在 `~/.ssh` 中进行任何设置。它制作一次性密钥，向 API 请求短期证书（15 分钟，对该沙箱有效），并通过 MIOSA 的经过身份验证的隧道运行系统 `ssh`。如果服务器没有证书颁发机构，它会在沙箱中授权一次性密钥。密钥、证书和主机文件位于私有临时目录中，当 ssh 结束时该目录将被删除。

```bash
miosa ssh my-box                                  # interactive login shell
miosa ssh my-box -- bash -lc "cd app && npm test"  # run a command
miosa ssh my-box -- bash -s < ./setup.sh           # pipe a script in
```

标准输入通过管道传输。如果 `ssh` 或 `ssh-keygen` 不可用，或者 `--api` 不可用，则该命令将通过 API 运行。

> **警告：没有公共 SSH 端点**
>
> `ssh user@host` 和 `scp` 直接指向沙箱不起作用。隧道是唯一的 SSH 路径。它也可用于代码：Go SDK 公开 `sandboxssh.Dial`，Python SDK 将 `miosa.ssh_tunnel` 作为 OpenSSH `ProxyCommand` 和 `paramiko` / `asyncssh` 套接字提供。
>
> ```text
> # ~/.ssh/config
> Host miosa
>     HostName <sandbox id>
>     User root
>     ProxyCommand python3 -m miosa.ssh_tunnel %h
> ```

## 4. 预览 URL

要让浏览器访问沙箱内运行的服务，请将其端口公开为 HTTPS 预览 URL。预览是唯一的入站路径；沙箱中的其他内容均无法访问。

```bash
miosa preview create my-box 3000 --visibility public
miosa url my-box
```

预览带有自己的用于嵌入和共享的令牌模型。请参阅[预览](/docs/develop/previews)。

## 使用哪个

- **自动化和应用程序：** SDK。
- **您现在在终端：** `miosa console` 用于交互式工作，`miosa exec` 用于脚本。
- **需要 SSH 的工具：** `miosa ssh`，或来自您自己的代码的 SDK 的 SSH 隧道。
- **需要打开页面的人或浏览器：**预览 URL。

## 另请参阅

- [文件和执行](/docs/develop/files-and-exec)：使用 SDK 运行命令、流输出和移动文件。
- [运行命令并移动文件](/docs/cli/work-inside)：CLI 中的 exec、控制台、ssh、文件、进程、服务和端口。
- [预览](/docs/develop/previews)：将沙箱端口公开为实时 HTTPS URL。
