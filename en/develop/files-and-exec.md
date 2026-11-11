---
slug: "develop/files-and-exec"
title: "Files & Exec"
description: "Read, write, watch files inside a sandbox. Run commands, stream stdout/stderr, manage processes."
section: "Sandboxes"
order: 2005
---

The Files and Exec APIs are how you (or an AI agent) interact with a Sandbox. Together they're the substrate for everything else: generating code, running builds, starting servers.

## Files

### Write

**TypeScript**

```typescript
await sandbox.files.write("/workspace/src/index.ts", `
  console.log("Hello from MIOSA")
`)

// Binary
await sandbox.files.write("/workspace/logo.png", pngBuffer, {
  encoding: "binary",
})
```

**cURL**

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SBX/files/write \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -d '{
    "path": "/workspace/src/index.ts",
    "content": "console.log(\"Hello from MIOSA\")"
  }'
```

Parent directories are created as needed. Existing files are overwritten.

### Read
```typescript
const content = await sandbox.files.read("/workspace/package.json")
const parsed = JSON.parse(content)
```
```bash
curl https://api.miosa.ai/api/v1/sandboxes/$SBX/files/read?path=/workspace/package.json \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

### Export generated artifacts

Use exports when an agent creates files that a user should download from your
product: HTML, PDFs, CSVs, ZIPs, screenshots, reports, or build folders.

**TypeScript**

```typescript
const exportInfo = await sandbox.createExport({
  paths: ["/workspace/dist/index.html", "/workspace/dist/assets/app.js"],
  label: "homepage build",
})

console.log(exportInfo.archiveDownloadUrl)

const archive = await sandbox.downloadExport([
  "/workspace/dist/index.html",
  "/workspace/dist/assets/app.js",
])
```

**Python**

```python
export_info = sandbox.create_export(
    ["/workspace/dist/index.html", "/workspace/dist/assets/app.js"],
    label="homepage build",
)

print(export_info["archive_download_url"])

archive = sandbox.download_export([
    "/workspace/dist/index.html",
    "/workspace/dist/assets/app.js",
])
```

**CLI**

```bash
miosa files cp sb_abc123:/workspace/dist/index.html ./index.html --json

miosa files cp -r sb_abc123:/workspace/dist ./homepage-build --json
```

For one file, exports return a direct download URL for that file. For multiple
files, exports return an archive download URL. This is the same low-level model
as copying or downloading sandbox files: the sandbox remains the working
filesystem, and your product decides which paths become user-visible artifacts.

Use this pattern for artifact products:

```text
1. Agent writes files under /workspace/artifacts or /workspace/dist.
2. Your backend calls createExport with the exact paths users should receive.
3. Your UI stores the returned export descriptor on the task or run.
4. User clicks Download.
5. Your backend streams the direct file or tar archive from MIOSA.
```

Examples:

| Agent output | Export path |
|---|---|
| Landing page HTML | `/workspace/dist/index.html` |
| Built app folder | `/workspace/dist` |
| PDF report | `/workspace/artifacts/report.pdf` |
| Sales CSV | `/workspace/artifacts/leads.csv` |
| Screenshot set | `/workspace/artifacts/screenshots` |
| Generated project | `/workspace` or a filtered build folder |

Do not make users scrape files out of terminal output. If a run produces a
deliverable, turn it into an export and attach it to the run, task, or project
record in your product.

For agent platforms, treat the sandbox filesystem as the working surface and
exports as the user-facing contract. The agent can create many intermediate
files under `/workspace`, but your backend should promote only approved paths
into named artifacts.

| Product pattern | Working paths | Exported artifact |
|---|---|---|
| Lovable/Replit/Genspark-style app builder | `/workspace`, `/workspace/dist` | source ZIP, build folder, preview, deployment URL |
| Polsia-style operator | `/workspace/research`, `/workspace/artifacts` | reports, CSVs, campaign copy, generated app bundles |
| cofounder.co-style company OS | `/workspace/agents`, `/workspace/receipts`, `/workspace/artifacts` | milestone documents, decisions, status reports |
| Nebula-style virtual device | shared `/workspace` plus computer screenshots | selected files, screenshots, logs, device receipts |

Store the export id, label, paths, size, MIME type when available, and download
URL on your own run record. That gives users a stable artifact history even if
the sandbox later pauses, forks, or continues generating new files.

