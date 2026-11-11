---
slug: "cookbook/streaming-code-execution"
title: "流式代码执行"
description: "在沙箱中运行长 Python 脚本，并通过 SSE 将标准输出实时传输到您的 UI，并进行反压处理。"
section: "Sandboxes"
order: 2033
---

`~8 min` `Python` `TypeScript`

> **目标**：在沙箱内运行长时间运行的脚本，并将每个标准输出行实时传输到前端 UI - 不轮询，不缓冲完整输出。
>
> **您将使用什么**：沙箱、SSE 事件、执行流

## 你将构建什么

最小的代码执行控制台：后端创建一个沙箱，运行一个脚本，并将沙箱 SSE 事件流通过管道传输回浏览器。浏览器会在每一行到达时显示它。在终端中看起来像这样：

```text
[00:01] Starting data pipeline...
[00:03] Loaded 50,000 rows
[00:05] Processing batch 1/10...
[00:09] Processing batch 2/10...
...
[01:12] Done. Output written to /workspace/output.csv
```

## 先决条件

- MIOSA 工作区 API 密钥 (`msk_u_*`) - 请参阅 [API 密钥](/docs/platform/api-keys)
- 节点 22+ 或 Python 3.11+

## 步骤 1 - 安装和配置

**Python**

```bash
pip install miosa
export MIOSA_API_KEY="msk_u_..."
```

**打字稿**

```bash
npm install @miosa/sdk
export MIOSA_API_KEY="msk_u_..."
```

## 步骤 2 - 将长时间运行的脚本写入沙箱

创建一个沙箱并编写要执行的脚本。这里的脚本模拟了十批数据管道，并且批之间有睡眠。

**Python**

```python
import os
from miosa import Miosa

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

sbx = client.sandboxes.create(template_id="miosa-sandbox", timeout_sec=300)

script = """\
import time, sys

BATCHES = 10
print("Starting data pipeline...", flush=True)
time.sleep(1)

for i in range(1, BATCHES + 1):
    print(f"Processing batch {i}/{BATCHES}...", flush=True)
    # simulate work
    time.sleep(0.8)

print("Done. Output written to /workspace/output.csv", flush=True)
"""

sbx.files.write("/workspace/pipeline.py", script)
```

**打字稿**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

const sbx = await client.sandboxes.create({
  templateId: 'miosa-sandbox',
  timeoutSec: 300,
});

const script = `
import time, sys

BATCHES = 10
print("Starting data pipeline...", flush=True)
time.sleep(1)

for i in range(1, BATCHES + 1):
    print(f"Processing batch {i}/{BATCHES}...", flush=True)
    time.sleep(0.8)

print("Done. Output written to /workspace/output.csv", flush=True)
`;

await sbx.files.write('/workspace/pipeline.py', script);
```

## 步骤 3 - 流式执行输出

使用 `exec.stream(...)`（Python 和 TypeScript）在行到达时接收行，而不是等待进程退出。

**Python**

```python
import sys, datetime

def ts() -> str:
    return datetime.datetime.now().strftime("%M:%S")

# exec.stream yields ExecEvent objects as lines arrive
for event in sbx.exec.stream("python3 /workspace/pipeline.py"):
    if event.type == "stdout":
        print(f"[{ts()}] {event.line}")
    elif event.type == "stderr":
        print(f"[{ts()}] ERR: {event.line}", file=sys.stderr)
    elif event.type == "exit":
        print(f"Process exited with code {event.exit_code}")
        break
```

**打字稿**

```typescript
const now = () => new Date().toISOString().substring(14, 19);

