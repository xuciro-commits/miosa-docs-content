---
slug: "opencomputers/connect"
title: "连接您的第一台主机"
description: "安装主机智能体、注册计算机、验证该主机上的命令，并为 Linux、macOS 或 Windows 配置服务启动。"
section: "Computers"
order: 3014
---

注册您作为 OpenComputer 运行的计算机。
您的第一次成功是在该机器上完成了 `hostname` 命令，然后重新启动了经过验证的服务。
仅保存的密钥或在线徽章并不是终点线。

## 开始之前

- 端口 443 上的出站 HTTPS 和 WebSocket 访问，包括访问 GitHub 版本下载和 `api.miosa.ai`。
- 作为应运行智能体的用户的 Shell 访问权限，以及 Linux 或 macOS 上的 `curl` 和 POSIX shell。
- 有权访问 OpenComputers 的 MIOSA 帐户，以及为目标组织授权的 `miosa` CLI 或平台 API 密钥。
- `sudo` 访问和 systemd（如果您遵循 Linux 后台服务部分）。

谨慎选择智能体的操作系统用户：其权限决定主机可以访问哪些本地文件和服务。
连接敏感工作负载之前，请参阅[安全模型](/docs/opencomputers/security)。

> **注意：预构建目标并不通用**
>
> 预构建的 Linux 智能体目标是 x86-64。
> 在 Linux ARM64 上或在没有预构建智能体的平台上，请遵循 [OSA 源安装说明 ](https://github.com/Miosa-osa/OSA#install-in-one-command)，而不是假定安装程序支持它。
> 注册中出现的操作系统并不能保证对每种体系结构的桌面支持。

## 1.在机器上安装智能体

**Linux / macOS**

```bash
curl -fsSL https://raw.githubusercontent.com/Miosa-osa/OSA/main/scripts/install.sh -o /tmp/osa-install.sh
sh /tmp/osa-install.sh
osa version
```

如果未找到 `osa`，请打开一个新 shell 并再次运行 `osa version`。
请遵循安装程序打印的任何特定于平台的说明。
对于固定版本，请使用 [OSA 安装指南 ](https://github.com/Miosa-osa/OSA#install-in-one-command) 中的版本选择说明。

**Windows**

从 [OSA 存储库](https://github.com/Miosa-osa/OSA) 中的发布资产安装 OSA 智能体，然后在新终端中确认智能体答案：

```powershell
osa version
```

注册接受`--platform windows`。
桌面流式传输取决于您平台的智能体构建；已注册的 Windows 或 macOS 主机不会自动拥有工作桌面。

## 2. 注册主机并保存其密钥

**MIOSA CLI**

```bash
miosa host register studio --platform linux
```

`--platform` 采用 `macos`、`linux` 或 `windows`。
`register` 创建主机记录并打印一次主机 ID 和主机密钥。
添加 `--quiet-key` 以仅打印密钥，并添加 `--region` 以固定区域。

**MIOSA 用户界面**

在 MIOSA 中打开 OpenComputers，选择 **连接计算机**，然后在目标组织中注册计算机。
为其指定一个可识别的名称，例如 `lab-server`，然后选择操作系统。
保存返回的主机 ID 和一次性主机密钥。

**主机密钥**验证该机器的智能体。
您的 **平台 API 密钥** 授权来自您的终端或后端的 REST 请求。
它们是不同的凭证并且不可互换。

## 3. 将智能体指向 MIOSA

在计算机本身上，配置与 OSA 智能体的连接：

```bash
osa opencomputers login
osa opencomputers enable --no-profile
```

当出现 `login` 提示时粘贴主机密钥。
提示符接受凭证作为终端输入；使用私人终端会话。
配置写入智能体用户的`~/.osa`目录下，包括`open_computers.toml`和`open_computers.ed25519`身份文件。
更新智能体时保留该身份。

`login` 保存密钥，`enable` 启用主机模式。
这两个命令都不能证明实时会话已开始。

## 4. 启动主机会话

在机器的终端中启动无头智能体：

```bash
OSA_OPEN_COMPUTERS_ENABLED=true osa serve
```

保持其运行，同时从另一个终端完成第一次检查。
智能体启动到 MIOSA 的经过身份验证的出站 WebSocket 连接。
不需要入站管理端口或公共 IP。

在 MIOSA 中打开主机，或从终端列出它，并确认新的心跳和命令执行所需的功能：

```bash
miosa host list --tag studio
miosa host get <host>
```

如果主机未连接，请按照[主机保持离线状态](/docs/opencomputers/troubleshooting#host-stays-offline)操作。

## 5. 在所选主机上完成命令

**MIOSA CLI**

```bash
miosa host jobs <host>
miosa host job list <host>
miosa host fs ls <host> --path /tmp
```

`miosa host jobs <host>` 列出最近的作业。 `miosa host job get <host> <job>` 显示一项作业及其输出。

**休息API**

在单独的受信任终端中，设置您的平台 API 密钥和返回的主机 ID：

```bash
export MIOSA_API_KEY="<your-platform-api-key>"
export HOST_ID="<your-host-id>"
```

确认主机记录：

```bash
curl --fail-with-body -sS "https://api.miosa.ai/api/v1/opencomputers/hosts/$HOST_ID" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```响应包含一个 `host` 对象。
确认其ID、名称、心跳和能力与本机相对应。
然后运行只读命令：

```bash
curl --fail-with-body -sS \
  -X POST "https://api.miosa.ai/api/v1/opencomputers/hosts/$HOST_ID/exec" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"cmd":"hostname","stream":false,"timeout_ms":10000}'
```

预期结果，使用您自己的作业 ID 和主机名：

```json
{
  "job_id": "<job-id>",
  "exit_code": 0,
  "stdout": "lab-server\n",
  "stderr": ""
}
```

将 `stdout` 与直接在机器上运行的 `hostname` 进行比较。
使用 `stream: false` API 收集输出；默认值为 `stream: true`，它返回 SSE 事件，直到最终结果。
对于较长的命令，请使用 [streaming execcontract](/docs/api-reference/open-computers#run-a-job) 而不是这个简短的收集输出检查。

使用您自己的请求中的 ID 和相同的授权组织上下文。
接受的请求并不能证明命令已完成。
将 API 密钥保存在您的后端或终端环境中。

## 6. 将智能体作为服务运行

命令通过后，在启动第二个实例之前停止前台 `osa serve` 进程。

**Linux（系统）**

该单元在保存配置身份的同一用户和主目录下运行。
以该用户身份运行这些命令，而不是从 root 登录 shell 运行。
如果该装置已存在，请在更换前对其进行检查；保留任何特定于站点的限制。

```bash
OSA_HOST_BIN="$(command -v osa)"
OSA_HOST_USER="$(id -un)"

sudo tee /etc/systemd/system/osa-opencomputers.service > /dev/null <<EOF_UNIT
[Unit]
Description=MIOSA OSA OpenComputers Host
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=${OSA_HOST_USER}
Environment=HOME=${HOME}
Environment=OSA_OPEN_COMPUTERS_ENABLED=true
ExecStart=${OSA_HOST_BIN} serve
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF_UNIT

sudo systemctl daemon-reload
sudo systemctl enable --now osa-opencomputers.service
systemctl status osa-opencomputers.service --no-pager
```

**其他服务经理**

在同一智能体用户下使用平台的服务管理器，并在服务中保留两件事：

- `OSA_OPEN_COMPUTERS_ENABLED=true` 环境变量，因此智能体以主机模式启动；
- `~/.osa` 目录，包括 `open_computers.toml` 和 `open_computers.ed25519` 身份文件。

macOS 上的启动智能体和 Windows 服务都以这种方式工作。
然后在 MIOSA 中确认新的心跳。

等待活动服务，然后在 MIOSA 中验证新的心跳并重复 `hostname` 请求。
启用的服务应在启动时启动，但在依赖无人值守工作之前​​在计划的重新启动后验证该行为。

## 7. 断开或退出主机

对于临时维护，停止调度新作业，检查正在运行的作业，并停止智能体：

```bash
miosa host jobs <host>
sudo systemctl stop osa-opencomputers.service
```

该设备在下次启动时保持启用状态。
要停用，请禁用启动、注销智能体并撤销主机：

```bash
sudo systemctl disable --now osa-opencomputers.service
osa opencomputers logout
miosa host rm <host> --yes
```

撤销和注销不会删除机器的文件，也不能证明每个子进程都已停止。
在删除访问权限或在其他地方重试之前，请验证活动工作。

- [主机和功能](/docs/opencomputers/hosts)：检查主机并在一台计算机或一组计算机上运行工作。

- [OpenComputers 故障排除](/docs/opencomputers/troubleshooting)：诊断脱机主机、失败的命令、身份拒绝和隧道。
