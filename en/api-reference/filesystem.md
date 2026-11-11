---
slug: "api-reference/filesystem"
title: "Filesystem API"
description: "Complete API reference for file and directory operations on MIOSA computers - upload, download, stat, mkdir, rename, copy, chmod, and readdir."
section: "API reference"
order: 12020
---

The Filesystem API gives you full programmatic access to files and directories inside a running computer. All operations are proxied through the computer command service and restricted to allowed paths.

Base path: `/api/v1/computers/{id}/files`

**Allowed paths:** `/workspace`, `/home`, `/root`, `/tmp`, `/opt`, `/srv`

> **Note**
>
> The computer must be in `"running"` status. Paths outside the allowed prefixes return `403 FORBIDDEN_PATH`.

---

## Quick Start
```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// Write a file
await client.files.write(computerId, {
  path: '/workspace/hello.py',
  content: 'print("Hello from MIOSA")',
});

// List the directory
const { files } = await client.files.list(computerId, '/workspace');

// Download it back
const content = await client.files.download(computerId, '/workspace/hello.py');
```

---

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/files/upload` | Upload a file (multipart) |
| `POST` | `/files/write` | Write text/binary content directly |
| `GET` | `/files` | List directory (`readdir`) |
| `GET` | `/files/download` | Download a file |
| `POST` | `/files/export` | Download with metadata (base64) |
| `DELETE` | `/files` | Delete a file |
| `GET` | `/files/stat` | Stat a file or directory |
| `POST` | `/files/mkdir` | Create a directory |
| `POST` | `/files/rename` | Rename or move a file/directory |
| `POST` | `/files/copy` | Copy a file |
| `POST` | `/files/chmod` | Change file permissions |

---

## Upload a File

**`POST /api/v1/computers/{id}/files/upload`**

Multipart upload  -  use this for binary files or large payloads.

### Request  -  `multipart/form-data`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file` | file | Yes | File to upload |
| `path` | string | No | Remote path. If ends with `/`, filename is appended. Default: `/workspace/` |

### Response  -  `201 Created`
```json
{
  "success": true,
  "file": {
    "path": "/workspace/script.py",
    "filename": "script.py",
    "size_bytes": 1234
  }
}
```

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 400 | `MISSING_FILE` | No file field in request |
| 403 | `FORBIDDEN_PATH` | Path outside allowed directories |
| 413 | `FILE_TOO_LARGE` | File exceeds 10 MB limit |
| 502 | `AGENT_UNAVAILABLE` | In-VM agent unreachable |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/files/upload \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -F "file=@./script.py" \
  -F "path=/workspace/scripts/"
```

---

## Write a File

**`POST /api/v1/computers/{id}/files/write`**

Write text or base64-encoded binary content. Faster than multipart for programmatic use.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `path` | string | Yes | Full destination path |
| `content` | string | Yes | File content (UTF-8 text or base64 when `encoding="base64"`) |
| `encoding` | string | No | `"utf8"` (default) or `"base64"` |

### Response  -  `201 Created`
```json
{
  "success": true,
  "file": {
    "path": "/workspace/config.json",
    "size_bytes": 256
  }
}
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/files/write \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"path": "/workspace/config.json", "content": "{\"key\": \"value\"}"}'
```

---

## List Directory (readdir)

**`GET /api/v1/computers/{id}/files`**

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | No | Directory to list. Default: `/workspace` |

### Response  -  `200 OK`
```json
{
  "files": [
    {
      "name": "Documents",
      "type": "directory",
      "size_bytes": 4096,
      "permissions": "drwxr-xr-x",
      "modified_at": "2026-04-11T10:30:00Z"
    },
    {
      "name": "script.py",
      "type": "file",
      "size_bytes": 1234,
      "permissions": "-rw-r--r--",
      "modified_at": "2026-04-11T10:25:00Z"
    }
  ],
  "path": "/workspace"
}
```
```bash
curl "https://api.miosa.ai/api/v1/computers/{id}/files?path=/workspace" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Download a File

**`GET /api/v1/computers/{id}/files/download`**

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Full path to the file |

### Response  -  `200 OK`

Binary file content. `Content-Disposition` is set for browser download.

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 400 | `MISSING_PATH` | No path parameter |
| 403 | `FORBIDDEN_PATH` | Path outside allowed directories |
| 404 | `FILE_NOT_FOUND` | File does not exist |
| 502 | `AGENT_UNAVAILABLE` | In-VM agent unreachable |
```bash
curl "https://api.miosa.ai/api/v1/computers/{id}/files/download?path=/workspace/output.txt" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  --output output.txt
```

---

## Export a File

**`POST /api/v1/computers/{id}/files/export`**

Returns metadata and base64-encoded content in a single JSON response. Useful when you need the file and its metadata together without handling binary responses.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `path` | string | Yes | Full path to the file |

### Response  -  `200 OK`
```json
{
  "success": true,
  "file": {
    "path": "/workspace/data.json",
    "filename": "data.json",
    "size_bytes": 5678,
    "content_type": "application/json",
    "modified_at": "2026-04-11T10:30:00Z"
  },
  "content_base64": "eyJrZXkiOiAidmFsdWUifQ=="
}
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/files/export \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"path": "/workspace/data.json"}'
```