// execStream returns an async iterable of exec events
for await (const event of sbx.execStream('python3 /workspace/pipeline.py')) {
  if (event.type === 'stdout') {
    process.stdout.write(`[${now()}] ${event.line}\n`);
  } else if (event.type === 'stderr') {
    process.stderr.write(`[${now()}] ERR: ${event.line}\n`);
  } else if (event.type === 'exit') {
    console.log(`Process exited with code ${event.exitCode}`);
    break;
  }
}
```

## 步骤 4 - 通过 SSE 将流转发到浏览器

在真实的应用程序中，您的后端运行沙箱执行程序并通过 HTTP SSE 响应将每一行转发到浏览器。下面是 Python ASGI 服务器 (FastAPI) 和 Node HTTP 处理程序的模式。

**Python（FastAPI）**

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
from miosa import Miosa
import os, asyncio, json

app = FastAPI()
client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

@app.post("/run")
async def run_script(body: dict):
    """
    Accepts { "script": "..." } in the request body.
    Returns an SSE stream of execution events.
    """
    sbx = client.sandboxes.create(template_id="miosa-sandbox", timeout_sec=120)
    sbx.files.write("/workspace/script.py", body["script"])

    def event_generator():
        try:
            for event in sbx.exec.stream("python3 /workspace/script.py"):
                if event.type == "stdout":
                    payload = json.dumps({"type": "stdout", "line": event.line})
                    yield f"data: {payload}\n\n"
                elif event.type == "stderr":
                    payload = json.dumps({"type": "stderr", "line": event.line})
                    yield f"data: {payload}\n\n"
                elif event.type == "exit":
                    payload = json.dumps({"type": "exit", "code": event.exit_code})
                    yield f"data: {payload}\n\n"
                    break
        finally:
            sbx.delete()

    return StreamingResponse(
        event_generator(),
        media_type="text/event-stream",
        headers={
            "Cache-Control": "no-cache",
            "X-Accel-Buffering": "no",  # disable nginx buffering
        },
    )
```

**TypeScript（节点 HTTP）**

```typescript
import http from 'node:http';
import { Miosa } from '@miosa/sdk';

const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

const server = http.createServer(async (req, res) => {
  if (req.method !== 'POST' || req.url !== '/run') {
    res.writeHead(404).end();
    return;
  }

  // Read request body
  const chunks: Buffer[] = [];
  for await (const chunk of req) chunks.push(chunk as Buffer);
  const { script } = JSON.parse(Buffer.concat(chunks).toString());

  // SSE headers  -  must be set before any write
  res.writeHead(200, {
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
    'X-Accel-Buffering': 'no',
    Connection: 'keep-alive',
  });

  const sbx = await client.sandboxes.create({
    templateId: 'miosa-sandbox',
    timeoutSec: 120,
  });

  try {
    await sbx.files.write('/workspace/script.py', script);

    for await (const event of sbx.execStream('python3 /workspace/script.py')) {
      const payload = JSON.stringify(event);
      res.write(`data: ${payload}\n\n`);

      if (event.type === 'exit') break;
    }
  } finally {
    res.end();
    await sbx.delete();
  }
});

server.listen(3000, () => console.log('Listening on http://localhost:3000'));
```

> **注意**
>
> 在 nginx 后面运行时设置 `X-Accel-Buffering: no`。如果没有它，nginx 会缓冲 SSE 响应，浏览器会分块接收事件，而不是一个接一个地接收事件。

## 步骤 5 - 在浏览器中使用

连接到 SSE 端点并将每一行附加到 `<pre>` 元素的最小浏览器片段：

```html
<!doctype html>
<html>
  <body>
    <button id="run">Run pipeline</button>
    <pre id="output" style="background:#111;color:#0f0;padding:1rem;min-height:200px"></pre>
    <script>
      document.getElementById('run').addEventListener('click', () => {
        const output = document.getElementById('output');
        output.textContent = '';

        const es = new EventSource('/run-sse'); // GET endpoint that starts a job
        es.onmessage = (e) => {
          const event = JSON.parse(e.data);
          if (event.type === 'stdout' || event.type === 'stderr') {
            output.textContent += event.line + '\n';
          } else if (event.type === 'exit') {
            output.textContent += `\n[exit ${event.code}]`;
            es.close();
          }
        };
        es.onerror = () => es.close();
      });
    </script>
  </body>
</html>
```

