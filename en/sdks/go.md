---
slug: "sdks/go"
title: "Go SDK"
description: "Install, authenticate and use the miosa-go v2 SDK - sandboxes, exec, files, snapshots, computers, agent runs and typed errors."
section: "CLI, SDKs & MCP"
order: 11017
---

The official Go SDK for MIOSA. Module path `github.com/Miosa-osa/miosa-go/v2`, version 2.2.0. Requires Go 1.21 or newer.

## Install

```bash
go get github.com/Miosa-osa/miosa-go/v2
```

```go
import miosa "github.com/Miosa-osa/miosa-go/v2"
```

Subpackages: `github.com/Miosa-osa/miosa-go/v2/sandboxssh` and `github.com/Miosa-osa/miosa-go/v2/computeruse`.

## Authentication

```go
client := miosa.NewClient("msk_u_...")
```

`NewClient(apiKey, opts...)` falls back to `MIOSA_API_KEY` when the key is empty. It also reads `MIOSA_BASE_URL`, `MIOSA_ACCESS_TOKEN` and `MIOSA_TENANT`. Defaults: base URL `https://api.miosa.ai/api/v1`, timeout 60s, 3 retries.

Options: `WithBaseURL`, `WithHTTPClient`, `WithTimeout`, `WithMaxRetries`, `WithTenant`, `WithAccessToken`, `WithBillTo`, `WithDefaultHeader`, `WithUserAgent`. Derive a scoped client with `client.With(opts...)`, `client.AsUser(token)` or `client.ForTenant(id)`.

```go
ctx := miosa.WithBillToContext(context.Background(), "acme")
```

## Sandboxes

Sandbox calls live on `client.Sandboxes`. The `*Sandbox` value is data, not a handle: it has no methods.

```go
sbx, err := client.Sandboxes.Create(ctx, miosa.CreateSandboxInput{Name: "my-box"})
ok, err := client.Sandboxes.WaitUntilReady(ctx, sbx.ID, miosa.WaitUntilReadyOptions{})
```

| Method | What it does |
|---|---|
| `client.Sandboxes.Create(ctx, input)` | Create a sandbox |
| `client.Sandboxes.CreateDetailed(ctx, input)` | Create, plus wait outcome and billed org |
| `client.Sandboxes.Get(ctx, id)` | Get one |
| `client.Sandboxes.GetByName(ctx, name)` | Get by name |
| `client.Sandboxes.List(ctx, input)` / `ListPage` / `ListAll` | List (paged or all) |
| `client.Sandboxes.Delete(ctx, id)` | Delete |
| `client.Sandboxes.Update(ctx, id, in)` | Update |
| `client.Sandboxes.Stop/Pause/Resume/Extend` | Lifecycle |
| `client.Sandboxes.Usage(ctx, id)` | Usage |
| `client.Sandboxes.WaitUntilReady(ctx, id, opts)` | Wait until ready |

> **Note**
>
> The `*Sandbox` value has fields such as `ID` and `PreviewURL` but no methods. Call exec, destroy and readiness on `client.Sandboxes` with the sandbox id.

## Exec and files

```go
res, err := client.Sandboxes.Exec(ctx, sbx.ID, miosa.ExecSandboxInput{Command: "node -v"})
events, err := client.Sandboxes.ExecStream(ctx, sbx.ID, miosa.ExecSandboxInput{Command: "npm run dev"})
```

Computer exec uses a WebSocket service:

```go
out, err := computer.Exec.Bash(ctx, "ls -la")
out, err = computer.Exec.Python(ctx, "print(2 + 2)")
```

Files are computer-scoped. Scope an unscoped client with `For`:

```go
files := client.Files.For(computer.ID)
_ = files.Upload(ctx, "./notes.txt", "/home/user/notes.txt")
data, _ := files.Download(ctx, "/home/user/notes.txt")
list, _ := files.List(ctx, "/home/user")
_ = files.Mkdir(ctx, "/home/user/data")
_ = files.Rename(ctx, "/home/user/a", "/home/user/b")
```