---

## Delete a File

**`DELETE /api/v1/computers/{id}/files`**

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `path` | string | Yes | Full path to delete |

### Response  -  `200 OK`
```json
{ "success": true, "path": "/workspace/old-file.txt" }
```
```bash
curl -X DELETE https://api.miosa.ai/api/v1/computers/{id}/files \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"path": "/workspace/old-file.txt"}'
```

---

## Stat a File or Directory

**`GET /api/v1/computers/{id}/files/stat`**

Returns metadata without downloading file content.

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Full path |

### Response  -  `200 OK`
```json
{
  "path": "/workspace/script.py",
  "type": "file",
  "size_bytes": 1234,
  "permissions": "-rw-r--r--",
  "owner": "user",
  "group": "user",
  "modified_at": "2026-04-11T10:25:00Z",
  "created_at": "2026-04-11T09:00:00Z"
}
```

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 404 | `FILE_NOT_FOUND` | Path does not exist |
```bash
curl "https://api.miosa.ai/api/v1/computers/{id}/files/stat?path=/workspace/script.py" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Create a Directory

**`POST /api/v1/computers/{id}/files/mkdir`**

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `path` | string | Yes | Directory path to create |
| `recursive` | boolean | No | Create parent directories if missing (default: `false`) |

### Response  -  `201 Created`
```json
{ "success": true, "path": "/workspace/project/src" }
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/files/mkdir \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"path": "/workspace/project/src", "recursive": true}'
```

---

## Rename or Move

**`POST /api/v1/computers/{id}/files/rename`**

Renames a file or directory, or moves it to a different path. Both source and destination must be within allowed paths.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `source` | string | Yes | Current path |
| `destination` | string | Yes | Target path |

### Response  -  `200 OK`
```json
{ "success": true, "source": "/workspace/old.txt", "destination": "/workspace/new.txt" }
```

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 404 | `FILE_NOT_FOUND` | Source path does not exist |
| 409 | `DESTINATION_EXISTS` | Destination already exists |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/files/rename \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"source": "/workspace/app.py", "destination": "/workspace/app_v2.py"}'
```

---

## Copy a File

**`POST /api/v1/computers/{id}/files/copy`**

Copies a file to a new path. Directory copy is not supported  -  copy individual files.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `source` | string | Yes | Source file path |
| `destination` | string | Yes | Destination file path |

### Response  -  `201 Created`
```json
{ "success": true, "source": "/workspace/template.py", "destination": "/workspace/project/main.py" }
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/files/copy \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"source": "/workspace/template.py", "destination": "/workspace/project/main.py"}'
```

---

## Change Permissions

**`POST /api/v1/computers/{id}/files/chmod`**

Sets UNIX file permissions.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `path` | string | Yes | File path |
| `mode` | string | Yes | Octal permission string, e.g. `"755"` or `"644"` |

### Response  -  `200 OK`
```json
{ "success": true, "path": "/workspace/script.sh", "mode": "755" }
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/files/chmod \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"path": "/workspace/script.sh", "mode": "755"}'
```

---

## Path Security

All paths are validated before execution:

1. **Expansion**  -  `Path.expand` resolves `..`, `~`, and symlinks
2. **Prefix check**  -  Must start with `/workspace`, `/home`, `/root`, `/tmp`, `/opt`, or `/srv`
3. **Rejection**  -  Paths outside these prefixes return `403 FORBIDDEN_PATH`

`/workspace/../../etc/passwd` expands to `/etc/passwd` and is rejected.

---

## Common Recipes

### Scaffold a project directory
```typescript
// Create structure
await client.files.mkdir(computerId, { path: '/workspace/app/src', recursive: true });
await client.files.mkdir(computerId, { path: '/workspace/app/tests', recursive: true });

// Write files
await client.files.write(computerId, { path: '/workspace/app/src/main.py', content: mainPy });
await client.files.write(computerId, { path: '/workspace/app/requirements.txt', content: reqs });

// Make entrypoint executable
await client.files.chmod(computerId, { path: '/workspace/app/src/main.py', mode: '755' });
```

### Retrieve build artifacts
```python
# List the build output directory
files = client.files.list(computer_id, path="/workspace/app/dist")
for f in files["files"]:
    if f["type"] == "file":
        content = client.files.export(computer_id, path=f"/workspace/app/dist/{f['name']}")
        save_artifact(f["name"], content["content_base64"])
```

---

## Sandbox filesystem

The sandbox primitive exposes the same list and stat operations under `/api/v1/sandboxes/{id}`.

### List Files (sandbox)

**`GET /api/v1/sandboxes/{id}/files?path=/workspace`**

Lists files and directories in a sandbox directory.
```bash
curl "https://api.miosa.ai/api/v1/sandboxes/{id}/files?path=/workspace" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

Response:
```json
{
  "data": {
    "path": "/workspace",
    "entries": [
      {"name": "index.html", "path": "/workspace/index.html", "is_dir": false, "size": 128}
    ]
  }
}
```

### Stat a File (sandbox)

**`POST /api/v1/sandboxes/{id}/files/stat`**

Returns metadata for one file or directory.
```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/{id}/files/stat \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"path":"/workspace/index.html"}'
```
