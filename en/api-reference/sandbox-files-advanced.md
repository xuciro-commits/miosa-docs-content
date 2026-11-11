---
slug: "api-reference/sandbox-files-advanced"
title: "Sandbox Files (Advanced) API"
description: "Upload a file, write bytes as well as text, traverse the file tree, batch write, and watch file-change events in a sandbox."
section: "API reference"
order: 12022
---

These advanced file operations extend the core [Files API](/docs/api-reference/files) with IDE-oriented capabilities: recursive tree listing, atomic batch writes, and a live file-change event stream.

Base path: `/api/v1/sandboxes/{id}/files`, and `/api/v1/sandboxes/{id}/fs` for the upload route.

> **Note**
>
> These routes use `:sandbox_id` as the path parameter internally but are accessed as `/sandboxes/{id}/files/...`  -  the `{id}` segment is the sandbox UUID.

> **Note: On SOMA**
>
> File read, write, list, mkdir, and remove work on a SOMA sandbox through these routes.
> They no longer answer `501 FILES_UNSUPPORTED_ON_RUNTIME`.
> Files on SOMA are kept only while the sandbox runs, because SOMA does not persist state across a stop yet.
> See [Choosing a runtime](/docs/soma/choosing-a-runtime).

---

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/sandboxes/{id}/files/tree` | Recursive file tree |
| `POST` | `/api/v1/sandboxes/{id}/files/write-many` | Batch write multiple files |
| `GET` | `/api/v1/sandboxes/{id}/files/watch` | SSE stream of file-change events |
| `POST` | `/api/v1/sandboxes/{id}/fs/upload` | Upload a file (multipart) |

---

## Upload a file

**`POST /api/v1/sandboxes/{id}/fs/upload`**

Send the bytes as multipart form data.
This is the route the SDKs use, and the only upload route today.

| Field | Type | Required | Description |
|---|---|---|---|
| `file` | file | Yes | The bytes to write. A missing part answers `400 MISSING_FILE`. |
| `path` | string | No | Destination. Defaults to `/home/user/`. A trailing slash appends the uploaded filename; without one, `path` is the exact destination. |

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SBX/fs/upload \
  -H "Authorization: Bearer msk_u_..." \
  -F "path=/workspace/assets/" \
  -F "file=@./logo.png"
```

### Errors

| Status | Code | Cause |
|---|---|---|
| 400 | `MISSING_FILE` | No `file` part. |
| 400 | `INVALID_PATH` | `path` contains a null byte. |
| 413 | `FILE_TOO_LARGE` | The upload exceeds the size limit. |

> **Note**
>
> `/api/v1/sandboxes/{id}/files/upload` is an alias of this route.

---

## Write bytes, not text

`/files/write` stores `content` verbatim, so it is for text.
For anything binary, send `content_base64` instead; `/files/write-many` takes it per file.

```bash
# macOS uses `base64 -i`, GNU coreutils uses `base64 -w0`
B64=$(base64 -i ./logo.png | tr -d '\n')
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SBX/files/write \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  --data "$(jq -n --arg p /workspace/logo.png --arg b "$B64" '{path: $p, content_base64: $b}')"
```

---

## File Tree

**`GET /api/v1/sandboxes/{id}/files/tree`**

Returns a recursive directory listing up to a configurable depth. Useful for populating an IDE file-explorer panel.

### Auth

```http
Authorization: Bearer msk_...
```

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `path` | string | `/workspace` | Root directory to list |
| `depth` | integer | `3` | Max traversal depth (1-10) |
| `include_hidden` | boolean | `false` | Include dot-files and dot-directories |

### Response  -  `200 OK`

```json
{
  "data": {
    "path": "/workspace",
    "type": "directory",
    "children": [
      {
        "name": "src",
        "path": "/workspace/src",
        "type": "directory",
        "children": [
          {
            "name": "index.ts",
            "path": "/workspace/src/index.ts",
            "type": "file",
            "size": 1842,
            "modified_at": "2026-05-26T09:12:01Z"
          }
        ]
      },
      {
        "name": "package.json",
        "path": "/workspace/package.json",
        "type": "file",
        "size": 512,
        "modified_at": "2026-05-25T18:00:00Z"
      }
    ]
  }
}
```

