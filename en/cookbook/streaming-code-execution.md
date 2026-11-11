---
slug: "cookbook/streaming-code-execution"
title: "Streaming code execution"
description: "Run a long Python script in a sandbox and stream stdout to your UI in real time via SSE, with backpressure handling."
section: "Sandboxes"
order: 2033
---

`~8 min` `Python` `TypeScript`

> **Goal**: Run a long-running script inside a sandbox and stream every stdout line to your frontend UI in real time  -  no polling, no buffering the full output.
>
> **What you'll use**: Sandboxes, SSE events, exec streaming

## What you'll build

A minimal code-execution console: your backend creates a sandbox, runs a script, and pipes the sandbox SSE event stream back to the browser. The browser displays each line as it arrives. Looks like this in the terminal:

```text
[00:01] Starting data pipeline...
[00:03] Loaded 50,000 rows
[00:05] Processing batch 1/10...
[00:09] Processing batch 2/10...
...
[01:12] Done. Output written to /workspace/output.csv
```

## Prerequisites

- A MIOSA workspace API key (`msk_u_*`)  -  see [API Keys](/docs/platform/api-keys)
- Node 22+ or Python 3.11+

## Step 1  -  Install and configure

**Python**

```bash
pip install miosa
export MIOSA_API_KEY="msk_u_..."
```

**TypeScript**

```bash
npm install @miosa/sdk
export MIOSA_API_KEY="msk_u_..."
```

## Step 2  -  Write a long-running script into the sandbox

Create a sandbox and write the script you want to execute. The script here simulates a ten-batch data pipeline with a sleep between batches.

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

**TypeScript**

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

## Step 3  -  Stream exec output

Use `exec.stream(...)` (Python and TypeScript) to receive lines as they arrive rather than waiting for the process to exit.

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

**TypeScript**

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

## Step 4  -  Forward the stream to a browser via SSE

In a real application your backend runs the sandbox exec and forwards each line to the browser over an HTTP SSE response. Below is the pattern for both a Python ASGI server (FastAPI) and a Node HTTP handler.

**Python (FastAPI)**

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

**TypeScript (Node HTTP)**

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

> **Note**
>
> Set `X-Accel-Buffering: no` when running behind nginx. Without it, nginx buffers the SSE response and the browser receives events in chunks rather than one by one.

## Step 5  -  Consume in the browser

A minimal browser snippet that connects to the SSE endpoint and appends each line to a `<pre>` element:

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

> **Tip**
>
> `EventSource` only supports GET requests. If you need to pass a script body, use a two-step flow: POST `/run` to create a job and get back a `job_id`, then `GET /run/:job_id/stream` over EventSource.

## Backpressure

The MIOSA SSE stream pushes events as they arrive. If your client (browser tab, UI component) is slow, the network layer buffers them. For most exec workloads this is fine.

For very high-throughput scripts (millions of log lines), apply backpressure explicitly:

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

**TypeScript**

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

## Long-running server processes

For servers or daemons that must survive across multiple `exec` calls, use `processes.start` instead of exec with a background flag. The process keeps running even when your driver script moves on.

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

**TypeScript**

```typescript
// processes is Python-first in the current SDK.
// For TypeScript, use exec with a detached background command:
await sbx.exec('python3 -m http.server 8080 &', { cwd: '/workspace' });

// Check it is up:
const check = await sbx.exec('curl -s http://localhost:8080/', { cwd: '/workspace' });
console.log('Server responded:', check.stdout.slice(0, 80));
```

> **Tip**
>
> Use `processes.start` (Python) when you need to tail logs or stop the process programmatically later. Use a backgrounded `exec` command for simpler cases where you only need the process to keep running.

## What you learned

- `exec` with streaming yields events as the process runs  -  no waiting for exit.
- `stdout`, `stderr`, and `exit` are the three event types to handle.
- SSE responses need `X-Accel-Buffering: no` behind nginx to avoid buffering.
- For high-throughput output, a token-bucket rate limiter protects the browser client.
- `processes.start` (Python) keeps a server or daemon alive independently of exec calls, with named process management and log tailing.

## Next

- [Agent builds an app](/docs/cookbook/agent-builds-app): Wire a streaming exec into an LLM agent loop that self-corrects on build errors.
- [Sandbox events in real time](/docs/cookbook/sandbox-events-realtime): React to sandbox lifecycle events via webhooks or the SSE stream.
- [Files and Exec](/docs/develop/files-and-exec): Complete reference for exec options, working directories, environment variables, and timeouts.
