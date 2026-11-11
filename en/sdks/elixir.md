---
slug: "sdks/elixir"
title: "Elixir SDK"
description: "Install, authenticate and use the miosa Elixir SDK 1.1.0 - sandboxes, exec, files, snapshots, computers, agents and typed errors."
section: "CLI, SDKs & MCP"
order: 11018
---

The official Elixir SDK for MIOSA. Hex package `miosa`, version 1.1.0. Requires Elixir 1.15 or newer. Every resource returns `{:ok, value} | {:error, %Miosa.Error{}}`.

## Install

```elixir
{:miosa, "~> 1.1"}
```

The OTP application starts a shared named Finch pool (`:miosa_finch_pool`), so there is no config block to add. A client can also be started as a supervised child:

```elixir
{Miosa.Client, api_key: System.fetch_env!("MIOSA_API_KEY")}
```

## Authentication

```elixir
client = Miosa.client("msk_u_...")
client = Miosa.client_from_env()
```

`Miosa.client(api_key, opts)` takes `:base_url` (default `https://api.miosa.ai/api/v1`), `:timeout` (default 30s), `:receive_timeout` (default 60s), `:retry` (default `false`), `:tenant` (sent as `X-MIOSA-Tenant`), `:user_agent` and `:app_info`. The constructor raises `ArgumentError` when the credential is not `msk_`, `mat_` or JWT shaped.

`Miosa.client_from_env()` reads `MIOSA_API_KEY`, then `MIOSA_ACCESS_TOKEN`, `MIOSA_BASE_URL` and `MIOSA_TENANT`. `Miosa.Client.with_tenant(client, tenant)` returns a client acting in another organization.

> **Note**
>
> Retry defaults to `false` in this SDK, unlike Java, which retries three times by default.

## Sandboxes

A sandbox is a computer with `template_id: "miosa-sandbox"`, created through `Miosa.Sandboxes`.

```elixir
{:ok, sandbox} = Miosa.Sandboxes.create(client, %{name: "my-box"})
{:ok, _} = Miosa.Sandboxes.wait_until_ready(client, sandbox["id"])
```

| Function | What it does |
|---|---|
| `Miosa.Sandboxes.create(client, params, opts \\ [])` | Create (`:wait`, `:bill_to`, `:idempotency_key`) |
| `Miosa.Sandboxes.create_with_meta(client, params, opts)` | Create, plus wait and bill-to meta |
| `Miosa.Sandboxes.list(client, opts)` / `list_page/2` | List |
| `Miosa.Sandboxes.get(client, id)` | Get one |
| `Miosa.Sandboxes.delete(client, id)` | Delete |
| `Miosa.Sandboxes.extend/3`, `stop/2`, `pause/2`, `resume/2`, `fork/3` | Lifecycle |
| `Miosa.Sandboxes.wait_until_ready(client, id, opts)` | Wait until ready |

## Exec and files

```elixir
{:ok, %{"stdout" => out}} = Miosa.Sandbox.Exec.run(client, sandbox_id, "node -v")
:ok = Miosa.Sandbox.Exec.stream(client, sandbox_id, "npm run dev", fn ev -> IO.inspect(ev) end)
```

Computer exec is a different module:

```elixir
{:ok, _} = Miosa.Exec.bash(client, computer_id, "ls -la")
{:ok, _} = Miosa.Exec.python(client, computer_id, "print(2 + 2)")
{:ok, cmd} = Miosa.Exec.spawn(client, computer_id, "bash")
```

Files: computer files live in `Miosa.Files`; sandbox files live in `Miosa.Sandbox.Fs` and `Miosa.Sandbox.Files`.

```elixir
:ok = Miosa.Files.write(client, computer_id, "/home/user/notes.txt", "hi")
{:ok, data} = Miosa.Files.download(client, computer_id, "/home/user/notes.txt")
{:ok, entries} = Miosa.Files.list(client, computer_id, "/home/user")

{:ok, _} = Miosa.Sandbox.Fs.write(client, sandbox_id, "/workspace/app.ex", code)
{:ok, _} = Miosa.Sandbox.Fs.read(client, sandbox_id, "/workspace/app.ex")
{:ok, _} = Miosa.Sandbox.Files.tree(client, sandbox_id, "/workspace")
```