---

## Batch Write

**`POST /api/v1/sandboxes/{id}/files/write-many`**

Writes multiple files in a single request. All writes are applied atomically from the guest's perspective  -  either all succeed or an error is returned for the failing file.

> **Note**
>
> Maximum 50 files per request. Files larger than 90 KB should be uploaded individually via `POST /files/write`.

### Request Body

```json
{
  "files": [
    {
      "path": "/workspace/src/index.ts",
      "content": "export const hello = () => 'world';\n",
      "encoding": "utf8"
    },
    {
      "path": "/workspace/src/utils.ts",
      "content": "ZXhwb3J0IGZ1bmN0aW9uIG5vb3AoKSB7fQo=",
      "encoding": "base64"
    }
  ]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `files[].path` | string | Yes | Absolute path inside the sandbox |
| `files[].content` | string | Yes | File content (UTF-8 text or base64-encoded binary) |
| `files[].encoding` | string | No | `"utf8"` (default) or `"base64"` |

### Response  -  `200 OK`

```json
{
  "data": {
    "written": 2,
    "paths": [
      "/workspace/src/index.ts",
      "/workspace/src/utils.ts"
    ]
  }
}
```

### Errors

| Status | Code | Cause |
|--------|------|-------|
| 400 | `INVALID_BODY` | `files` is missing or not an array |
| 400 | `TOO_MANY_FILES` | More than 50 files in one request |
| 400 | `INVALID_PATH` | A path contains a null byte or traverses outside the sandbox |

---

## File Watch (SSE)

**`GET /api/v1/sandboxes/{id}/files/watch`**

Streams file-system change events as SSE. Useful for syncing editor state in real time.

### Auth

```http
Authorization: Bearer msk_...
```

(Browser `EventSource` cannot set this header  -  use a server-side client or a scoped session token.)

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `path` | string | `/workspace` | Directory to watch |

### Response  -  `200 OK` (`text/event-stream`)

```text
id: wev_001
event: file.changed
data: {"path":"/workspace/src/index.ts","event":"modified","ts":"2026-05-26T10:14:23Z"}

id: wev_002
event: file.changed
data: {"path":"/workspace/src/new-file.ts","event":"created","ts":"2026-05-26T10:14:24Z"}

: heartbeat
```

**Event values for `data.event`:**

| Value | Trigger |
|-------|---------|
| `created` | New file or directory |
| `modified` | File content changed |
| `deleted` | File or directory removed |
| `renamed` | File moved or renamed |

---

## Examples

**cURL**

```bash
# File tree (depth 2)
curl "https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/files/tree?depth=2" \
  -H "Authorization: Bearer msk_u_..."

# Batch write
curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/files/write-many \
  -H "Authorization: Bearer msk_u_..." \
  -H "Content-Type: application/json" \
  -d '{
    "files": [
      {"path": "/workspace/hello.ts", "content": "console.log(\"hello\");\n"}
    ]
  }'

# File watch (SSE)
curl -N -H "Authorization: Bearer msk_u_..." \
  -H "Accept: text/event-stream" \
  "https://api.miosa.ai/api/v1/sandboxes/sbx_abc123/files/watch?path=/workspace/src"
```

**Python**

```python
import miosa

client = miosa.Miosa()

# Tree
tree = client.sandboxes.files.tree("sbx_abc123", path="/workspace", depth=3)

# Batch write
client.sandboxes.files.write_many("sbx_abc123", [
    {"path": "/workspace/index.ts", "content": "export {};"},
    {"path": "/workspace/utils.ts", "content": "export function noop() {}"},
])
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// File tree
const tree = await client.sandboxes.files.tree('sbx_abc123', {
  path: '/workspace',
  depth: 3,
  includeHidden: false,
});

// Batch write
await client.sandboxes.files.writeMany('sbx_abc123', [
  { path: '/workspace/index.ts', content: 'export {};' },
  { path: '/workspace/utils.ts', content: 'export function noop() {}' },
]);
```
