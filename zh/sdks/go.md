---
slug: "sdks/go"
title: "Go SDK"
description: "安装、验证并使用 miosa-go v2 SDK——沙箱、exec、文件、快照、计算机、智能体运行和类型化错误。"
section: "CLI, SDKs & MCP"
order: 11017
---

MIOSA 官方 Go SDK。模块路径为 `github.com/Miosa-osa/miosa-go/v2`，版本 2.2.0。需要 Go 1.21 或更高版本。

## 安装

```bash
go get github.com/Miosa-osa/miosa-go/v2
```

```go
import miosa "github.com/Miosa-osa/miosa-go/v2"
```

子包：`github.com/Miosa-osa/miosa-go/v2/sandboxssh` 和 `github.com/Miosa-osa/miosa-go/v2/computeruse`。

## 身份验证

```go
client := miosa.NewClient("msk_u_...")
```

当密钥为空时，`NewClient(apiKey, opts...)` 会回退到 `MIOSA_API_KEY`。它还会读取 `MIOSA_BASE_URL`、`MIOSA_ACCESS_TOKEN` 和 `MIOSA_TENANT`。默认值：基础 URL 为 `https://api.miosa.ai/api/v1`，超时 60 秒，重试 3 次。

选项：`WithBaseURL`、`WithHTTPClient`、`WithTimeout`、`WithMaxRetries`、`WithTenant`、`WithAccessToken`、`WithBillTo`、`WithDefaultHeader`、`WithUserAgent`。可通过 `client.With(opts...)`、`client.AsUser(token)` 或 `client.ForTenant(id)` 派生一个限定范围的客户端。

```go
ctx := miosa.WithBillToContext(context.Background(), "acme")
```

## 沙箱

沙箱调用位于 `client.Sandboxes`。`*Sandbox` 值只是数据，不是句柄：它没有任何方法。

```go
sbx, err := client.Sandboxes.Create(ctx, miosa.CreateSandboxInput{Name: "my-box"})
ok, err := client.Sandboxes.WaitUntilReady(ctx, sbx.ID, miosa.WaitUntilReadyOptions{})
```

| 方法 | 作用 |
|---|---|
| `client.Sandboxes.Create(ctx, input)` | 创建沙箱 |
| `client.Sandboxes.CreateDetailed(ctx, input)` | 创建，并返回等待结果和计费组织 |
| `client.Sandboxes.Get(ctx, id)` | 获取单个沙箱 |
| `client.Sandboxes.GetByName(ctx, name)` | 按名称获取 |
| `client.Sandboxes.List(ctx, input)` / `ListPage` / `ListAll` | 列出（分页或全部） |
| `client.Sandboxes.Delete(ctx, id)` | 删除 |
| `client.Sandboxes.Update(ctx, id, in)` | 更新 |
| `client.Sandboxes.Stop/Pause/Resume/Extend` | 生命周期操作 |
| `client.Sandboxes.Usage(ctx, id)` | 用量 |
| `client.Sandboxes.WaitUntilReady(ctx, id, opts)` | 等待就绪 |

> **注意**
>
> `*Sandbox` 值包含 `ID` 和 `PreviewURL` 等字段，但没有方法。请使用沙箱 ID 在 `client.Sandboxes` 上调用 exec、销毁和就绪等待操作。

## Exec 与文件

```go
res, err := client.Sandboxes.Exec(ctx, sbx.ID, miosa.ExecSandboxInput{Command: "node -v"})
events, err := client.Sandboxes.ExecStream(ctx, sbx.ID, miosa.ExecSandboxInput{Command: "npm run dev"})
```

计算机 exec 使用 WebSocket 服务：

```go
out, err := computer.Exec.Bash(ctx, "ls -la")
out, err = computer.Exec.Python(ctx, "print(2 + 2)")
```

文件以计算机为范围。可用 `For` 给未限定范围的客户端设定范围：

```go
files := client.Files.For(computer.ID)
_ = files.Upload(ctx, "./notes.txt", "/home/user/notes.txt")
data, _ := files.Download(ctx, "/home/user/notes.txt")
list, _ := files.List(ctx, "/home/user")
_ = files.Mkdir(ctx, "/home/user/data")
_ = files.Rename(ctx, "/home/user/a", "/home/user/b")
```