`Miosa.Files` methods: `upload`, `download`, `download_to`, `list`, `write`, `export`, `delete`, plus `mkdir`, `rename`, `copy`, `chmod`, `stat`, `readdir`.

## Snapshots

```elixir
{:ok, cp} = Miosa.Checkpoints.create(client, sandbox_id, %{comment: "before deps"})
{:ok, snaps} = Miosa.Snapshots.list(client, limit: 50)
{:ok, _} = Miosa.NamedSnapshots.create(client, %{name: "before-upgrade"})
```

| Surface | Functions |
|---|---|
| `Miosa.Checkpoints` | `create`, `list`, `get`, `delete`, `restore`, `restore_stream` |
| `Miosa.Sandboxes` | `create_snapshot`, `list_snapshots`, `restore_snapshot`, `delete_snapshot` |
| `Miosa.Snapshots` | `list`, `groups`, `get`, `tree`, `wait_tree`, `download`, `fork`, `delete`, `delete_many`, `delete_history` |
| `Miosa.NamedSnapshots` | `list`, `create`, `get`, `delete`, `deploy` |

## Computers

```elixir
{:ok, computer} = Miosa.Computers.create(client, %{name: "dev-box"})
{:ok, _} = Miosa.Computer.start(client, computer["id"])
```

| Function | What it does |
|---|---|
| `Miosa.Computers.create(client, params)` | Create (`:template_type` default `miosa-desktop`, `:size`) |
| `Miosa.Computers.list(client)` | List |
| `Miosa.Computers.get(client, id)` | Get one |
| `Miosa.Computers.delete(client, id, opts)` | Delete (`force: true` for a running computer) |
| `Miosa.Computers.update(client, id, changes)` | Update |

The `Miosa.Computer` handle adds `start`, `stop`, `restart`, `destroy`, `wait_until_running`, `wait_until_stopped`, `clone`, `resize`, `move`, `preview_url`, `public_url`, `urls`, `stream_token`. Desktop control is in `Miosa.Desktop`: `screenshot`, `click`, `double_click`, `right_click`, `drag`, `type`, `key`, `hotkey`, `scroll`, `move`, `windows`, `cursor`, `focus_window`, `launch`, `accessibility_tree`, `get_clipboard`, `set_clipboard`, `screenshot_region`, and the `window_*` helpers.

## Agents

This SDK has full agent definition CRUD.

```elixir
{:ok, agents} = Miosa.Agents.list(client)
{:ok, agent} = Miosa.Agents.create(client, %{name: "reviewer"})
{:ok, _} = Miosa.Agents.duplicate(client, agent["id"], %{name: "reviewer-2"})
```

| Surface | Functions |
|---|---|
| `Miosa.Agents` | `list`, `create`, `get`, `update`, `delete`, `restore`, `duplicate`, `list_runs`, `list_harnesses`, `list_harness_versions`, `list_templates`, `list_defaults`, `put_default` |
| `Miosa.Agents.Settings` / `Credentials` / `Models` / `Signins` / `Chats` / `Triggers` / `Versions` | Sub-namespaces |
| `Miosa.Runs` | Run dispatch and reads |
| `Miosa.Connections` | Model provider and tool connections |

## Errors

```elixir
case Miosa.Sandboxes.get(client, id) do
  {:ok, sandbox} -> sandbox
  {:error, %Miosa.Error{} = err} ->
    if Miosa.Error.retryable?(err), do: :retry
    {err.status, err.code, err.message}
end
```

`Miosa.Error` is a `defexception` with `:message`, `:status`, `:code`, `:body`, `:details`, `:retryable`, `:retry_after_ms` and `:request_id`. Classify with `Miosa.Error.kind/1` (`:unauthorized`, `:insufficient_credits`, `:forbidden`, `:not_found`, `:conflict`, `:validation`, `:rate_limited`, `:server`, `:client`, `:network`), `retryable?/1`, `starting?/1` and `dependents/1`. The bang helpers `Miosa.Exec.bash!/4` and `python!/4` raise instead of returning an error tuple.

> **Note**
>
> `Miosa.Error` parses every server envelope, including the canonical `{"error": {...}}` map form and the 409 bulk-delete `data.blocked` case.
