---
slug: "develop/sandboxes/long-running-tasks"
title: "长时间运行的任务"
description: "比单个执行程序寿命更长的工作：后台进程、它们的日志和状态、始终在线的沙箱以及暂停和恢复后返回的内容。"
section: "Sandboxes"
order: 2011
---

单个 `exec` 调用是有界的：沙箱 `exec` 端点接受 1 到 86,400 秒（默认 3,600）的 `timeout`，与沙箱自身的超时无关。运行时间较长的工作（开发服务器、构建、观察者、工作人员）应作为后台进程启动并单独检查。

## 启动后台进程

**Python**

```python
proc = client.sandboxes.processes.start(
    "sbx_abc123",
    cmd="npm run dev -- --port 3000",
    cwd="/workspace",
    name="dev-server",
)
print(proc.pid)   # proc_...
```

**打字稿**

```typescript
const proc = await client.sandboxes.processes.start('sbx_abc123', {
  cmd: 'npm run dev -- --port 3000',
  cwd: '/workspace',
  name: 'dev-server',
});
```

**命令行界面**

```bash
miosa process start --name build --cwd /workspace/app -- npm run build
miosa ps            # list processes (alias for: miosa process list)
```

正文接受 `cmd`（必需）、`cwd`、`env` 和 `name`。沙箱必须是 `running` 才能启动进程。

## 检查状态和日志

|状态 |意义|
|---|---|
| `running` |进程还活着 |
| `exited` |进程正常退出 |
| `killed` |进程已通过 `DELETE` | 停止
| `error` |进程启动失败 |

获取 stdout+stderr 的最后 N 行，或实时流式传输它们：

```bash
# Last 50 lines
curl "https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/processes/proc_01hwxyz/logs?tail=50" \
  -H "Authorization: Bearer $MIOSA_API_KEY"

# Live stream (SSE)
curl "https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/processes/proc_01hwxyz/stream" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

使用 `DELETE` 停止进程，该进程会在宽限期后发送 `SIGTERM`，然后发送 `SIGKILL`。

## 长期服务的守护进程

沙箱守护进程是一个命名的长期进程：启动一次，然后列出它，读取
它的输出并通过名称重新启动或停止它，而不是寻找 pid。

```bash
miosa daemon create my-box --name web --command "npm run dev" --cwd /workspace/app
miosa daemon list my-box
miosa daemon logs my-box web
miosa daemon restart my-box web
miosa daemon stop my-box web
```

相同的调用是 `POST /api/v1/sandboxes/:id/daemons`、`GET .../daemons` 和
`GET .../daemons/:name`; [进程 API](/docs/api-reference/sandbox-processes)
记录他们所依赖的流程模型。

在计算机上，长寿命守护进程属于 systemd，并在失败时重新启动
而不是进程表；请参阅[服务 API](/docs/api-reference/services)。

## 保持沙箱存活

后台进程仅在沙箱运行时运行。有两个设置可以控制：

- **`always_on: true`** 禁用超时和空闲超时强制执行，直到执行明确的生命周期操作为止，具体取决于租户策略。
- **`idle_timeout_sec`** 在一段时间不活动后停止废弃的持久会话； `0` 禁用空闲自动停止。如果未设置，则对于 2026 年 10 月 11 日或之后创建的帐户，默认为 `0` 或 `1800`（30 分钟）。已停止的沙箱将在下一个请求时唤醒。

超时是空闲锚定的。
对于持久性沙箱，到期会停止沙箱而不是销毁它，并且停止永远不会保持正在运行的进程。
明确的非持久性沙箱被破坏。请参阅[超时和自动停止](/docs/develop/sandboxes/timeouts)。

> **警告：进程不会在停止、快照或分叉后继续存在**
>
> 后台进程的生命周期与其运行会话的生命周期相同。它永远不会在停止后幸存下来，并且不会在快照、恢复或分叉中保留。只有暂停（仅限 Firecracker）才能保留进程，并且最多只能持续 30 分钟，之后沙箱的行为就像停止一样。沙箱再次运行后重新启动进程，或者将其设为已声明的服务，以便于重新启动。

## 另请参阅

- [进程 API](/docs/api-reference/sandbox-processes)：端点、请求字段和状态值。
- [服务 API](/docs/api-reference/services)：计算机上由 systemd 支持的服务。
- [超时和自动停止](/docs/develop/sandboxes/timeouts)：活动会话超时以及如何测量使用情况。
