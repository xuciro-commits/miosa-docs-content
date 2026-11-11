---
slug: "api-reference/files"
title: "Files API"
description: "API reference for file upload, download, list, export, and delete operations."
section: "API reference"
order: 12021
---

Manage files on a running computer. All paths are restricted to `/home/user`, `/home/ubuntu`, and `/tmp`.

Base path: `/api/v1/computers/{id}/files`

> **Note**
>
> The computer must be in `"running"` status. All operations go through the computer command service.

---

## Upload a File

**`POST /api/v1/computers/{id}/files/upload`**

Uploads a file to the computer via multipart form data.

### Request

Content-Type: `multipart/form-data`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file` | file | Yes | File to upload |
| `path` | string | No | Remote path. If ends with `/`, filename is appended. Default: `/home/ubuntu/` |

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
| 413 | `FILE_TOO_LARGE` | File exceeds the 500 MB upload limit |
| 502 | `AGENT_UNAVAILABLE` | In-VM agent unreachable |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/files/upload \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -F "file=@./script.py" \
  -F "path=/workspace/scripts/"
```

---

## List Directory

**`GET /api/v1/computers/{id}/files`**

Lists files and directories at a given path.

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | No | Directory to list. Default: `/home/ubuntu` |

### Response  -  `200 OK`
```json
{
  "files": [
    {
      "name": "Documents",
      "type": "directory",
      "size_bytes": 4096,
      "modified_at": "2026-04-11T10:30:00Z"
    },
    {
      "name": "script.py",
      "type": "file",
      "size_bytes": 1234,
      "modified_at": "2026-04-11T10:25:00Z"
    }
  ],
  "path": "/workspace"
}
```

### File Entry Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | File or directory name |
| `type` | string | `"file"` or `"directory"` |
| `size_bytes` | integer | Size in bytes |
| `modified_at` | ISO 8601 | Last modification time |
```bash
curl "https://api.miosa.ai/api/v1/computers/{id}/files?path=/workspace" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Download a File

**`GET /api/v1/computers/{id}/files/download`**

Downloads a file from the computer as binary data.

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Full path to the file |

### Response  -  `200 OK`

Binary file content. Content-Type is inferred from the file extension. The `Content-Disposition` header is set for download.

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 400 | `MISSING_PATH` | No path parameter |
| 403 | `FORBIDDEN_PATH` | Path outside allowed directories |
| 502 | `AGENT_UNAVAILABLE` | In-VM agent unreachable |
```bash
curl "https://api.miosa.ai/api/v1/computers/{id}/files/download?path=/workspace/output.txt" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  --output output.txt
```

---

## Export a File

**`POST /api/v1/computers/{id}/files/export`**

Returns file metadata and base64-encoded content in a single response.

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

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 400 | `MISSING_PATH` | No path in request body |
| 403 | `FORBIDDEN_PATH` | Path outside allowed directories |
| 404 | `FILE_NOT_FOUND` | File does not exist or is empty |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/files/export \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"path": "/workspace/data.json"}'
```

---

## Delete a File

**`DELETE /api/v1/computers/{id}/files`**

Removes a file from the computer.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `path` | string | Yes | Full path to the file to delete |

### Response  -  `200 OK`
```json
{
  "success": true,
  "path": "/workspace/old-file.txt"
}
```

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 400 | `MISSING_PATH` | No path in request body |
| 403 | `FORBIDDEN_PATH` | Path outside allowed directories |
| 502 | `AGENT_UNAVAILABLE` | In-VM agent unreachable |
```bash
curl -X DELETE https://api.miosa.ai/api/v1/computers/{id}/files \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"path": "/workspace/old-file.txt"}'
```

---

## Path Security

All paths are validated before execution:

1. **Expansion**  -  `Path.expand` resolves `..`, `~`, and symlinks
2. **Prefix check**  -  Expanded path must start with `/home/user`, `/home/ubuntu`, or `/tmp`
3. **Rejection**  -  Paths outside these prefixes return `403 FORBIDDEN_PATH`

This prevents path traversal attacks. For example, `/workspace/../../etc/passwd` expands to `/etc/passwd` and is rejected.

---

## Sandbox files

Sandbox files live under `/api/v1/sandboxes/{id}`. The sandbox must be in `running` state.

### Upload a File (sandbox)

**`POST /api/v1/sandboxes/{id}/files`**

Writes a file to the sandbox filesystem. Two request formats are accepted.

