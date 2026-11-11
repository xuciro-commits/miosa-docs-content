---
slug: "cli/install"
title: "安装并验证"
description: "从 macOS、Linux 和 Windows 上的 npm、Homebrew、安装脚本或发布存档安装 miosa CLI，验证 2.3，然后登录。"
section: "Get started"
order: 5
---

`miosa` 是官方 MIOSA 命令行工具。有一个 `miosa` CLI：版本 2.3。它可以在 macOS、Linux 和 Windows 上运行。

## 1.安装

**macOS / Linux**

```bash
curl -fsSL https://raw.githubusercontent.com/Miosa-osa/miosa-cli-go/main/install.sh | sh
```

macOS 和 Linux。对于非 root 用户，安装到 `~/.local/bin/miosa`；对于 root 用户，安装到 `/usr/local/bin/miosa`。用 `INSTALL_DIR=/usr/local/bin` 覆盖。

**Windows**

```bash
npm i -g @miosa/cli
```

安装 `miosa` 2.3 CLI。该软件包为您的平台（包括 Windows x64）解决了正确的本机构建问题。在 macOS、Linux 和 Windows 上的工作方式相同。

安装程序执行以下步骤：

1. 检测您的操作系统和CPU。
2. 下载匹配的 `miosa` 二进制文件并检查其 SHA-256。
3. 将二进制文件安装到您的 `$PATH` 上。

然后使用 `miosa login` 登录（下面的步骤 3）：它会打开您的浏览器，并且本机的密钥存储在 `~/.miosa/config.toml` 中。

更喜欢其他安装路径？

**自制**

```bash
brew install --cask Miosa-osa/homebrew-tap/miosa
```

仅限 macOS。

**手册**