> **提示**
>
> `EventSource` 仅支持 GET 请求。如果需要传递脚本主体，请使用两步流程：POST `/run` 创建作业并返回 `job_id`，然后通过 EventSource 返回 `GET /run/:job_id/stream`。

## 背压

MIOSA SSE 流在事件到达时推送事件。如果您的客户端（浏览器选项卡、UI 组件）速度很慢，网络层会对它们进行缓冲。对于大多数执行工作负载来说，这很好。

对于非常高吞吐量的脚本（数百万条日志行），显式应用反压：

**Python**

```python
import asyncio

# Rate-limit to 200 lines/sec to avoid overwhelming a browser client
MAX_LINES_PER_SEC = 200
bucket = MAX_LINES_PER_SEC

for event in sbx.exec.stream("python3 /workspace/heavy.py"):
    if event.type == "stdout":
        if bucket <= 0:
            # Drop or summarize  -  never block the generator
            continue
        bucket -= 1
        yield f"data: {json.dumps({'line': event.line})}\n\n"
    elif event.type == "exit":
        break
```

**打字稿**

```typescript
let bucket = 200; // refill every second
const refill = setInterval(() => { bucket = 200; }, 1000);

try {
  for await (const event of sbx.execStream('python3 /workspace/heavy.py')) {
    if (event.type === 'stdout') {
      if (bucket > 0) {
        bucket--;
        res.write(`data: ${JSON.stringify({ line: event.line })}\n\n`);
      }
      // else: drop the line  -  client is keeping up with summaries
    } else if (event.type === 'exit') {
      break;
    }
  }
} finally {
  clearInterval(refill);
}
```

## 长时间运行的服务器进程

对于必须在多个 `exec` 调用中生存的服务器或守护程序，请使用 `processes.start` 而不是带有后台标志的 exec。即使您的驱动程序脚本继续运行，该进程也会继续运行。

**Python**

```python
# Start a long-running server  -  stays alive between exec calls
proc = sbx.processes.start(
    command="python3 -m http.server 8080",
    name="http-server",
)
print(f"Server running  -  pid {proc['pid']}")

# Do other work…

# Tail the last 50 lines of output
logs = sbx.processes.logs(proc["pid"], tail=50)
print(logs)

# List all running processes
for p in sbx.processes.list():
    print(f"  {p['pid']}  {p['command']}  {p['status']}")

# Stop cleanly when done
sbx.processes.stop(proc["pid"])
```

**打字稿**

```typescript
// processes is Python-first in the current SDK.
// For TypeScript, use exec with a detached background command:
await sbx.exec('python3 -m http.server 8080 &', { cwd: '/workspace' });

// Check it is up:
const check = await sbx.exec('curl -s http://localhost:8080/', { cwd: '/workspace' });
console.log('Server responded:', check.stdout.slice(0, 80));
```

> **提示**
>
> 当您稍后需要尾部日志或以编程方式停止进程时，请使用 `processes.start` (Python)。对于仅需要进程继续运行的简单情况，请使用后台 `exec` 命令。

## 你学到了什么

- `exec` 在进程运行时流式传输会产生事件 - 无需等待退出。
- `stdout`、`stderr` 和 `exit` 是要处理的三种事件类型。
- SSE 响应需要 `X-Accel-Buffering: no` 在 nginx 后面以避免缓冲。
- 对于高吞吐量输出，令牌桶速率限制器可以保护浏览器客户端。
- `processes.start` (Python) 通过命名进程管理和日志跟踪，使服务器或守护进程独立于 exec 调用而保持活动状态。

## 下一步

- [智能体构建应用程序](/docs/cookbook/agent-builds-app)：将流式执行程序连接到 LLM 智能体循环中，该循环可以自我更正构建错误。
- [实时沙箱事件](/docs/cookbook/sandbox-events-realtime)：通过 webhooks 或 SSE 流对沙箱生命周期事件做出反应。
- [文件和执行](/docs/develop/files-and-exec)：执行选项、工作目录、环境变量和超时的完整参考。
