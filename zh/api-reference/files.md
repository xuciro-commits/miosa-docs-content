---
slug: "api-reference/files"
title: "文件API"
description: "文件上传、下载、列出、导出和删除操作的 API 参考。"
section: "API reference"
order: 12021
---

管理正在运行的计算机上的文件。所有路径都限制为 `/home/user`、`/home/ubuntu` 和 `/tmp`。

基本路径：`/api/v1/computers/{id}/files`

> **注意**
>
> 计算机必须处于 `"running"` 状态。所有操作都通过计算机命令服务进行。

---

## 上传文件

**`POST /api/v1/computers/{id}/files/upload`**

通过多部分表单数据将文件上传到计算机。

### 请求

内容类型：`multipart/form-data`

|字段 |类型 |必填 |描述 |
|--------|------|----------|-------------|
| `file` |文件|是的 |要上传的文件 |
| `path` |字符串|没有 |远程路径。如果以 `/` 结尾，则附加文件名。默认值：`/home/ubuntu/` |

### 响应 - `201 Created`
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

### 错误

|状态 |代码|描述 |
|--------|------|-------------|
| 400 | `MISSING_FILE` |请求中没有文件字段 |
| 403 | `FORBIDDEN_PATH` | 允许目录之外的路径 |
| 413 | `FILE_TOO_LARGE` | 文件超过 500 MB 上传限制 |
| 502 | `AGENT_UNAVAILABLE` | 虚拟机内智能体无法访问 |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/files/upload \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -F "file=@./script.py" \
  -F "path=/workspace/scripts/"
```

---

## 列出目录

**`GET /api/v1/computers/{id}/files`**

列出给定路径中的文件和目录。

### 查询参数

|参数|类型 |必填 |描述 |
|------------|------|----------|-------------|
| `path` |字符串|没有 |要列出的目录。默认值：`/home/ubuntu` |

### 响应 - `200 OK`
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

### 文件输入字段

|字段 |类型 |描述 |
|--------|------|-------------|
| `name` |字符串 |文件或目录名 |
| `type` |字符串| `"file"` 或 `"directory"` |
| `size_bytes` |整数 |大小（以字节为单位）|
| `modified_at` | ISO 8601 |最后修改时间 |
```bash
curl "https://api.miosa.ai/api/v1/computers/{id}/files?path=/workspace" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## 下载文件

**`GET /api/v1/computers/{id}/files/download`**

从计算机下载文件作为二进制数据。

### 查询参数

|参数|类型 |必填 |描述 |
|------------|------|----------|-------------|
| `path` |字符串|是的 |文件的完整路径 |

### 响应 - `200 OK`

二进制文件内容。 Content-Type 是从文件扩展名推断出来的。 `Content-Disposition` 标头设置为下载。

### 错误

|状态 |代码|描述 |
|--------|------|-------------|
| 400 | `MISSING_PATH` |无路径参数|
| 403 | `FORBIDDEN_PATH` | 允许目录之外的路径 |
| 502 | `AGENT_UNAVAILABLE` | 虚拟机内智能体无法访问 |
```bash
curl "https://api.miosa.ai/api/v1/computers/{id}/files/download?path=/workspace/output.txt" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  --output output.txt
```

---

## 导出文件

**`POST /api/v1/computers/{id}/files/export`**

在单个响应中返回文件元数据和 Base64 编码的内容。

### 请求正文

|字段 |类型 |必填 |描述 |
|--------|------|----------|-------------|
| `path` |字符串|是的 |文件的完整路径 |

### 响应 - `200 OK`
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

### 错误

