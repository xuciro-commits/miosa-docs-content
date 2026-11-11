---
slug: "sdks/rust"
title: "Rust SDK"
description: "Install, authenticate and use the miosa Rust SDK 0.1.0 - sandboxes, exec, files, computers, agents and a typed error enum."
section: "CLI, SDKs & MCP"
order: 11020
---

The crate is named `miosa`, version 0.1.0. It targets Rust 1.85 or newer (edition 2021). The default `ws` feature adds WebSocket support for the terminal, port tunnels and the SSH tunnel; the `blocking` feature adds `BlockingMiosa`, which owns its own Tokio runtime.

## Install

```bash
cargo add miosa
```

```toml
[dependencies]
miosa = "0.1"
```

## Authentication

```rust
use miosa::Miosa;

let client = Miosa::new("msk_u_...")?;
let client = Miosa::from_env()?;
```

`Miosa::new(api_key)` and `Miosa::from_env()` read `MIOSA_API_KEY`, then `MIOSA_ACCESS_TOKEN`, `MIOSA_BASE_URL` and `MIOSA_TENANT`. Defaults: base URL `https://api.miosa.ai/api/v1`, timeout 60s, 3 retries.

The builder sets everything explicitly:

```rust
let client = Miosa::builder()
    .api_key("msk_u_...")
    .tenant("acme")
    .timeout(Duration::from_secs(45))
    .build()?;
```

Builder methods: `api_key`, `access_token`, `base_url`, `tenant`, `workspace`, `bill_to`, `user_agent`, `unbranded`, `timeout`, `max_retries`, `http_client`. Derived clients (which share the connection pool): `with_tenant`, `with_workspace`, `with_bill_to`, `as_user`, `anonymous`, `with_timeout`. Passing both an API key and an access token is an `Error::Config`.

## Sandboxes

```rust
use miosa::CreateSandbox;

let sbx = client.sandboxes().create(CreateSandbox::new().name("my-box")).await?;
let handle = client.sandboxes().handle(sbx.id.clone());
handle.wait_until_ready(Default::default()).await?;
```

| Method | What it does |
|---|---|
| `client.sandboxes().create(options)` | Create (mints an `Idempotency-Key`) |
| `client.sandboxes().create_detailed(options)` | Create, plus wait outcome |
| `client.sandboxes().get(id)` | Get one |
| `client.sandboxes().get_by_name(name)` | Get by name |
| `client.sandboxes().list(filter)` / `list_all` / `list_every` | List |
| `client.sandboxes().delete(id)` | Delete |
| `client.sandboxes().handle(id)` | A `SandboxHandle` for the sandbox |

`CreateSandbox` builder: `name`, `size`, `template_id`, `metadata`, `env`, `timeout_sec`, `idle_timeout_sec`, `persistent`, `always_on`, `environment`, `no_env`, `setup_file`, `wait`, `idempotency_key`. `SandboxHandle` adds `exec`, `pause`, `resume`, `stop`, `extend`, `destroy`, `usage`, `wait_until_ready`, `wait_until_idle`, `fork`, `logs`, `metrics`, `files`, and the snapshot methods.

## Exec and files

```rust
use miosa::ExecOptions;

let r = handle.exec("node -v", ExecOptions::new()).await?;
let mut stream = handle.exec_stream("npm run dev", ExecOptions::new()).await?;
```

`ExecOptions` builder: `cwd`, `env`, `timeout`. Computer exec: `computer.bash("ls -la", ExecOptions::new()).await?` and `computer.python(code, opts)`.

Files are reached through a machine-scoped `Files`:

```rust
let files = handle.files();
files.write("/workspace/app.rs", code.as_bytes()).await?;
let text = files.read("/workspace/app.rs").await?;
let list = files.list(Some("/workspace")).await?;
files.mkdir("/workspace/out", true, None).await?;
files.delete("/workspace/old").await?;
```

`Files` methods: `list`, `readdir`, `stat`, `download`, `read`, `write`, `write_many`, `upload`, `mkdir`, `rename`, `copy`, `chmod`, `delete`. `computer.files()` returns the same type, scoped to that computer.

