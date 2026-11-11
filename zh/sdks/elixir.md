---
slug: "sdks/elixir"
title: "Elixir SDK"
description: "安装、验证并使用 miosa Elixir SDK 1.1.0——沙箱、exec、文件、快照、计算机、智能体和类型化错误。"
section: "CLI, SDKs & MCP"
order: 11018
---

MIOSA 官方 Elixir SDK。Hex 包 `miosa`，版本 1.1.0。需要 Elixir 1.15 或更高版本。每个资源都返回 `{:ok, value} | {:error, %Miosa.Error{}}`。

## 安装

```elixir
{:miosa, "~> 1.1"}
```

OTP 应用会启动一个共享的具名 Finch 连接池（`:miosa_finch_pool`），因此无需添加配置块。客户端也可以作为受监督的子进程启动：

```elixir
{Miosa.Client, api_key: System.fetch_env!("MIOSA_API_KEY")}
```

## 身份验证

```elixir
client = Miosa.client("msk_u_...")
client = Miosa.client_from_env()
```

`Miosa.client(api_key, opts)` 接受 `:base_url`（默认 `https://api.miosa.ai/api/v1`）、`:timeout`（默认 30 秒）、`:receive_timeout`（默认 60 秒）、`:retry`（默认 `false`）、`:tenant`（作为 `X-MIOSA-Tenant` 发送）、`:user_agent` 和 `:app_info`。当凭据不是 `msk_`、`mat_` 或 JWT 形式时，构造函数会抛出 `ArgumentError`。

`Miosa.client_from_env()` 会读取 `MIOSA_API_KEY`，然后是 `MIOSA_ACCESS_TOKEN`、`MIOSA_BASE_URL` 和 `MIOSA_TENANT`。`Miosa.Client.with_tenant(client, tenant)` 返回一个在其他组织中操作的客户端。

> **注意**
>
> 此 SDK 中重试默认为 `false`，这一点与 Java 不同——Java 默认重试三次。

## 沙箱

沙箱是一台带 `template_id: "miosa-sandbox"` 的计算机，通过 `Miosa.Sandboxes` 创建。

```elixir
{:ok, sandbox} = Miosa.Sandboxes.create(client, %{name: "my-box"})
{:ok, _} = Miosa.Sandboxes.wait_until_ready(client, sandbox["id"])
```

| 函数 | 作用 |
|---|---|
| `Miosa.Sandboxes.create(client, params, opts \\ [])` | 创建（`:wait`、`:bill_to`、`:idempotency_key`） |
| `Miosa.Sandboxes.create_with_meta(client, params, opts)` | 创建，并附带等待和账单归属元数据 |
| `Miosa.Sandboxes.list(client, opts)` / `list_page/2` | 列出 |
| `Miosa.Sandboxes.get(client, id)` | 获取单个 |
| `Miosa.Sandboxes.delete(client, id)` | 删除 |
| `Miosa.Sandboxes.extend/3`, `stop/2`, `pause/2`, `resume/2`, `fork/3` | 生命周期 |
| `Miosa.Sandboxes.wait_until_ready(client, id, opts)` | 等待直至就绪 |

## Exec 与文件

```elixir
{:ok, %{"stdout" => out}} = Miosa.Sandbox.Exec.run(client, sandbox_id, "node -v")
:ok = Miosa.Sandbox.Exec.stream(client, sandbox_id, "npm run dev", fn ev -> IO.inspect(ev) end)
```

计算机 exec 使用不同的模块：

```elixir
{:ok, _} = Miosa.Exec.bash(client, computer_id, "ls -la")
{:ok, _} = Miosa.Exec.python(client, computer_id, "print(2 + 2)")
{:ok, cmd} = Miosa.Exec.spawn(client, computer_id, "bash")
```

文件：计算机文件位于 `Miosa.Files`；沙箱文件位于 `Miosa.Sandbox.Fs` 和 `Miosa.Sandbox.Files`。

```elixir
:ok = Miosa.Files.write(client, computer_id, "/home/user/notes.txt", "hi")
{:ok, data} = Miosa.Files.download(client, computer_id, "/home/user/notes.txt")
{:ok, entries} = Miosa.Files.list(client, computer_id, "/home/user")

{:ok, _} = Miosa.Sandbox.Fs.write(client, sandbox_id, "/workspace/app.ex", code)
{:ok, _} = Miosa.Sandbox.Fs.read(client, sandbox_id, "/workspace/app.ex")
{:ok, _} = Miosa.Sandbox.Files.tree(client, sandbox_id, "/workspace")
```