从 [GitHub Releases](https://github.com/Miosa-osa/miosa-cli-go/releases/latest) 下载适合您平台的存档，并将二进制文件放在 `$PATH` 上的目录中。

```text
miosa_2.3.1_darwin_arm64.tar.gz
miosa_2.3.1_darwin_amd64.tar.gz
miosa_2.3.1_linux_arm64.tar.gz
miosa_2.3.1_linux_amd64.tar.gz
miosa_2.3.1_windows_amd64.zip
```

## 2. 验证

```bash
miosa --version
miosa version
```

这些打印 `2.3.1`。如果它们打印 `1.x`，请运行 `npm i -g @miosa/cli@latest` 或 `brew upgrade` 以移至 2.x 行。

## 窗口

npm install 是 Windows 上最简单的路径： `npm i -g @miosa/cli` 解析 `win32-x64` 构建 (`@miosa/cli-win32-x64`) 并将 `miosa` 放在您的 PATH 上。

要手动安装，请从 [GitHub Releases](https://github.com/Miosa-osa/miosa-cli-go/releases/latest) 下载 `miosa_2.3.1_windows_amd64.zip`，解压 `miosa.exe`，并将其文件夹添加到 `PATH`。在 PowerShell 中：

```powershell
$dir = "$env:USERPROFILE\miosa"
Expand-Archive -Path "$env:USERPROFILE\Downloads\miosa_2.3.1_windows_amd64.zip" -DestinationPath $dir
[Environment]::SetEnvironmentVariable("Path", "$env:Path;$dir", "User")

# open a new terminal, then
miosa --version
```

`SetEnvironmentVariable` 与 `User` 范围使更改永久生效，无需管理员权限；它只影响运行后打开的终端。仅对于当前会话，请使用 `$env:Path += ";$dir"` 代替。

Windows 版本仅限 x64。没有 windows/arm64 存档。

## 从 1.x 升级

1.x 是 Node CLI。 2.x 是相同的 `miosa` 命令，但具有重写的命令表面，上面的安装现在都为您提供 2.3.x。

改变了什么：

- **沙盒命令丢失了 `sandbox` 前缀。** `miosa sandbox create` 现在是 `miosa create`。
- **部署工作移至 `miosa deploy` 下。** 旧的 `miosa app` 和顶级 `releases`、`rollback` 和 `builds` 命令现在是 `miosa deploy` 子命令。 2.2 中的新增功能：`miosa deploy create` 发布沙箱、git 存储库或本地目录。
- **上下文变成了配置文件。** `miosa context` 现在是 `miosa profile`，存储在 `~/.miosa/config.toml` 中。
- **在 2.1 中恢复。** 多个 1.x 命令系列回归，并重命名为：`miosa device`（任何运行时类型）、`miosa mcp` 以及用于容器、网格、GitHub Actions 运行器、备份和隧道的 `miosa host` 系列。
- **仍然从公共 CLI 中删除：**内部 OSA 工具 `actions`、`blueprint`、`changes`、`drift`、`evidence`、`capabilities` 和 `command-overview`。操作权限是服务器端的，`miosa --help`取代了`command-overview`。

重命名的命令：

| 1.x | 2.3 |
|---|---|
| `miosa sandbox create` | `miosa create` |
| `miosa sandbox exec` | `miosa exec` |
| `miosa sandbox stop` / `resume` | `miosa stop` / `miosa resume` |
| `miosa sandbox fork` | `miosa fork` |
| `miosa sandbox wait` | `miosa wait` |
| `miosa sandbox prompt` | `miosa prompt` |
| `miosa sandbox write-file` | `miosa files write` |
| `miosa sandbox dev` | `miosa console` |
| `miosa sandbox doctor` | `miosa doctor` |
| `miosa sandbox publish` | `miosa deploy` |
| `miosa ls` | `miosa list` |
| `miosa rm` | `miosa destroy` |
| `miosa cp` | `miosa files cp` |
| `miosa watch` | `miosa events` |
| `miosa context` | `miosa profile` |
| `miosa app link` / `promote` / `rollback` | `miosa deploy` (`promote`, `rollback`) |
| `miosa releases` | `miosa deploy releases` |
| `miosa rollback` | `miosa deploy rollback` |
| `miosa builds` | `miosa deploy builds` |
| `miosa deploy destroy` | `miosa deploy rm` |
| `miosa deploy domain` | `miosa deploy domains` / `miosa deploy domain-add` |
| `miosa deploy env` | `miosa deploy env` / `miosa deploy env-set` |
| `miosa databases` | `miosa db` |
| `miosa api-keys` | `miosa api-key` |
| `miosa checkpoints` | `miosa checkpoint` |
| `miosa templates` | `miosa template` |
| `miosa schedules` / `miosa cron` | `miosa schedule` |
| `miosa connectors` | `miosa connections` |
| `miosa workspaces` / `miosa teams` | `miosa workspace` |
| `miosa devices` | `miosa device` |
| `miosa opencomputers` | `miosa host` |
| `miosa containers` | `miosa host container` |
| `miosa meshes` | `miosa host mesh` |
| `miosa gha-runners` | `miosa host gha-runner` |
| `miosa backups` | `miosa host backup`（快照位于 `miosa host backup snapshot` 下） |
| `miosa tunnel open` | `miosa host tunnel` |
| `miosa mcp ...` | `miosa mcp ...`（在 2.1 中恢复） |
| `miosa capabilities` | `miosa catalog` |
| `miosa command-overview` | `miosa --help` |
| `miosa actions ...` | 已移除（权限由服务端管理） |

## 更新

```bash
miosa update            # download and replace the binary
miosa update --check    # report only
miosa version
```

`miosa update` 根据版本 `checksums.txt` 验证下载的 SHA-256，并拒绝替换 Homebrew 管理的二进制文件（在那里使用 `brew upgrade`）。当有新版本存在时，每天最多出现一次单行通知；使用 `--no-update-notice` 或 `MIOSA_NO_UPDATE_NOTICE=1` 将其静音。

## 外壳完成

```bash
miosa completion zsh      # bash, fish, powershell
```

沙箱名称、配置文件、模板和智能体均由实时 API 完成。

## 3. 登录

```bash
miosa login               # opens the browser, device-code sign in
miosa whoami              # organization, plan, credits, and the scopes your key holds
```

`miosa whoami` 显示用户、组织、工作空间、计划和密钥所包含的范围。

### 使用密钥登录

`miosa login --with-key` 请求具有隐藏输入的键，并且 `--key-stdin` 从标准输入读取它。密钥永远不会被接受为参数。

```bash
miosa login --with-key
echo "$MIOSA_KEY" | miosa login --key-stdin
```

### 凭证解析

凭据按以下顺序解析：

1. `--api-key`
2. `MIOSA_API_KEY`
3. `~/.miosa/config.toml` 中的活动配置文件

> **提示**
>
> CI 和其他非交互式环境使用 `MIOSA_API_KEY`。请参阅[配置和配置文件](/docs/cli/config)了解多帐户设置。

## 退出

```bash
miosa logout
```

删除活动配置文件的存储密钥。 `miosa logout --all` 删除每个配置文件的配置文件。

## 诊断

```bash
miosa doctor
```

检查配置、连接和身份验证，并打印每个故障的修复程序。

- [配置和配置文件](/docs/cli/config)：命名登录、api_url、工作区默认值。
- [原始API访问](/docs/cli/api)：miosa api，用于高级命令未涵盖的任何内容。