### List
```typescript
const entries = await sandbox.files.list("/workspace")
for (const e of entries) {
  console.log(e.type, e.path, e.size)
}
```
```bash
curl https://api.miosa.ai/api/v1/sandboxes/$SBX/files/list?path=/workspace \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

### Watch

Stream filesystem events:
```typescript
for await (const event of sandbox.files.watch("/workspace")) {
  console.log(event.type, event.path)
  // event.type: "create" | "modify" | "delete"
}
```

Watch is implemented as Server-Sent Events. See [Events](/docs/api-reference/events) for the SSE event shape.

### Filesystem layout

| Path | Purpose |
|---|---|
| `/workspace` | Working directory; this is where agents write app code |
| `/home/sandbox` | User home |
| `/tmp` | Temporary scratch (cleared on sandbox destroy) |
| `/opt/venv` | Pre-installed Python virtualenv (`miosa-sandbox` and Python templates) |
| `/usr/local/bin` | System binaries |

Generally, write inside `/workspace` and let your build tooling handle
node_modules, virtualenvs, etc. Production publish freezes `/workspace`
(excluding cache dirs) as the source snapshot. See
[Publishing](/docs/docker-host/publishing).

## Exec

Run commands inside the sandbox.

### One-shot

**TypeScript**

```typescript
const result = await sandbox.exec("npm install", {
  cwd: "/workspace",
  env: { NODE_ENV: "development" },
  timeoutSec: 120,
})

console.log(result.exit_code, result.stdout, result.stderr)
```

**cURL**

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SBX/exec \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -d '{
    "cmd": "npm",
    "args": ["install"],
    "cwd": "/workspace",
    "timeout_ms": 120000
  }'
```

Blocks until the command exits (or `timeoutMs` elapses). Returns `exit_code`, `stdout`, `stderr`, and `duration_ms`.

### Long-running servers
```typescript
// Keep the process attached and consume its output as it runs.
const events = sandbox.exec.stream("node /workspace/server.js")
for await (const event of events) console.log(event)
```

Use [Previews](/docs/develop/previews) to expose its port.

### Streaming

For real-time stdout/stderr (e.g. tail an agent's compile output):
```typescript
for await (const chunk of sandbox.exec.stream("pnpm dev")) {
  if (chunk.type === "stdout") process.stdout.write(chunk.data)
  if (chunk.type === "stderr") process.stderr.write(chunk.data)
  if (chunk.type === "exit") console.log("exit:", chunk.exit_code)
}
```

Implemented as SSE. See [Streaming Exec](/docs/api-reference/streaming-exec).

### Terminal

Need an interactive PTY (for remote dev or AI agents driving CLIs)?
```typescript
const session = await sandbox.terminal.create({})
// Attach over the session's WebSocket channel. See API Reference: Terminal for the protocol.
```

Terminals are WebSocket-based with `xterm.js`-compatible ANSI passthrough. See [API Reference: Terminal](/docs/api-reference/streaming-exec) for protocol details.

## Process management

`exec` blocks until the command exits; `exec.stream` keeps it running while you consume output. To inspect or stop processes, run the shell commands inside the sandbox:

```typescript
await sandbox.exec("ps aux")
await sandbox.exec("kill <pid>")
```

## Idempotency

Exec is **not** idempotent. Running `npm install` twice runs it twice. If you
need idempotent retries, store a marker file:
```typescript
const marker = "/workspace/.deps-installed"
const exists = await sandbox.files.exists(marker)
if (!exists) {
  await sandbox.exec("npm ci")
  await sandbox.files.write(marker, new Date().toISOString())
}
```

File writes are idempotent: same path, same content, same result.

## Limits

| Limit | Default | Tunable |
|---|---|---|
| Max file size per write | 500 MB | Yes (plan-dependent) |
| Max files per sandbox | 1,000,000 | Yes |
| Max exec stdout buffer | 10 MB | Stream mode bypasses this |
| Max exec duration | `timeout_ms` parameter, default 30 s; pass `120000` for npm/pip installs | Yes; detached has no limit |
| Max concurrent detached jobs | 100 | Yes |

> **Note: No raw shell escaping concerns**
>
> `cmd` and `args` are passed as a tuple to `execve` directly, with no shell
> interpolation. You don't need to quote arguments or escape special characters.
> If you actually want a shell, set `cmd: "bash"` and
> `args: ["-c", "&lt;your shell command&gt;"]`.

## See also

- [Sandboxes](/docs/develop/sandboxes): the VM you're acting on
- [Previews](/docs/develop/previews): expose ports started via exec
- [API Reference: Streaming Exec](/docs/api-reference/streaming-exec): SSE wire format
- [API Reference: Files](/docs/api-reference/files): file API wire format
