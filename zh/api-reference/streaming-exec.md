---
slug: "api-reference/streaming-exec"
title: "流式执行 API"
description: "通过 SSE 从 MIOSA 计算机实时流式传输命令输出的 API 参考。"
section: "API reference"
order: 12019
---

Streaming exec 在计算机上运行命令并通过服务器发送的事件逐行传送输出。将其用于长时间运行的构建、安装或任何需要增量输出而不是等待完成的命令。

基本路径：`/api/v1/computers/{id}/exec/stream`

> **注意**
>
> 标准 `POST /computers/{id}/exec` 在返回之前等待命令完成。流式执行更适合运行时间超过几秒的命令或您想要交互式监控的命令。

---

## 快速入门
```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// Stream a long-running build
for await (const event of client.exec.stream(computerId, {
  command: 'pip install torch',
  timeout: 300,
})) {
  if (event.type === 'output') {
    process.stdout.write(event.data.line);
  } else if (event.type === 'exit') {
    console.log(`Exit code: ${event.data.exitCode}`);
    break;
  }
}
```
```bash
# Obtain a streaming ticket first, then connect
TICKET=$(curl -s -X POST https://api.miosa.ai/api/v1/computers/{id}/exec/stream \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"command": "pip install torch", "timeout": 300}' | jq -r .ticket)

curl -N "https://api.miosa.ai/api/v1/computers/{id}/exec/stream/events?ticket=$TICKET" \
  -H "Accept: text/event-stream"
```

---

## 端点

|方法|路径|描述 |
|--------|------|-------------|
| `POST` | `/computers/{id}/exec/stream` |启动流式执行程序并获取票证 |
| `GET` | `/computers/{id}/exec/stream/events` | SSE 直播（需要门票）|
| `POST` | `/computers/{id}/exec` |非流式执行（等待完成）|
| `POST` | `/computers/{id}/exec/python` |非流式Python exec |

---

## 开始流式执行

**`POST /api/v1/computers/{id}/exec/stream`**

启动命令并返回 SSE 流的短期票证。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `command` |字符串|是的 |执行的 Shell 命令 |
| `timeout` |整数 |没有 |超时以秒为单位（默认值：30，最大值：86400）|
| `shell` |字符串|没有 |要使用的 shell（默认值：`/bin/bash`）|

### 响应 - `202 Accepted`
```json
{
  "ticket": "exec_short_lived_token",
  "expires_at": 1712700060,
  "stream_url": "/api/v1/computers/{id}/exec/stream/events?ticket=exec_short_lived_token"
}
```

票证将在 60 秒后过期 - 立即打开 EventSource 连接。

### 错误

|状态 |错误 |原因 |
|--------|--------|--------|
| 409 | `COMPUTER_NOT_RUNNING`|计算机未运行 |
| 502 | `AGENT_UNAVAILABLE`|虚拟机内智能体无法访问 |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/exec/stream \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"command": "npm install", "timeout": 120}'
```

---

## 连接到 SSE 流

**`GET /api/v1/computers/{id}/exec/stream/events?ticket={ticket}`**

### 事件类型

|活动 |有效负载字段 |描述 |
|--------|-------------|-------------|
| `output` | `line`, `stream` |一行 stdout 或 stderr |
| `exit` | `exit_code`、`duration_ms` |命令完成；此后流关闭 |
| `timeout` | `timeout_seconds` |超时后命令被终止 |
| `error` | `message` |命令启动前出现内部错误 |

### 事件负载示例
```text

event: output
data: {"line": "Collecting torch\n", "stream": "stdout"}

event: output
data: {"line": "  Downloading torch-2.3.0-...\n", "stream": "stdout"}

event: exit
data: {"exit_code": 0, "duration_ms": 45312}
```

### `stream` 值

|价值|描述 |
|--------|-------------|
| `stdout` |标准输出|
| `stderr` |标准错误|
```javascript
// Browser
const es = new EventSource(`/api/v1/computers/${id}/exec/stream/events?ticket=${ticket}`);

es.addEventListener('output', e => {
  const { line, stream } = JSON.parse(e.data);
  console.log(`[${stream}] ${line}`);
});

es.addEventListener('exit', e => {
  const { exit_code } = JSON.parse(e.data);
  console.log(`Exited with code ${exit_code}`);
  es.close();
});

es.addEventListener('error', () => {
  // Connection dropped or ticket expired
  es.close();
});
```

---

## 比较：流式传输与非流式传输

|方面| `POST /exec` | `POST /exec/stream` |
|--------|-------------|--------------------|
|响应时间|命令完成后 |立即（门票）|
|输出交付|一次全部|SSE逐条 |
|最大超时 | 86400 秒 | 86400 秒 |
|最适合 |快速命令|构建、安装时间长 |
| SDK方法| `exec.run()` | `exec.stream()` |

---

## 常见食谱

### 监控构建
```python
for event in client.exec.stream(computer_id, command="make build", timeout=300):
    if event.type == "output":
        print(event.data["line"], end="")
    elif event.type == "exit":
        if event.data["exit_code"] != 0:
            raise RuntimeError(f"Build failed with exit code {event.data['exit_code']}")
        print("Build succeeded")
        break
```

### 分别捕获两个流
```typescript
const stdout: string[] = [];
const stderr: string[] = [];

for await (const event of client.exec.stream(computerId, { command: 'myapp 2>&1 | tee /tmp/output.log' })) {
  if (event.type === 'output') {
    if (event.data.stream === 'stdout') stdout.push(event.data.line);
    else stderr.push(event.data.line);
  } else if (event.type === 'exit') {
    break;
  }
}
```

### 超时处理
```python
try:
    for event in client.exec.stream(computer_id, command="long-task", timeout=60):
        if event.type == "timeout":
            print(f"Command killed after {event.data['timeout_seconds']}s")
            break
        elif event.type == "exit":
            break
except Exception as e:
    print(f"Stream error: {e}")
```

---

## 沙盒流执行

沙箱直接通过 SSE 流式传输命令输出，无需单独的票据步骤。

**`POST /api/v1/sandboxes/{id}/exec/stream`**

运行命令并将 stdout/stderr 事件作为服务器发送的事件进行流式传输。将此用于安装程序输出、长时间运行的智能体任务以及应在命令退出之前显示进度的构建日志。请求正文与`/exec`相同。

```bash
curl -N -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_01j9xr2t4fk8me3n5q/exec/stream \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Accept: text/event-stream" \
  -H "Content-Type: application/json" \
  -d '{"command":"npm install && npm run build","working_dir":"/workspace","timeout":300}'
```

事件示例：
```text
event: stdout
data: {"line":"added 342 packages"}

event: exit
data: {"exit_code":0}
```
