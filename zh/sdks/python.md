---
slug: "sdks/python"
title: "Python SDK"
description: "安装、认证并使用 miosa Python SDK 2.16.5——同步与异步、沙箱、exec、文件、快照、计算机、智能体和类型化错误。"
section: "CLI, SDKs & MCP"
order: 11016
---

MIOSA 官方 Python SDK。每个资源都有同步类和异步类：构造时选择 `Miosa` 或 `AsyncMiosa`。需要 Python 3.9 或更高版本。

## 安装

```bash
pip install miosa
```

```python
from miosa import Miosa, AsyncMiosa, Sandbox, Computer, Forge
```

可选依赖：`miosa[events]`（websockets）、`miosa[http2]`、`miosa[dev]`。智能体运行时是独立模块：`from miosa.agent import Agent`。

## 身份验证

```python
from miosa import Miosa

client = Miosa(api_key="msk_u_...")           # sync
async_client = AsyncMiosa(api_key="msk_u_...") # async
```

`api_key` 和 `access_token` 互斥：同时传入会引发 `ValueError`。密钥解析顺序为 `access_token`、`api_key`、`MIOSA_ACCESS_TOKEN`、`MIOSA_API_KEY`。其他选项：`tenant`（`MIOSA_TENANT`）、`base_url`（`MIOSA_BASE_URL`，默认 `https://api.miosa.ai/api/v1`）、`timeout`（默认 60 秒）、`max_retries`（默认 3）、`unbranded`。

两个客户端都是上下文管理器。`client.with_tenant(tenant)` 返回一个在其他组织中操作的客户端。使用 `client.get_balance()`、`client.get_usage()` 和 `client.get_transactions()` 读取账户信息。

## 沙箱

```python
sbx = client.sandboxes.create(name="my-box", size="small")
sbx.wait_until_ready()
```

| 方法 | 作用 |
|---|---|
| `client.sandboxes.create(template_id=None, *, name=None, size=None, ...)` | 创建沙箱 |
| `client.sandboxes.list(*, state=None, tags=None, ...)` | 列出沙箱 |
| `client.sandboxes.get(sandbox_id)` | 获取单个沙箱 |
| `client.sandboxes.get_by_name(name)` | 按名称获取 |
| `client.sandboxes.get_or_create(name, *, resume=True, ...)` | 复用或创建 |
| `client.sandboxes.delete(sandbox_id)` | 删除 |

在句柄上，`sandbox.stop()` 会停止沙箱（保留磁盘，不保留进程），`sandbox.resume()` 会将其恢复；`sandbox.destroy()` 会删除沙箱（`delete` 是别名）。规格为 `xs`、`small`、`medium`、`large`、`xl`。

> **注意**
>
> `Sandbox` 和 `Computer` 会在 `__exit__` 时销毁自身。`with client.sandboxes.create(...) as sbx:` 代码块会在退出时删除沙箱。

## Exec 与文件

```python
r = sbx.exec.run("node -v")      # ExecResult(stdout, stderr, exit_code, duration_ms)
sbx.exec("npm test")             # the exec runner is callable
sbx.run("npm test")              # delegates to exec.run
for ev in sbx.exec.stream("npm run dev"):
    print(ev)
sbx.commands.run("ls -la")       # richer: background processes, pid
```

计算机的 exec 是独立的：

```python
computer.bash("ls -la")
computer.python("print(2 + 2)")
```

沙箱和计算机的文件接口不同。沙箱文件没有 `mkdir` 或 `rm`：

```python
sbx.files.write("/workspace/app.py", code)
sbx.files.read_text("/workspace/app.py")
sbx.files.list("/workspace")
# directory create/remove: use exec
sbx.exec.run("mkdir -p /workspace/out")

computer.files.write_file("/home/user/notes.txt", "hi")
computer.files.mkdir("/home/user/data", recursive=True)
computer.files.delete("/home/user/old")
```

沙箱文件方法：`list`、`read`、`read_text`、`write`、`write_many`、`upload`、`download`、`stat`、`tree`、`watch`。句柄上的扁平封装：`write_file`、`read_file`、`list_files`、`stat_file`、`upload`、`download`。

## 快照