`client.Files` 和 `computer.Files` 是同一服务；`computer.Files` 已预先限定范围。方法：`Upload`、`UploadReader`、`Download`、`DownloadTo`、`List`、`Stat`、`Mkdir`、`Rename`、`Copy`、`Chmod`、`Readdir`、`Delete`、`Export`。`FS(workingDir)` 返回按路径限定范围的 `ScopedFS`。

## 快照

共有五个接口面。

```go
snap, err := client.Sandboxes.CreateSnapshot(ctx, sbx.ID, miosa.CreateSandboxSnapshotInput{})
snaps, err := client.Sandboxes.ListSnapshots(ctx, sbx.ID)
_, err = client.Sandboxes.RestoreSnapshot(ctx, sbx.ID, snap.ID)
```

| 接口面 | 方法 |
|---|---|
| `client.Sandboxes` | `CreateSnapshot`、`ListSnapshots`、`RestoreSnapshot`、`DeleteSnapshot` |
| `computer.Snapshots` | `Create`、`List`、`Get`、`Delete`、`Events` |
| `client.Snapshots` | `List`、`Groups`、`Get`、`Delete`、`DeleteMany`、`Fork`、`Tree`、`Download` |
| `client.NamedSnapshots` | `List`、`Get`、`Save`、`Remove`、`Deploy` |
| `client.SnapshotsStandalone` | `List`、`Get`（无类型数据） |

## 计算机

```go
computer, err := client.Computers.Create(ctx, miosa.CreateComputerInput{Name: "dev-box"})
err = computer.Start(ctx)
```

| 方法 | 作用 |
|---|---|
| `client.Computers.Create(ctx, input)` | 创建 |
| `client.Computers.List(ctx, input)` | 列出 |
| `client.Computers.Get(ctx, id)` | 获取单个 |
| `client.Computers.Delete(ctx, id)` | 删除 |

`Computer` 句柄还提供 `Start`、`Stop`、`Restart`、`Destroy`、`Wait`、`Bash`、`Python`、`PreviewURL`、`PublicURL`、`Clone`、`Resize`、`Move`、`VncCredentials`、`StreamToken`。桌面操作：`Screenshot`（字节）、`Click`、`LeftClick`、`DoubleClick`、`RightClick`、`Drag`、`Scroll`、`Type`、`Key`、`Hotkey`、`MoveMouse`、`Launch`、`Windows`、`Cursor`、`Clipboard`、`SetWallpaper`、`AccessibilityTree`、`ViewerURL`。

## 智能体

`client.Agents` 仅提供设置和目录。此 SDK 不包含智能体定义的 CRUD。

```go
settings, err := client.Agents.Settings(ctx, workspaceID)
catalog, err := client.Agents.Harnesses(ctx)
```

| 接口面 | 方法 |
|---|---|
| `client.Agents` | `Settings`、`SetDefaultHarness`、`UpdateHarness`、`Models`、`Credentials`、`PutCredential`、`DeleteCredential`、`Harnesses`、`ChatContext`、`CompactChat` |
| `client.AgentAccounts` | `List`、`CreateAPIKey`、`Delete`、`StartSignin`、`GetSignin`、`SubmitSigninCode`、`CancelSignin` |
| `client.Runs` | `Run(ctx, RunCreateInput)`、`Usage(ctx, opts)` |
| `client.Sandboxes` | `QueueAgentPrompt`、`StopAgent` |

**此 SDK 尚未提供：** 创建、列出、更新和删除智能体定义。

## 错误

调用返回普通 Go 错误。使用 `errors.As` 检查它们：

```go
var apiErr *miosa.MiosaError
if errors.As(err, &apiErr) {
    if miosa.IsCode(err, "SANDBOX_STARTING") { /* retry */ }
    fmt.Println(apiErr.StatusCode, apiErr.Code, apiErr.RequestID)
}
```

`MiosaError` 包含 `StatusCode`、`Message`、`RequestID`、`Body`、`Code`、`Details`、`Retryable` 和 `RetryDelay`，以及 `ShouldRetry()`。类型化子类有：`AuthenticationError`（401）、`InsufficientCreditsError`（402）、`PermissionError`（403）、`NotFoundError`（404）、`ValidationError`（422）、`RateLimitError`（429，额外提供 `RetryAfter`）、`ServerError`（5xx）、`ConnectionError`。辅助函数：`AsMiosaError`、`ErrorCode`、`IsCode`、`IsStarting`、`RetryAfter`。
