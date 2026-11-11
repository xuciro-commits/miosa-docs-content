---
slug: "sdks/python"
title: "Python SDK"
description: "Install, authenticate and use the miosa Python SDK 2.16.5 - sync and async, sandboxes, exec, files, snapshots, computers, agents and typed errors."
section: "CLI, SDKs & MCP"
order: 11016
---

The official Python SDK for MIOSA. Every resource has a sync and an async class: pick `Miosa` or `AsyncMiosa` at construction. Requires Python 3.9 or newer.

## Install

```bash
pip install miosa
```

```python
from miosa import Miosa, AsyncMiosa, Sandbox, Computer, Forge
```

Extras: `miosa[events]` (websockets), `miosa[http2]`, `miosa[dev]`. The agent runtime is a separate module: `from miosa.agent import Agent`.

## Authentication

```python
from miosa import Miosa

client = Miosa(api_key="msk_u_...")           # sync
async_client = AsyncMiosa(api_key="msk_u_...") # async
```

`api_key` and `access_token` are mutually exclusive: passing both raises `ValueError`. Key resolution order is `access_token`, `api_key`, `MIOSA_ACCESS_TOKEN`, `MIOSA_API_KEY`. Other options: `tenant` (`MIOSA_TENANT`), `base_url` (`MIOSA_BASE_URL`, default `https://api.miosa.ai/api/v1`), `timeout` (default 60s), `max_retries` (default 3), `unbranded`.

Both clients are context managers. `client.with_tenant(tenant)` returns a client acting in another organization. Use `client.get_balance()`, `client.get_usage()` and `client.get_transactions()` for account reads.

## Sandboxes

```python
sbx = client.sandboxes.create(name="my-box", size="small")
sbx.wait_until_ready()
```

| Method | What it does |
|---|---|
| `client.sandboxes.create(template_id=None, *, name=None, size=None, ...)` | Create a sandbox |
| `client.sandboxes.list(*, state=None, tags=None, ...)` | List |
| `client.sandboxes.get(sandbox_id)` | Get one |
| `client.sandboxes.get_by_name(name)` | Get by name |
| `client.sandboxes.get_or_create(name, *, resume=True, ...)` | Reuse or create |
| `client.sandboxes.delete(sandbox_id)` | Delete |

On the handle, `sandbox.stop()` stops it (the disk is kept, processes are not) and `sandbox.resume()` brings it back; `sandbox.destroy()` deletes it (`delete` is an alias). Sizes are `xs`, `small`, `medium`, `large`, `xl`.

> **Note**
>
> `Sandbox` and `Computer` destroy themselves on `__exit__`. A `with client.sandboxes.create(...) as sbx:` block deletes the sandbox at block exit.

## Exec and files

```python
r = sbx.exec.run("node -v")      # ExecResult(stdout, stderr, exit_code, duration_ms)
sbx.exec("npm test")             # the exec runner is callable
sbx.run("npm test")              # delegates to exec.run
for ev in sbx.exec.stream("npm run dev"):
    print(ev)
sbx.commands.run("ls -la")       # richer: background processes, pid
```

Computer exec is separate:

```python
computer.bash("ls -la")
computer.python("print(2 + 2)")
```

Files differ between sandboxes and computers. Sandbox files have no `mkdir` or `rm`:

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

Sandbox file methods: `list`, `read`, `read_text`, `write`, `write_many`, `upload`, `download`, `stat`, `tree`, `watch`. Flat shims on the handle: `write_file`, `read_file`, `list_files`, `stat_file`, `upload`, `download`.

## Snapshots

```python
sbx.snapshots.create(comment="before deps")
sbx.snapshots.list()
sbx.snapshots.restore(snapshot_id)
sbx.create_snapshot()            # convenience
```

| Surface | Methods |
|---|---|
| `sbx.snapshots` | `create`, `list`, `restore`, `delete` |
| `client.snapshots` | `list`, `get` |
| `client.snapshot_history` | `list`, `groups`, `get`, `delete`, `bulk_delete`, `tree`, `file`, `fork` |
| `client.named_snapshots` | `list`, `create`, `get`, `delete`, `deploy` |

## Computers

```python
computer = client.computers.create(name="dev-box", size="small")
computer.start()
computer.wait_until_ready()
```

| Method | What it does |
|---|---|
| `client.computers.create(name, *, template_type="miosa-desktop", size="small", ...)` | Create |
| `client.computers.list(*, workspace_id=None)` | List |
| `client.computers.get(computer_id)` | Get one |
| `client.computers.update(computer_id, ...)` | Update |
| `client.computers.delete(computer_id)` | Delete |

Desktop control: `screenshot()` (bytes), `screenshot_base64()`, `click`, `right_click`, `double_click`, `type`, `key`, `hotkey`, `scroll`, `drag`, `move_cursor`, `launch`, `windows`, `accessibility_tree`, `set_clipboard`, `set_wallpaper`.

## Agents

There is no `client.agents`. Agent surfaces:

```python
computer.agent.run("open the editor")     # computer control session
client.runs.run(instruction="...", target_kind="sandbox", target_id=sbx.id)
sbx.run_agent("fix tests")                # convenience, runner "claude-code"
sbx.prompt("fix tests")                   # convenience, provider "claude"
```

| Surface | Purpose |
|---|---|
| `computer.agent` | Control sessions: `run`, `list`, `get`, `cancel`, `pause`, `resume`, `task` |
| `client.runs` | `run`, `list`, `get`, `cancel`, `outputs`, `files`, `messages`, `diagnostics` |
| `client.agent_runs` | `run`, `list`, `get`, `cancel`, `artifacts`, `events`, `wait_for_completion` |
| `miosa.agent` | ADK: `Agent`, `miosa_tools`, provider factories |

`miosa.agent.Agent` needs `tools=` or `miosa_api_key=`; it raises `ValueError` otherwise.

## Errors

```python
from miosa import Miosa, RateLimitError

try:
    client.sandboxes.get("missing")
except RateLimitError as err:
    print(err.retry_after)
```

All errors extend `MiosaError`, which carries `status_code`, `code`, `body`, `details`, `request_id`, `retryable` and `retry_after_ms`. Branch on `err.code` (a string), not `err.status_code`. Status-mapped subclasses (`AuthenticationError`, `PermissionError`, `NotFoundError`, `ValidationError`, `RateLimitError`, `InsufficientCreditsError`, `ServerError`, `TimeoutError`, `ConnectionError`) plus around 55 typed errors are exported from the package root.

> **Note**
>
> `PermissionError`, `TimeoutError` and `ConnectionError` shadow Python builtins and derive from `MiosaError`, not from the builtin of the same name. Catch `MiosaError` for a universal handler.

## Services and data, in this SDK

The same client reaches the Services and data surfaces. The method names below
are the ones in 2.19.0, on both the sync and the async client:

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

Protection and WAF are `services.set_protection`, `services.share_link`,
`services.set_waf` and `services.set_attack_mode`.
