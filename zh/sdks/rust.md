---
slug: "sdks/rust"
title: "Rust SDK"
description: "安装、认证并使用 miosa Rust SDK 0.1.0——沙箱、exec、文件、计算机、智能体和类型化错误枚举。"
section: "CLI, SDKs & MCP"
order: 11020
---

该 crate 名为 `miosa`，版本 0.1.0。它面向 Rust 1.85 或更高版本（edition 2021）。默认的 `ws` feature 为终端、端口隧道和 SSH 隧道添加 WebSocket 支持；`blocking` feature 添加 `BlockingMiosa`，它拥有自己的 Tokio 运行时。

## 安装

```bash
cargo add miosa
```

```toml
[dependencies]
miosa = "0.1"
```

## 身份验证

```rust
use miosa::Miosa;

let client = Miosa::new("msk_u_...")?;
let client = Miosa::from_env()?;
```

`Miosa::new(api_key)` 和 `Miosa::from_env()` 会依次读取 `MIOSA_API_KEY`、`MIOSA_ACCESS_TOKEN`、`MIOSA_BASE_URL` 和 `MIOSA_TENANT`。默认值：基础 URL 为 `https://api.miosa.ai/api/v1`，超时 60 秒，重试 3 次。

构建器会显式设置所有内容：

```rust
let client = Miosa::builder()
    .api_key("msk_u_...")
    .tenant("acme")
    .timeout(Duration::from_secs(45))
    .build()?;
```

构建器方法：`api_key`、`access_token`、`base_url`、`tenant`、`workspace`、`bill_to`、`user_agent`、`unbranded`、`timeout`、`max_retries`、`http_client`。派生客户端（共享连接池）：`with_tenant`、`with_workspace`、`with_bill_to`、`as_user`、`anonymous`、`with_timeout`。同时传入 API 密钥和访问令牌会返回 `Error::Config`。

## 沙箱

```rust
use miosa::CreateSandbox;

let sbx = client.sandboxes().create(CreateSandbox::new().name("my-box")).await?;
let handle = client.sandboxes().handle(sbx.id.clone());
handle.wait_until_ready(Default::default()).await?;
```

| 方法 | 作用 |
|---|---|
| `client.sandboxes().create(options)` | 创建（生成 `Idempotency-Key`） |
| `client.sandboxes().create_detailed(options)` | 创建，并返回等待结果 |
| `client.sandboxes().get(id)` | 获取单个 |
| `client.sandboxes().get_by_name(name)` | 按名称获取 |
| `client.sandboxes().list(filter)` / `list_all` / `list_every` | 列出 |
| `client.sandboxes().delete(id)` | 删除 |
| `client.sandboxes().handle(id)` | 用于该沙箱的 `SandboxHandle` |

`CreateSandbox` 构建器：`name`、`size`、`template_id`、`metadata`、`env`、`timeout_sec`、`idle_timeout_sec`、`persistent`、`always_on`、`environment`、`no_env`、`setup_file`、`wait`、`idempotency_key`。`SandboxHandle` 额外提供 `exec`、`pause`、`resume`、`stop`、`extend`、`destroy`、`usage`、`wait_until_ready`、`wait_until_idle`、`fork`、`logs`、`metrics`、`files`，以及快照方法。

## Exec 与文件

```rust
use miosa::ExecOptions;

let r = handle.exec("node -v", ExecOptions::new()).await?;
let mut stream = handle.exec_stream("npm run dev", ExecOptions::new()).await?;
```

`ExecOptions` 构建器：`cwd`、`env`、`timeout`。计算机 exec：`computer.bash("ls -la", ExecOptions::new()).await?` 和 `computer.python(code, opts)`。

文件通过机器作用域的 `Files` 访问：

```rust
let files = handle.files();
files.write("/workspace/app.rs", code.as_bytes()).await?;
let text = files.read("/workspace/app.rs").await?;
let list = files.list(Some("/workspace")).await?;
files.mkdir("/workspace/out", true, None).await?;
files.delete("/workspace/old").await?;
```

`Files` 方法：`list`、`readdir`、`stat`、`download`、`read`、`write`、`write_many`、`upload`、`mkdir`、`rename`、`copy`、`chmod`、`delete`。`computer.files()` 返回相同类型，作用域限定在该计算机。