```python
sbx.snapshots.create(comment="before deps")
sbx.snapshots.list()
sbx.snapshots.restore(snapshot_id)
sbx.create_snapshot()            # convenience
```

| 接口 | 方法 |
|---|---|
| `sbx.snapshots` | `create`、`list`、`restore`、`delete` |
| `client.snapshots` | `list`、`get` |
| `client.snapshot_history` | `list`、`groups`、`get`、`delete`、`bulk_delete`、`tree`、`file`、`fork` |
| `client.named_snapshots` | `list`、`create`、`get`、`delete`、`deploy` |

## 计算机

```python
computer = client.computers.create(name="dev-box", size="small")
computer.start()
computer.wait_until_ready()
```

| 方法 | 作用 |
|---|---|
| `client.computers.create(name, *, template_type="miosa-desktop", size="small", ...)` | 创建 |
| `client.computers.list(*, workspace_id=None)` | 列出 |
| `client.computers.get(computer_id)` | 获取单个 |
| `client.computers.update(computer_id, ...)` | 更新 |
| `client.computers.delete(computer_id)` | 删除 |

桌面控制：`screenshot()`（字节）、`screenshot_base64()`、`click`、`right_click`、`double_click`、`type`、`key`、`hotkey`、`scroll`、`drag`、`move_cursor`、`launch`、`windows`、`accessibility_tree`、`set_clipboard`、`set_wallpaper`。

## 智能体

没有 `client.agents`。智能体相关接口如下：

```python
computer.agent.run("open the editor")     # computer control session
client.runs.run(instruction="...", target_kind="sandbox", target_id=sbx.id)
sbx.run_agent("fix tests")                # convenience, runner "claude-code"
sbx.prompt("fix tests")                   # convenience, provider "claude"
```

| 接口 | 用途 |
|---|---|
| `computer.agent` | 控制会话：`run`、`list`、`get`、`cancel`、`pause`、`resume`、`task` |
| `client.runs` | `run`、`list`、`get`、`cancel`、`outputs`、`files`、`messages`、`diagnostics` |
| `client.agent_runs` | `run`、`list`、`get`、`cancel`、`artifacts`、`events`、`wait_for_completion` |
| `miosa.agent` | ADK：`Agent`、`miosa_tools`、提供商工厂 |

`miosa.agent.Agent` 需要 `tools=` 或 `miosa_api_key=`；否则会引发 `ValueError`。

## 错误

```python
from miosa import Miosa, RateLimitError

try:
    client.sandboxes.get("missing")
except RateLimitError as err:
    print(err.retry_after)
```

所有错误都继承 `MiosaError`，其中携带 `status_code`、`code`、`body`、`details`、`request_id`、`retryable` 和 `retry_after_ms`。请根据 `err.code`（字符串）分支，而不是 `err.status_code`。按状态码映射的子类（`AuthenticationError`、`PermissionError`、`NotFoundError`、`ValidationError`、`RateLimitError`、`InsufficientCreditsError`、`ServerError`、`TimeoutError`、`ConnectionError`）以及约 55 个类型化错误会从包根导出。

> **注意**
>
> `PermissionError`、`TimeoutError` 和 `ConnectionError` 会遮蔽 Python 内置异常，它们继承自 `MiosaError`，而不是同名内置异常。要使用通用处理器，请捕获 `MiosaError`。

## 此 SDK 中的服务与数据

同一个客户端可以访问 Services 和数据相关接口。以下方法名来自 2.19.0，同步和异步客户端均适用：

```python
# settings as code, replicas and rolling deploys
client.services.update(service_id, settings={"strategy": "rolling"}, shape="dedicated-4", lock_version=3)

# a pull request preview per repository
client.services.set_preview_settings(project_id, repository="acme/api", enabled=True, secret="whsec_...")
metrics = client.services.metrics(project_id, service_id, range="24h")

# log drains
client.services.create_log_drain(project_id, environment_id, name="ops", url="https://logs.example.test/ingest")

# public database endpoint
client.databases.enable_public_networking(database_id, ["203.0.113.0/24"])
```

保护和 WAF 对应 `services.set_protection`、`services.share_link`、
`services.set_waf` 和 `services.set_attack_mode`。