#### Request Body  -  JSON

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `path` | string | Yes | Absolute destination path inside the sandbox. |
| `content` | string | Yes | Base64-encoded file content. |
```json
{
  "path": "/workspace/script.py",
  "content": "cHJpbnQoJ2hlbGxvJyk="
}
```

#### Request Body  -  Multipart

Alternatively, send `multipart/form-data` with fields:

| Field | Description |
|-------|-------------|
| `path` | Absolute destination path. |
| `file` | File part containing the raw content. |

#### Response  -  `200 OK`
```json
{
  "data": {
    "sandbox_id": "sbx_01j9xr2t4fk8me3n5q",
    "path": "/workspace/script.py",
    "size": 16
  }
}
```

#### Errors

| Status | Code | Cause |
|--------|------|-------|
| 409 | `SANDBOX_NOT_RUNNING` | Sandbox is not in `running` state. |
| 413 |  -  | File exceeds the 500 MB upload limit. |
```bash
# JSON (base64-encoded content)
curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_01j9xr2t4fk8me3n5q/files \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"path\": \"/workspace/script.py\", \"content\": \"$(base64 -w0 script.py)\"}"

# Multipart
curl -X POST https://api.miosa.ai/api/v1/sandboxes/sbx_01j9xr2t4fk8me3n5q/files \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -F "path=/workspace/script.py" \
  -F "file=@script.py"
```

### Download a File (sandbox)

**`GET /api/v1/sandboxes/{id}/files/{path}`**

Downloads a file from the sandbox filesystem.

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | string | Sandbox ID. Must be in `running` state. |
| `path` | string | URL-encoded path to the file inside the sandbox (leading `/` stripped). |

#### Response  -  `200 OK`

Raw file bytes with `Content-Type: application/octet-stream`.

#### Errors

| Status | Code | Cause |
|--------|------|-------|
| 404 | `FILE_NOT_FOUND` | Path does not exist inside the sandbox. |
| 409 | `SANDBOX_NOT_RUNNING` | Sandbox is not in `running` state. |
```bash
curl -o output.json \
  "https://api.miosa.ai/api/v1/sandboxes/sbx_01j9xr2t4fk8me3n5q/files/workspace%2Foutput.json" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

### Export Files (sandbox)

Exports are for files an agent created inside the sandbox that your product
needs to show or download: static HTML, PDFs, CSVs, screenshots, generated
reports, ZIPs, and build folders.

**`POST /api/v1/sandboxes/{id}/exports`**

Creates a ready export descriptor. This does not move the file out of the
sandbox; it returns download URLs backed by the sandbox filesystem.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `path` | string | Yes, unless `paths` is set | Single file path inside the sandbox. |
| `paths` | array | Yes, unless `path` is set | Multiple file paths to package as a tar.gz archive. |
| `label` | string | No | Human-readable export label. |
| `filename` | string | No | Suggested filename for download responses. |

#### Response  -  `200 OK`

```json
{
  "data": {
    "id": "exp_01j9xr2t4fk8me3n5q",
    "sandbox_id": "sbx_01j9xr2t4fk8me3n5q",
    "label": "homepage build",
    "status": "ready",
    "files": [
      {
        "path": "/workspace/dist/index.html",
        "filename": "index.html",
        "download_url": "https://api.miosa.ai/api/v1/sandboxes/sbx_.../exports/download?path=..."
      }
    ],
    "archive_download_url": "https://api.miosa.ai/api/v1/sandboxes/sbx_.../exports/download?paths%5B%5D=...",
    "created_at": "2026-06-14T21:00:00Z"
  }
}
```

**`GET /api/v1/sandboxes/{id}/exports/download?path=/workspace/report.pdf`**

Downloads one file as binary.

**`GET /api/v1/sandboxes/{id}/exports/download?paths[]=/workspace/dist/index.html&paths[]=/workspace/dist/app.js`**

Downloads multiple paths as a tar.gz archive.

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SBX/exports \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"paths":["/workspace/dist/index.html","/workspace/dist/app.js"],"label":"build"}'

curl "https://api.miosa.ai/api/v1/sandboxes/$SBX/exports/download?path=/workspace/report.pdf" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -o report.pdf
```

### Read Logs (sandbox)

**`GET /api/v1/sandboxes/{id}/logs`**

Returns recent sandbox/template lifecycle logs. Pass `lines` to control the
tail length.
```bash
curl "https://api.miosa.ai/api/v1/sandboxes/{id}/logs?lines=200" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

Stream logs with SSE:
```bash
curl -N "https://api.miosa.ai/api/v1/sandboxes/{id}/logs/stream" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Accept: text/event-stream"
```