## 快照

没有顶层快照命名空间。快照按机器挂在各个句柄上。

```rust
let snap = handle.create_snapshot(SnapshotOptions::default()).await?;
let snaps = handle.list_snapshots().await?;
handle.restore_snapshot(&snap.id).await?;
```

| 句柄 | 方法 |
|---|---|
| `SandboxHandle` | `create_snapshot`、`list_snapshots`、`restore_snapshot`、`delete_snapshot`、`fork` |
| `ComputerHandle` | `create_snapshot`、`list_snapshots`、`restore_snapshot`、`delete_snapshot`、`duplicate(name)` |

**本 SDK 尚未提供：** 账户级快照历史、命名快照，以及快照树或下载。

## 计算机

```rust
let computer = client.computers().create(CreateComputer::default()).await?;
let handle = client.computers().handle(computer.id.clone());
handle.start().await?;
handle.wait_until_ready(Default::default()).await?;
```

| 方法 | 作用 |
|---|---|
| `client.computers().create(options)` | 创建 |
| `client.computers().get(id)` | 获取单个 |
| `client.computers().list(filter)` / `list_all` | 列出 |
| `client.computers().delete(id)` | 删除 |
| `client.computers().handle(id)` | 返回一个 `ComputerHandle` |

`ComputerHandle` 额外提供 `start`、`stop`、`restart`、`resize`、`move_to`、`destroy`、`bash`、`python`、`files`、快照方法、`viewer_password`、`rotate_viewer_password`、`urls`、`vnc_credentials`、`stream_token` 和 `embed`。桌面操作：`screenshot`、`screenshot_region`、`click`、`double_click`、`drag`、`type_text`、`key`、`hotkey`、`key_down`、`key_up`、`scroll`、`move_cursor`、`launch`、`focus_window`、`close_window`、`maximize_window`、`minimize_window`、`set_wallpaper`、`windows`、`clipboard`、`set_clipboard`、`accessibility_tree`、`act(action)`、`look()`。

## 智能体

设置、目录和触发器。智能体定义的增删改查和运行分发不在本 SDK 中。

```rust
let harnesses = client.agents().harnesses().await?;
let triggers = client.agents().triggers(&agent_id).await?;
let profile = client.agent_runtime().profiles().await?;
```

| 接口 | 方法 |
|---|---|
| `client.agents()` | `harnesses`、`harness_versions`、`templates`、`models`、`defaults`、`set_defaults`、`settings`、`set_default_harness`、`update_harness`、`credentials`、`put_credential`、`delete_credential`、`triggers`、`create_trigger`、`update_trigger`、`delete_trigger`、`rotate_trigger_secret`、`fire_trigger`、`chat_context`、`compact_chat` |
| `client.agent_runtime()` | `profiles`、`profile`、`create_profile`、`update_profile`、`delete_profile`、`start_signin`、`signin`、`submit_signin_code`、`cancel_signin` |

**本 SDK 尚未提供：** 创建智能体、运行智能体和读取运行输出。许多 `agents` 方法返回未类型化的 `serde_json::Value`。

## 错误

该 crate 只有一个错误类型，即一个 `#[non_exhaustive]` 枚举：

```rust
use miosa::Error;

match result {
    Ok(sbx) => println!("{}", sbx.id),
    Err(e) if e.is_retryable() => { /* back off */ }
    Err(e) => eprintln!("{} {} {}", e.kind_name(), e.status().unwrap_or(0), e.code().unwrap_or("")),
}
```

变体：`Api(ApiError)`、`Transport`、`Decode`、`Config`、`ToolsetConfig`、`ClientRequired`、`Internal`。`ApiError` 包含 `kind`、`status`、`code`、`message`、`retryable`、`retry_after`、`request_id`、`details` 和 `body`，另有 `is_code(&str)`。`Error` 辅助方法：`status()`、`code()`、`is_code()`、`retry_after()`、`is_retryable()`、`request_id()`、`api()`、`is_not_found()`、`is_conflict()`、`is_timeout()`、`kind_name()`。对 `Error` 进行匹配时必须包含通配分支，因为该枚举是非穷尽的。

> **注意**
>
> 仅当请求带有 `Idempotency-Key` 时，POST 和 PATCH 才会重试。`sandboxes().create` 会自动生成一个。