## Snapshots

There is no top-level snapshots namespace. Snapshots are per-machine on the handles.

```rust
let snap = handle.create_snapshot(SnapshotOptions::default()).await?;
let snaps = handle.list_snapshots().await?;
handle.restore_snapshot(&snap.id).await?;
```

| Handle | Methods |
|---|---|
| `SandboxHandle` | `create_snapshot`, `list_snapshots`, `restore_snapshot`, `delete_snapshot`, `fork` |
| `ComputerHandle` | `create_snapshot`, `list_snapshots`, `restore_snapshot`, `delete_snapshot`, `duplicate(name)` |

**Not yet in this SDK:** account-wide snapshot history, named snapshots, and snapshot tree or download.

## Computers

```rust
let computer = client.computers().create(CreateComputer::default()).await?;
let handle = client.computers().handle(computer.id.clone());
handle.start().await?;
handle.wait_until_ready(Default::default()).await?;
```

| Method | What it does |
|---|---|
| `client.computers().create(options)` | Create |
| `client.computers().get(id)` | Get one |
| `client.computers().list(filter)` / `list_all` | List |
| `client.computers().delete(id)` | Delete |
| `client.computers().handle(id)` | A `ComputerHandle` |

`ComputerHandle` adds `start`, `stop`, `restart`, `resize`, `move_to`, `destroy`, `bash`, `python`, `files`, the snapshot methods, `viewer_password`, `rotate_viewer_password`, `urls`, `vnc_credentials`, `stream_token` and `embed`. Desktop: `screenshot`, `screenshot_region`, `click`, `double_click`, `drag`, `type_text`, `key`, `hotkey`, `key_down`, `key_up`, `scroll`, `move_cursor`, `launch`, `focus_window`, `close_window`, `maximize_window`, `minimize_window`, `set_wallpaper`, `windows`, `clipboard`, `set_clipboard`, `accessibility_tree`, `act(action)`, `look()`.

## Agents

Settings, catalog and triggers. Agent definition CRUD and run dispatch are not in this SDK.

```rust
let harnesses = client.agents().harnesses().await?;
let triggers = client.agents().triggers(&agent_id).await?;
let profile = client.agent_runtime().profiles().await?;
```

| Surface | Methods |
|---|---|
| `client.agents()` | `harnesses`, `harness_versions`, `templates`, `models`, `defaults`, `set_defaults`, `settings`, `set_default_harness`, `update_harness`, `credentials`, `put_credential`, `delete_credential`, `triggers`, `create_trigger`, `update_trigger`, `delete_trigger`, `rotate_trigger_secret`, `fire_trigger`, `chat_context`, `compact_chat` |
| `client.agent_runtime()` | `profiles`, `profile`, `create_profile`, `update_profile`, `delete_profile`, `start_signin`, `signin`, `submit_signin_code`, `cancel_signin` |

**Not yet in this SDK:** creating agents, running agents, and reading run outputs. Many `agents` methods return untyped `serde_json::Value`.

## Errors

The crate has one error type, a `#[non_exhaustive]` enum:

```rust
use miosa::Error;

match result {
    Ok(sbx) => println!("{}", sbx.id),
    Err(e) if e.is_retryable() => { /* back off */ }
    Err(e) => eprintln!("{} {} {}", e.kind_name(), e.status().unwrap_or(0), e.code().unwrap_or("")),
}
```

Variants: `Api(ApiError)`, `Transport`, `Decode`, `Config`, `ToolsetConfig`, `ClientRequired`, `Internal`. `ApiError` carries `kind`, `status`, `code`, `message`, `retryable`, `retry_after`, `request_id`, `details` and `body`, plus `is_code(&str)`. `Error` helpers: `status()`, `code()`, `is_code()`, `retry_after()`, `is_retryable()`, `request_id()`, `api()`, `is_not_found()`, `is_conflict()`, `is_timeout()`, `kind_name()`. Matches on `Error` must include a wildcard arm because the enum is non-exhaustive.

> **Note**
>
> POST and PATCH are only retried when the request carries an `Idempotency-Key`. `sandboxes().create` mints one automatically.