`Miosa.Files` 的方法：`upload`、`download`、`download_to`、`list`、`write`、`export`、`delete`，以及 `mkdir`、`rename`、`copy`、`chmod`、`stat`、`readdir`。

## 快照

```elixir
{:ok, cp} = Miosa.Checkpoints.create(client, sandbox_id, %{comment: "before deps"})
{:ok, snaps} = Miosa.Snapshots.list(client, limit: 50)
{:ok, _} = Miosa.NamedSnapshots.create(client, %{name: "before-upgrade"})
```

| 接口 | 函数 |
|---|---|
| `Miosa.Checkpoints` | `create`、`list`、`get`、`delete`、`restore`、`restore_stream` |
| `Miosa.Sandboxes` | `create_snapshot`、`list_snapshots`、`restore_snapshot`、`delete_snapshot` |
| `Miosa.Snapshots` | `list`、`groups`、`get`、`tree`、`wait_tree`、`download`、`fork`、`delete`、`delete_many`、`delete_history` |
| `Miosa.NamedSnapshots` | `list`、`create`、`get`、`delete`、`deploy` |

## 计算机

```elixir
{:ok, computer} = Miosa.Computers.create(client, %{name: "dev-box"})
{:ok, _} = Miosa.Computer.start(client, computer["id"])
```

| 函数 | 作用 |
|---|---|
| `Miosa.Computers.create(client, params)` | 创建（`:template_type` 默认 `miosa-desktop`，以及 `:size`） |
| `Miosa.Computers.list(client)` | 列出 |
| `Miosa.Computers.get(client, id)` | 获取单个 |
| `Miosa.Computers.delete(client, id, opts)` | 删除（对运行中的计算机使用 `force: true`） |
| `Miosa.Computers.update(client, id, changes)` | 更新 |

`Miosa.Computer` 句柄增加了 `start`、`stop`、`restart`、`destroy`、`wait_until_running`、`wait_until_stopped`、`clone`、`resize`、`move`、`preview_url`、`public_url`、`urls`、`stream_token`。桌面控制位于 `Miosa.Desktop`：`screenshot`、`click`、`double_click`、`right_click`、`drag`、`type`、`key`、`hotkey`、`scroll`、`move`、`windows`、`cursor`、`focus_window`、`launch`、`accessibility_tree`、`get_clipboard`、`set_clipboard`、`screenshot_region`，以及 `window_*` 辅助函数。

## 智能体

此 SDK 提供完整的智能体定义 CRUD。

```elixir
{:ok, agents} = Miosa.Agents.list(client)
{:ok, agent} = Miosa.Agents.create(client, %{name: "reviewer"})
{:ok, _} = Miosa.Agents.duplicate(client, agent["id"], %{name: "reviewer-2"})
```

| 接口 | 函数 |
|---|---|
| `Miosa.Agents` | `list`、`create`、`get`、`update`、`delete`、`restore`、`duplicate`、`list_runs`、`list_harnesses`、`list_harness_versions`、`list_templates`、`list_defaults`、`put_default` |
| `Miosa.Agents.Settings` / `Credentials` / `Models` / `Signins` / `Chats` / `Triggers` / `Versions` | 子命名空间 |
| `Miosa.Runs` | 运行分发与读取 |
| `Miosa.Connections` | 模型提供商和工具连接 |

## 错误

```elixir
case Miosa.Sandboxes.get(client, id) do
  {:ok, sandbox} -> sandbox
  {:error, %Miosa.Error{} = err} ->
    if Miosa.Error.retryable?(err), do: :retry
    {err.status, err.code, err.message}
end
```

`Miosa.Error` 是一个 `defexception`，包含 `:message`、`:status`、`:code`、`:body`、`:details`、`:retryable`、`:retry_after_ms` 和 `:request_id`。使用 `Miosa.Error.kind/1`（`:unauthorized`、`:insufficient_credits`、`:forbidden`、`:not_found`、`:conflict`、`:validation`、`:rate_limited`、`:server`、`:client`、`:network`）、`retryable?/1`、`starting?/1` 和 `dependents/1` 进行分类。bang 辅助函数 `Miosa.Exec.bash!/4` 和 `python!/4` 会抛出异常，而不是返回错误元组。

> **注意**
>
> `Miosa.Error` 会解析每个服务器信封，包括规范的 `{"error": {...}}` 映射形式和 409 批量删除的 `data.blocked` 情况。
