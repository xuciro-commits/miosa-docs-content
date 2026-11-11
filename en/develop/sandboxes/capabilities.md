---
slug: "develop/sandboxes/capabilities"
title: "Capabilities"
description: "What sandboxes are for, what a sandbox is concretely, and the core exec, files, preview, snapshot, and lifecycle capabilities."
section: "Sandboxes"
order: 2001
---

This page is the reference behind the [Sandboxes overview](/docs/develop/sandboxes): what people build with sandboxes, what one actually is, and the capabilities every sandbox exposes.

## What Sandboxes are for

- **Agent build loops**: Generate code, edit files, install dependencies, run tests, and stream logs back to the agent.
- **Preview servers**: Run a web server in the sandbox and expose only the needed port as an HTTPS preview URL.
- **Safe execution**: Isolate untrusted commands from your backend while keeping filesystem state scoped to the sandbox.
- **Reproducible state**: Snapshot working state, resume later, or fork from a known point for parallel agent work.

## What a Sandbox is, concretely

- Isolated compute for code generation, builds, tests, and dev servers.
- Named, versioned CPU, memory, and disk contracts plus per-sandbox timeout and environment parameters.
- `/workspace` is the editable filesystem root.
- Boots from a versioned template, usually `miosa-sandbox`.
- Persistent by default, so `timeout_sec` stops compute and preserves state instead of deleting the workspace.
- Network-isolated by default.
- Preview URLs open only the specific ports you expose.
- Destroy is permanent and removes the saved state.

## Core capabilities

| Capability | What it gives you |
|---|---|
| `exec.run` | Blocking command execution with stdout, stderr, exit code, and duration |
| `exec.stream` | Real-time stdout/stderr events for long builds or installs |
| `files.write/read/list/stat` | Direct filesystem access under `/workspace` and other allowed paths |
| `previews.create` | Public HTTPS URL for a running process on a specific sandbox port |
| `snapshot` | Durable checkpoint for resume, fork, or rollback |
| `pause` / `resume` | Stop compute without discarding saved state |

## See it work

**TypeScript**

```typescript
const sandbox = await miosa.sandboxes.create({ templateId: "miosa-sandbox" })

await sandbox.files.write(
  "/workspace/index.html",
  `<!doctype html>
<html lang="en">
  <head><meta charset="utf-8"><title>My App</title></head>
  <body><h1>Hello from MIOSA</h1><p>Built in a sandbox.</p></body>
</html>`
)

const result = await sandbox.exec("python3 --version")
console.log(result.stdout)

const preview = await sandbox.previews.create(3000)
console.log(preview.url)
// https://3000-abc12345.sandbox.miosa.ai
```

**Python**

```python
sandbox = miosa.sandboxes.create(template_id="miosa-sandbox")

sandbox.files.write(
    "/workspace/index.html",
    """<!doctype html>
<html lang="en">
  <head><meta charset="utf-8"><title>My App</title></head>
  <body><h1>Hello from MIOSA</h1><p>Built in a sandbox.</p></body>
</html>""",
)

result = sandbox.exec("python3 --version")
print(result.stdout)

preview = sandbox.previews.create(port=3000)
print(preview.url)
# https://5173-abc12345.sandbox.miosa.ai
```