`client.Files` and `computer.Files` are the same service; `computer.Files` is pre-scoped. Methods: `Upload`, `UploadReader`, `Download`, `DownloadTo`, `List`, `Stat`, `Mkdir`, `Rename`, `Copy`, `Chmod`, `Readdir`, `Delete`, `Export`. `FS(workingDir)` returns a path-scoped `ScopedFS`.

## Snapshots

Five surfaces.

```go
snap, err := client.Sandboxes.CreateSnapshot(ctx, sbx.ID, miosa.CreateSandboxSnapshotInput{})
snaps, err := client.Sandboxes.ListSnapshots(ctx, sbx.ID)
_, err = client.Sandboxes.RestoreSnapshot(ctx, sbx.ID, snap.ID)
```

| Surface | Methods |
|---|---|
| `client.Sandboxes` | `CreateSnapshot`, `ListSnapshots`, `RestoreSnapshot`, `DeleteSnapshot` |
| `computer.Snapshots` | `Create`, `List`, `Get`, `Delete`, `Events` |
| `client.Snapshots` | `List`, `Groups`, `Get`, `Delete`, `DeleteMany`, `Fork`, `Tree`, `Download` |
| `client.NamedSnapshots` | `List`, `Get`, `Save`, `Remove`, `Deploy` |
| `client.SnapshotsStandalone` | `List`, `Get` (untyped data) |

## Computers

```go
computer, err := client.Computers.Create(ctx, miosa.CreateComputerInput{Name: "dev-box"})
err = computer.Start(ctx)
```

| Method | What it does |
|---|---|
| `client.Computers.Create(ctx, input)` | Create |
| `client.Computers.List(ctx, input)` | List |
| `client.Computers.Get(ctx, id)` | Get one |
| `client.Computers.Delete(ctx, id)` | Delete |

The `Computer` handle adds `Start`, `Stop`, `Restart`, `Destroy`, `Wait`, `Bash`, `Python`, `PreviewURL`, `PublicURL`, `Clone`, `Resize`, `Move`, `VncCredentials`, `StreamToken`. Desktop: `Screenshot` (bytes), `Click`, `LeftClick`, `DoubleClick`, `RightClick`, `Drag`, `Scroll`, `Type`, `Key`, `Hotkey`, `MoveMouse`, `Launch`, `Windows`, `Cursor`, `Clipboard`, `SetWallpaper`, `AccessibilityTree`, `ViewerURL`.

## Agents

`client.Agents` is settings and catalog only. There is no agent definition CRUD in this SDK.

```go
settings, err := client.Agents.Settings(ctx, workspaceID)
catalog, err := client.Agents.Harnesses(ctx)
```

| Surface | Methods |
|---|---|
| `client.Agents` | `Settings`, `SetDefaultHarness`, `UpdateHarness`, `Models`, `Credentials`, `PutCredential`, `DeleteCredential`, `Harnesses`, `ChatContext`, `CompactChat` |
| `client.AgentAccounts` | `List`, `CreateAPIKey`, `Delete`, `StartSignin`, `GetSignin`, `SubmitSigninCode`, `CancelSignin` |
| `client.Runs` | `Run(ctx, RunCreateInput)`, `Usage(ctx, opts)` |
| `client.Sandboxes` | `QueueAgentPrompt`, `StopAgent` |

**Not yet in this SDK:** creating, listing, updating and deleting agent definitions.

## Errors

Calls return ordinary Go errors. Inspect them with `errors.As`:

```go
var apiErr *miosa.MiosaError
if errors.As(err, &apiErr) {
    if miosa.IsCode(err, "SANDBOX_STARTING") { /* retry */ }
    fmt.Println(apiErr.StatusCode, apiErr.Code, apiErr.RequestID)
}
```

`MiosaError` has `StatusCode`, `Message`, `RequestID`, `Body`, `Code`, `Details`, `Retryable` and `RetryDelay`, plus `ShouldRetry()`. Typed subclasses: `AuthenticationError` (401), `InsufficientCreditsError` (402), `PermissionError` (403), `NotFoundError` (404), `ValidationError` (422), `RateLimitError` (429, adds `RetryAfter`), `ServerError` (5xx), `ConnectionError`. Helpers: `AsMiosaError`, `ErrorCode`, `IsCode`, `IsStarting`, `RetryAfter`.