|状态 |代码|描述 |
|--------|------|-------------|
| 400 | `MISSING_PATH` |请求正文中没有路径 |
| 403 | `FORBIDDEN_PATH` | 允许目录之外的路径 |
| 404 | `FILE_NOT_FOUND` | 文件不存在或为空 |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/files/export \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"path": "/workspace/data.json"}'
```

---

## 删除文件

**`DELETE /api/v1/computers/{id}/files`**

从计算机中删除文件。

### 请求正文

|字段 |类型 |必填 |描述 |
|--------|------|----------|-------------|
| `path` |字符串|是的 |要删除的文件的完整路径 |

### 响应 - `200 OK`
```json
{
  "success": true,
  "path": "/workspace/old-file.txt"
}
```

### 错误

|状态 |代码|描述 |
|--------|------|-------------|
| 400 | `MISSING_PATH` |请求正文中没有路径 |
| 403 | `FORBIDDEN_PATH` | 允许目录之外的路径 |
| 502 | `AGENT_UNAVAILABLE` | 虚拟机内智能体无法访问 |
```bash
curl -X DELETE https://api.miosa.ai/api/v1/computers/{id}/files \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"path": "/workspace/old-file.txt"}'
```

---

## 路径安全

所有路径在执行前都会经过验证：

1. **扩展** - `Path.expand` 解析 `..`、`~` 和符号链接
2. **前缀检查** - 扩展路径必须以 `/home/user`、`/home/ubuntu` 或 `/tmp` 开头
3. **拒绝** - 这些前缀之外的路径返回 `403 FORBIDDEN_PATH`

这可以防止路径遍历攻击。例如，`/workspace/../../etc/passwd` 扩展为 `/etc/passwd` 并被拒绝。

---

## 沙盒文件

沙箱文件位于 `/api/v1/sandboxes/{id}` 下。沙箱必须处于 `running` 状态。

### 上传文件（沙盒）

**`POST /api/v1/sandboxes/{id}/files`**

将文件写入沙箱文件系统。接受两种请求格式。

#### 请求正文 - JSON

|字段 |类型 |必填 |描述 |
|--------|------|----------|-------------|
| `path` |字符串|是的 |沙箱内的绝对目标路径。 |
| `content` |字符串|是的 | Base64 编码的文件内容。 |
```json
{
  "path": "/workspace/script.py",
  "content": "cHJpbnQoJ2hlbGxvJyk="
}
```

#### 请求正文 - 多部分

或者，发送带有字段的 `multipart/form-data`：

|字段 |描述 |
|--------|-------------|
| `path` |绝对目标路径。 |
| `file` |包含原始内容的文件部分。 |

#### 响应 - `200 OK`
```json
{
  "data": {
    "sandbox_id": "sbx_01j9xr2t4fk8me3n5q",
    "path": "/workspace/script.py",
    "size": 16
  }
}
```

#### 错误

|状态 |代码|原因 |
|--------|------|--------|
| 409 | `SANDBOX_NOT_RUNNING` | 沙盒不处于 `running` 状态。 |
| 413 | - | 文件超过 500 MB 上传限制。 |
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

### 下载文件（沙箱）

**`GET /api/v1/sandboxes/{id}/files/{path}`**

从沙箱文件系统下载文件。

|参数|类型 |描述 |
|------------|------|-------------|
| `id` |字符串|沙箱 ID。必须处于 `running` 状态。 |
| `path` |字符串|沙箱内文件的 URL 编码路径（去除前导 `/`）。 |

#### 响应 - `200 OK`

带有 `Content-Type: application/octet-stream` 的原始文件字节。

#### 错误

|状态 |代码|原因 |
|--------|------|--------|
| 404 | `FILE_NOT_FOUND` | 沙箱内不存在路径。 |
| 409 | `SANDBOX_NOT_RUNNING` | 沙盒不处于 `running` 状态。 |
```bash
curl -o output.json \
  "https://api.miosa.ai/api/v1/sandboxes/sbx_01j9xr2t4fk8me3n5q/files/workspace%2Foutput.json" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

### 导出文件（沙箱）

导出用于智能体在您的产品的沙箱内创建的文件
需要显示或下载：静态 HTML、PDF、CSV、屏幕截图、生成
报告、ZIP 和构建文件夹。

**`POST /api/v1/sandboxes/{id}/exports`**

创建一个准备好的导出描述符。这不会将文件移出
沙箱；它返回沙箱文件系统支持的下载 URL。

|字段 |类型 |必填 |描述 |
|--------|------|----------|-------------|
| `path` |字符串|是的，除非设置了 `paths` |沙箱内的单个文件路径。 |
| `paths` |数组|是的，除非设置了 `path` |打包为 tar.gz 存档的多个文件路径。 |
| `label` |字符串|没有 |人类可读的出口标签。 |
| `filename` |字符串|没有 |下载响应的建议文件名。 |

#### 响应 - `200 OK`

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

以二进制形式下载一个文件。

**`GET /api/v1/sandboxes/{id}/exports/download?paths[]=/workspace/dist/index.html&paths[]=/workspace/dist/app.js`**

下载多个路径作为 tar.gz 存档。

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SBX/exports \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"paths":["/workspace/dist/index.html","/workspace/dist/app.js"],"label":"build"}'

curl "https://api.miosa.ai/api/v1/sandboxes/$SBX/exports/download?path=/workspace/report.pdf" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -o report.pdf
```

### 读取日志（沙箱）

**`GET /api/v1/sandboxes/{id}/logs`**

返回最近的沙箱/模板生命周期日志。通过`lines`来控制
尾长。
```bash
curl "https://api.miosa.ai/api/v1/sandboxes/{id}/logs?lines=200" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

使用 SSE 流式传输日志：
```bash
curl -N "https://api.miosa.ai/api/v1/sandboxes/{id}/logs/stream" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Accept: text/event-stream"
```
